<script>
  import Quote from "#lib/components/Quote.svelte";
</script>

<main class="min-h-screen bg-gradient-to-br from-slate-50 via-blue-50 to-indigo-100 px-4 py-16">
  <div class="mx-auto max-w-3xl">
    <h1 class="mb-2 text-center text-4xl font-bold tracking-tight text-slate-900">
      Daily Quotes
    </h1>
    <p class="mb-12 text-center text-slate-500">Copy or share the ones you like.</p>

    <div class="space-y-8">
      <Quote author="Steve Jobs">
        The only way to do great work is to love what you do.
      </Quote>

      <Quote author="Rei Alijaj">
        Don't stop when you are tired. Stop when you are done.
      </Quote>

      <Quote author="Paul Opitz">
        Code is like humor. When you have to explain it, it's bad.😈
      </Quote>

      <Quote author="John Johnson">
        First, solve the problem. Then, write the code.
      </Quote>

      <Quote author="Ron Volaj">
        Don't die with a dream.Die with a story.
      </Quote>

    </div>
  </div>
</main>