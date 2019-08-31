<script>
  import {
    width,
    height,
    fontSize,
    maxLength,
    backgroundColor,
    fontColor,
    fontFamily,
    fonts,
    isLoading,
    isAllowedToCreateNew,
    showResultsPanel,
    response
  } from "../stores.js";

  import TileBox from "../components/TileBox.svelte";
  import ColorPicker from "../components/ColorPicker.svelte";
  import NumericInput from "../components/NumericInput.svelte";

  import { randomNumber, randomColor } from "../lib/utils.js";
  import { API_BASE } from "../shared.js";

  const selectRandomFont = () => {
    const randomFontIndex = Math.floor(Math.random() * $fonts.length);
    fontFamily.set($fonts[randomFontIndex]);
  };

  const handleRandomizeClick = () => {
    width.set(randomNumber(600));
    height.set(randomNumber(600));
    fontSize.set(randomNumber(50));
    maxLength.set(randomNumber(13));
    backgroundColor.set(randomColor());
    fontColor.set(randomColor());
    selectRandomFont();
  };

  let newFont;
  const addNewFontToList = () => {
    if (!newFont || newFont == "") return;
    fonts.update(f => [...f, newFont]);
    fontFamily.set(newFont);
    newFont = "";
  };

  const createCounter = async () => {
    isLoading.set(true);

    const res = await fetch(
      `https://cors-anywhere.herokuapp.com/${API_BASE}/counters`,
      {
        method: "post",
        body: JSON.stringify({
          width: $width,
          height: $height,
          fontSize: $fontSize,
          maxLength: $maxLength,
          backgroundColor: $backgroundColor,
          fontColor: $fontColor,
          fontFamily: $fontFamily
        }),
        headers: {
          "Content-Type": "application/json"
        }
      }
    );
    const json = await res.json();

    if (res.ok) {
      isAllowedToCreateNew.set(false);
      response.set(json);
      showResultsPanel.set(true);
    }

    isLoading.set(false);
  };
</script>

<style>
  .field-label {
    flex-grow: 2;
  }
</style>

<TileBox title="Customize">

  <ColorPicker label="Background color" bind:value={$backgroundColor} />
  <ColorPicker label="Font color" bind:value={$fontColor} />

  <div class="field is-horizontal">
    <div class="field-label is-normal">
      <label class="label">Font family</label>
    </div>
    <div class="field-body">
      <div class="field has-addons ">

        <p class="control">
          <span class="select is-fullwidth">
            <select
              bind:value={$fontFamily}
              disabled={!$isAllowedToCreateNew || $isLoading}>
              {#each $fonts as font}
                <option value={font}>{font}</option>
              {/each}
            </select>
          </span>
        </p>
        <p class="control">
          <button
            class="button"
            on:click={() => selectRandomFont()}
            class:is-static={!$isAllowedToCreateNew || $isLoading}>
            <span class="icon">
              <i class="fas fa-random" />
            </span>
          </button>
        </p>

      </div>
    </div>
  </div>

  <div class="field is-horizontal">
    <div class="field-label" />
    <div class="field-body">
      <div class="field is-expanded">
        <div class="field has-addons">
          <p class="control is-expanded">
            <input
              class="input"
              type="text"
              bind:value={newFont}
              placeholder="Add new font to the list"
              disabled={!$isAllowedToCreateNew || $isLoading} />
          </p>
          <p class="control">
            <button
              class="button"
              on:click={() => addNewFontToList()}
              class:is-static={!$isAllowedToCreateNew || $isLoading}>
              <i class="fas fa-plus" />
            </button>
          </p>
        </div>
        <p class="help">
          Make sure the new font is available on the page where the counter will
          be placed
        </p>
      </div>
    </div>
  </div>

  <NumericInput label="Font size" max={50} bind:value={$fontSize} />
  <NumericInput label="Max digits" max={13} bind:value={$maxLength} />
  <NumericInput label="Width" max={600} bind:value={$width} />
  <NumericInput label="Height" max={600} bind:value={$height} />

  <hr />
  <div class="field is-grouped is-grouped-centered">

    <p class="control">
      <button
        class="button is-light has-icons has-icons-left"
        class:is-static={!$isAllowedToCreateNew || $isLoading}
        on:click={() => handleRandomizeClick()}>
        <span class="icon">
          <i class="fas fa-random" />
        </span>
        <p>Randomize All</p>
      </button>
    </p>

    <p class="control">
      <button
        class="button is-primary has-icons has-icons-left"
        class:is-static={!$isAllowedToCreateNew}
        class:is-loading={$isLoading}
        on:click={() => createCounter()}>
        >
        <span class="icon">
          <i class="fas fa-space-shuttle" />
        </span>
        <p>Create Counter</p>
      </button>
    </p>

  </div>
</TileBox>