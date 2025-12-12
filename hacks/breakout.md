---
layout: opencs
title: Breakout
permalink: /breakout/
---

<style>
  canvas {
    background: #000000ff;
    display: block;
    margin: 0 auto;
    border: 1px solid #333;
  }
  h2 {
    margin-top: 5px !important;
  }
  p {
    margin-bottom: 5px !important;
  }
</style>

<canvas id="gameCanvas" width="600" height="400"></canvas>

<div style="max-width:600px;margin:15px auto;font-family:system-ui,Arial;">
  <h3>🎨 Live Color Controls</h3>

  <label><strong>Ball Color:</strong></label>
  <input type="color" id="ballColorPicker" value="#de0000" style="margin-left:10px;">
  <br><br>

  <label><strong>Brick Color:</strong></label>
  <input type="color" id="brickColorPicker" value="#de0000" style="margin-left:10px;">
</div>

<!-- NEW: Next Level buttons -->
<button id="nextLevelBtn" style="display:none;margin:10px auto 0;padding:10px 16px;font-family:system-ui,Arial;font-size:16px;font-weight:600;border:1px solid #222;background:#fff;cursor:pointer;border-radius:8px;display:block;max-width:600px;color:#111 !important;">
  Next Level ▶
</button>

<!-- NEW: Win message -->
<div id="winMessage" style="display:none; text-align:center; margin-top:20px; font-family:system-ui,Arial;">
  <h2> you win yay </h2>
</div>


<!-- Hack/info sections here (unchanged) -->

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");
const nextLevelBtn = document.getElementById("nextLevelBtn");

// --- Levels / pause ---
let level = 1;
const levelSpeedScale = 2;
let paused = false;

// Paddle
let paddleHeight = 10;
let basePaddleWidth = 75;
let paddleWidth = basePaddleWidth;
let paddleX = (canvas.width - paddleWidth) / 2;

let rightPressed = false;
let leftPressed = false;

// Ball
let ballRadius = 8;
let x = canvas.width / 2;
let y = canvas.height - 30;
let dx = 4;
let dy = -4;

let ballColor = "#de0000";
let brickColor = "#de0000";

// Score and Lives
let score = 0;
let lives = 999;

// Blocks
let brickRowCount = 4;
const brickColumnCount = 6;
const brickWidth = 75;
const brickHeight = 20;
const brickPadding = 10;
const brickOffsetTop = 30;
const brickOffsetLeft = 50;

let bricks = [];
const powerUpChance = 0.3;

function initBricks() {
  bricks = [];
  for (let c = 0; c < brickColumnCount; c++) {
    bricks[c] = [];
    for (let r = 0; r < brickRowCount; r++) {
      const hasPowerUp = Math.random() < powerUpChance;
      bricks[c][r] = { x: 0, y: 0, status: 1, powerUp: hasPowerUp };
    }
  }
}
initBricks();

// Powerups
let powerUps = [];
const powerUpSize = 20;
const powerUpFallSpeed = 1.5;

let activePowerUp = null;
let powerUpTimer = 0;
const powerUpDuration = 5000;

let slowMotion = false;

// Input handling
document.addEventListener("keydown", keyDownHandler);
document.addEventListener("keyup", keyUpHandler);
document.addEventListener("mousemove", mouseMoveHandler);

function keyDownHandler(e) {
  if (e.key === "Right" || e.key === "ArrowRight" || e.key === "d" || e.key === "D") {
    rightPressed = true;
  } else if (e.key === "Left" || e.key === "ArrowLeft" || e.key === "a" || e.key === "A") {
    leftPressed = true;
  }
  
  // --- Slow motion toggle ---
  if (e.key === "s" || e.key === "S") {
    slowMotion = !slowMotion;
    if (slowMotion) {
      dx /= 2;
      dy /= 2;
    } else {
      dx *= 2;
      dy *= 2;
    }
  }
}

function keyUpHandler(e) {
  if (e.key === "Right" || e.key === "ArrowRight" || e.key === "d" || e.key === "D") {
    rightPressed = false;
  } else if (e.key === "Left" || e.key === "ArrowLeft" || e.key === "a" || e.key === "A") {
    leftPressed = false;
  }
}

