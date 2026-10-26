<script>
  let { children, author = "Anonymous" } = $props();
  let quoteElement = $state();
  let canShare = $state(false);
  let copied = $state(false);

  $effect(() => {
    canShare = !!navigator.share;
  });

  function getText() {
    return `"${quoteElement.innerText.trim()}" — ${author}`;
  }

  async function notify(message) {
    if (!("Notification" in window)) return;
    let permission = Notification.permission;
    if (permission === "default") {
      permission = await Notification.requestPermission();
    }
    if (permission === "granted") new Notification(message);
  }

  async function copyQuote() {
    await navigator.clipboard.writeText(getText());
    copied = true;
    setTimeout(() => (copied = false), 2000);
    await notify("Quote copied to clipboard!");
  }

  async function shareQuote() {
    if (!navigator.share) return copyQuote();
    try {
      await navigator.share({
        title: "Please check out this quote",
        text: getText(),
        url: window.location.href
      });
    } catch (err) {
      if (err.name !== "AbortError") console.error("Share failed:", err);
    }
  }
</script>

<figure
  class="group relative mx-auto max-w-xl overflow-hidden rounded-2xl bg-white p-8 shadow-lg ring-1 ring-slate-200 transition hover:-translate-y-1 hover:shadow-xl"
>
  <!-- accent bar -->
  <div class="absolute inset-x-0 top-0 h-1 bg-gradient-to-r from-blue-500 via-indigo-500 to-purple-500"></div>

  <!-- big quote mark -->
  <svg class="absolute right-6 top-6 h-16 w-16 text-slate-100" fill="currentColor" viewBox="0 0 24 24">
    <path d="M9.983 3v7.391c0 5.704-3.731 9.57-8.983 10.609l-.995-2.151c2.432-.917 3.995-3.638 3.995-5.849h-4v-10h9.983zm14.017 0v7.391c0 5.704-3.748 9.571-9 10.609l-.996-2.151c2.433-.917 3.996-3.638 3.996-5.849h-3.983v-10h9.983z"/>
  </svg>

  <blockquote
    bind:this={quoteElement}
    class="relative text-xl font-medium leading-relaxed text-slate-800 sm:text-2xl"
  >
    {@render children?.()}
  </blockquote>

  <figcaption class="relative mt-6 flex items-center justify-between gap-4">
    <div class="flex items-center gap-3">
      <div
        class="flex h-10 w-10 items-center justify-center rounded-full bg-gradient-to-br from-blue-500 to-purple-500 font-semibold text-white"
      >
        {author.charAt(0).toUpperCase()}
      </div>
      <span class="font-semibold text-slate-700">{author}</span>
    </div>

    <div class="flex gap-2">
      <button
        onclick={copyQuote}
        class="flex items-center gap-1.5 rounded-full bg-slate-100 px-4 py-2 text-sm font-medium text-slate-700 transition hover:bg-slate-200 active:scale-95"
      >
        {#if copied}
          <svg class="h-4 w-4 text-green-600" fill="none" stroke="currentColor" stroke-width="2.5" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" d="M5 13l4 4L19 7" />
          </svg>
          Copied!
        {:else}
          <svg class="h-4 w-4" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
            <rect x="9" y="9" width="13" height="13" rx="2" />
            <path d="M5 15H4a2 2 0 01-2-2V4a2 2 0 012-2h9a2 2 0 012 2v1" />
          </svg>
          Copy
        {/if}
      </button>

      <button
        onclick={shareQuote}
        class="flex items-center gap-1.5 rounded-full bg-blue-600 px-4 py-2 text-sm font-medium text-white shadow-sm transition hover:bg-blue-700 active:scale-95"
      >
        <svg class="h-4 w-4" fill="none" stroke="currentColor" stroke-width="2" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" d="M4 12v7a2 2 0 002 2h12a2 2 0 002-2v-7M16 6l-4-4-4 4M12 2v13" />
        </svg>
        {canShare ? "Share" : "Share (copy)"}
      </button>
    </div>
  </figcaption>
</figure>