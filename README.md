<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<title>Rubik's Timer</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<!-- Chart.js для графика -->
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<style>
body {
  margin: 0;
  background: linear-gradient(to bottom, #0f172a, #1e1b4b);
  font-family: Arial, sans-serif;
  color: white;
  display: flex;
  justify-content: center;
  padding: 20px;
}

.container {
  max-width: 800px;
  width: 100%;
}

/* Таймер-блок */
.timer-box {
  background: rgba(255,255,255,0.07);
  padding: 30px;
  border-radius: 20px;
  text-align: center;
  animation: fadeIn 0.6s ease;
}

/* Анимации */
@keyframes fadeIn {
  from { opacity: 0; transform: translateY(20px); }
  to   { opacity: 1; transform: translateY(0); }
}

@keyframes pulseIdle {
  0%   { transform: scale(1);    opacity: 0.85; }
  50%  { transform: scale(1.03); opacity: 1; }
  100% { transform: scale(1);    opacity: 0.85; }
}

@keyframes blinkStart {
  0%   { color: #38ff74; }
  100% { color: white; }
}

@keyframes stopPop {
  0%   { transform: scale(1); }
  40%  { transform: scale(1.12); }
  100% { transform: scale(1); }
}

.timer {
  font-size: 60px;
  margin-top: 20px;
  font-weight: bold;
  font-family: monospace;
  transition: color 0.2s;
}

/* Стиль состояний */
.timer.idle {
  animation: pulseIdle 1.8s infinite;
}

.timer.start {
  animation: blinkStart 0.4s;
  color: #38ff74;
}

.timer.stop {
  animation: stopPop 0.35s;
  color: #ffea61;
}

/* Скрамбл */
.scramble {
  margin-top: 20px;
  background: rgba(255,255,255,0.1);
  padding: 15px;
  border-radius: 10px;
  font-size: 16px;
  word-wrap: break-word;
  animation: fadeIn 0.7s ease;
}

button {
  padding: 10px 15px;
  margin: 5px;
  border: none;
  border-radius: 10px;
  background: rgba(255,255,255,0.2);
  color: white;
  font-size: 15px;
}

/* История */
.history {
  margin-top: 20px;
  background: rgba(255,255,255,0.07);
  padding: 20px;
  border-radius: 15px;
  max-height: 250px;
  overflow-y: auto;
}

.time-item {
  background: rgba(255,255,255,0.12);
  padding: 10px;
  border-radius: 8px;
  margin-bottom: 10px;
  font-family: monospace;
  animation: fadeIn 0.4s ease;
}

/* 3D куб */
.cube-wrapper {
  margin: 30px auto;
  width: 120px;
  height: 120px;
  perspective: 600px;
}

.cube {
  width: 100%;
  height: 100%;
  position: relative;
  transform-style: preserve-3d;
  animation: spin 6s infinite linear;
}

.cube div {
  position: absolute;
  width: 120px;
  height: 120px;
  background: rgba(255,255,255,0.2);
  border: 2px solid white;
}

@keyframes spin {
  from { transform: rotateX(0deg) rotateY(0deg); }
  to   { transform: rotateX(360deg) rotateY(360deg); }
}

/* Стороны куба */
.front  { transform: translateZ(60px); }
.back   { transform: rotateY(180deg) translateZ(60px); }
.right  { transform: rotateY(90deg)  translateZ(60px); }
.left   { transform: rotateY(-90deg) translateZ(60px); }
.top    { transform: rotateX(90deg)  translateZ(60px); }
.bottom { transform: rotateX(-90deg) translateZ(60px); }

</style>
</head>

<body>
<div class="container">

  <div class="timer-box">
    <h1>Rubik's Timer</h1>
    <p>Таймер запускается ПОСЛЕ отпускания пробела</p>

    <div class="cube-wrapper">
      <div class="cube">
        <div class="front"></div>
        <div class="back"></div>
        <div class="right"></div>
        <div class="left"></div>
        <div class="top"></div>
        <div class="bottom"></div>
      </div>
    </div>

    <div id="timer" class="timer idle">0.00</div>

    <div class="scramble">
      <span id="scramble"></span>
    </div>

    <button onclick="newScramble()">Новый скрамбл</button>
    <button onclick="copyScramble()">Скопировать</button>
    <button onclick="clearHistory()">Очистить историю</button>
  </div>

  <canvas id="chart" style="margin-top:30px;"></canvas>

  <div class="history" id="history"></div>

</div>

<script>
let running = false;
let startTime = 0;
let elapsed = 0;
let interval = null;
const timerEl = document.getElementById("timer");

function format(t) {
  return (t/1000).toFixed(2);
}

function startTimer() {
  running = true;
  startTime = performance.now();

  timerEl.classList.remove("idle","stop");
  timerEl.classList.add("start");

  interval = setInterval(() => {
    timerEl.textContent = format(performance.now() - startTime);
  }, 10);

  setTimeout(() => timerEl.classList.remove("start"), 300);
}

function stopTimer() {
  running = false;
  clearInterval(interval);
  elapsed = performance.now() - startTime;
  timerEl.textContent = format(elapsed);

  timerEl.classList.add("stop");

  saveTime(format(elapsed));
  updateChart();
  newScramble();

  setTimeout(() => {
    timerEl.classList.remove("stop");
    timerEl.classList.add("idle");
  }, 400);
}

window.addEventListener("keyup", e => {
  if (e.code === "Space") {
    if (!running) startTimer();
    else stopTimer();
  }
});

/* Скрамблы */
function generateScramble(len = 20) {
  const moves = ["U","D","L","R","F","B"];
  const mod = ["","'","2"];
  let out = [];
  let last = "";

  for (let i = 0; i < len; i++) {
    let m;
    do {
      m = moves[Math.floor(Math.random()*moves.length)];
    } while (m === last);
    last = m;
    out.push(m + mod[Math.floor(Math.random()*mod.length)]);
  }

  return out.join(" ");
}

function newScramble() {
  const scr = document.getElementById("scramble");
  scr.style.opacity = 0;
  setTimeout(() => {
    scr.textContent = generateScramble();
    scr.style.opacity = 1;
  }, 150);
}

function copyScramble() {
  navigator.clipboard.writeText(document.getElementById("scramble").textContent);
}

/* История */
let history = [];

function saveTime(t) {
  history.push(parseFloat(t));
  updateHistory();
}

function updateHistory() {
  let el = document.getElementById("history");
  el.innerHTML = "";
  history.slice().reverse().forEach(time => {
    el.innerHTML += `<div class="time-item">${time.toFixed(2)}</div>`;
  });
}

function clearHistory() {
  history = [];
  updateHistory();
  updateChart();
}

/* График прогресса */
const ctx = document.getElementById("chart");

const chart = new Chart(ctx, {
  type: "line",
  data: {
    labels: [],
    datasets: [{
      label: "Прогресс (сек)",
      data: [],
      borderWidth: 3
    }]
  },
  options: {
    responsive: true,
    scales: {
      y: { beginAtZero: false }
    }
  }
});

function updateChart() {
  chart.data.labels = history.map((_, i) => i+1);
  chart.data.datasets[0].data = history;
  chart.update();
}

/* Init */
newScramble();
</script>

</body>
</html>