function mouseMoveHandler(e) {
  let relativeX = e.clientX - canvas.offsetLeft;
  if (relativeX > 0 && relativeX < canvas.width) {
    paddleX = relativeX - paddleWidth / 2;
  }
}

// Collision detection
function collisionDetection() {
  for (let c = 0; c < brickColumnCount; c++) {
    for (let r = 0; r < brickRowCount; r++) {
      let b = bricks[c][r];
      if (b.status === 1) {
        if (
          x > b.x &&
          x < b.x + brickWidth &&
          y > b.y &&
          y < b.y + brickHeight
        ) {
          dy = -dy;

          // --- Hack #2: Increase speed ---
          dx *= 1.005;
          dy *= 1.005;

          b.status = 0;
          score++;

          if (b.powerUp) {
            powerUps.push({ x: b.x + brickWidth / 2, y: b.y, active: true });
          }
        }
      }
    }
  }
}

// Remaining bricks
function remainingBricks() {
  let count = 0;
  for (let c = 0; c < brickColumnCount; c++) {
    for (let r = 0; r < brickRowCount; r++) {
      if (bricks[c][r].status === 1) count++;
    }
  }
  return count;
}

// Powerup mechanics
function drawPowerUps() {
  for (let i = 0; i < powerUps.length; i++) {
    let p = powerUps[i];
    if (p.active) {
      let gradient = ctx.createRadialGradient(p.x, p.y, 5, p.x, p.y, powerUpSize);
      gradient.addColorStop(0, "yellow");
      gradient.addColorStop(1, "red");

      ctx.beginPath();
      ctx.arc(p.x, p.y, powerUpSize / 2, 0, Math.PI * 2);
      ctx.fillStyle = gradient;
      ctx.fill();
      ctx.closePath();

      ctx.fillStyle = "black";
      ctx.font = "bold 14px Arial";
      ctx.textAlign = "center";
      ctx.textBaseline = "middle";
      ctx.fillText("P", p.x, p.y);

      p.y += powerUpFallSpeed;

      if (
        p.y + powerUpSize / 2 >= canvas.height - paddleHeight &&
        p.x > paddleX &&
        p.x < paddleX + paddleWidth
      ) {
        p.active = false;
        paddleWidth = basePaddleWidth + 40;
        activePowerUp = "Wide Paddle";
        powerUpTimer = Date.now();
      }

      if (p.y > canvas.height) p.active = false;
    }
  }
}

function drawPowerUpTimer() {
  if (activePowerUp) {
    let elapsed = Date.now() - powerUpTimer;
    let remaining = Math.max(0, powerUpDuration - elapsed);
    let barHeight = 100;
    let barWidth = 10;
    let fillHeight = (remaining / powerUpDuration) * barHeight;

    ctx.fillStyle = "gray";
    ctx.fillRect(canvas.width - 20, 20, barWidth, barHeight);

    ctx.fillStyle = "lime";
    ctx.fillRect(canvas.width - 20, 20 + (barHeight - fillHeight), barWidth, fillHeight);

    ctx.strokeStyle = "black";
    ctx.strokeRect(canvas.width - 20, 20, barWidth, barHeight);

    if (remaining <= 0) {
      activePowerUp = null;
      paddleWidth = basePaddleWidth;
    }
  }
}

// Draw functions
function drawBall() {
  ctx.beginPath();
  ctx.arc(x, y, ballRadius, 0, Math.PI * 2);
  ctx.fillStyle = ballColor;
  ctx.fill();
  ctx.closePath();
}

function drawPaddle() {
  ctx.beginPath();
  ctx.rect(paddleX, canvas.height - paddleHeight, paddleWidth, paddleHeight);
  ctx.fillStyle = "#de0000ff";
  ctx.fill();
  ctx.closePath();
}

