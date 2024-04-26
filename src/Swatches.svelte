<script>
  import { writable } from 'svelte/store';

  let showCopyIcon = writable(false);
  export let displayedColorSwatches = [];

    async function copyToClipboard(hex) {

      await navigator.clipboard.writeText(hex);
      showCopyIcon.set(true);

      setTimeout(() => {
          showCopyIcon.set(false);
      }, 1000);
  }
</script>

{#if displayedColorSwatches.length}
<div class="title-container">
  <span class="title">Extracted Colors:</span>
</div>
<div class="colors">
    {#each displayedColorSwatches as { color, hex }}
        <!-- svelte-ignore a11y-click-events-have-key-events -->
        <!-- svelte-ignore a11y-no-static-element-interactions -->
        <div class="color-swatch" style="background-color: {color};" on:click={() => copyToClipboard(hex)}>

            {#if $showCopyIcon}
            <span>
            <svg
              xmlns="http://www.w3.org/2000/svg"
              height="24px"
              viewBox="0 -960 960 960"
              width="24px"
              fill="var(--white)">
              
              <path d="M300-200q-24 0-42-18t-18-42v-560q0-24 18-42t42-18h440q24 0 42 18t18 42v560q0 24-18 42t-42 18H300Zm0-60h440v-560H300v560ZM180-80q-24 0-42-18t-18-42v-620h60v620h500v60H180Zm120-180v-560 560Z"/>
            </svg>
          </span>
            {:else}
                <span>{hex}</span>
            {/if}
        </div>
    {/each}
</div>
{/if}

<style>

  .title-container {
    display: flex;
    justify-content: center;
    margin-bottom: 5px;
  }

  .title {
    font-size: 25px;
    text-align: center;
  }

  .colors {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 12px;
  }
  .color-swatch {
      width: 60px;
      height: 60px;
      border-radius: 5px;
      border: 1px solid var(--transparent);
      position: relative;
      display: flex;
      align-items: center;
      justify-content: center;
      color: var(--white);
      cursor: pointer;
      position: relative;
  }
  .color-swatch span {
      font-size: .75rem;
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      opacity: 0;
      transition: opacity .5s ease;
  }
  .color-swatch:hover span {
      opacity: 1;
  }
</style>