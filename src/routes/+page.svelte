<script>
  import Quote from "#lib/components/Quote.svelte";
  import { flip } from "svelte/animate";

  let lists = $state({
    all: [
      { id: 1, author: "Steve Jobs", text: "The only way to do great work is to love what you do." },
      { id: 2, author: "Rei Alijaj", text: "Don't stop when you are tired. Stop when you are done." },
      { id: 3, author: "Paul Opitz", text: "Code is like humor. When you have to explain it, it's bad.😈" },
      { id: 4, author: "John Johnson", text: "First, solve the problem. Then, write the code." },
      { id: 5, author: "Ron Volaj", text: "Don't die with a dream. Die with a story." }
    ],
    favorites: []
  });

  const columns = [
    { key: "all", title: "Alle Zitate", color: "bg-red-100", ring: "ring-red-400" },
    { key: "favorites", title: "Favoriten ⭐", color: "bg-green-100", ring: "ring-green-400" }
  ];

  let dragged = $state(null); 
  let overColumn = $state(null);

  function onDragStart(e, id, from) {
    dragged = { id, from };
    e.dataTransfer.effectAllowed = "move";
    e.dataTransfer.setData("text/plain", String(id)); 
  }

  function onDragEnd() {
    dragged = null;
    overColumn = null;
  }

  function drop(to, index = lists[to].length) {
    if (!dragged) return;
    const { id, from } = dragged;

    const fromIndex = lists[from].findIndex((q) => q.id === id);
    const [item] = lists[from].splice(fromIndex, 1);

    
    if (from === to && fromIndex < index) index--;

    lists[to].splice(index, 0, item);
    onDragEnd();
  }
</script>

<main class="min-h-screen bg-gradient-to-br from-slate-50 via-blue-50 to-indigo-100 px-4 py-16">
  <div class="mx-auto max-w-6xl">
    <h1 class="mb-2 text-center text-4xl font-bold tracking-tight text-slate-900">
      Daily Quotes
    </h1>
    <p class="mb-12 text-center text-slate-500">
      Zieh die Zitate in deine Favoriten oder ändere die Reihenfolge.
    </p>

    <div class="grid gap-6 md:grid-cols-2">
      {#each columns as col (col.key)}
        <section
          role="list"
          class="min-h-64 rounded-3xl p-4 transition {col.color}
                 {overColumn === col.key ? `ring-4 ${col.ring}` : ''}"
          ondragover={(e) => { e.preventDefault(); overColumn = col.key; }}
          ondragleave={(e) => { if (!e.currentTarget.contains(e.relatedTarget)) overColumn = null; }}
          ondrop={(e) => { e.preventDefault(); drop(col.key); }}
        >
          <h2 class="mb-4 flex items-center justify-between px-2 text-lg font-semibold text-slate-700">
            {col.title}
            <span class="rounded-full bg-white/70 px-3 py-0.5 text-sm">{lists[col.key].length}</span>
          </h2>

          <div class="space-y-6">
            {#each lists[col.key] as q, i (q.id)}
              <div
                role="listitem"
                draggable="true"
                animate:flip={{ duration: 250 }}
                class="cursor-grab active:cursor-grabbing transition
                       {dragged?.id === q.id ? 'opacity-40 scale-95' : ''}"
                ondragstart={(e) => onDragStart(e, q.id, col.key)}
                ondragend={onDragEnd}
                ondragover={(e) => e.preventDefault()}
                ondrop={(e) => { e.preventDefault(); e.stopPropagation(); drop(col.key, i); }}
              >
                <Quote author={q.author}>{q.text}</Quote>
              </div>
            {/each}

            {#if lists[col.key].length === 0}
              <div class="rounded-2xl border-2 border-dashed border-slate-300 p-10 text-center text-slate-400">
                Zitat hierher ziehen
              </div>
            {/if}
          </div>
        </section>
      {/each}
    </div>
  </div>
</main>