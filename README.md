<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>紙張單價計算機</title>
  <style>
    :root {
      --primary-color: #007aff;
      --bg-color: #f2f2f7;
      --card-bg: #ffffff;
      --text-color: #1c1c1e;
      --border-color: #e5e5ea;
    }
    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      background-color: var(--bg-color);
      color: var(--text-color);
      margin: 0;
      padding: 16px;
    }
    .container {
      max-width: 500px;
      margin: 0 auto;
    }
    h2 {
      text-align: center;
      margin-bottom: 20px;
    }
    .card {
      background: var(--card-bg);
      border-radius: 12px;
      padding: 16px;
      margin-bottom: 16px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.05);
    }
    .form-group {
      margin-bottom: 12px;
      display: flex;
      flex-direction: column;
    }
    label {
      font-size: 14px;
      font-weight: 600;
      margin-bottom: 4px;
      color: #666;
    }
    input {
      padding: 10px;
      font-size: 16px;
      border: 1px solid var(--border-color);
      border-radius: 8px;
      outline: none;
    }
    input:focus {
      border-color: var(--primary-color);
    }
    .btn-group {
      display: flex;
      flex-direction: column;
      gap: 10px;
      margin-top: 10px;
    }
    button {
      width: 100%;
      border: none;
      padding: 14px;
      font-size: 18px;
      font-weight: bold;
      border-radius: 10px;
      cursor: pointer;
    }
    .btn-calc {
      background-color: var(--primary-color);
      color: white;
    }
    .btn-clear {
      background-color: #e5e5ea;
      color: #3a3a3c;
    }
    .btn-clear:active {
      background-color: #d1d1d6;
    }
    .result-row {
      display: flex;
      justify-content: space-between;
      padding: 8px 0;
      border-bottom: 1px solid var(--border-color);
    }
    .result-row:last-child {
      border-bottom: none;
    }
    .result-value {
      font-weight: bold;
      color: var(--primary-color);
    }
  </style>
</head>
<body>

<div class="container">
  <h2>紙張單價計算機</h2>
  
  <div class="card">
    <div class="form-group"><label>長</label><input type="number" id="length" placeholder="請輸入長"></div>
    <div class="form-group"><label>寬</label><input type="number" id="width" placeholder="請輸入寬"></div>
    <div class="form-group"><label>規格</label><input type="text" id="spec" placeholder="例如：全紙"></div>
    <div class="form-group"><label>盎司 (Oz)</label><input type="number" id="ounce" placeholder="請輸入盎司"></div>
    <div class="form-group"><label>客訂張數</label><input type="number" id="customSheet" placeholder="請輸入客訂張數"></div>
    <div class="form-group"><label>噸價 (KG)</label><input type="number" id="tonPrice" placeholder="請輸入噸價"></div>
    <div class="form-group"><label>各式加工費 (一噸)</label><input type="number" id="procTon" placeholder="請輸入各式加工費"></div>
    <div class="form-group"><label>基本加工費 (一批)</label><input type="number" id="procBatch" placeholder="請輸入基本加工費"></div>
    <div class="form-group"><label>裁工 (單張)</label><input type="number" id="cutSheet" placeholder="請輸入裁工(單張)"></div>
    <div class="form-group"><label>裁工 (一批)</label><input type="number" id="cutBatch" placeholder="請輸入裁工(一批)"></div>
    
    <div class="btn-group">
      <button class="btn-calc" onclick="calculate()">開始計算</button>
      <button class="btn-clear" onclick="clearFields()">清除資料</button>
    </div>
  </div>

  <div class="card">
    <h3>計算結果</h3>
    <div class="result-row"><span>張數 (盎司):</span><span class="result-value" id="resOunceSheet">0</span></div>
    <div class="result-row"><span>一噸張數:</span><span class="result-value" id="resTonSheet">0</span></div>
    <div class="result-row"><span>客訂重量 (KG):</span><span class="result-value" id="resCustomWeight">0</span></div>
    <div class="result-row"><span>單張價格:</span><span class="result-value" id="resUnitPrice">0</span></div>
  </div>
