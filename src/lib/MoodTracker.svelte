<script>
  import { onMount } from 'svelte';

  let moodData = $state([]);
  let currentMood = $state(5);
  let note = $state('');
  let isLoaded = $state(false);

  onMount(() => {
    const saved = localStorage.getItem('mood_logs');
    if (saved) {
      moodData = JSON.parse(saved);
    }
    isLoaded = true;
  });

  const getMoodColor = (val) => {
    // Ensuring the colors are "loud" by keeping green at 0 and scaling R and B
    const red = Math.floor(255 - (val - 1) * 28);
    const blue = Math.floor((val - 1) * 28);
    return `rgb(${red}, 0, ${blue})`;
  };

  const saveEntry = () => {
    const newEntry = {
      date: new Date().toLocaleDateString(),
      timestamp: new Date().toISOString(),
      mood: currentMood,
      note: note,
    };

    moodData = [...moodData, newEntry];
    localStorage.setItem('mood_logs', JSON.stringify(moodData));
    note = '';
  };

  const exportJSON = () => {
    const dataStr = `data:text/json;charset=utf-8,${encodeURIComponent(JSON.stringify(moodData, null, 2))}`;
    const downloadAnchorNode = document.createElement('a');
    downloadAnchorNode.setAttribute('href', dataStr);
    downloadAnchorNode.setAttribute('download', 'mood_history.json');
    document.body.appendChild(downloadAnchorNode);
    downloadAnchorNode.click();
    downloadAnchorNode.remove();
  };
</script>

{#if isLoaded}
  <div class="mx-auto mt-10 mb-20 max-w-2xl rounded-3xl border border-gray-100 bg-white p-8 shadow-2xl">
    <h1 class="mb-8 text-center text-4xl font-black tracking-tighter text-gray-900 uppercase">Mood Tracker</h1>

    <div class="mb-10 flex flex-col items-center justify-center rounded-2xl border-4 border-dashed border-gray-200 bg-gray-50 py-10">
      <div
        class="leading-none font-black transition-all duration-150 select-none"
        style="color: {getMoodColor(currentMood)}; font-size: 10rem; text-shadow: 0 10px 30px {getMoodColor(currentMood)}44;"
      >
        {currentMood}
      </div>
      <p class="mt-4 text-sm font-bold tracking-widest text-gray-400 uppercase">Current Vibration</p>
    </div>

    <div class="mb-12">
      <input
        type="range"
        min="1"
        max="10"
        bind:value={currentMood}
        class="h-6 w-full cursor-pointer appearance-none rounded-full bg-gray-200 accent-current transition-colors"
        style="color: {getMoodColor(currentMood)}"
      />
      <div class="mt-4 flex justify-between px-2 text-2xl font-black uppercase italic">
        <span style="color: {getMoodColor(1)}">Low</span>
        <span style="color: {getMoodColor(10)}">Peak</span>
      </div>
    </div>

    <div class="mb-6">
      <textarea
        bind:value={note}
        placeholder="Why do you feel this way? Log it..."
        class="w-full resize-none rounded-2xl border-2 border-gray-200 p-5 text-lg transition-all outline-none focus:border-gray-800"
        rows="3"></textarea>
    </div>

    <div class="flex gap-4">
      <button
        onclick={saveEntry}
        type="button"
        class="flex-[2] rounded-2xl bg-gray-900 py-5 text-xl font-black tracking-wider text-white uppercase shadow-xl transition-all hover:scale-[1.02] active:scale-95"
      >
        Log Entry
      </button>
      <button
        onclick={exportJSON}
        type="button"
        class="flex-1 rounded-2xl border-2 border-gray-200 text-xs font-bold text-gray-500 uppercase transition-all hover:bg-gray-50"
      >
        Export
      </button>
    </div>

    <div class="mt-16 border-t-4 border-double border-gray-100 pt-10">
      <h2 class="mb-6 text-2xl font-black text-gray-800 uppercase">The Archive</h2>
      <div class="space-y-4">
        {#each moodData.slice().reverse() as entry}
          <div class="flex items-center gap-6 rounded-2xl border-2 border-gray-50 bg-white p-5 shadow-sm">
            <div
              class="flex h-14 w-14 flex-shrink-0 items-center justify-center rounded-xl text-2xl font-black text-white shadow-lg"
              style="background-color: {getMoodColor(entry.mood)}"
            >
              {entry.mood}
            </div>
            <div class="flex-1">
              <p class="mb-1 text-[10px] font-black tracking-widest text-gray-400 uppercase">{entry.date}</p>
              <p class="leading-tight font-medium text-gray-800">{entry.note || 'No context provided.'}</p>
            </div>
          </div>
        {/each}
      </div>
    </div>
  </div>
{:else}
  <div class="flex min-h-screen items-center justify-center text-2xl font-black tracking-tighter text-gray-300 uppercase">Syncing...</div>
{/if}
