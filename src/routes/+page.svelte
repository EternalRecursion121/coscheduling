<script>
  import { onMount } from 'svelte';
  import confetti from 'canvas-confetti';
  import { oxfordLibraries } from '$lib/libraries.js';

  let visitedLibraries = new Set();
  let importString = '';
  let showImport = false;
  let showExportMessage = false;
  let exportString = '';
  let previousProgress = 0;
  let searchQuery = '';
  let plusPressed = false;

  onMount(() => {
    const saved = localStorage.getItem('visitedLibraries');
    if (saved) {
      visitedLibraries = new Set(JSON.parse(saved));
      previousProgress = visitedLibraries.size;
    }

    window.addEventListener('keydown', handleKeyDown);

    return () => {
      window.removeEventListener('keydown', handleKeyDown);
    };
  });

  function handleKeyDown(event) {
    if (event.key === '+') {
      event.preventDefault();
      selectAllLibraries();
    } else if (event.key === '-') {
      event.preventDefault();
      removeAllLibraries();
    }
  }

  function removeAllLibraries() {
    visitedLibraries = new Set();
    localStorage.setItem('visitedLibraries', JSON.stringify([...visitedLibraries]));
    previousProgress = 0;
  }

  function selectAllLibraries() {
    const allButFirst = oxfordLibraries.slice(1);
    visitedLibraries = new Set(allButFirst);
    localStorage.setItem('visitedLibraries', JSON.stringify([...visitedLibraries]));
    
    previousProgress = visitedLibraries.size;
  }

  function celebrate() {
    const duration = 3 * 1000;
    const end = Date.now() + duration;

    // Gold and Oxford Blue colors
    const colors = ['#002147', '#B59410'];

    (function frame() {
      confetti({
        particleCount: 2,
        angle: 60,
        spread: 55,
        origin: { x: 0 },
        colors: colors
      });
      confetti({
        particleCount: 2,
        angle: 120,
        spread: 55,
        origin: { x: 1 },
        colors: colors
      });

      if (Date.now() < end) {
        requestAnimationFrame(frame);
      }
    }());
  }

  function toggleLibrary(library) {
    if (visitedLibraries.has(library)) {
      visitedLibraries.delete(library);
    } else {
      visitedLibraries.add(library);
    }
    visitedLibraries = visitedLibraries;
    localStorage.setItem('visitedLibraries', JSON.stringify([...visitedLibraries]));

    // Check if we just reached 100%
    if (visitedLibraries.size === oxfordLibraries.length && previousProgress !== oxfordLibraries.length) {
      celebrate();
    }
    previousProgress = visitedLibraries.size;
  }

  function handleExport() {
    exportString = btoa(JSON.stringify([...visitedLibraries]));
    navigator.clipboard.writeText(exportString);
    showExportMessage = true;
    setTimeout(() => {
      showExportMessage = false;
    }, 2000);
  }

  function handleImport() {
    showImport = !showImport;
    importString = '';
  }

  function importProgress() {
    try {
      const decoded = JSON.parse(atob(importString));
      visitedLibraries = new Set(decoded);
      localStorage.setItem('visitedLibraries', JSON.stringify([...visitedLibraries]));
      importString = '';
      showImport = false;
    } catch (error) {
      alert('Invalid import string');
    }
  }

  $: progressPercentage = (visitedLibraries.size / oxfordLibraries.length) * 100;
  $: filteredLibraries = oxfordLibraries.filter(library => 
    library.toLowerCase().includes(searchQuery.toLowerCase())
  );
</script>

