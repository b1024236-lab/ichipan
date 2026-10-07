<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
  <title>ラッピングテクスチャ合成エンジン</title>
  <style>
    /* --- CSSデザイン --- */
    * {
      box-sizing: border-box;
      -webkit-tap-highlight-color: transparent;
    }

    body {
      font-family: -apple-system, BlinkMacSystemFont, "SF Pro Display", "SF Pro Text", "Helvetica Neue", Arial, sans-serif;
      margin: 0;
      padding: 16px 12px;
      background-color: #f2f2f7;
      color: #1c1c1e;
      overscroll-behavior: none;
    }

    h2 {
      font-size: 20px;
      font-weight: 700;
      margin: 0 0 12px 0;
      text-align: center;
      letter-spacing: -0.5px;
    }

    #canvas-container {
      width: 100%;
      background: #000;
      border-radius: 16px;
      overflow: hidden;
      box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);
      margin-bottom: 16px;
    }

    #preview-canvas {
      width: 100%;
      height: auto;
      display: block;
      touch-action: none;
      cursor: move;
    }

    .controls {
      background: #ffffff;
      padding: 16px;
      border-radius: 16px;
      display: flex;
      flex-direction: column;
      gap: 12px;
      box-shadow: 0 4px 16px rgba(0, 0, 0, 0.05);
    }

    .row {
      display: flex;
      gap: 10px;
    }

    .row > * {
      flex: 1;
    }

    .control-group {
      display: flex;
      flex-direction: column;
      gap: 6px;
    }

    label {
      font-size: 12px;
      font-weight: 600;
      color: #8e8e93;
      padding-left: 2px;
    }

    input[type="text"], select, input[type="file"] {
      font-size: 15px;
      padding: 10px 12px;
      border: 1px solid #e5e5ea;
      border-radius: 10px;
      background: #f2f2f7;
      color: #000;
      outline: none;
      width: 100%;
    }

    .slider-box {
      display: flex;
      align-items: center;
      justify-content: space-between;
      background: #f2f2f7;
      padding: 8px 12px;
      border-radius: 10px;
    }

    .slider-box label {
      font-size: 13px;
      color: #1c1c1e;
      margin: 0;
    }

    .slider-box input[type="range"] {
      width: 50%;
      accent-color: #007aff;
    }

    .slider-val {
      font-size: 12px;
      font-weight: 700;
      color: #007aff;
      min-width: 40px;
      text-align: right;
    }

    .button-group {
      display: flex;
      flex-direction: column;
      gap: 8px;
      margin-top: 4px;
    }

    button {
      font-size: 15px;
      font-weight: 600;
      padding: 12px;
      border: none;
      border-radius: 12px;
      cursor: pointer;
    }

    button:active {
      opacity: 0.7;
    }

    .btn-primary {
      background: #34c759;
      color: #ffffff;
    }

    .btn-secondary {
      background: #8e8e93;
      color: #ffffff;
    }
  </style>
