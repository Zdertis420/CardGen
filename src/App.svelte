<script lang="ts">
  import { marked } from "marked";
  import html2canvas from "html2canvas";

  // === Состояние визитки ===
  let markdown = $state(`Type here in Markdown`);

  let htmlContent = $derived(marked.parse(markdown));

  let previewRef: HTMLElement | null = $state(null);

  // === Настройки оформления ===
  let selectedGradient = $state(0);
  let textColor = $state("#ffffff");
  let fontFamily = $state("system-ui, sans-serif");

  const gradients = [
    "linear-gradient(135deg, #3b82f6, #8b5cf6)",
    "linear-gradient(135deg, #ec4899, #f43f5e)",
    "linear-gradient(135deg, #14b8a6, #0ea5e9)",
    "linear-gradient(135deg, #f59e0b, #ef4444)",
    "linear-gradient(135deg, #8b5cf6, #d946ef)",
    "linear-gradient(135deg, #22c55e, #eab308)",
  ];

  const fonts = [
    { name: "System", value: "system-ui, sans-serif" },
    { name: "Inter", value: "Inter, system-ui, sans-serif" },
    { name: "Poppins", value: "Poppins, sans-serif" },
    { name: "Playfair Display", value: "Playfair Display, serif" },
  ];

  let cardStyle = $derived(
    `background: ${gradients[selectedGradient]}; ` +
      `color: ${textColor}; ` +
      `font-family: ${fontFamily};`,
  );

  async function exportToPNG() {
    if (!previewRef) return;

    const canvas = await html2canvas(previewRef, {
      scale: 2.5,
      backgroundColor: "#1f2937",
      useCORS: true,
    });

    const link = document.createElement("a");
    link.download = "business-card.png";
    link.href = canvas.toDataURL("image/png");
    link.click();
  }
</script>