<div class="min-h-screen bg-[#f5f3ef] py-8 px-4 sm:px-6 lg:px-8 font-serif">
  <div class="max-w-4xl mx-auto">
    <div class="bg-white shadow-md rounded-lg p-8 border-t-4 border-[#002147]">
      <header class="mb-12 text-center">
        <h1 class="text-4xl font-bold text-[#002147] mb-3 font-serif tracking-tight">Oxford Libraries</h1>
        <div class="w-16 h-1 bg-[#002147] mx-auto mb-4"></div>
      </header>
      
      <div class="mb-10 bg-[#f8f7f5] p-6 rounded-lg border border-gray-200">
        <div class="flex flex-col sm:flex-row items-center justify-between gap-4 mb-6">
          <div class="text-center sm:text-left">
            <p class="text-sm uppercase tracking-wider text-gray-500 mb-1">Progress</p>
            <p class="text-2xl font-bold text-[#002147]">
              {visitedLibraries.size} <span class="text-gray-400">/</span> {oxfordLibraries.length}
            </p>
          </div>
          <div class="flex gap-3">
            <button
              on:click={handleExport}
              class="px-4 py-2 bg-[#002147] text-white rounded hover:bg-[#003167] transition-colors duration-200 text-sm tracking-wide"
            >
              Export Progress
            </button>
            <button
              on:click={handleImport}
              class="px-4 py-2 border border-[#002147] text-[#002147] rounded hover:bg-[#002147] hover:text-white transition-colors duration-200 text-sm tracking-wide"
            >
              Import Progress
            </button>
          </div>
        </div>

        <div class="mb-6">
          <div class="relative">
            <input
              type="text"
              bind:value={searchQuery}
              placeholder="Search libraries..."
              class="w-full px-4 py-2.5 pl-10 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-[#002147] bg-white text-sm"
            />
            <svg 
              class="absolute left-3 top-1/2 transform -translate-y-1/2 w-4 h-4 text-gray-400" 
              fill="none" 
              stroke="currentColor" 
              viewBox="0 0 24 24"
            >
              <path 
                stroke-linecap="round" 
                stroke-linejoin="round" 
                stroke-width="2" 
                d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"
              />
            </svg>
          </div>
          {#if searchQuery && filteredLibraries.length === 0}
            <p class="mt-2 text-sm text-gray-500">No libraries found matching "{searchQuery}"</p>
          {/if}
          {#if searchQuery && filteredLibraries.length > 0}
            <p class="mt-2 text-sm text-gray-500">Found {filteredLibraries.length} {filteredLibraries.length === 1 ? 'library' : 'libraries'}</p>
          {/if}
        </div>

        {#if showExportMessage && exportString}
          <div class="mb-4 p-4 bg-white border border-gray-200 rounded">
            <p class="text-sm text-gray-600 mb-2">Your progress string:</p>
            <p class="text-xs font-mono bg-gray-50 p-2 rounded break-all">{exportString}</p>
          </div>
        {/if}

        {#if showImport}
          <div class="flex gap-2">
            <input
              type="text"
              bind:value={importString}
              placeholder="Paste import string here"
              class="flex-1 px-4 py-2.5 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-[#002147] bg-white text-sm"
            />
            <button
              on:click={importProgress}
              class="px-4 py-2 bg-[#002147] text-white rounded hover:bg-[#003167] transition-colors duration-200 text-sm tracking-wide whitespace-nowrap"
            >
              Import
            </button>
          </div>
        {/if}
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
        {#each filteredLibraries as library}
          <div
            class="group flex items-center p-5 border rounded-lg cursor-pointer transition-all duration-200 hover:shadow-md
            {visitedLibraries.has(library) ? 
              'bg-[#002147]/5 border-[#002147]/20' : 
              'bg-white border-gray-200 hover:border-[#002147]/30'}"
            on:click={() => toggleLibrary(library)}
          >
            <div class="flex-1">
              <h3 class="text-lg font-medium text-gray-900 group-hover:text-[#002147] transition-colors duration-200">
                {#if searchQuery}
                  {@html library.replace(
                    new RegExp(searchQuery, 'gi'),
                    match => `<span class="bg-yellow-100">${match}</span>`
                  )}
                {:else}
                  {library}
                {/if}
              </h3>
            </div>
            <div class="ml-4">
              {#if visitedLibraries.has(library)}
                <svg class="w-6 h-6 text-[#002147]" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
                </svg>
              {:else}
                <div class="w-6 h-6 rounded-full border-2 border-gray-200 group-hover:border-[#002147] transition-colors duration-200"></div>
              {/if}
            </div>
          </div>
        {/each}
      </div>
    </div>
  </div>
</div>
