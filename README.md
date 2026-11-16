
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="theme-color" content="#090d14">
  <title>نبراس للتداول V27</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    * { box-sizing: border-box; }
    body {
      margin: 0;
      background: #090d14;
      color: #e5e7eb;
      font-family: Arial, sans-serif;
    }
    .panel {
      background: #111925;
      border: 1px solid #263244;
      border-radius: 14px;
      padding: 14px;
    }
    .gold { color: #d6b45c; }
    .green { color: #22c55e; }
    .red { color: #ef4444; }
    .muted { color: #91a0b5; }
    button, select, input {
      border-radius: 9px;
      padding: 10px;
      background: #1b2737;
      color: white;
      border: 1px solid #34445a;
    }
    button:active { transform: scale(.98); }
    .active {
      border-color: #d6b45c;
      color: #d6b45c;
    }
  </style>
</head>

<body>
  <header class="p-4 border-b border-slate-700">
    <div class="flex items-center justify-between">
      <div>
        <h1 class="text-2xl font-bold">
          <span class="green">N</span> نبراس
          <span class="gold">PRO</span>
        </h1>
        <p class="muted text-xs mt-1">V27 | تداول ذكي</p>
      </div>
      <div class="text-left">
        <span class="text-xs gold">BINANCE FUTURES</span>
        <p class="text-xs green">محرك التحليل</p>
      </div>
    </div>
  </header>

  <main class="p-3 space-y-3">

    <section class="panel">
      <div class="flex justify-between items-center mb-3">
        <h2 class="font-bold gold">لوحة التداول</h2>
        <span class="text-xs muted">تحليل تجريبي</span>
      </div>

      <label class="text-sm muted">العملة</label>
      <input id="symbol" value="BTCUSDT"
        class="w-full mt-1 mb-3" placeholder="مثال BTCUSDT">

      <label class="text-sm muted">الفريم الزمني</label>
      <div class="grid grid-cols-3 gap-2 mt-2">
        <button class="tf active" data-tf="15m">15 دقيقة</button>
        <button class="tf" data-tf="5m">5 دقائق</button>
        <button class="tf" data-tf="1h">ساعة</button>
      </div>

      <button id="analyzeBtn"
        class="w-full mt-4 font-bold"
        style="background:#166534;border-color:#22c55e">
        ابدأ التحليل
      </button>
    </section>

    <section class="panel">
      <h2 class="gold font-bold mb-3">نتيجة الإشارة</h2>
      <div class="text-center py-4">
        <div id="signal" class="text-3xl font-bold muted">
          انتظار
        </div>
        <p id="signalNote" class="text-xs muted mt-2">
          اضغط ابدأ التحليل لقراءة السوق
        </p>
      </div>
      <div class="grid grid-cols-2 gap-3 text-sm">
        <div>
          <p class="muted">سعر الدخول</p>
          <p id="entry" class="font-bold">—</p>
        </div>
        <div>
          <p class="muted">وقف الخسارة</p>
          <p id="stop" class="red font-bold">—</p>
        </div>
        <div>
          <p class="muted">الهدف الأول</p>
          <p id="target1" class="green font-bold">—</p>
        </div>
        <div>
          <p class="muted">الهدف الثاني</p>
          <p id="target2" class="green font-bold">—</p>
        </div>
        <div>
          <p class="muted">الهدف الثالث</p>
          <p id="target3" class="green font-bold">—</p>
        </div>
        <div>
          <p class="muted">قوة الإشارة</p>
          <p id="strength" class="gold font-bold">—</p>
        </div>
      </div>
    </section>

    <section class="panel">
      <h2 class="gold font-bold mb-3">تأكيد الاتجاه</h2>
      <div class="grid grid-cols-3 gap-2 text-center text-sm">
        <div class="bg-slate-800 rounded-lg p-3">
          <p class="muted">4 ساعات</p>
          <p id="trend4h">—</p>
        </div>
        <div class="bg-slate-800 rounded-lg p-3">
          <p class="muted">ساعة</p>
          <p id="trend1h">—</p>
        </div>
        <div class="bg-slate-800 rounded-lg p-3">
          <p class="muted">الفريم المختار</p>
          <p id="trendMain">—</p>
        </div>
      </div>
    </section>

    <section class="panel">
      <h2 class="gold font-bold mb-2">مخطط العملة</h2>
      <div id="chart" class="w-full" style="height:420px"></div>
    </section>

    <footer class="text-center muted text-xs py-4">
      نبراس للتداول — تداول ذكي
      <p class="mt-1">الإشارات تحليلية وليست ضماناً للربح</p>
    </footer>
  </main>

  <script>
    let selectedTimeframe = "15m";

    document.querySelectorAll(".tf").forEach(button => {
      button.addEventListener("click", () => {
        document.querySelectorAll(".tf").forEach(b =>
          b.classList.remove("active")
        );
        button.classList.add("active");
        selectedTimeframe = button.dataset.tf;
      });
    });
    
// ===== محرك تحليل نبراس V27 =====

function calculateEMA(values, period) {
  const k = 2 / (period + 1);
  let ema = values[0];

  for (let i = 1; i < values.length; i++) {
    ema = values[i] * k + ema * (1 - k);
  }

  return ema;
}

function calculateRSI(values, period = 14) {
  if (values.length <= period) return 50;

  let gain = 0;
  let loss = 0;

  for (let i = 1; i <= period; i++) {
    const change = values[i] - values[i - 1];
    gain += Math.max(change, 0);
    loss += Math.max(-change, 0);
  }

  let avgGain = gain / period;
  let avgLoss = loss / period;

  for (let i = period + 1; i < values.length; i++) {
    const change = values[i] - values[i - 1];
    avgGain =
      (avgGain * (period - 1) + Math.max(change, 0)) / period;
    avgLoss =
      (avgLoss * (period - 1) + Math.max(-change, 0)) / period;
  }

  if (avgLoss === 0) return 100;

  const rs = avgGain / avgLoss;
  return 100 - (100 / (1 + rs));
}

function calculateATR(candles, period = 14) {
  const trs = [];

  for (let i = 1; i < candles.length; i++) {
    const current = candles[i];
    const previous = candles[i - 1];

    const tr = Math.max(
      current.high - current.low,
      Math.abs(current.high - previous.close),
      Math.abs(current.low - previous.close)
    );

    trs.push(tr);
  }

  const recent = trs.slice(-period);
  return recent.reduce((a, b) => a + b, 0) / recent.length;
}

async function getMarketData(symbol, interval) {
  const url =
    `https://fapi.binance.com/fapi/v1/klines?symbol=${encodeURIComponent(symbol)}&interval=${interval}&limit=150`;

  const response = await fetch(url);

  if (!response.ok) {
    throw new Error(`تعذر جلب بيانات ${interval}`);
  }

  const raw = await response.json();

  if (!Array.isArray(raw) || raw.length < 60) {
    throw new Error("بيانات السوق غير كافية");
  }

  // استبعاد الشمعة الحالية غير المغلقة
  const candles = raw.slice(0, -1).map(item => ({
    high: Number(item[2]),
    low: Number(item[3]),
    close: Number(item[4])
  }));

  const closes = candles.map(c => c.close);

  const ema20 = calculateEMA(closes, 20);
  const ema50 = calculateEMA(closes, 50);
  const rsi = calculateRSI(closes, 14);
  const atr = calculateATR(candles, 14);
  const price = closes[closes.length - 1];

  let trend = "WAIT";

  if (ema20 > ema50) trend = "LONG";
  if (ema20 < ema50) trend = "SHORT";

  return {
    price,
    ema20,
    ema50,
    rsi,
    atr,
    trend
  };
}

function showTrend(elementId, trend) {
  const element = document.getElementById(elementId);

  const labels = {
    LONG: "صاعد ▲",
    SHORT: "هابط ▼",
    WAIT: "محايد"
  };

  element.textContent = labels[trend] || "—";
  element.className =
    trend === "LONG" ? "green font-bold" :
    trend === "SHORT" ? "red font-bold" :
    "muted font-bold";
}

function formatPrice(value) {
  if (!Number.isFinite(value)) return "—";

  return value.toLocaleString("en-US", {
    maximumFractionDigits: 8
  });
}

async function runNebrasAnalysis() {
  const button = document.getElementById("analyzeBtn");
  const symbol = document.getElementById("symbol")
    .value.trim().toUpperCase();

  if (!/^[A-Z0-9]{5,30}$/.test(symbol)) {
    alert("أدخل رمز عملة صحيحاً مثل BTCUSDT");
    return;
  }

  button.disabled = true;
  button.textContent = "جارٍ تحليل السوق...";

  document.getElementById("signal").textContent = "تحليل";
  document.getElementById("signalNote").textContent =
    "جارٍ قراءة الفريمات الثلاثة";

  try {
    const [data4h, data1h, dataMain] = await Promise.all([
      getMarketData(symbol, "4h"),
      getMarketData(symbol, "1h"),
      getMarketData(symbol, selectedTimeframe)
    ]);

    showTrend("trend4h", data4h.trend);
    showTrend("trend1h", data1h.trend);
    showTrend("trendMain", dataMain.trend);

    const allLong =
      data4h.trend === "LONG" &&
      data1h.trend === "LONG" &&
      dataMain.trend === "LONG";

    const allShort =
      data4h.trend === "SHORT" &&
      data1h.trend === "SHORT" &&
      dataMain.trend === "SHORT";

    let signal = "WAIT";

    if (allLong && dataMain.rsi >= 50 && dataMain.rsi < 70) {
      signal = "LONG";
    }

    if (allShort && dataMain.rsi <= 50 && dataMain.rsi > 30) {
      signal = "SHORT";
    }

    const entry = dataMain.price;
    const risk = dataMain.atr * 1.5;

    let stop = null;
    let target1 = null;
    let target2 = null;
    let target3 = null;

    if (signal === "LONG") {
      stop = entry - risk;
      target1 = entry + risk;
      target2 = entry + risk * 2;
      target3 = entry + risk * 3;
    }

    if (signal === "SHORT") {
      stop = entry + risk;
      target1 = entry - risk;
      target2 = entry - risk * 2;
      target3 = entry - risk * 3;
    }

    const signalElement = document.getElementById("signal");

    signalElement.textContent =
      signal === "LONG" ? "شراء LONG" :
      signal === "SHORT" ? "بيع SHORT" :
      "انتظار WAIT";

    signalElement.className =
      signal === "LONG" ? "text-3xl font-bold green" :
      signal === "SHORT" ? "text-3xl font-bold red" :
      "text-3xl font-bold muted";

    document.getElementById("signalNote").textContent =
      signal === "WAIT"
        ? "لا يوجد توافق كافٍ بين الفريمات وشروط RSI"
        : `اتجاه متوافق على الفريمات الثلاثة | RSI: ${dataMain.rsi.toFixed(1)}`;

    document.getElementById("entry").textContent =
      formatPrice(signal === "WAIT" ? null : entry);

    document.getElementById("stop").textContent =
      formatPrice(stop);

    document.getElementById("target1").textContent =
      formatPrice(target1);

    document.getElementById("target2").textContent =
      formatPrice(target2);

    document.getElementById("target3").textContent =
      formatPrice(target3);

    document.getElementById("strength").textContent =
      signal === "WAIT" ? "غير مكتملة" : "3 من 3 فريمات";

  } catch (error) {
    document.getElementById("signal").textContent = "تعذر التحليل";
    document.getElementById("signalNote").textContent =
      error.message || "تحقق من الاتصال ورمز العملة";

    console.error(error);

  } finally {
    button.disabled = false;
    button.textContent = "ابدأ التحليل";
  }
}

document.getElementById("analyzeBtn")
  .addEventListener("click", runNebrasAnalysis);

  </script>
</body>
</html>
