<!DOCTYPE html>
<html lang="ko">
<head>
    
  <meta charset="UTF-8">
  <title>bownce</title>
  <style>
    * { box-sizing: border-box; }
    body, html {
      margin: 0;
      padding: 0;
      font-family: sans-serif;
      background: #111;
      color: #fff;
      text-align: center;
    }
    #gameCanvas {
      display: block;
      margin: 0 auto;
      background: #333;
    }
    .screen {
      display: none;
      padding: 20px;
    }
    .active {
      display: block;
    }
    button {
      margin: 10px;
      padding: 10px 20px;
      font-size: 16px;
      border: none;
      border-radius: 8px;
      background-color: #1abc9c;
      color: white;
      cursor: pointer;
    }
    button:hover {
      background-color: #16a085;
    }
  </style>
</head>
<body>

<div id="mainScreen" class="screen active">
  <h1>공 튀기는걸 조종하자</h1>
  <p>스테이지를 선택:</p>
  <div id="stageButtons"></div>
</div>

<canvas id="gameCanvas" width="800" height="600"></canvas>

<div id="resultScreen" class="screen">
  <h1 id="resultMessage">게임 결과</h1>
  <p id="timeDisplay"></p>
  <button onclick="restartStage()">다시하기</button>
  <button onclick="goToMain()">메인으로</button>
</div>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

const gravity = 0.4;
const bounce = -0.8;
let currentStage = 0;
let stages = [];
let gameInterval;
let startTime;
let keys = {};

const ball = {
  x: 0,
  y: 0,
  radius: 15,
  vx: 0,
  vy: -10, // 초기 점프
  alive: true,
};

const GOAL_SIZE = 40;
const spikeSize = 20;

function defineStages() {
  stages = [
    {
      platforms: [
        {x: 100, y: 500, width: 200, height: 10},
        {x: 350, y: 400, width: 200, height: 10},
      ],
      spikes: [
        {x: 300, y: 580, width: spikeSize, height: spikeSize},
      ],
      goal: {x: 700, y: 550, width: GOAL_SIZE, height: GOAL_SIZE},
    },
    {
      platforms: [
        {x: 50, y: 550, width: 150, height: 10},
        {x: 250, y: 450, width: 150, height: 10},
        {x: 500, y: 350, width: 200, height: 10},
      ],
      spikes: [
        {x: 400, y: 580, width: spikeSize, height: spikeSize},
      ],
      goal: {x: 750, y: 550, width: GOAL_SIZE, height: GOAL_SIZE},
    },
    {
      platforms: [
        {x: 50, y: 500, width: 100, height: 10},
        {x: 200, y: 400, width: 100, height: 10},
        {x: 350, y: 300, width: 100, height: 10},
        {x: 500, y: 200, width: 100, height: 10},
      ],
      spikes: [],
      goal: {x: 750, y: 150, width: GOAL_SIZE, height: GOAL_SIZE},
    },
    {
      platforms: [
        {x: 100, y: 500, width: 600, height: 10},
      ],
      spikes: [
        {x: 400, y: 580, width: spikeSize, height: spikeSize},
        {x: 450, y: 580, width: spikeSize, height: spikeSize},
        {x: 500, y: 580, width: spikeSize, height: spikeSize},
      ],
      goal: {x: 700, y: 460, width: GOAL_SIZE, height: GOAL_SIZE},
    },
  ];
}

function drawPlatform(p) {
  ctx.fillStyle = "#0f0";
  ctx.fillRect(p.x, p.y, p.width, p.height);
}

function drawSpike(s) {
  ctx.fillStyle = "red";
  ctx.fillRect(s.x, s.y, s.width, s.height);
}

function drawGoal(g) {
  ctx.fillStyle = "gold";
  ctx.fillRect(g.x, g.y, g.width, g.height);
}

function drawBall() {
  ctx.beginPath();
  ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
  ctx.fillStyle = "#f39c12";
  ctx.fill();
  ctx.closePath();
}

function updateGame() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  const stage = stages[currentStage];

  if (keys["ArrowLeft"]) ball.vx = -3;
  else if (keys["ArrowRight"]) ball.vx = 3;
  else ball.vx = 0;

  ball.vy += gravity;
  ball.x += ball.vx;
  ball.y += ball.vy;

  if (ball.y + ball.radius > canvas.height) {
    endGame(false);
    return;
  }

  for (let s of stage.spikes) {
    if (ball.x > s.x && ball.x < s.x + s.width && ball.y + ball.radius > s.y) {
      endGame(false);
      return;
    }
  }

  const g = stage.goal;
  if (ball.x > g.x && ball.x < g.x + g.width && ball.y > g.y && ball.y < g.y + g.height) {
    const timeTaken = ((Date.now() - startTime) / 1000).toFixed(2);
    document.getElementById("resultMessage").textContent = "🎉 클리어!";
    document.getElementById("timeDisplay").textContent = `소요 시간: ${timeTaken}초`;
    endGame(true);
    return;
  }

  for (let p of stage.platforms) {
    if (ball.x > p.x && ball.x < p.x + p.width && ball.y + ball.radius > p.y && ball.y < p.y + p.height) {
      ball.y = p.y - ball.radius;
      ball.vy = -10; // 튕기기 유지
    }
  }

  stage.platforms.forEach(drawPlatform);
  stage.spikes.forEach(drawSpike);
  drawGoal(stage.goal);
  drawBall();
}

function startGame(stageIndex) {
  document.getElementById("mainScreen").classList.remove("active");
  document.getElementById("resultScreen").classList.remove("active");
  canvas.style.display = "block";

  currentStage = stageIndex;
  const stage = stages[stageIndex];
  ball.x = 20;
  ball.y = 50;
  ball.vx = 0;
  ball.vy = -10;
  ball.alive = true;

  startTime = Date.now();
  gameInterval = setInterval(updateGame, 1000 / 60);
}

function endGame(success) {
  clearInterval(gameInterval);
  canvas.style.display = "none";
  document.getElementById("resultScreen").classList.add("active");
  if (!success) {
    document.getElementById("resultMessage").textContent = "💥 게임 오버!";
    document.getElementById("timeDisplay").textContent = "";
  }
}

function restartStage() {
  startGame(currentStage);
}

function goToMain() {
  document.getElementById("resultScreen").classList.remove("active");
  document.getElementById("mainScreen").classList.add("active");
}

function createStageButtons() {
  const container = document.getElementById("stageButtons");
  for (let i = 0; i < stages.length; i++) {
    const btn = document.createElement("button");
    btn.textContent = `스테이지 ${i + 1}`;
    btn.onclick = () => startGame(i);
    container.appendChild(btn);
  }
}

defineStages();
createStageButtons();

document.addEventListener("keydown", e => keys[e.key] = true);
document.addEventListener("keyup", e => keys[e.key] = false);
</script>

</body>
</html>
