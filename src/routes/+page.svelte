<script>
  import Quote from "#lib/components/Quote.svelte";
  import { flip } from "svelte/animate";
  import { untrack } from "svelte";
  import { getDistance, isPointWithinRadius } from "geolib";

  // ---------- Geofence ----------
  const ZONE = {
    lat: 42.05910523134263,
    lng: 19.517268301553923,
    radius: 15 // meters
  };

  const ZONE_QUOTE = {
    id: "zone-quote",
    author: "DataExchange",
    text: "You found the secret spot! 📍 Some quotes only appear where you stand.",
    zone: true
  };

  let position = $state(null); // { lat, lng, accuracy }
  let geoError = $state(null);

  const zoneCenter = { latitude: ZONE.lat, longitude: ZONE.lng };

  let distance = $derived(
    position ? getDistance({ latitude: position.lat, longitude: position.lng }, zoneCenter) : null
  );
  let inZone = $derived(
    position
      ? isPointWithinRadius({ latitude: position.lat, longitude: position.lng }, zoneCenter, ZONE.radius)
      : false
  );

  // Start watching the position when the page loads, stop when it is left
  $effect(() => {
    if (!("geolocation" in navigator)) {
      geoError = "Geolocation wird von diesem Browser nicht unterstützt.";
      return;
    }

    const watchId = navigator.geolocation.watchPosition(
      (pos) => {
        position = {
          lat: pos.coords.latitude,
          lng: pos.coords.longitude,
          accuracy: pos.coords.accuracy
        };
        geoError = null;
      },
      (err) => {
        geoError = err.message;
      },
      { enableHighAccuracy: true, maximumAge: 0, timeout: 15000 }
    );

    return () => navigator.geolocation.clearWatch(watchId);
  });

  // Add the zone quote when entering, remove it when leaving
  $effect(() => {
    const inside = inZone; // only this value triggers the effect

    untrack(() => {
      const isZoneQuote = (q) => q.id === ZONE_QUOTE.id;
      const hasQuote = lists.all.some(isZoneQuote) || lists.favorites.some(isZoneQuote);

      if (inside && !hasQuote) {
        lists.all.unshift({ ...ZONE_QUOTE });
      } else if (!inside && hasQuote) {
        lists.all = lists.all.filter((q) => !isZoneQuote(q));
        lists.favorites = lists.favorites.filter((q) => !isZoneQuote(q));
      }
    });
  });

  // ---------- Quotes + Drag & Drop ----------
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
    if (fromIndex === -1) return onDragEnd(); // quote vanished while dragging (left the zone)
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
    <p class="mb-6 text-center text-slate-500">
      Zieh die Zitate in deine Favoriten oder ändere die Reihenfolge.
    </p>

    <!-- Location status -->
    <div class="mb-10 flex justify-center">
      <div
        class="flex items-center gap-3 rounded-full px-5 py-2 text-sm font-medium shadow-sm ring-1 transition
               {inZone
                 ? 'bg-amber-100 text-amber-800 ring-amber-300'
                 : 'bg-white/80 text-slate-600 ring-slate-200'}"
      >
        <span class="relative flex h-3 w-3">
          {#if inZone}
            <span class="absolute inline-flex h-full w-full animate-ping rounded-full bg-amber-400 opacity-75"></span>
          {/if}
          <span class="relative inline-flex h-3 w-3 rounded-full {inZone ? 'bg-amber-500' : geoError ? 'bg-red-500' : 'bg-slate-400'}"></span>
        </span>

        {#if geoError}
          Standort: {geoError}
        {:else if distance === null}
          Standort wird gesucht…
        {:else if inZone}
          Du bist in der Zone! Neues Zitat freigeschaltet 🎉
        {:else}
          Noch {Math.round(distance - ZONE.radius)} m bis zur Zone
          <span class="text-slate-400">(GPS ±{Math.round(position.accuracy)} m)</span>
        {/if}
      </div>
    </div>

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
                class="cursor-grab rounded-2xl transition active:cursor-grabbing
                       {q.zone ? 'ring-4 ring-amber-400 ring-offset-2' : ''}
                       {dragged?.id === q.id ? 'scale-95 opacity-40' : ''}"
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