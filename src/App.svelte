<script>
  import { on } from "svelte/events";
  import { pccs_color_map } from "./lib/pccs_color";

  // 利用可能なトーンのリスト
  const tones = Array.from(pccs_color_map.keys());

  // 選択した色を配列で管理（初期値はランダム）
  const initialRandom = getRandomColor();

  let selectedTone = initialRandom.tone;
  let selectedColor = initialRandom.color;

  // 各パネルはトーンと色のペアで管理
  let colorPanels = [{ tone: initialRandom.tone, color: initialRandom.color }];
  let selectedPanelIndex = 0;

  // ----------------
  // ヘルパー関数
  // ---------------

  // ランダムなトーンと色を取得する関数
  function getRandomColor() {
    const randomTone = tones[Math.floor(Math.random() * tones.length)];
    const colors = pccs_color_map.get(randomTone);
    const randomColor = colors[Math.floor(Math.random() * colors.length)];
    return { tone: randomTone, color: randomColor };
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

  function isPanelSelected() {
    return selectedPanelIndex >= 0;
  }

  // ----------------
  // イベントハンドラ
  // ----------------

  // トーンが変更されたときの処理
  function onToneChanged(tone) {
    selectedTone = tone;

    if (isPanelSelected()) {
      // アクティブなパネルのトーンと色を更新
      colorPanels[selectedPanelIndex] = {
        tone: selectedTone,
        color: pccs_color_map.get(selectedTone)[0],
      };
      colorPanels = [...colorPanels]; // 配列の変更を検知させる
    }
  }

  function onColorChanged(color) {
    if (isPanelSelected()) {
      selectedColor = color;

      // アクティブなパネルの色を更新
      colorPanels[selectedPanelIndex] = {
        tone: selectedTone,
        color: selectedColor,
      };
      colorPanels = [...colorPanels]; // 配列の変更を検知させる
    }
  }

  function onPanelClicked(panelIndex) {
    if (selectedPanelIndex === panelIndex) {
      selectedPanelIndex = -1;
    } else {
      selectedPanelIndex = panelIndex;
    }
  }

  function onAddPanel() {
    const randomResult = getRandomColor();
    colorPanels = [
      ...colorPanels,
      { tone: randomResult.tone, color: randomResult.color },
    ];
    onPanelClicked(colorPanels.length - 1); // 新しいパネルを選択状態にする
  }

  function onDeletePanel() {
    // 選択したパネルを削除
    colorPanels = colorPanels.filter(
      (_, index) => index !== selectedPanelIndex,
    );
    // インデックスを調整
    if (selectedPanelIndex >= colorPanels.length) {
      selectedPanelIndex = colorPanels.length - 1;
      selectedTone = colorPanels[selectedPanelIndex].tone;
      selectedColor = colorPanels[selectedPanelIndex].color;
    }
  }
</script>

<main>
  <h1>PCCS 配色</h1>

  <div class="container">
    <div class="selector-panel">
      <section class="tone-selector">
        <h3>トーン</h3>
        <div class="tone-swatches">
          {#each tones as tone}
            <button
              class="tone-item"
              style={((c) =>
                `background-color: ${c}; color: ${getTextColor(c)};`)(
                pccs_color_map.get(tone)[6],
              )}
              on:click={() => onToneChanged(tone)}
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
        <h3>色</h3>
        <div class="color-swatches">
          {#if selectedTone && pccs_color_map.has(selectedTone)}
            {#each pccs_color_map.get(selectedTone) as color, index}
              <button
                class="color-item"
                style="background-color: {color}; color: {getTextColor(color)};"
                on:click={() => onColorChanged(color)}
                class:selected={colorPanels.some(
                  (panel) => panel.color === color,
                )}
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
      <!-- 色パネルのコンテナ -->
      <div class="panels-container-wrapper">
        <div class="panels-container">
          {#each colorPanels as panel, i}
            <div class="panel-wrapper">
              <button
                class="color-panel"
                style="background-color: {panel.color};"
                class:active={i === selectedPanelIndex}
                on:click={(e) => {
                  e.stopPropagation();
                  onPanelClicked(i);
                }}
                aria-label="カラーパネル {i + 1} を選択"
                type="button"
              >
                <div class="panel-info">
                  <p>
                    {panel.tone}-{pccs_color_map
                      .get(panel.tone)
                      .indexOf(panel.color) + 1}
                  </p>
                  <p>{panel.color}</p>
                </div>
              </button>

              <!-- アクティブなパネルの下に削除ボタンを表示 -->
              <div
                class="delete-btn-container"
                class:show={i === selectedPanelIndex}
              >
                <button
                  class="delete-panel-btn"
                  on:click={() => onDeletePanel()}
                  disabled={colorPanels.length <= 1}
                >
                  削除
                </button>
              </div>
            </div>
          {/each}

          <!-- 新しいパネル追加ボタン -->
          <button
            class="add-panel-btn"
            on:click={() => onAddPanel()}
            aria-label="新しい色パネルを追加"
            type="button"
          >
            <span>+</span>
          </button>
        </div>

        <!-- 各パネル毎の削除ボタンは表示しない -->
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
    align-items: stretch; /* 子要素の高さを揃える */
  }

  .selector-panel {
    flex: 1.5;
    display: flex;
    flex-direction: column;
  }

  .preview-panel {
    flex: 1;
    display: flex;
    flex-direction: column;
  }

  .color-list {
    margin-top: 1.5rem;
    flex: 1; /* 残りのスペースを埋める */
    display: flex;
    flex-direction: column;
  }

  .tone-selector {
    margin-bottom: 1.5rem;
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
    flex: 1; /* 親要素の残りのスペースを埋める */
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
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: bold;
    font-size: 0.9rem;
    transition: transform 0.2s;
  }

  .color-item:hover {
    transform: scale(1.05);
  }

  .add-panel-btn {
    width: 40px;
    margin-left: 12px;
    background-color: #f0f0f0;
    border: 1px dashed #aaa;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    transition: all 0.2s;
    border-radius: 4px;
  }

  .add-panel-btn span {
    font-size: 24px;
    color: #555;
  }

  .add-panel-btn:hover {
    background-color: #e5e5e5;
    transform: scale(1.05);
  }

  .panels-container-wrapper {
    flex: 1;
    display: flex;
    flex-direction: column;
  }

  .panels-container {
    display: flex;
    width: 100%;
    flex: 1;
    margin-bottom: 0.5rem;
  }

  .panel-wrapper {
    flex: 1;
    position: relative;
    display: flex;
    flex-direction: column;
    align-items: stretch;
  }

  .color-panel {
    flex: 1;
    height: 100%;
    position: relative;
    /* 色の変化からtransitionを除去 */
    transition-property: transform, margin-top;
    transition-duration: 0.16s;
    transition-timing-function: ease-in-out;
    cursor: pointer;
    padding: 0;
    border: none;
    margin: 0;
  }

  .color-panel.active {
    margin-top: -24px; /* そのままの高さを維持 */
    z-index: 10;
  }

  .delete-btn-container {
    height: 0;
    margin-top: 0;
    overflow: hidden;
    transition:
      height 0.16s ease-in-out,
      margin-top 0.16s ease-in-out;
  }

  .delete-btn-container.show {
    height: 24px;
    margin-top: 2px; /* パネルと削除ボタンの間のマージン */
  }

  .delete-panel-btn {
    background-color: #ff4757;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 11px;
    height: 24px;
    width: 100%;
    transition-property: transform;
    transition-duration: 0.16s;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .delete-panel-btn:hover {
    background-color: #ff6b81;
  }

  .delete-panel-btn:disabled {
    background-color: #aaaaaa;
    cursor: not-allowed;
  }

  .panel-info {
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

  .panel-info p {
    margin: 2px 0;
  }
</style>
