<script lang="ts">
	// Extend the Window interface to include Chart for TypeScript
	declare global {
		interface Window {
			Chart?: any;
		}
	}

	import PlayerCard from './components/PlayerCard.svelte';
	import LastWinner from './components/LastWinner.svelte';
	import StatisticsSection from './components/StatisticsSection.svelte';
	let gameStats: any[] = [];
	let avgDuration: number = 0;
	let streaks: { Felicia: number; Jorge: number; max: number; maxPlayer: string | null } = {
		Felicia: 0,
		Jorge: 0,
		max: 0,
		maxPlayer: null
	};
	let unoShoutsPerGame: number[] = []; // Array to hold UNO shouts per game

	// Helper to get player name from winner key
	function winnerName(winner: string) {
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
	let chart: { destroy: () => void } | null = null;
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

	function saveGameToHistory(winner: any) {
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
			player1: player1,
			player2: player2,
			uno1,
			uno2,
			shouts: uno1 + uno2,
			date: new Date().toISOString()
		});
		localStorage.setItem('uno-game-history', JSON.stringify(games));
		// Reset uno shouts for new game
		uno1 = 0;
		uno2 = 0;
		localStorage.setItem('uno-shout1', '0');
		localStorage.setItem('uno-shout2', '0');
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
	let timerInterval: number | null | undefined = null;
	let timerRunning = false;
	let prevGameDuration: string | number | null = null; // duration of previous game

	// Last winner state
	let lastWinner: string | null = null;
	let lastWinTime: number | null = null;

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
	function formatTime(s: number) {
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

	function handleKey(e: { key: string }) {
		if (e.key === 'a' || e.key === 'A') inc1();
		if (e.key === 'l' || e.key === 'L') inc2();
		if (e.key === 'q' || e.key === 'Q') incUno1();
		if (e.key === 'p' || e.key === 'P') incUno2();
	}

	function saveTimerToSet(winner: string | null) {
		if (typeof window !== 'undefined') {
			// Save only the last game duration and winner
			prevGameDuration = timer;
			localStorage.setItem('uno-prev-game-duration', String(prevGameDuration));
			lastWinner = winner;
			lastWinTime = timer;
			saveGameToHistory(winner);
		}
	}

	function inc1() {
		player1++;
		if (typeof window !== 'undefined') {
			localStorage.setItem('uno-player1', String(player1));
		}
		saveTimerToSet('player1');
		resetTimer();
	}
	function inc2() {
		player2++;
		if (typeof window !== 'undefined') {
			localStorage.setItem('uno-player2', String(player2));
		}
		saveTimerToSet('player2');
		resetTimer();
	}
	function incUno1() {
		uno1++;
		if (typeof window !== 'undefined') {
			localStorage.setItem('uno-shout1', String(uno1));
		}
	}
	function incUno2() {
		uno2++;
		if (typeof window !== 'undefined') {
			localStorage.setItem('uno-shout2', String(uno2));
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

<main
	class="flex h-full min-h-screen flex-col items-center justify-center gap-2 bg-black font-sans"
>
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

	<LastWinner {lastWinner} {prevGameDuration} {formatTime} />

	<div class="flex flex-col gap-8 md:flex-row md:gap-16">
		<PlayerCard
			name="Felicia"
			score={player1}
			unoCount={uno1}
			onVictory={inc1}
			onUno={incUno1}
			isWinner={player1 > player2}
			color="red"
			victoryKey="A"
		/>
		<PlayerCard
			name="Jorge"
			score={player2}
			unoCount={uno2}
			onVictory={inc2}
			onUno={incUno2}
			isWinner={player2 > player1}
			unoColor="yellow"
			victoryKey="L"
		/>
	</div>
</main>

<StatisticsSection {avgDuration} {streaks} {formatTime} {gameStats} />
