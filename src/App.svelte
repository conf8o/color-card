<script>
  import { pccs_color_map } from "./lib/pccs_color";

  // 利用可能なトーンのリスト
  const tones = Array.from(pccs_color_map.keys());

  // 選択した色を配列で管理（初期値はランダム）
  const initialRandom = getRandomColor();

  // ----------------
  // 状態管理
  // ----------------

  let selectedTone = initialRandom.tone;
  let selectedColor = initialRandom.color;

  // 各パネルは (トーン, 色, 色相番号) の組で管理
  let colorPanels = [
    {
      tone: initialRandom.tone,
      color: initialRandom.color,
      colorIndex: initialRandom.colorIndex,
    },
  ];
  let selectedPanelIndex = 0;

  // ----------------
  // ヘルパー関数
  // ----------------

  function getRandomColor() {
    const randomTone = tones[Math.floor(Math.random() * tones.length)];
    const colors = pccs_color_map.get(randomTone);
    const randomColorIndex = Math.floor(Math.random() * colors.length);
    const randomColor = colors[randomColorIndex];
    return {
      tone: randomTone,
      color: randomColor,
      colorIndex: randomColorIndex,
    };
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

  // PCCS記号を生成する関数
  function pccsCode(tone, index) {
    return `${tone}-${index + 1}`;
  }

  function copyToClipboard(s) {
    navigator.clipboard
      .writeText(s)
      .then(() => {
        console.log("クリップボードにコピーしました:", s);
      })
      .catch((err) => {
        console.error("クリップボードへのコピーに失敗しました:", err);
      });
  }

  // ----------------
  // イベントハンドラ
  // ----------------

  function onToneChanged(tone) {
    selectedTone = tone;
  }

  function onColorChanged(color, colorIndex) {
    if (isPanelSelected()) {
      selectedColor = color;

      // アクティブなパネルの色を更新
      colorPanels[selectedPanelIndex] = {
        tone: selectedTone,
        color: selectedColor,
        colorIndex: colorIndex,
      };
      colorPanels = [...colorPanels]; // 状態更新のため再代入
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
      {
        tone: randomResult.tone,
        color: randomResult.color,
        colorIndex: randomResult.colorIndex,
      },
    ];
    onPanelClicked(colorPanels.length - 1); // 新しいパネルを選択状態にする
  }

  function onDeletePanel() {
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

  // PCCS記号一覧をクリップボードにコピーする
  function onCopyPccsToClipboard() {
    const clipboardText = colorPanels
      .map((panel) => pccsCode(panel.tone, panel.colorIndex))
      .join(",");

    copyToClipboard(clipboardText);
  }

  // コピー成功時のフィードバック表示用
  let showCopyFeedback = false;
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
                on:click={() => onColorChanged(color, index)}
                class:selected={colorPanels.some(
                  (panel) => panel.color === color,
                )}
                aria-label="{pccsCode(selectedTone, index)}: {color} を選択"
                type="button"
              >
                {pccsCode(selectedTone, index)}
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
                    {pccsCode(panel.tone, panel.colorIndex)}
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
      </div>

      <!-- PCCSコードをコピーするボタン -->
      <div class="copy-button-container">
        <button
          class="copy-button"
          on:click={() => onCopyPccsToClipboard()}
          aria-label="PCCS記号をクリップボードにコピー"
          type="button"
        >
          PCCS記号をコピー
        </button>
        {#if showCopyFeedback}
          <div class="copy-feedback">コピーしました！</div>
        {/if}
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
    grid-template-columns: repeat(4, 1fr);
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

  .copy-button-container {
    display: flex;
    align-items: center;
    margin-top: 10px;
    position: relative;
  }

  .copy-button {
    background-color: #4a6cf7;
    color: white;
    border: none;
    border-radius: 4px;
    padding: 8px 16px;
    font-size: 14px;
    cursor: pointer;
    transition: background-color 0.2s;
  }

  .copy-button:hover {
    background-color: #3a57d7;
  }

  .copy-feedback {
    position: absolute;
    right: 0;
    background-color: rgba(0, 0, 0, 0.7);
    color: white;
    padding: 6px 12px;
    border-radius: 4px;
    font-size: 12px;
    animation:
      fadeIn 0.3s,
      fadeOut 0.3s 1.7s;
  }

  @keyframes fadeIn {
    from {
      opacity: 0;
    }
    to {
      opacity: 1;
    }
  }

  @keyframes fadeOut {
    from {
      opacity: 1;
    }
    to {
      opacity: 0;
    }
  }
</style>