<div class="app">
  <header>
    <h1>Markdown → красивая визитка → PNG</h1>
  </header>

  <div class="content">
    <div class="workspace">
      <!-- ЛЕВАЯ КОЛОНКА: Настройки + кнопка -->
      <div class="panel sidebar-panel">
        <h2>Настройки оформления</h2>
        <div class="settings-content">
          <!-- Градиенты -->
          <div>
            <div class="setting-label">Градиент фона</div>
            <div class="gradients">
              {#each gradients as gradient, i}
                <button
                  type="button"
                  class="gradient-swatch"
                  style="background: {gradient};"
                  class:selected={selectedGradient === i}
                  onclick={() => (selectedGradient = i)}
                  aria-label="Градиент {i + 1}"
                ></button>
              {/each}
            </div>
          </div>

          <!-- Цвет текста -->
          <div>
            <label for="textColor">Цвет текста</label>
            <input id="textColor" type="color" bind:value={textColor} />
          </div>

          <!-- Шрифт -->
          <div>
            <label for="fontFamily">Шрифт</label>
            <select id="fontFamily" bind:value={fontFamily}>
              {#each fonts as font}
                <option value={font.value}>{font.name}</option>
              {/each}
            </select>
          </div>
        </div>

        <!-- Кнопка экспорта -->
        <div class="sidebar-footer">
          <button onclick={exportToPNG} class="export-btn">
            💾 Запечь в PNG
          </button>
        </div>
      </div>

      <!-- Markdown редактор -->
      <div class="panel editor-panel">
        <h2>Markdown редактор</h2>
        <textarea bind:value={markdown} spellcheck="false"></textarea>
      </div>

      <!-- Превью визитки -->
      <div class="panel preview-panel">
        <h2>Live превью</h2>
        <div class="card-wrapper">
          <div bind:this={previewRef} class="business-card" style={cardStyle}>
            {@html htmlContent}
          </div>
        </div>
      </div>
    </div>
  </div>
</div>

<style>
  /* Глобальный сброс */
  *,
  *::before,
  *::after {
    box-sizing: border-box;
  }

  .app {
    height: 100vh;
    width: 100vw;
    display: flex;
    flex-direction: column;
    overflow: hidden;
    background: #0f172a;
  }

  .content {
    flex: 1;
    width: 100%;
    padding: 1rem 1.5rem;       /* уменьшили, чтобы не вылезало */
    overflow: hidden;
    display: flex;
    flex-direction: column;
  }

  header {
    text-align: center;
    margin-bottom: 1rem;
    padding-top: 0.75rem;
    flex-shrink: 0;
  }

  header h1 {
    font-size: 1.2rem;
    color: #94a3b8;
    margin: 0;
  }

  /* Основной workspace — здесь была главная проблема */
  .workspace {
    flex: 1;
    display: flex;
    gap: 1rem;                  /* уменьшили gap */
    overflow: hidden;
    min-height: 0;
  }

  .panel {
    display: flex;
    flex-direction: column;
    background: #1e2937;
    border-radius: 12px;
    overflow: hidden;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.25);
  }

  .sidebar-panel {
    flex: 0 0 320px;            /* фиксированная ширина — надёжно */
  }

  .editor-panel {
    flex: 1;
    min-width: 0;
  }

  .preview-panel {
    flex: 1.5;                  /* превью получает больше пространства */
    display: flex;
    flex-direction: column;
    min-width: 0;
    min-height: 0;
  }

  .panel h2 {
    padding: 1rem 1.25rem;
    margin: 0;
    background: #334155;
    font-size: 1.12rem;
    color: #94a3b8;

    font-weight: 600;
    flex-shrink: 0;
  }

  /* Card wrapper и карточка */
  .card-wrapper {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 2rem;
    background: #1e2937;
    overflow: hidden;
    min-height: 0;
  }

  .business-card {
    width: 100%;
    max-width: 580px;
    min-width: 460px;
    height: 390px;
    padding: 2.8rem 2.6rem;
    border-radius: 20px;
    box-shadow: 0 25px 60px rgba(0, 0, 0, 0.45);
    text-align: center;
    box-sizing: border-box;

    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    gap: 0.55rem;

    overflow: hidden;
    word-break: break-word;
    hyphens: auto;
  }

  /* Markdown стили внутри карточки */
  :global(.business-card h1),
  :global(.business-card h2),
  :global(.business-card h3) {
    font-size: 2.15rem;
    margin: 0 0 0.35rem 0;
    line-height: 1.1;
    font-weight: 700;
    word-break: break-word;
    overflow-wrap: break-word;
  }

  :global(.business-card p) {
    margin: 0.18rem 0;
    font-size: 1.18rem;
    line-height: 1.4;
    word-break: break-word;
  }

  textarea {
    flex: 1;
    background: transparent;
    color: #e2e8f0;
    border: none;
    padding: 1.25rem;
    font-family: ui-monospace, monospace;
    font-size: 1.05rem;
    line-height: 1.55;
    resize: none;
    min-height: 0;
  }

  /* Настройки */
  .settings-content {
    flex: 1;
    padding: 1.25rem;
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
    overflow-y: auto;
  }

  .gradients {
    display: flex;
    gap: 12px;
    flex-wrap: wrap;
  }

  .gradient-swatch {
    width: 58px;
    height: 58px;
    border-radius: 12px;
    cursor: pointer;
    box-shadow: 0 4px 15px rgba(0, 0, 0, 0.25);
    transition: all 0.2s ease;
  }

  .gradient-swatch.selected {
    box-shadow: 0 0 0 4px #22c55e;
    transform: scale(1.08);
  }

  select, input[type="color"] {
    background: #334155;
    color: #e2e8f0;
    border: none;
    padding: 0.75rem 1rem;
    border-radius: 8px;
    font-size: 1rem;
    width: 100%;
    cursor: pointer;
  }

  input[type="color"] {
    width: 64px;
    height: 64px;
    padding: 4px;
  }

  .sidebar-footer {
    padding: 1.25rem;
    background: #1e2937;
    border-top: 1px solid #334155;
    display: flex;
    justify-content: center;
  }

  .export-btn {
    padding: 14px 36px;
    font-size: 1.05rem;
    background: #22c55e;
    color: #0f172a;
    border: none;
    border-radius: 12px;
    font-weight: 600;
    cursor: pointer;
    width: 100%;
  }

  .export-btn:hover {
    background: #16a34a;
    transform: translateY(-2px);
  }
</style>