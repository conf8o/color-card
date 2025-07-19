<script>
  import { pccs_color_map } from "./lib/pccs_color";

  // 利用可能なトーンのリスト
  const tones = Array.from(pccs_color_map.keys());

  // 選択されたトーンと色
  let selectedTone = tones[0];
  let selectedColor = pccs_color_map.get(selectedTone)[0];

  // トーンが変更されたときに最初の色を選択
  function handleToneChange() {
    selectedColor = pccs_color_map.get(selectedTone)[0];
  }

  // 色の明るさを判定してテキスト色を決定する
  function getTextColor(hexColor) {
    // カラーコードからRGB値を取得
    const r = parseInt(hexColor.slice(1, 3), 16);
    const g = parseInt(hexColor.slice(3, 5), 16);
    const b = parseInt(hexColor.slice(5, 7), 16);

    // 輝度の計算（YIQ方式）
    const brightness = (r * 299 + g * 587 + b * 114) / 1000;

    // 輝度が128より大きい場合は暗いテキスト、そうでなければ明るいテキスト
    return brightness > 128 ? "#000000" : "#ffffff";
  }
</script>

<main>
  <h1>PCCS 色見本</h1>

  <div class="container">
    <div class="selector-panel">
      <section class="tone-selector">
        <h3>トーン選択</h3>
        <div class="tone-swatches">
          {#each tones as tone}
            <button
              class="tone-item"
              style="background-color: {pccs_color_map.get(
                tone,
              )[9]}; color: {getTextColor(pccs_color_map.get(tone)[9])};"
              on:click={() => {
                selectedTone = tone;
                handleToneChange();
              }}
              class:selected={tone === selectedTone}
              aria-label="{tone}トーンを選択"
              type="button"
            >
              {tone}
            </button>
          {/each}
        </div>
      </section>

      <section class="color-list">
        <h3>色リスト</h3>
        <div class="color-swatches">
          {#if selectedTone && pccs_color_map.has(selectedTone)}
            {#each pccs_color_map.get(selectedTone) as color, index}
              <button
                class="color-item"
                style="background-color: {color}; color: {getTextColor(color)};"
                on:click={() => (selectedColor = color)}
                class:selected={color === selectedColor}
                aria-label="{selectedTone}-{index + 1}: {color} を選択"
                type="button"
              >
                {selectedTone}-{index + 1}
              </button>
            {/each}
          {/if}
        </div>
      </section>
    </div>

    <div class="preview-panel">
      <h3>選択した色</h3>
      <div class="color-preview" style="background-color: {selectedColor};">
        <div class="preview-info">
          <p>
            {selectedTone}-{pccs_color_map
              .get(selectedTone)
              .indexOf(selectedColor) + 1}
          </p>
          <p>{selectedColor}</p>
        </div>
      </div>
    </div>
  </div>
</main>

<style>
  main {
    font-family: sans-serif;
    padding: 2rem;
    max-width: 1000px;
    margin: 0 auto;
  }

  section {
    margin-bottom: 1rem;
  }

  .container {
    display: flex;
    gap: 2rem;
    margin-top: 2rem;
  }

  .selector-panel {
    flex: 1.5;
  }

  .preview-panel {
    flex: 1;
    display: flex;
    flex-direction: column;
  }

  .color-list {
    margin-top: 1.5rem;
  }

  .tone-swatches {
    display: grid;
    grid-template-columns: repeat(6, 1fr);
    gap: 0.5rem;
    padding: 0.5rem;
    border: 1px solid #eee;
    border-radius: 4px;
  }

  .tone-item {
    width: 100%;
    height: 40px;
    border-radius: 4px;
    cursor: pointer;
    border: 1px solid #ccc;
    transition: transform 0.2s;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: bold;
    font-size: 0.9rem;
  }

  .tone-item:hover {
    transform: scale(1.05);
  }

  .tone-item.selected {
    border: 2px solid #000;
  }

  .color-swatches {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 0.5rem;
    max-height: 500px;
    overflow-y: auto;
    padding: 0.5rem;
    border: 1px solid #eee;
    border-radius: 4px;
  }

  .color-item {
    width: 100%;
    height: 45px;
    border-radius: 4px;
    cursor: pointer;
    border: 1px solid #ccc;
    transition: transform 0.2s;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: bold;
    font-size: 0.9rem;
  }

  .color-item:hover {
    transform: scale(1.05);
  }

  .color-item.selected {
    border: 2px solid #000;
  }

  .color-preview {
    width: 100%;
    height: 300px;
    position: relative;
    margin-bottom: 1rem;
  }

  .preview-info {
    position: absolute;
    right: 10px;
    bottom: 10px;
    background-color: rgba(0, 0, 0, 0.2);
    padding: 5px 8px;
    border-radius: 4px;
    font-size: 10px;
    text-align: right;
    color: #ffffff;
    text-shadow: 0 0 2px rgba(0, 0, 0, 0.5);
  }

  .preview-info p {
    margin: 2px 0;
  }
</style>