function drawBricks() {
  for (let c = 0; c < brickColumnCount; c++) {
    for (let r = 0; r < brickRowCount; r++) {
      if (bricks[c][r].status === 1) {
        let brickX = c * (brickWidth + brickPadding) + brickOffsetLeft;
        let brickY = r * (brickHeight + brickPadding) + brickOffsetTop;
        bricks[c][r].x = brickX;
        bricks[c][r].y = brickY;

        ctx.beginPath();
        ctx.rect(brickX, brickY, brickWidth, brickHeight);

        if (bricks[c][r].powerUp) {
          ctx.fillStyle = "#0095DD";
          ctx.shadowColor = "#6ecfffff";
          ctx.shadowBlur = 10;
        } else {
          ctx.fillStyle = brickColor;
          ctx.shadowBlur = 0;
        }

        ctx.fill();
        ctx.closePath();
      }
    }
  }
}

function resetBallAndPaddle() {
  const speed = Math.hypot(dx, dy);
  x = canvas.width / 2;
  y = canvas.height - 30;
  const angle = (Math.PI / 6) + Math.random() * (Math.PI / 3);
  const sign = Math.random() < 0.5 ? -1 : 1;
  dx = sign * speed * Math.cos(angle);
  dy = -Math.abs(speed * Math.sin(angle));
  paddleX = (canvas.width - paddleWidth) / 2;

  powerUps = [];
  activePowerUp = null;
  paddleWidth = basePaddleWidth;
}

function nextLevel() {
  const currentSpeed = Math.hypot(dx, dy) * levelSpeedScale;
  const theta = Math.atan2(dy, dx);
  dx = currentSpeed * Math.cos(theta);
  dy = currentSpeed * Math.sin(theta);

  level++;
  if (brickRowCount < 8) brickRowCount++;

  initBricks();
  resetBallAndPaddle();

  paused = false;
  nextLevelBtn.style.display = "none";
  draw();
}

nextLevelBtn.addEventListener("click", nextLevel);

function drawScore() {
  ctx.save();
  ctx.shadowBlur = 0;
  ctx.font = "16px Arial";
  ctx.fillStyle = "#0095DD";
  ctx.textAlign = "left";
  ctx.textBaseline = "top";
  ctx.fillText("Score: " + score, 10, 10);
  ctx.restore();
}

function drawLives() {
  ctx.save();
  ctx.shadowBlur = 0;
  ctx.font = "16px Arial";
  ctx.fillStyle = "#0095DD";
  ctx.textAlign = "right";
  ctx.textBaseline = "top";
  ctx.fillText("Lives: " + lives, canvas.width - 10, 10);
  ctx.restore();
}

function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  drawBricks();
  drawBall();
  drawPaddle();
  drawPowerUps();
  drawPowerUpTimer();
  drawScore();
  drawLives();
  collisionDetection();

  if (!paused && remainingBricks() === 0) {
    paused = true;
    // --- Show You Win message immediately ---
    document.getElementById("winMessage").style.display = "block";
    return;
  }

  if (x + dx > canvas.width - ballRadius || x + dx < ballRadius) dx = -dx;
  if (y + dy < ballRadius) dy = -dy;
  else if (y + dy > canvas.height - ballRadius) {
    if (x > paddleX && x < paddleX + paddleWidth) {
      dy = -dy;
      dx *= 1.025;
      dy *= 1.025;
    } else {
      lives--;
      if (!lives) {
        alert("GAME OVER");
        document.location.reload();
      } else {
        x = canvas.width / 2;
        y = canvas.height - 30;
        dx = 4 * Math.sign(dx);
        dy = -4;
        paddleX = (canvas.width - paddleWidth) / 2;
      }
    }
  }

  if (rightPressed && paddleX < canvas.width - paddleWidth) paddleX += 7;
  else if (leftPressed && paddleX > 0) paddleX -= 7;

  x += dx;
  y += dy;

  if (!paused) requestAnimationFrame(draw);
}

// Start the game
draw();

// Reset button logic
document.getElementById("resetBtn").addEventListener("click", () => {
  score = 0;
  lives = 999;
  paused = false;
  document.getElementById("winMessage").style.display = "none";
  initBricks();
  resetBallAndPaddle();
  draw();
});

// --- Color Picker Event Listeners ---
const ballPicker = document.getElementById("ballColorPicker");
const brickPicker = document.getElementById("brickColorPicker");

ballPicker.addEventListener("input", (e) => ballColor = e.target.value);
brickPicker.addEventListener("input", (e) => brickColor = e.target.value);

