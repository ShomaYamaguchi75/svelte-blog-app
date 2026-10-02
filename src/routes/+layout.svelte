<script>
  import "./layout.css";
  let { children } = $props();
</script>

<div
  class="min-h-screen bg-neutral-950 text-neutral-100 font-sans selection:bg-cyan-500/30 selection:text-cyan-200"
>
  <header
    class="border-b border-neutral-800/80 bg-neutral-900/50 backdrop-blur-md sticky top-0 z-40"
  >
    <div class="max-w-5xl mx-auto px-6 h-16 flex items-center justify-between">
      <div class="flex items-center gap-2">
        <div
          class="w-3 h-3 rounded-full bg-cyan-400 shadow-[0_0_10px_#22d3ee]"
        ></div>
        <span class="font-bold tracking-tight text-lg text-white"
          >DevPulse<span class="text-cyan-400">.log</span></span
        >
      </div>
      <span
        class="text-xs text-neutral-400 bg-neutral-800/80 px-2.5 py-1 rounded-full border border-neutral-700/50"
      >
        API Demo
      </span>
    </div>
  </header>

  <main class="max-w-5xl mx-auto px-6 py-10">
    {@render children()}
  </main>
</div>
