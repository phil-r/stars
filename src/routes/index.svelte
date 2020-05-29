<script>
  import { createDice, createCoin } from "unfair-dice";

  const TIME_LIMIT = 2; // seconds

  let currentStep = "beforeGame";
  let symbol;
  let correct;
  let answer;
  let mode;

  let lines;
  const dice = createDice(["lines", "symbols"]);
  // const dice = createDice(["lines"]);

  const symbolsDice = createDice(["🌟", "☁️", "🌕"]);
  const LINES = [
    "Atque id ullam culpa doloribus placeat.\n",
    "Quasi cum ut voluptatem sunt voluptatem similique repellat odio.\n",
    "Laudantium est quisquam non.\n",
    "Eos et totam quidem recusandae autem facere quia.\n",
    "Provident explicabo a quasi.\n"
  ];
  const linesDice = createDice(LINES);
  const coin = createCoin();
  const d10 = createDice(10);
  const d20 = createDice(20);
  const d30 = createDice(30);
  const d50 = createDice(50);
  newGame();

  function generateLines() {
    const addPercentages = coin.flip();
    let percentage = 0;
    correct = d20.roll() + 5;
    lines = "";
    for (let i = 0; i < correct; i++) {
      if (addPercentages) {
        if (i === correct - 1) {
          lines += "100% ";
        } else {
          percentage +=
            (((100 - percentage) * Math.random()) / (correct - i)) * 2;
          percentage = parseFloat(percentage.toFixed(2));
          lines += `${percentage}% `;
        }
      }
      lines += linesDice.roll();
    }
  }

  function generateSymbols() {
    symbol = symbolsDice.roll();
    const addRandomSymbols = coin.flip();
    correct = d30.roll() + 10;
    lines = "";
    for (let i = 0; i < correct; ) {
      let spaces = d10.roll();
      for (let j = 0; j < spaces; j++) {
        lines += " ";
      }
      if (addRandomSymbols && coin.flip() == 1) {
        if (symbol === "🌟") {
          lines += "🌕";
        } else {
          lines += "💩";
        }
      } else {
        lines += symbol;
        i++;
      }
      spaces = d10.roll();
      for (let j = 0; j < spaces; j++) {
        lines += " ";
      }
    }
  }

  function newGame() {
    mode = dice.roll();
    if (mode === "lines") {
      generateLines();
    } else if (mode === "symbols") {
      generateSymbols();
    }
  }

  function handleStart() {
    currentStep = "game";
    setTimeout(() => {
      currentStep = "enter";
      answer = 1;
    }, TIME_LIMIT * 1000);
  }

  function submitAnswer() {
    currentStep = "afterGame";
  }

  function restart() {
    newGame();
    currentStep = "beforeGame";
  }
</script>

<style>
  .footer {
    position: fixed;
    bottom: 0;
  }

  .symbols {
    width: 300px;
    white-space: break-spaces;
    margin: 0 auto;
  }
</style>

<svelte:head>
  <title>🌟 [Counting] stars</title>
</svelte:head>

{#if currentStep === 'beforeGame'}
  <div>

    {#if mode === 'lines'}
      Please count the number of
      <b>lines of text</b>
      that you'll see.
    {:else if mode === 'symbols'}
      Please count the number of
      <b>{symbol} symbols</b>
      that you'll see.
    {/if}
    You will only have {TIME_LIMIT} seconds.
  </div>
  <button on:click|preventDefault={handleStart}>Start</button>
{/if}

{#if currentStep === 'game'}
  {#if mode === 'lines'}
    <pre>
      <code>{lines}</code>
    </pre>
  {:else if mode === 'symbols'}
    <pre class="symbols">{lines}</pre>
  {/if}
{/if}

{#if currentStep === 'enter'}
  <div>
    How many
    {#if mode === 'lines'}
      lines
    {:else if mode === 'symbols'}{symbol} symbols{/if}
    did you see?
  </div>
  <form action="" on:submit|preventDefault={submitAnswer}>
    <input type="number" min="1" bind:value={answer} />
    <input
      type="submit"
      value="Submit"
      on:click|preventDefault={submitAnswer} />
  </form>
{/if}
{#if currentStep === 'afterGame'}
  <div>Your number: {answer}, actual number: {correct}</div>
  <button on:click|preventDefault={restart}>Play again</button>
{/if}
<!-- Debug -->
<!-- <div class="footer">{currentStep} - {mode} - {correct} - {answer}</div> -->

<div class="footer">
  <a href="https://github.com/phil-r/stars">Source code</a>
</div>
