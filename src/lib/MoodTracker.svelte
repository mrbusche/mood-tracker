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
    return `rgb(${red}, 20, ${blue})`;
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
  <div class="max-w-2xl mx-auto p-8 bg-white rounded-3xl shadow-2xl mt-10 border border-gray-100 mb-20">
    <h1 class="text-4xl font-black text-gray-900 mb-8 text-center uppercase tracking-tighter">Mood Tracker</h1>

    <div class="flex flex-col items-center justify-center py-10 mb-10 rounded-2xl bg-gray-50 border-4 border-dashed border-gray-200">
      <div
        class="font-black transition-all duration-150 leading-none select-none"
        style="color: {getMoodColor(currentMood)}; font-size: 10rem; text-shadow: 0 10px 30px {getMoodColor(currentMood)}44;"
      >
        {currentMood}
      </div>
      <p class="text-sm font-bold uppercase tracking-widest mt-4 text-gray-400">Current Vibration</p>
    </div>

    <div class="mb-12">
      <input
        type="range"
        min="1"
        max="10"
        bind:value={currentMood}
        class="w-full h-6 bg-gray-200 rounded-full appearance-none cursor-pointer accent-current transition-colors"
        style="color: {getMoodColor(currentMood)}"
      />
      <div class="flex justify-between mt-4 px-2 font-black text-2xl uppercase italic">
        <span style="color: {getMoodColor(1)}">Low</span>
        <span style="color: {getMoodColor(10)}">Peak</span>
      </div>
    </div>

    <div class="mb-6">
      <textarea
        bind:value={note}
        placeholder="Why do you feel this way? Log it..."
        class="w-full p-5 text-lg border-2 border-gray-200 rounded-2xl focus:border-gray-800 outline-none transition-all resize-none"
        rows="3"
      ></textarea>
    </div>

    <div class="flex gap-4">
      <button
        onclick={saveEntry}
        type="button"
        class="flex-[2] bg-gray-900 text-white py-5 rounded-2xl font-black text-xl uppercase tracking-wider hover:scale-[1.02] active:scale-95 transition-all shadow-xl"
      >
        Log Entry
      </button>
      <button
        onclick={exportJSON}
        type="button"
        class="flex-1 border-2 border-gray-200 rounded-2xl font-bold text-gray-500 hover:bg-gray-50 transition-all uppercase text-xs"
      >
        Export
      </button>
    </div>

    <div class="mt-16 border-t-4 border-double border-gray-100 pt-10">
      <h2 class="text-2xl font-black text-gray-800 mb-6 uppercase">The Archive</h2>
      <div class="space-y-4">
        {#each moodData.slice().reverse() as entry}
          <div class="p-5 rounded-2xl bg-white border-2 border-gray-50 shadow-sm flex items-center gap-6">
            <div
              class="w-14 h-14 rounded-xl flex-shrink-0 flex items-center justify-center text-white text-2xl font-black shadow-lg"
              style="background-color: {getMoodColor(entry.mood)}"
            >
              {entry.mood}
            </div>
            <div class="flex-1">
              <p class="text-[10px] font-black text-gray-400 uppercase tracking-widest mb-1">{entry.date}</p>
              <p class="text-gray-800 font-medium leading-tight">{entry.note || 'No context provided.'}</p>
            </div>
          </div>
        {/each}
      </div>
    </div>
  </div>
{:else}
  <div class="flex items-center justify-center min-h-screen text-2xl font-black text-gray-300 uppercase tracking-tighter">Syncing...</div>
{/if}
