<script>
	let gameStats = [];
	let avgDuration = 0;
	let streaks = { Felicia: 0, Jorge: 0, max: 0, maxPlayer: null };
	let unoShoutsPerGame = [];

	// Helper to get player name from winner key
	function winnerName(winner) {
		return winner === 'player1' ? 'Felicia' : 'Jorge';
	}

	// Calculate stats from localStorage
	function calculateStats() {
		if (typeof window === 'undefined') return;
		const raw = localStorage.getItem('uno-game-history');
		if (!raw) return;
		let games = [];
		try {
			games = JSON.parse(raw);
		} catch {}
		if (!Array.isArray(games) || games.length === 0) return;
		gameStats = games.slice(-10);
		// Average duration
		avgDuration = Math.round(games.reduce((a, g) => a + g.duration, 0) / games.length) || 0;
		// UNO shouts per game
		unoShoutsPerGame = games.slice(-10).map((g) => g.uno1 + g.uno2);
		// Winner streaks
		let max = 1,
			cur = 1,
			maxPlayer = games[0].winner,
			prev = games[0].winner;
		for (let i = 1; i < games.length; i++) {
			if (games[i].winner === prev) {
				cur++;
				if (cur > max) {
					max = cur;
					maxPlayer = prev;
				}
			} else {
				cur = 1;
				prev = games[i].winner;
			}
		}
		streaks = { Felicia: 0, Jorge: 0, max, maxPlayer: winnerName(maxPlayer) };
		// Current streak
		let last = games[games.length - 1].winner,
			streak = 1;
		for (let i = games.length - 2; i >= 0; i--) {
			if (games[i].winner === last) streak++;
			else break;
		}
		streaks[winnerName(last)] = streak;
	}

	// Save each game to history and update stats/graph
	let chart = null;
	import { tick } from 'svelte';
	async function renderChart() {
		if (typeof window === 'undefined') return;
		await tick();
		if (!gameStats.length) return;
		let ChartCtor = null;
		if (!window.Chart) {
			ChartCtor = (await import('chart.js/auto')).Chart;
			window.Chart = ChartCtor;
		} else {
			ChartCtor = window.Chart;
		}
		const ctx = document.getElementById('gameChart');
		if (!ctx) return;
		if (chart) chart.destroy();
		chart = new ChartCtor(ctx, {
			type: 'bar',
			data: {
				labels: gameStats.map((g, i) => `G${gameStats.length - i}`),
				datasets: [
					{
						label: 'Game Duration (s)',
						data: gameStats.map((g) => g.duration),
						backgroundColor: gameStats.map((g) => (g.winner === 'player1' ? '#ef4444' : '#facc15')),
						borderRadius: 8,
						borderSkipped: false,
						borderWidth: 2,
						borderColor: '#fff',
						barPercentage: 0.7,
						categoryPercentage: 0.7
					},
					{
						label: 'UNO Shouts',
						data: gameStats.map((g) => g.uno1 + g.uno2),
						backgroundColor: 'rgba(34,197,94,0.2)',
						borderColor: '#22c55e',
						pointBackgroundColor: '#22c55e',
						pointBorderColor: '#fff',
						pointRadius: 7,
						pointHoverRadius: 10,
						borderWidth: 3,
						type: 'line',
						yAxisID: 'y1',
						tension: 0.3
					}
				]
			},
			options: {
				responsive: true,
				plugins: {
					legend: { labels: { color: '#fff', font: { size: 16, weight: 'bold' } } },
					tooltip: {
						backgroundColor: '#222',
						titleColor: '#facc15',
						bodyColor: '#fff',
						borderColor: '#fff',
						borderWidth: 1,
						padding: 12,
						cornerRadius: 8
					}
				},
				scales: {
					x: {
						ticks: { color: '#fff', font: { size: 14, weight: 'bold' } },
						grid: { color: '#333', borderColor: '#fff' }
					},
					y: {
						title: {
							display: true,
							text: 'Seconds',
							color: '#fff',
							font: { size: 16, weight: 'bold' }
						},
						ticks: { color: '#fff', font: { size: 14 } },
						grid: { color: '#333', borderColor: '#fff' }
					},
					y1: {
						position: 'right',
						title: {
							display: true,
							text: 'UNO Shouts',
							color: '#22c55e',
							font: { size: 16, weight: 'bold' }
						},
						ticks: { color: '#22c55e', font: { size: 14 } },
						grid: { drawOnChartArea: false }
					}
				}
			}
		});
	}

	function saveGameToHistory(winner) {
		if (typeof window === 'undefined') return;
		let games = [];
		const raw = localStorage.getItem('uno-game-history');
		if (raw) {
			try {
				games = JSON.parse(raw);
			} catch {}
		}
		games.push({
			winner,
			duration: timer,
			uno1,
			uno2,
			date: new Date().toISOString()
		});
		localStorage.setItem('uno-game-history', JSON.stringify(games));
		calculateStats();
		renderChart();
	}

	import { onMount, onDestroy } from 'svelte';

	let player1 = 0;
	let player2 = 0;
	let uno1 = 0;
	let uno2 = 0;

	// Timer state
	let timer = 0; // seconds (current game clock)
	let timerInterval = null;
	let timerRunning = false;
	let prevGameDuration = null; // duration of previous game

	// Last winner state
	let lastWinner = null;
	let lastWinTime = null;

	onMount(() => {
		if (typeof window !== 'undefined') {
			const p1 = localStorage.getItem('uno-player1');
			const p2 = localStorage.getItem('uno-player2');
			const u1 = localStorage.getItem('uno-shout1');
			const u2 = localStorage.getItem('uno-shout2');
			const prev = localStorage.getItem('uno-prev-game-duration');
			if (p1 !== null && !isNaN(Number(p1))) player1 = Number(p1);
			if (p2 !== null && !isNaN(Number(p2))) player2 = Number(p2);
			if (u1 !== null && !isNaN(Number(u1))) uno1 = Number(u1);
			if (u2 !== null && !isNaN(Number(u2))) uno2 = Number(u2);
			if (prev !== null && !isNaN(Number(prev))) prevGameDuration = Number(prev);
			startTimer();
			window.addEventListener('keydown', handleKey);
		}
	});
	function formatTime(s) {
		const m = Math.floor(s / 60);
		const sec = s % 60;
		return `${m.toString().padStart(2, '0')}:${sec.toString().padStart(2, '0')}`;
	}

	function startTimer() {
		if (timerRunning) return;
		timerRunning = true;
		timerInterval = setInterval(() => {
			timer++;
		}, 1000);
	}

	function pauseTimer() {
		timerRunning = false;
		if (timerInterval) clearInterval(timerInterval);
		timerInterval = null;
	}

	function resetTimer() {
		pauseTimer();
		timer = 0;
		startTimer();
	}

	function handleKey(e) {
		if (e.key === 'a' || e.key === 'A') inc1();
		if (e.key === 'l' || e.key === 'L') inc2();
		if (e.key === 'q' || e.key === 'Q') incUno1();
		if (e.key === 'p' || e.key === 'P') incUno2();
	}

	function saveTimerToSet(winner) {
		if (typeof window !== 'undefined') {
			// Save only the last game duration and winner
			prevGameDuration = timer;
			localStorage.setItem('uno-prev-game-duration', prevGameDuration);
			lastWinner = winner;
			lastWinTime = timer;
			saveGameToHistory(winner);
		}
	}

	function inc1() {
		player1++;
		if (typeof window !== 'undefined') {
			localStorage.setItem('uno-player1', player1);
		}
		saveTimerToSet('player1');
		resetTimer();
	}
	function inc2() {
		player2++;
		if (typeof window !== 'undefined') {
			localStorage.setItem('uno-player2', player2);
		}
		saveTimerToSet('player2');
		resetTimer();
	}
	function incUno1() {
		uno1++;
		if (typeof window !== 'undefined') {
			localStorage.setItem('uno-shout1', uno1);
		}
	}
	function incUno2() {
		uno2++;
		if (typeof window !== 'undefined') {
			localStorage.setItem('uno-shout2', uno2);
		}
	}

	// Listen for keydown events on mount
	onMount(() => {
		if (typeof window !== 'undefined') {
			window.addEventListener('keydown', handleKey);
			calculateStats();
			renderChart();
		}
	});
	onDestroy(() => {
		if (typeof window !== 'undefined') {
			window.removeEventListener('keydown', handleKey);
		}
		if (timerInterval) clearInterval(timerInterval);
	});
