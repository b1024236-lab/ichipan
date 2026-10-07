# ichipan
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
  <title>ラッピングテクスチャ合成エンジン</title>
  <!-- CSSファイルの読み込み -->
  <link rel="stylesheet" href="style.css">
</head>
<body>

  <h2>ラッピング合成</h2>

  <!-- 合成プレビューCanvas -->
  <div id="canvas-container">
    <canvas id="preview-canvas" width="2048" height="1024"></canvas>
  </div>

  <!-- 操作パネル -->
  <div class="controls">
    <!-- 行1: 背景選択 & 画像ファイル挿入 -->
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

    <!-- 行2: タイトル ＆ サブタイトル -->
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

    <!-- 行3: 背景拡大率 -->
    <div class="slider-box">
      <label>背景拡大</label>
      <input type="range" id="bg-scale" min="10" max="300" value="100">
      <span id="bg-scale-val" class="slider-val">100%</span>
    </div>

    <!-- 行4: タイトル＆サブタイトルサイズ -->
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

    <!-- 行5: ボタン群 -->
    <div class="button-group">
      <button id="btn-download" class="btn-primary">生成画像をダウンロード</button>
      <button id="btn-reset" class="btn-secondary">初期位置・サイズにリセット</button>
    </div>
  </div>

  <!-- JavaScriptファイルの読み込み -->
  <script src="script.js"></script>
</body>
</html>
