<script>
  import { pccs_color_map } from "./lib/pccs_color";

  // 利用可能なトーンのリスト
  const tones = Array.from(pccs_color_map.keys());

  // ランダムなトーンと色を取得する関数
  function getRandomColor() {
    const randomTone = tones[Math.floor(Math.random() * tones.length)];
    const colors = pccs_color_map.get(randomTone);
    const randomColor = colors[Math.floor(Math.random() * colors.length)];
    return { tone: randomTone, color: randomColor };
  }

  // 選択されたトーンと色（UI表示用）
  let selectedTone = tones[0];

  // 選択した色を配列で管理（初期値はランダム）
  const initialRandom = getRandomColor();
  selectedTone = initialRandom.tone;
  // 各パネルはトーンと色のペアで管理
  let selectedPanels = [
    { tone: initialRandom.tone, color: initialRandom.color },
  ];
  // 現在アクティブなパネルのインデックス
  let activeColorIndex = 0;

  // トーンが変更されたときの処理
  function handleToneChange() {
    const newColor = pccs_color_map.get(selectedTone)[0];
    if (activeColorIndex < selectedPanels.length) {
      // アクティブなパネルのトーンと色を更新
      selectedPanels[activeColorIndex] = {
        tone: selectedTone,
        color: newColor,
      };
      selectedPanels = [...selectedPanels]; // 配列の変更を検知させる
    } else {
      // 選択されていない場合は最初の色を追加
      selectedPanels = [{ tone: selectedTone, color: newColor }];
      activeColorIndex = 0;
    }
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
  <h1>PCCS 配色</h1>

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
              )[17]}; color: {getTextColor(pccs_color_map.get(tone)[17])};"
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
                on:click={() => {
                  if (activeColorIndex < selectedPanels.length) {
                    selectedPanels[activeColorIndex] = {
                      tone: selectedTone,
                      color: color,
                    };
                    selectedPanels = [...selectedPanels]; // 配列の変更を検知させる
                  } else {
                    selectedPanels = [
                      ...selectedPanels,
                      { tone: selectedTone, color: color },
                    ];
                    activeColorIndex = selectedPanels.length - 1;
                  }
                }}
                class:selected={selectedPanels.some(
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
      <h3>選択した色</h3>

      <!-- 色パネルの数を設定するコントロール -->
      <div class="panel-controls">
        <button
          class="control-btn"
          on:click={() => {
            if (selectedPanels.length > 1) {
              selectedPanels.pop();
              selectedPanels = [...selectedPanels];
              if (activeColorIndex >= selectedPanels.length) {
                activeColorIndex = selectedPanels.length - 1;
              }
            }
          }}
          disabled={selectedPanels.length <= 1}
        >
          -
        </button>
        <span>{selectedPanels.length} 色</span>
        <button
          class="control-btn"
          on:click={() => {
            // ランダムな新しい色を追加
            const randomResult = getRandomColor();
            selectedPanels = [
              ...selectedPanels,
              { tone: randomResult.tone, color: randomResult.color },
            ];
          }}
        >
          +
        </button>
      </div>

      <!-- 色パネルのコンテナ -->
      <div class="panels-container-wrapper">
        <div class="panels-container">
          {#each selectedPanels as panel, i}
            <button
              class="color-panel"
              style="background-color: {panel.color};"
              class:active={i === activeColorIndex}
              on:click={(e) => {
                e.stopPropagation();
                // 既に選択されているパネルをクリックしたら選択解除
                if (i === activeColorIndex) {
                  activeColorIndex = -1;
                } else {
                  activeColorIndex = i;
                  // 現在のパネルのトーンに合わせてトーン選択UIも更新
                  selectedTone = panel.tone;
                }
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
          {/each}
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

  /* パネル関連のスタイル */
  .panel-controls {
    display: flex;
    align-items: center;
    margin-bottom: 2rem;
    gap: 0.5rem;
  }

  .control-btn {
    padding: 0.25rem 0.75rem;
    border: 1px solid #ccc;
    background: #f5f5f5;
    border-radius: 4px;
    font-size: 1.2rem;
    cursor: pointer;
  }

  .control-btn:disabled {
    opacity: 0.5;
    cursor: not-allowed;
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

  .color-panel {
    flex: 1;
    height: 100%;
    position: relative;
    transition: all 0.2s ease;
    cursor: pointer;
    padding: 0;
    border: none;
    margin: 0;
  }

  .color-panel.active {
    margin: -12px;
    z-index: 10;
    box-shadow: 0 0 15px rgba(0, 0, 0, 0.2);
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