</script>

<main class="flex min-h-screen flex-col items-center justify-center gap-2 bg-black font-sans">
	<h1
		class="mb-2 text-4xl font-extrabold tracking-tight text-yellow-400 uppercase drop-shadow-lg md:text-5xl"
		style="text-shadow: 2px 2px 0 #e3342f, 4px 4px 0 #fff;"
	>
		UNO Victory Counter
	</h1>

	<div class="mb-4 flex flex-col items-center">
		<div class="flex items-center gap-4">
			<span
				class="rounded-lg border-2 border-white bg-black px-6 py-2 font-mono text-3xl font-bold tracking-widest text-white shadow-lg md:text-4xl"
				>{formatTime(timer)}</span
			>
		</div>
	</div>

	{#if lastWinner}
		<div class="mb-2 flex flex-col items-center">
			<span
				class="rounded-lg border-2 border-white bg-black px-4 py-1 text-lg font-bold text-white shadow"
				>Last Winner: <span class={lastWinner === 'player1' ? 'text-red-400' : 'text-yellow-300'}
					>{lastWinner === 'player1' ? 'Felicia' : 'Jorge'}</span
				></span
			>
			<span class="mt-1 text-xs text-green-300"
				>Previous game duration: {formatTime(prevGameDuration ?? 0)}</span
			>
		</div>
	{/if}

	<div class="flex flex-col gap-8 md:flex-row md:gap-16">
		<div
			class="relative flex w-72 flex-col items-center rounded-2xl border-4 border-white bg-gradient-to-br from-red-600 to-yellow-400 p-8 shadow-2xl"
		>
			{#if player1 > player2}
				<!-- Crown SVG for Felicia -->
				<div class="absolute -top-15 left-0 z-10 -translate-x-1/2 -rotate-25 text-8xl">👑</div>
			{/if}
			<h2
				class="mb-2 text-2xl font-extrabold text-white drop-shadow"
				style="text-shadow: 1px 1px 0 #e3342f;"
			>
				Felicia
			</h2>
			<div
				class="mb-4 text-6xl font-black text-white drop-shadow"
				style="text-shadow: 2px 2px 0 #e3342f;"
			>
				{player1}
			</div>
			<button
				on:click={inc1}
				class="w-full rounded-lg border-2 border-white bg-red-600 px-8 py-1 text-xl font-extrabold text-white shadow-lg transition-all duration-150 hover:bg-yellow-400 hover:text-black focus:ring-4 focus:ring-yellow-400 focus:outline-none"
			>
				Victory <div class="text-sm text-white">
					(Press <span class="font-mono font-semibold text-yellow-300">A</span>)
				</div></button
			>
			<div class="mt-6 flex flex-col items-center">
				<div class="mb-1 text-lg font-bold text-yellow-300">UNO Shouts</div>
				<div class="flex items-center gap-2">
					<span class="text-3xl font-black text-white" style="text-shadow: 1px 1px 0 #e3342f;"
						>{uno1}</span
					>
					<button
						on:click={incUno1}
						class="rounded border-2 border-white bg-yellow-400 px-3 py-1 text-lg font-extrabold text-black shadow transition-all duration-150 hover:bg-red-600 hover:text-white focus:ring-2 focus:ring-red-400 focus:outline-none"
						>UNO!</button
					>
				</div>
				<div class="mt-1 text-xs text-white">
					(Press <span class="font-mono font-semibold text-yellow-300">Q</span>)
				</div>
			</div>
		</div>
		<div
			class="relative flex w-72 flex-col items-center rounded-2xl border-4 border-white bg-gradient-to-br from-yellow-400 to-red-600 p-8 shadow-2xl"
		>
			{#if player2 > player1}
				<!-- Crown SVG for Jorge -->
				<div class="absolute -top-15 -right-10 z-10 rotate-25 text-8xl">👑</div>
			{/if}
			<h2
				class="mb-2 text-2xl font-extrabold text-black drop-shadow"
				style="text-shadow: 1px 1px 0 #fff;"
			>
				Jorge
			</h2>
			<div
				class="mb-4 text-6xl font-black text-black drop-shadow"
				style="text-shadow: 2px 2px 0 #fff;"
			>
				{player2}
			</div>
			<button
				on:click={inc2}
				class="w-full rounded-lg border-2 border-white bg-yellow-400 px-8 py-1 text-xl font-extrabold text-black shadow-lg transition-all duration-150 hover:bg-red-600 hover:text-white focus:ring-4 focus:ring-red-400 focus:outline-none"
				>Victory <div class="text-sm text-black">
					(Press <span class="font-mono font-semibold text-red-500">L</span>)
				</div></button
			>
			<div class="mt-6 flex flex-col items-center">
				<div class="mb-1 text-lg font-bold text-red-600">UNO Shouts</div>
				<div class="flex items-center gap-2">
					<span class="text-3xl font-black text-black" style="text-shadow: 1px 1px 0 #fff;"
						>{uno2}</span
					>
					<button
						on:click={incUno2}
						class="rounded border-2 border-white bg-red-600 px-3 py-1 text-lg font-extrabold text-white shadow transition-all duration-150 hover:bg-yellow-400 hover:text-black focus:ring-2 focus:ring-yellow-400 focus:outline-none"
						>UNO!</button
					>
				</div>
				<div class="mt-1 text-xs text-black">
					(Press <span class="font-mono font-semibold text-red-600">P</span>)
				</div>
			</div>
		</div>
	</div>
</main>

<!-- Statistics Section -->
<div class="mx-auto mt-4 mb-6 w-full max-w-2xl rounded-xl border-2 border-white bg-black/80 p-4">
	<h3 class="mb-2 text-xl font-bold text-yellow-300">Game Statistics</h3>
	<div class="mb-4 flex flex-wrap gap-6">
		<div class="flex flex-col items-center">
			<span class="text-lg font-semibold text-white">Avg. Duration</span>
			<span class="font-mono text-2xl text-green-300">{formatTime(avgDuration)}</span>
		</div>
		<div class="flex flex-col items-center">
			<span class="text-lg font-semibold text-white">Current Streak</span>
			<span
				class="text-2xl font-bold {streaks.Felicia > streaks.Jorge
					? 'text-red-400'
					: 'text-yellow-300'}"
				>{streaks.Felicia > streaks.Jorge
					? `Felicia (${streaks.Felicia})`
					: `Jorge (${streaks.Jorge})`}</span
			>
		</div>
		<div class="flex flex-col items-center">
			<span class="text-lg font-semibold text-white">Longest Streak</span>
			<span
				class="text-2xl font-bold {streaks.maxPlayer === 'Felicia'
					? 'text-red-400'
					: 'text-yellow-300'}">{streaks.maxPlayer} ({streaks.max})</span
			>
		</div>
	</div>
	<div class="h-64 w-full rounded-lg border border-white bg-black p-2">
		<canvas id="gameChart" width="400" height="200"></canvas>
	</div>
	<div class="mt-4 flex flex-col gap-1">
		<span class="text-sm font-semibold text-white">UNO Shouts (last 10 games):</span>
		<div class="flex flex-wrap gap-2">
			{#each gameStats as g, i}
				<span class="rounded bg-gray-800 px-2 py-1 text-xs text-white"
					>G{gameStats.length - i}: {g.uno1 + g.uno2} shouts</span
				>
			{/each}
		</div>
	</div>
</div>
