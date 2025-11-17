<script lang="ts">
  import { goto } from "$app/navigation";
  import { page } from "$app/state";
  import { browser } from "$app/environment";

  const selfdiscovery_prompts = [
    "What would be your dream job?",
    "What is the think you most care about?",
    "Prompt3",
  ] as const;

  let selected_prompt = $state(
    selfdiscovery_prompts[
      Math.floor(Math.random() * selfdiscovery_prompts.length)
    ]
  );

  // Pick a random prompt
  function pickRandomPrompt() {
    selected_prompt =
      selfdiscovery_prompts[
        Math.floor(Math.random() * selfdiscovery_prompts.length)
      ];
  }

  // Check if content should be shown based on URL
  const choice: string = $derived(page.url.searchParams.get("choice") ?? '');

  let stormId: number = Date.now();
  let textInputs: string[] = [];

  function toggleContent(p0: string) {
    const newUrl = new URL(page.url);

    if (p0 == "0") {
      stormId = 0;
      textInputs = [];
      // Hide content - remove the parameter
      newUrl.searchParams.delete("choice");
    } else if (p0 == "guided") {
      stormId = Date.now();
      textInputs = [];
      
      // Show content - add the parameter
      newUrl.searchParams.set("choice", "guided");
    } else if (p0 == "open") {
      stormId = Date.now();
      textInputs = [];

      // Show content - add the parameter
      newUrl.searchParams.set("choice", "open");
    }

    goto(newUrl, { keepFocus: true });
  }

  // let choice = $state("0");

  type Storm = {
    id: number;
    text: string[];
    choice: string;
  };

  let storms = $state<Storm[]>([]);

  // Load on mount
  if (browser) {
    const stored = localStorage.getItem("storms");
    if (stored) {
      storms = JSON.parse(stored);
    }
  }

  // Auto-save on change
  $effect(() => {
    if (browser) {
      localStorage.setItem("storms", JSON.stringify(storms));
    }
  });

  function saveStorm() {
    //storms = [...storms, { id: Date.now(), text: "New" }];
    const newItem:Storm = { id: stormId, text: textInputs, choice: choice };

    storms = storms.some((t) => t.id === newItem.id)
      ? storms.map((t) => (t.id === newItem.id ? { ...t, ...newItem } : t))
      : [...storms, newItem];
  }
</script>

{#if choice == "guided"}
  <a href="/">index</a>
  <button onclick={pickRandomPrompt}>different prompt</button>

  <div class="content">
    <div class="">
      <p class="center"><b>{selected_prompt}</b></p>
      <textarea>t1</textarea>
      <br />
      <br />
      <button onclick={saveStorm}>Save locally</button>
    </div>
  </div>
{:else if choice == "open"}
  <a href="/">index</a>
  <div class="content">
    <div class="">
      <p><b>Tough question I am overthinking:</b></p>
      <textarea class="ta-question">t1</textarea>
      <br />
      <br />
      <textarea>t2</textarea>
      <br />
      <br />
      <button onclick={saveStorm}>Save locally</button>
    </div>
  </div>
{:else}
  <div class="center content">
    <div class="">
      <h1>What do you want to clarify your thoughts about?</h1>
      <button onclick={() => toggleContent("guided")}
        >Self-discovery (guided)</button
      >
      <button onclick={() => toggleContent("open")}
        >Tough question I am overthinking (open question)</button
      >
      I hesitate between multiple options
    </div>
  </div>

  <details>
    <summary><b>Brainstorms archive</b></summary>
    {#each storms as storm}
      <div>
        {storm.text}. {storm.choice}. {storm.id} Brainstorm this question/prompt
        again
      </div>
    {/each}
  </details>

  <div class="footer">
    <p>
      Clarify is a simple text editor for clarifying your thoughts and avoiding
      cognitive overload and overthinking.<br /><a href="/">Open-source</a>
      and
      <i>fully private</i> (data never leaves your browser, it is saved to localStorage).
    </p>
  </div>

{/if}

<style>
  .content {
    max-width: var(--screen-sm);
    margin: auto;
  }
  h1 {
    font-size: var(--text-2xl);
  }

  h2 {
    font-size: var(--text-lg);
  }

  .center {
    text-align: center;
    /* display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
     */
  }
  .footer {
    margin-top: 100px;
    text-align: right;
    font-size: smaller;
  }

  textarea {
    width: 100%;
    height: 300px;
  }

  .ta-question {
    height: 56px !important;
  }
  details {
    margin-top: var(--space-2xl)
  }
</style>
