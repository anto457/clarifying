<script lang="ts">
  import { browser } from "$app/environment";

  const selfdiscovery_prompts = [
    "What would be your dream job?",
    "What is the think you most care about?",
    "Are you stressed these days? If so, why?",
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

  let stormId: number = 0;
  let textInputs: string[] = $state([]);

  let choice: number = $state(0);

  function selectStorm(val: number) {
    choice = val;
    stormId = Date.now();

    if (choice == 1) {
      pickRandomPrompt();
      textInputs = [""];
    } else if (choice == 2) {
      textInputs = ["", ""];
    } else {
      textInputs = [];
    }
  }

  type Storm = {
    id: number;
    text: string[];
    choice: number;
  };

  let storms = $state<Storm[]>([]);

  // Load data on mount
  if (browser) {
    const stored = localStorage.getItem("storms");
    if (stored) {
      storms = JSON.parse(stored);
    }
  }

  function saveStorm() {
    if (browser) {
      const newItem: Storm = { id: stormId, text: textInputs, choice: choice };

      storms = storms.some((t) => t.id === newItem.id)
        ? storms.map((t) => (t.id === newItem.id ? { ...t, ...newItem } : t))
        : [...storms, newItem];

      localStorage.setItem("storms", JSON.stringify(storms));
    }
  }
</script>

{#if choice == 1}
  <button onclick={() => (choice = 0)}>index</button>
  <button onclick={pickRandomPrompt}>different prompt</button>

  <div class="content">
    <div class="">
      <p class="center"><b>{selected_prompt}</b></p>
      <textarea bind:value={textInputs[0]}></textarea>
      <br />
      <br />
      <button onclick={saveStorm}>Save locally</button>
    </div>
  </div>
{:else if choice == 2}
  <button onclick={() => (choice = 0)}>index</button>
  <div class="content">
    <div class="">
      <p><b>Tough question I am overthinking:</b></p>
      <textarea class="ta-question" bind:value={textInputs[0]}></textarea>
      <br />
      <br />
      <textarea bind:value={textInputs[1]}></textarea>
      <br />
      <br />
      <button onclick={saveStorm}>Save locally</button>
    </div>
  </div>
{:else}
  <div class="center content">
    <div class="">
      <h1>What do you want to clarify your thoughts about?</h1>
      <button onclick={() => selectStorm(1)}>Self-discovery</button>
      <button onclick={() => selectStorm(2)}
        >Tough question I am overthinking</button
      >
      <!-- <button onclick={() => selectStorm(3)} disabled
        >I hesitate between multiple options</button
      > -->
    </div>
  </div>

  <details>
    <summary><b>Brainstorms archive</b></summary>
    {#each storms as storm}
      <div>
        {storm.choice} - {storm.text}.
      </div>
    {/each}
  </details>

  <div class="footer">
    <p>
      Clarify is a simple text editor for clarifying your thoughts and avoiding
      cognitive overload and overthinking.<br /><a href="https://github.com/anto457/clarifying/">Open-source</a>
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
    margin-top: var(--space-2xl);
  }
</style>