</head>
<body>

  <h2>ラッピング合成</h2>

  <!-- プレビューCanvas -->
  <div id="canvas-container">
    <canvas id="preview-canvas" width="2048" height="1024"></canvas>
  </div>

  <!-- 操作パネル -->
  <div class="controls">
    <div class="row">
      <div class="control-group">
        <label>🎨 背景プリセット</label>
        <select id="bg-style">
          <option value="wood">高級木目調</option>
          <option value="red-stripe">赤グラデーション</option>
          <option value="gold-pattern">ゴールド和柄</option>
        </select>
      </div>

      <div class="control-group">
        <label>📷 背景画像挿入</label>
        <input type="file" id="bg-file" accept="image/*">
      </div>
    </div>

    <div class="row">
      <div class="control-group">
        <label>タイトル</label>
        <input type="text" id="main-title" value="龍鳳">
      </div>
      <div class="control-group">
        <label>サブタイトル</label>
        <input type="text" id="sub-title" value="黄金塩ラーメン 800円">
      </div>
    </div>

    <div class="slider-box">
      <label>背景拡大</label>
      <input type="range" id="bg-scale" min="10" max="300" value="100">
      <span id="bg-scale-val" class="slider-val">100%</span>
    </div>

    <div class="row">
      <div class="slider-box">
        <label>タイトル大</label>
        <input type="range" id="title-size" min="40" max="220" value="130">
        <span id="title-size-val" class="slider-val">130px</span>
      </div>
      <div class="slider-box">
        <label>サブタイトル大</label>
        <input type="range" id="sub-size" min="20" max="100" value="48">
        <span id="sub-size-val" class="slider-val">48px</span>
      </div>
    </div>

    <div class="button-group">
      <button id="btn-download" class="btn-primary">生成画像をダウンロード</button>
      <button id="btn-reset" class="btn-secondary">初期位置・サイズにリセット</button>
    </div>
  </div>

  <!-- JavaScript処理 -->
  <script>
    const canvas = document.getElementById('preview-canvas');
    const ctx = canvas.getContext('2d');

    const bgScaleInput = document.getElementById('bg-scale');
    const titleSizeInput = document.getElementById('title-size');
    const subSizeInput = document.getElementById('sub-size');

    const bgScaleVal = document.getElementById('bg-scale-val');
    const titleSizeVal = document.getElementById('title-size-val');
    const subSizeVal = document.getElementById('sub-size-val');

    const bgFileInput = document.getElementById('bg-file');
    const bgStyleSelect = document.getElementById('bg-style');

    let customBgImage = null;
    let bgX = 0, bgY = 0;
    let titleX = 1024, titleY = 730;
    let subX = 1024, subY = 885;

    let activeDragTarget = null;
    let dragOffsetX = 0, dragOffsetY = 0;

    let titleBoundingBox = { x: 0, y: 0, w: 0, h: 0 };
    let subBoundingBox = { x: 0, y: 0, w: 0, h: 0 };

    function drawBackgroundPattern(theme) {
      const w = canvas.width, h = canvas.height;
      if (customBgImage) {
        const scale = parseInt(bgScaleInput.value, 10) / 100;
        ctx.drawImage(customBgImage, bgX, bgY, customBgImage.width * scale, customBgImage.height * scale);
        return;
      }
      if (theme === 'wood') {
        const grad = ctx.createLinearGradient(0, 0, 0, h);
        grad.addColorStop(0, '#2b1704'); grad.addColorStop(0.5, '#4a2a0c'); grad.addColorStop(1, '#1a0d02');
        ctx.fillStyle = grad; ctx.fillRect(0, 0, w, h);
      } else if (theme === 'red-stripe') {
        const grad = ctx.createLinearGradient(0, 0, 0, h);
        grad.addColorStop(0, '#8b0000'); grad.addColorStop(0.5, '#d32f2f'); grad.addColorStop(1, '#4a0000');
        ctx.fillStyle = grad; ctx.fillRect(0, 0, w, h);
      } else if (theme === 'gold-pattern') {
        const grad = ctx.createRadialGradient(w/2, h/2, 100, w/2, h/2, w/1.2);
        grad.addColorStop(0, '#ffe082'); grad.addColorStop(0.5, '#ffb300'); grad.addColorStop(1, '#ff6f00');
        ctx.fillStyle = grad; ctx.fillRect(0, 0, w, h);
      }
    }

    function drawLantern(x, y, text) {
      ctx.save();
      ctx.fillStyle = '#d32f2f';
      ctx.beginPath();
      ctx.roundRect(x, y, 120, 260, 25);
      ctx.fill();
      ctx.strokeStyle = '#000000';
      ctx.lineWidth = 5;
      ctx.stroke();

      ctx.fillStyle = '#ffffff';
      ctx.font = 'bold 36px sans-serif';
      ctx.textAlign = 'center';
      ctx.textBaseline = 'middle';
      
      const startY = y + 40;
      const stepY = 45;
      for (let i = 0; i < text.length; i++) {
        ctx.fillText(text[i], x + 60, startY + (i * stepY));
      }
      ctx.restore();
    }

    function drawContent(titleText, subtitleText) {
      const titleSize = parseInt(titleSizeInput.value, 10);
      const subSize = parseInt(subSizeInput.value, 10);

      ctx.save();
      ctx.font = `bold ${titleSize}px "Yu Mincho", "Hiragino Mincho ProN", serif`;
      ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
      const titleMetrics = ctx.measureText(titleText);
      const titleW = titleMetrics.width + 40, titleH = titleSize * 1.2;
      titleBoundingBox = { x: titleX - titleW / 2, y: titleY - titleH / 2, w: titleW, h: titleH };

      ctx.fillStyle = '#ffffff'; ctx.strokeStyle = '#000000';
      ctx.lineWidth = Math.max(6, titleSize * 0.14);
      ctx.strokeText(titleText, titleX, titleY);
      ctx.fillText(titleText, titleX, titleY);
      ctx.restore();

      ctx.save();
      ctx.font = `bold ${subSize}px sans-serif`;
      const subMetrics = ctx.measureText(subtitleText);
      const bannerW = Math.max(subMetrics.width + 120, 300), bannerH = subSize * 1.8;
      subBoundingBox = { x: subX - bannerW / 2, y: subY - bannerH / 2, w: bannerW, h: bannerH };

      ctx.fillStyle = 'rgba(255, 248, 225, 0.9)';
      ctx.fillRect(subBoundingBox.x, subBoundingBox.y, bannerW, bannerH);
      ctx.fillStyle = '#d32f2f'; ctx.textAlign = 'center'; ctx.textBaseline = 'middle';
      ctx.fillText(subtitleText, subX, subY);
      ctx.restore();

      drawLantern(80, 560, 'ジャッチュ');
      drawLantern(canvas.width - 200, 560, 'ジャッチュ');
    }

    function drawWindowAndDoorMask() {
      const w = canvas.width, windowY = 220, windowH = 320;
      ctx.save();
      const windowCount = 6, startX = 300, totalWidth = w - 600;
      const windowW = (totalWidth - (windowCount - 1) * 30) / windowCount;

      for (let i = 0; i < windowCount; i++) {
        const x = startX + i * (windowW + 30);
        ctx.fillStyle = '#111111';
        ctx.fillRect(x - 6, windowY - 6, windowW + 12, windowH + 12);
        const glassGrad = ctx.createLinearGradient(x, windowY, x + windowW, windowY + windowH);
        glassGrad.addColorStop(0, 'rgba(180, 220, 240, 0.65)');
        glassGrad.addColorStop(1, 'rgba(200, 240, 255, 0.7)');
        ctx.fillStyle = glassGrad;
        ctx.fillRect(x, windowY, windowW, windowH);
      }
      ctx.restore();
    }

    function generateTexture() {
      bgScaleVal.textContent = `${bgScaleInput.value}%`;
      titleSizeVal.textContent = `${titleSizeInput.value}px`;
      subSizeVal.textContent = `${subSizeInput.value}px`;

      drawBackgroundPattern(bgStyleSelect.value);
      drawContent(document.getElementById('main-title').value, document.getElementById('sub-title').value);
      drawWindowAndDoorMask();
    }

    function getCanvasCoords(e) {
      const rect = canvas.getBoundingClientRect();
      const scaleX = canvas.width / rect.width;
      const scaleY = canvas.height / rect.height;
      return {
        x: (e.clientX - rect.left) * scaleX,
        y: (e.clientY - rect.top) * scaleY
      };
    }

    function isInside(p, box) {
      return p.x >= box.x && p.x <= box.x + box.w && p.y >= box.y && p.y <= box.y + box.h;
    }

    canvas.addEventListener('pointerdown', (e) => {
      const coords = getCanvasCoords(e);
      if (isInside(coords, subBoundingBox)) {
        activeDragTarget = 'sub'; dragOffsetX = coords.x - subX; dragOffsetY = coords.y - subY;
      } else if (isInside(coords, titleBoundingBox)) {
        activeDragTarget = 'title'; dragOffsetX = coords.x - titleX; dragOffsetY = coords.y - titleY;
      } else if (customBgImage) {
        activeDragTarget = 'bg'; dragOffsetX = coords.x - bgX; dragOffsetY = coords.y - bgY;
      }
      if (activeDragTarget) canvas.setPointerCapture(e.pointerId);
    });

    canvas.addEventListener('pointermove', (e) => {
      if (!activeDragTarget) return;
      const coords = getCanvasCoords(e);
      const newX = Math.round(coords.x - dragOffsetX);
      const newY = Math.round(coords.y - dragOffsetY);

      if (activeDragTarget === 'title') { titleX = newX; titleY = newY; }
      else if (activeDragTarget === 'sub') { subX = newX; subY = newY; }
      else if (activeDragTarget === 'bg') { bgX = newX; bgY = newY; }
      generateTexture();
    });

    function stopDrag(e) {
      if (activeDragTarget) {
        activeDragTarget = null;
        try { canvas.releasePointerCapture(e.pointerId); } catch(err) {}
      }
    }
    canvas.addEventListener('pointerup', stopDrag);
    canvas.addEventListener('pointercancel', stopDrag);

    bgFileInput.addEventListener('change', (e) => {
      const file = e.target.files[0];
      if (file) {
        const img = new Image();
        img.onload = () => {
          customBgImage = img;
          const scaleX = canvas.width / img.width, scaleY = canvas.height / img.height;
          const fitScale = Math.max(scaleX, scaleY);
          bgScaleInput.value = Math.round(fitScale * 100);
          bgX = (canvas.width - img.width * fitScale) / 2;
          bgY = (canvas.height - img.height * fitScale) / 2;
          generateTexture();
        };
        img.src = URL.createObjectURL(file);
      }
    });

    bgStyleSelect.addEventListener('change', () => { customBgImage = null; generateTexture(); });
    bgScaleInput.addEventListener('input', generateTexture);
    titleSizeInput.addEventListener('input', generateTexture);
    subSizeInput.addEventListener('input', generateTexture);
    document.getElementById('main-title').addEventListener('input', generateTexture);
    document.getElementById('sub-title').addEventListener('input', generateTexture);

    document.getElementById('btn-download').addEventListener('click', () => {
      const link = document.createElement('a');
      link.download = 'tram-texture.png';
      link.href = canvas.toDataURL('image/png');
      link.click();
    });

    document.getElementById('btn-reset').addEventListener('click', () => {
      titleSizeInput.value = 130; subSizeInput.value = 48;
      titleX = 1024; titleY = 730; subX = 1024; subY = 885;
      bgScaleInput.value = 100; bgX = 0; bgY = 0; customBgImage = null;
      document.getElementById('main-title').value = "龍鳳";
      generateTexture();
    });

    generateTexture();
  </script>
</body>
</html>
