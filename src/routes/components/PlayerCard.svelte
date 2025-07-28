<script lang="ts">
	import { createEventDispatcher } from 'svelte';
	export let name: string;
	export let score: number;
	export let unoCount: number;
	export let onVictory: () => void;
	export let onUno: () => void;
	export let isWinner: boolean = false;
	export let color: 'red' | 'yellow' = 'red';
	export let victoryKey: string = '';

	let editing = false;
	let tempName: string = name;
	const dispatch = createEventDispatcher();

	$: if (!editing) tempName = name;

	function startEdit() {
		editing = true;
		tempName = name;
		setTimeout(() => {
			const input = document.getElementById(
				'player-name-input-' + victoryKey
			) as HTMLInputElement | null;
			if (input) input.focus();
		}, 0);
	}
	function finishEdit() {
		editing = false;
		const newName = tempName.trim();
		if (newName && newName !== name) {
			dispatch('nameChange', newName);
		}
		tempName = name; // reset if not changed
	}
	function handleKey(e: KeyboardEvent) {
		if (e.key === 'Enter') {
			(e.target as HTMLInputElement).blur();
		}
		if (e.key === 'Escape') {
			editing = false;
			tempName = name;
		}
	}
</script>

<div
	class="relative flex w-full min-w-72 flex-col items-center rounded-2xl border-4 border-white bg-gradient-to-br p-4 shadow-2xl"
	class:bg-gradient-to-br={color === 'red'}
	class:from-red-600={color === 'red'}
	class:to-yellow-400={color === 'red'}
	class:from-yellow-400={color === 'yellow'}
	class:to-red-600={color === 'yellow'}
>
	{#if isWinner}
		<div class="absolute -top-15 -right-10 z-10 rotate-30 text-8xl">👑</div>
	{/if}
	<button
		type="button"
		class="group mb-2 cursor-pointer border-none bg-transparent p-0 text-2xl font-extrabold drop-shadow outline-none"
		class:text-white={color === 'red'}
		class:text-black={color === 'yellow'}
		style={color === 'red' ? 'text-shadow: 1px 1px 0 #e3342f;' : 'text-shadow: 1px 1px 0 #fff;'}
		on:click={startEdit}
		on:mouseenter={() => (editing = false)}
		aria-label="Edit player name"
	>
		{#if editing}
			<input
				id={'player-name-input-' + victoryKey}
				class="rounded border border-yellow-300 bg-black/60 px-2 py-1 text-xl font-bold focus:ring-2 focus:ring-yellow-400 focus:outline-none"
				bind:value={tempName}
				on:blur={finishEdit}
				on:keydown={handleKey}
				on:click|stopPropagation
				on:mousedown|stopPropagation
				on:dblclick|stopPropagation
				style="width: 90%"
			/>
		{:else}
			<span
				class="cursor-pointer transition group-hover:underline group-hover:decoration-yellow-300"
				>{name}</span
			>
		{/if}
	</button>
	<div
		class="mb-4 text-6xl font-black drop-shadow"
		class:text-white={color === 'red'}
		class:text-black={color === 'yellow'}
		style={color === 'red' ? 'text-shadow: 2px 2px 0 #e3342f;' : 'text-shadow: 2px 2px 0 #fff;'}
	>
		{score}
	</div>
	<button
		on:click={onVictory}
		class="w-full rounded-lg border-2 border-white px-8 py-1 text-xl font-extrabold shadow-lg transition-all duration-150 focus:outline-none"
		class:bg-red-600={color === 'red'}
		class:text-white={color === 'red'}
		class:hover\:bg-yellow-400={color === 'red'}
		class:hover\:text-black={color === 'red'}
		class:focus\:ring-4={color === 'red'}
		class:focus\:ring-yellow-400={color === 'red'}
		class:bg-yellow-400={color === 'yellow'}
		class:text-black={color === 'yellow'}
		class:hover\:bg-red-600={color === 'yellow'}
		class:hover\:text-white={color === 'yellow'}
		class:focus\:ring-red-400={color === 'yellow'}
	>
		Victory <div
			class="text-sm"
			class:text-white={color === 'red'}
			class:text-black={color === 'yellow'}
		>
			(Press <span
				class="font-mono font-semibold"
				class:text-yellow-300={color === 'red'}
				class:text-red-500={color === 'yellow'}>{victoryKey}</span
			>)
		</div>
	</button>
	<div class="mt-6 flex flex-col items-center">
		<div
			class="mb-1 text-lg font-bold"
			class:text-yellow-300={color === 'red'}
			class:text-red-600={color === 'yellow'}
		>
			UNO Shouts
		</div>
		<div class="flex items-center gap-2">
			<span
				class="text-3xl font-black"
				class:text-white={color === 'red'}
				class:text-black={color === 'yellow'}
				style={color === 'red' ? 'text-shadow: 1px 1px 0 #e3342f;' : 'text-shadow: 1px 1px 0 #fff;'}
				>{unoCount}</span
			>
			<button
				on:click={onUno}
				class="rounded border-2 border-white px-3 py-1 text-lg font-extrabold shadow transition-all duration-150 focus:outline-none"
				class:bg-yellow-400={color === 'red'}
				class:text-black={color === 'red'}
				class:hover\:bg-red-600={color === 'red'}
				class:hover\:text-white={color === 'red'}
				class:focus\:ring-2={color === 'red'}
				class:focus\:ring-red-400={color === 'red'}
				class:bg-red-600={color === 'yellow'}
				class:text-white={color === 'yellow'}
				class:hover\:bg-yellow-400={color === 'yellow'}
				class:hover\:text-black={color === 'yellow'}
				class:focus\:ring-yellow-400={color === 'yellow'}
			>
				UNO!
			</button>
		</div>
		<div
			class="mt-1 text-xs"
			class:text-white={color === 'red'}
			class:text-black={color === 'yellow'}
		>
			(Press <span
				class="font-mono font-semibold"
				class:text-yellow-300={color === 'red'}
				class:text-red-600={color === 'yellow'}>{color === 'red' ? 'Q' : 'P'}</span
			>)
		</div>
	</div>
</div>