</div>

<script>
  // 盎司對應張數對照表
  const ounceMap = {
    8: 3760, 10: 3000, 12: 2480, 14: 2120, 16: 1880,
    18: 1680, 20: 1500, 22: 1365, 24: 1240, 26: 1156,
    28: 1060, 30: 1002, 32: 940,  34: 884,  36: 835,
    38: 791,  40: 752,  42: 716,  44: 683,  46: 653,
    48: 626,  50: 601,  52: 578,  54: 557,  56: 537
  };

  // 輔助函式：無條件進位到小數點第一位
  function ceilTo1Dec(val) {
    if (!val || isNaN(val) || val === Infinity) return 0;
    return Math.ceil(val * 10) / 10;
  }

  function calculate() {
    const L = parseFloat(document.getElementById('length').value) || 0;
    const W = parseFloat(document.getElementById('width').value) || 0;
    const ounce = parseFloat(document.getElementById('ounce').value) || 0;
    const customSheet = parseFloat(document.getElementById('customSheet').value) || 0;
    const tonPrice = parseFloat(document.getElementById('tonPrice').value) || 0;
    const procTon = parseFloat(document.getElementById('procTon').value) || 0;
    const procBatch = parseFloat(document.getElementById('procBatch').value) || 0;
    const cutSheet = parseFloat(document.getElementById('cutSheet').value) || 0;
    const cutBatch = parseFloat(document.getElementById('cutBatch').value) || 0;

    // 1. 張數(盎司)
    const ounceSheet = ounceMap[ounce] || 0;

    // 2. 一噸張數: 取整數(張數(盎司) / 長 / 寬 * 936)
    let tonSheet = 0;
    if (L > 0 && W > 0) {
      tonSheet = Math.floor((ounceSheet / L / W) * 936);
    }

    // 3. 客訂重量: 取整數(客訂張數 / 一噸張數 * 1000 + 0.7)
    let customWeight = 0;
    if (tonSheet > 0) {
      customWeight = Math.floor((customSheet / tonSheet) * 1000 + 0.7);
    }

    // 4. 單張價格: 無條件進位到小數點第一位計算
    let p1 = tonSheet > 0 ? ceilTo1Dec((tonPrice * 1000) / tonSheet) : 0;
    let p2 = tonSheet > 0 ? ceilTo1Dec(procTon / tonSheet) : 0;
    let p3 = cutSheet;
    let p4 = customSheet > 0 ? ceilTo1Dec(procBatch / customSheet) : 0;
    let p5 = customSheet > 0 ? ceilTo1Dec(cutBatch / customSheet) : 0;

    // 計算總和，並避免 JavaScript 浮點數精度問題
    const unitPrice = parseFloat((p1 + p2 + p3 + p4 + p5).toFixed(1));

    // 顯示結果
    document.getElementById('resOunceSheet').innerText = ounceSheet;
    document.getElementById('resTonSheet').innerText = tonSheet;
    document.getElementById('resCustomWeight').innerText = customWeight;
    document.getElementById('resUnitPrice').innerText = unitPrice;
  }

  // 清除資料函式：清空所有輸入框
  function clearFields() {
    const inputIds = [
      'length', 'width', 'spec', 'ounce', 'customSheet',
      'tonPrice', 'procTon', 'procBatch', 'cutSheet', 'cutBatch'
    ];
    inputIds.forEach(id => {
      document.getElementById(id).value = '';
    });

    // 歸零計算結果
    document.getElementById('resOunceSheet').innerText = '0';
    document.getElementById('resTonSheet').innerText = '0';
    document.getElementById('resCustomWeight').innerText = '0';
    document.getElementById('resUnitPrice').innerText = '0';
  }
</script>

</body>
</html>
