<script lang="ts">
	export let avgDuration: number;
	export let streaks: { Felicia: number; Jorge: number; maxPlayer: string; max: number };
	export let formatTime: (duration: number) => string;
	export let gameStats: { uno1?: number; uno2?: number }[];
</script>

<div
	class="m-4 mx-auto max-w-2xl min-w-72 items-center rounded-xl border-2 border-white bg-black/80 p-4"
>
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
			>
				{streaks.Felicia > streaks.Jorge
					? `Felicia (${streaks.Felicia})`
					: `Jorge (${streaks.Jorge})`}
			</span>
		</div>
		<div class="flex flex-col items-center">
			<span class="text-lg font-semibold text-white">Longest Streak</span>
			<span
				class="text-2xl font-bold {streaks.maxPlayer === 'Felicia'
					? 'text-red-400'
					: 'text-yellow-300'}">{streaks.maxPlayer} ({streaks.max})</span
			>
		</div>
		<div class="flex flex-col items-center">
			<span class="text-lg font-semibold text-white">Avg. UNO shouts p/game</span>
			<span class="font-mono text-2xl text-blue-300">
				{#if gameStats && gameStats.length > 0}
					{(() => {
						const totalShouts = gameStats.reduce(
							(a: number, g: { uno1?: number; uno2?: number }) => a + (g.uno1 ?? 0) + (g.uno2 ?? 0),
							0
						);
						console.log('Total Shouts:', totalShouts, 'Game Stats:', gameStats);
						const totalGames = gameStats.filter(
							(g: { uno1?: number; uno2?: number }) => (g.uno1 ?? 0) > 0 || (g.uno2 ?? 0) > 0
						).length;
						return totalGames > 0 ? Math.round(totalShouts / totalGames) : 0;
					})()}
				{:else}
					0
				{/if}
			</span>
		</div>
	</div>
	<div class="h-64 w-full rounded-lg border border-white bg-black p-2">
		<canvas id="gameChart" class="h-full w-full"></canvas>
	</div>
</div>
