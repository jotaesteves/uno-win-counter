<script>
	export let lastWinner;
	export let prevGameDuration;
	export let formatTime;
</script>

{#if lastWinner}
	<div class="mb-2 flex flex-col items-center">
		<span
			class="rounded-lg border-2 border-white bg-black px-4 py-1 text-lg font-bold text-white shadow"
		>
			Last Winner: <span class={lastWinner === 'player1' ? 'text-red-400' : 'text-yellow-300'}
				>{lastWinner === 'player1' ? 'Felicia' : 'Jorge'}</span
			>
		</span>
		<span class="mt-1 text-xs text-green-300"
			>Previous game duration: {formatTime(prevGameDuration ?? 0)}</span
		>
	</div>
{/if}
