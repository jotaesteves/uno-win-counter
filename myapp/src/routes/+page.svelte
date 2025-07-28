<script>

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

  // Restore from localStorage on mount
  import { onMount, onDestroy } from 'svelte';
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
    }
  });
  onDestroy(() => {
    if (typeof window !== 'undefined') {
      window.removeEventListener('keydown', handleKey);
    }
    if (timerInterval) clearInterval(timerInterval);
  });
</script>

<main class="min-h-screen flex flex-col items-center justify-center gap-2 bg-black font-sans">
  <h1 class="text-4xl md:text-5xl font-extrabold text-yellow-400 tracking-tight mb-2 drop-shadow-lg uppercase" style="text-shadow: 2px 2px 0 #e3342f, 4px 4px 0 #fff;">UNO Victory Counter</h1>


  <div class="flex flex-col items-center mb-4">
    <div class="flex items-center gap-4">
      <span class="text-3xl md:text-4xl font-mono font-bold text-white bg-black border-2 border-white rounded-lg px-6 py-2 shadow-lg tracking-widest">{formatTime(timer)}</span>
    </div>
    <div class="text-xs text-gray-300 mt-1">Current Game Timer</div>
  </div>

  {#if lastWinner}
    <div class="mb-2 flex flex-col items-center">
      <span class="text-lg font-bold text-white bg-black border-2 border-white rounded-lg px-4 py-1 shadow">Last Winner: <span class={lastWinner === 'player1' ? 'text-red-400' : 'text-yellow-300'}>{lastWinner === 'player1' ? 'Felicia' : 'Jorge'}</span></span>
      <span class="text-xs text-green-300 mt-1">Previous game duration: {formatTime(prevGameDuration ?? 0)}</span>
    </div>
  {/if}

<div class="flex flex-col md:flex-row gap-8 md:gap-16">
  <div class="flex flex-col items-center bg-gradient-to-br from-red-600 to-yellow-400 rounded-2xl shadow-2xl p-8 w-72 border-4 border-white">
    <h2 class="text-2xl font-extrabold text-white mb-2 drop-shadow" style="text-shadow: 1px 1px 0 #e3342f;">Felicia</h2>
    <div class="text-6xl font-black text-white mb-4 drop-shadow" style="text-shadow: 2px 2px 0 #e3342f;">{player1}</div>
    <button on:click={inc1} class="px-8 py-3 bg-red-600 hover:bg-yellow-400 text-white hover:text-black text-xl font-extrabold rounded-lg shadow-lg border-2 border-white transition-all duration-150 focus:outline-none focus:ring-4 focus:ring-yellow-400">+1</button>
    <div class="mt-3 text-sm text-white">(Press <span class="font-mono font-semibold text-yellow-300">A</span>)</div>
    <div class="flex flex-col items-center mt-6">
      <div class="text-lg font-bold text-yellow-300 mb-1">UNO Shouts</div>
      <div class="flex items-center gap-2">
        <span class="text-3xl font-black text-white" style="text-shadow: 1px 1px 0 #e3342f;">{uno1}</span>
        <button on:click={incUno1} class="px-3 py-1 bg-yellow-400 hover:bg-red-600 text-black hover:text-white text-lg font-extrabold rounded shadow border-2 border-white transition-all duration-150 focus:outline-none focus:ring-2 focus:ring-red-400">UNO!</button>
      </div>
      <div class="text-xs text-white mt-1">(Press <span class="font-mono font-semibold text-yellow-300">Q</span>)</div>
    </div>
  </div>
  <div class="flex flex-col items-center bg-gradient-to-br from-yellow-400 to-red-600 rounded-2xl shadow-2xl p-8 w-72 border-4 border-white">
    <h2 class="text-2xl font-extrabold text-black mb-2 drop-shadow" style="text-shadow: 1px 1px 0 #fff;">Jorge</h2>
    <div class="text-6xl font-black text-black mb-4 drop-shadow" style="text-shadow: 2px 2px 0 #fff;">{player2}</div>
    <button on:click={inc2} class="px-8 py-3 bg-yellow-400 hover:bg-red-600 text-black hover:text-white text-xl font-extrabold rounded-lg shadow-lg border-2 border-white transition-all duration-150 focus:outline-none focus:ring-4 focus:ring-red-400">+1</button>
    <div class="mt-3 text-sm text-black">(Press <span class="font-mono font-semibold text-red-600">L</span>)</div>
    <div class="flex flex-col items-center mt-6">
      <div class="text-lg font-bold text-red-600 mb-1">UNO Shouts</div>
      <div class="flex items-center gap-2">
        <span class="text-3xl font-black text-black" style="text-shadow: 1px 1px 0 #fff;">{uno2}</span>
        <button on:click={incUno2} class="px-3 py-1 bg-red-600 hover:bg-yellow-400 text-white hover:text-black text-lg font-extrabold rounded shadow border-2 border-white transition-all duration-150 focus:outline-none focus:ring-2 focus:ring-yellow-400">UNO!</button>
      </div>
      <div class="text-xs text-black mt-1">(Press <span class="font-mono font-semibold text-red-600">P</span>)</div>
    </div>
  </div>
</div>
</main>
