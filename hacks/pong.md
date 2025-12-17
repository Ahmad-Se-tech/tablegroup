---
layout: post
title: 🏓 Complete Pong Game Code Implementation
description: Complete HTML, CSS, and JavaScript code for building a fully functional 2-player Pong game
categories: ['Game Development', 'JavaScript', 'Canvas API', 'Code Implementation']
permalink: /custompong
menu: nav/tools_setup.html
toc: True
comments: True
---

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>🏓 Complete Pong Game Code Implementation</title>
  <meta name="description" content="Complete HTML, CSS, and JavaScript code for building a fully functional 2-player Pong game with difficulty settings">
  <style>
    body {
      margin: 0;
      padding: 20px;
      display: flex;
      flex-direction: column;
      align-items: center;
      background: #222;
      font-family: Arial, sans-serif;
      color: white;
    }
    
    .game-canvas-container {
      text-align: center;
      margin-top: 20px;
    }
    
    #pongCanvas {
      border: 10px solid #fff;
      background: #1e00ffff;
      display: block;
      margin: 0 auto;
    }
    
    .controls {
      margin: 20px 0;
      display: flex;
      gap: 10px;
      align-items: center;
    }
    
    .difficulty-btn {
      padding: 10px 20px;
      font-size: 16px;
      border: 2px solid #fff;
      border-radius: 6px;
      background: #444;
      color: white;
      cursor: pointer;
      transition: all 0.3s;
    }
    
    .difficulty-btn:hover {
      background: #666;
      transform: scale(1.05);
    }
    
    .difficulty-btn.active {
      background: #4caf50;
      border-color: #4caf50;
    }
    
    #restartBtn {
      display: none;
      padding: 10px 20px;
      font-size: 18px;
      border: none;
      border-radius: 6px;
      background: #4caf50;
      color: white;
      cursor: pointer;
      margin-top: 15px;
    }
    
    #restartBtn:hover {
      background: #45a049;
    }
    
    .difficulty-label {
      font-size: 18px;
      font-weight: bold;
      margin-right: 10px;
    }
  </style>
</head>
<body>
  <!-- Header Section -->
  <div style="text-align: center; margin-bottom: 10px;">
    <h1 style="margin-bottom: 5px;">🏓 Complete Pong Game Code Implementation</h1>
    <p style="color: #aaa; margin: 5px 0;">Complete HTML, CSS, and JavaScript code for building a fully functional 2-player Pong game</p>
    <p style="color: #888; font-size: 14px; margin: 5px 0;">
      <strong>Categories:</strong> Game Development | JavaScript | Canvas API | Code Implementation
    </p>
  </div>

  <h2 style="text-align: center; margin: 20px 0;">🎮 Pong Game Demo</h2>
  
  <div class="controls">
    <span class="difficulty-label">Difficulty:</span>
    <button class="difficulty-btn active" data-difficulty="easy">Easy</button>
    <button class="difficulty-btn" data-difficulty="medium">Medium</button>
    <button class="difficulty-btn" data-difficulty="hard">Hard</button>
  </div>
  
  <div class="game-canvas-container">
    <canvas id="pongCanvas" width="800" height="500"></canvas>
    <br>
    <button id="restartBtn">Restart Game</button>
  </div>

  <script>
// =============================
// Pong with Difficulty Settings
// =============================

// -----------------------------
// Config: Tweak these values
// -----------------------------
const Config = {
  canvas: { width: 800, height: 500 },
  paddle: { width: 10, height: 100, speed: 10},
  ball: { radius: 10, baseSpeedX: 5, maxRandomY: 2, spinFactor: 0.3 },
  rules: { winningScore: 10 },
  keys: {
    p1Up: "w", p1Down: "s",
    p2Up: "i", p2Down: "k"
  },
  visuals: { bg: "#1e00ffff", fg: "#fff", text: "#fff", gameOver: "red", win: "yellow" },
  // NEW: Difficulty settings
  difficulty: {
    easy: { speedMultiplier: 0.6, deadZone: 15 },
    medium: { speedMultiplier: 1.0, deadZone: 8 },
    hard: { speedMultiplier: 1.5, deadZone: 3 }
  },
  // NEW: Power-up settings
  powerup: {
    size: 20,
    spawnChance: 0.4, // 40% chance to spawn after each point
    duration: 10000, // 10 seconds in milliseconds
    paddleSizeMultiplier: 1.25,
    color: "#FFD700" // Gold color
  }
};

// Basic vector helper for clarity
class Vector2 {
  constructor(x = 0, y = 0) { this.x = x; this.y = y; }
}

class Paddle {
  constructor(x, y, width, height, speed, boundsHeight) {
    this.position = new Vector2(x, y);
    this.width = width;
    this.height = height;
    this.speed = speed;
    this.baseSpeed = speed; // Store original speed
    this.boundsHeight = boundsHeight;
    // NEW: Store original dimensions for power-up
    this.baseWidth = width;
    this.baseHeight = height;
  }
  move(dy) {
    this.position.y = Math.min(
      this.boundsHeight - this.height,
      Math.max(0, this.position.y + dy)
    );
  }
  // NEW: Method to apply power-up
  applyPowerup(multiplier) {
    this.width = this.baseWidth * multiplier;
    this.height = this.baseHeight * multiplier;
  }
  // NEW: Method to reset to normal size
  resetSize() {
    this.width = this.baseWidth;
    this.height = this.baseHeight;
  }
  rect() { return { x: this.position.x, y: this.position.y, w: this.width, h: this.height }; }
}

class Ball {
  constructor(radius, boundsWidth, boundsHeight) {
    this.radius = radius;
    this.boundsWidth = boundsWidth;
    this.boundsHeight = boundsHeight;
    this.position = new Vector2();
    this.velocity = new Vector2();
    this.reset(true);
  }
  reset(randomDirection = false) {
    this.position.x = this.boundsWidth / 2;
    this.position.y = this.boundsHeight / 2;
    const dir = randomDirection && Math.random() > 0.5 ? 1 : -1;
    this.velocity.x = dir * Config.ball.baseSpeedX;
    this.velocity.y = (Math.random() * (2 * Config.ball.maxRandomY)) - Config.ball.maxRandomY;
  }
  update() {
    this.position.x += this.velocity.x;
    this.position.y += this.velocity.y;
    // Bounce off top/bottom walls
    if (this.position.y + this.radius > this.boundsHeight || this.position.y - this.radius < 0) {
      this.velocity.y *= -1;
    }
  }
}

// NEW: PowerUp class
class PowerUp {
  constructor(boundsWidth, boundsHeight) {
    this.size = Config.powerup.size;
    this.boundsWidth = boundsWidth;
    this.boundsHeight = boundsHeight;
    this.position = new Vector2();
    this.active = false;
  }
  
  spawn() {
    // Random position, avoiding edges
    const margin = 100;
    this.position.x = margin + Math.random() * (this.boundsWidth - 2 * margin);
    this.position.y = margin + Math.random() * (this.boundsHeight - 2 * margin);
    this.active = true;
  }
  
  checkCollision(ball) {
    if (!this.active) return false;
    
    const dx = ball.position.x - this.position.x;
    const dy = ball.position.y - this.position.y;
    const distance = Math.sqrt(dx * dx + dy * dy);
    
    return distance < (ball.radius + this.size / 2);
  }
  
  deactivate() {
    this.active = false;
  }
  
  rect() {
    return {
      x: this.position.x - this.size / 2,
      y: this.position.y - this.size / 2,
      w: this.size,
      h: this.size
    };
  }
}

class Input {
  constructor() {
    this.keys = {};
    document.addEventListener("keydown", e => this.keys[e.key] = true);
    document.addEventListener("keyup", e => this.keys[e.key] = false);
  }
  isDown(k) { return !!this.keys[k]; }
}

class Renderer {
  constructor(ctx) { this.ctx = ctx; }
  clear(w, h) {
    this.ctx.fillStyle = Config.visuals.bg;
    this.ctx.fillRect(0, 0, w, h);
  }
  rect(r, color = Config.visuals.fg) {
    this.ctx.fillStyle = color;
    this.ctx.fillRect(r.x, r.y, r.w, r.h);
  }
  circle(ball, color = Config.visuals.fg) {
    this.ctx.fillStyle = color;
    this.ctx.beginPath();
    this.ctx.arc(ball.position.x, ball.position.y, ball.radius, 0, Math.PI * 2);
    this.ctx.closePath();
    this.ctx.fill();
  }
  text(t, x, y, color = Config.visuals.text) {
    this.ctx.fillStyle = color;
    this.ctx.font = "30px Arial";
    this.ctx.fillText(t, x, y);
  }
}

class Game {
  constructor(canvasEl, restartBtn) {
    // Canvas
    this.canvas = canvasEl;
    this.ctx = canvasEl.getContext('2d');
    this.renderer = new Renderer(this.ctx);

    // Systems
    this.input = new Input();

    // Entities
    const { width, height, speed } = Config.paddle;
    this.paddleLeft = new Paddle(0, (Config.canvas.height - height) / 2, width, height, speed, Config.canvas.height);
    this.paddleRight = new Paddle(Config.canvas.width - width, (Config.canvas.height - height) / 2, width, height, speed, Config.canvas.height);
    this.ball = new Ball(Config.ball.radius, Config.canvas.width, Config.canvas.height);

    // NEW: Power-up system
    this.powerup = new PowerUp(Config.canvas.width, Config.canvas.height);
    this.powerupTimer = null;

    // Rules/state
    this.scores = { p1: 0, p2: 0 };
    this.gameOver = false;
    this.restartBtn = restartBtn;
    this.restartBtn.addEventListener("click", () => this.restart());

    // NEW: Difficulty setting
    this.currentDifficulty = 'easy';
    this.updateAIDifficulty();

    this.loop = this.loop.bind(this);
  }

  // NEW: Method to update AI difficulty
  setDifficulty(difficulty) {
    this.currentDifficulty = difficulty;
    this.updateAIDifficulty();
  }

  updateAIDifficulty() {
    const settings = Config.difficulty[this.currentDifficulty];
    this.aiSpeedMultiplier = settings.speedMultiplier;
    this.aiDeadZone = settings.deadZone;
  }

  handleInput() {
    if (this.gameOver) return;

    // Player 1 Controls (Human)
    if (this.input.isDown(Config.keys.p1Up)) this.paddleLeft.move(-this.paddleLeft.speed);
    if (this.input.isDown(Config.keys.p1Down)) this.paddleLeft.move(this.paddleLeft.speed);

    // AI Controls for Player 2 with difficulty adjustment
    let paddleCenter = this.paddleRight.position.y + (this.paddleRight.height / 2);
    let error = this.ball.position.y - paddleCenter;

    // Apply difficulty-based speed
    const aiSpeed = this.paddleRight.baseSpeed * this.aiSpeedMultiplier;

    if (error > this.aiDeadZone) {
      this.paddleRight.move(aiSpeed);
    } else if (error < -this.aiDeadZone) {
      this.paddleRight.move(-aiSpeed);
    }
  }

  update() {
    if (this.gameOver) return;
    this.ball.update();

    // NEW: Check power-up collision
    if (this.powerup.checkCollision(this.ball)) {
      this.activatePowerup();
    }

    // Paddle collisions with ball
    const hitLeft = this.ball.position.x - this.ball.radius < this.paddleLeft.width &&
      this.ball.position.y > this.paddleLeft.position.y &&
      this.ball.position.y < this.paddleLeft.position.y + this.paddleLeft.height;

    if (hitLeft) {
      this.ball.velocity.x *= -1;
      const delta = this.ball.position.y - (this.paddleLeft.position.y + this.paddleLeft.height / 2);
      this.ball.velocity.y = delta * Config.ball.spinFactor;
    }

    const hitRight = this.ball.position.x + this.ball.radius > (Config.canvas.width - this.paddleRight.width) &&
      this.ball.position.y > this.paddleRight.position.y &&
      this.ball.position.y < this.paddleRight.position.y + this.paddleRight.height;

    if (hitRight) {
      this.ball.velocity.x *= -1;
      const delta = this.ball.position.y - (this.paddleRight.position.y + this.paddleRight.height / 2);
      this.ball.velocity.y = delta * Config.ball.spinFactor;
    }

    // Scoring
    if (this.ball.position.x - this.ball.radius < 0) {
      this.scores.p2++;
      this.checkWin() || this.handleScore();
    } else if (this.ball.position.x + this.ball.radius > Config.canvas.width) {
      this.scores.p1++;
      this.checkWin() || this.handleScore();
    }
  }

  // NEW: Handle scoring and potential power-up spawn
  handleScore() {
    this.ball.reset();
    // Random chance to spawn power-up
    if (Math.random() < Config.powerup.spawnChance) {
      this.powerup.spawn();
    }
  }

  // NEW: Activate power-up effect
  activatePowerup() {
    this.powerup.deactivate();
    
    // Clear existing timer if any
    if (this.powerupTimer) {
      clearTimeout(this.powerupTimer);
    }
    
    // Apply size increase to both paddles
    this.paddleLeft.applyPowerup(Config.powerup.paddleSizeMultiplier);
    this.paddleRight.applyPowerup(Config.powerup.paddleSizeMultiplier);
    
    // Set timer to revert after duration
    this.powerupTimer = setTimeout(() => {
      this.paddleLeft.resetSize();
      this.paddleRight.resetSize();
      this.powerupTimer = null;
    }, Config.powerup.duration);
  }

  checkWin() {
    if (this.scores.p1 >= Config.rules.winningScore || this.scores.p2 >= Config.rules.winningScore) {
      this.gameOver = true;
      this.restartBtn.style.display = "inline-block";
      // NEW: Deactivate power-up on game over
      this.powerup.deactivate();
      if (this.powerupTimer) {
        clearTimeout(this.powerupTimer);
        this.powerupTimer = null;
      }
      return true;
    }
    return false;
  }

  draw() {
    // Clear canvas
    this.renderer.clear(Config.canvas.width, Config.canvas.height);
    
    // Draw center stripe
    this.ctx.fillStyle = 'white';
    const stripeWidth = 5;
    const stripeX = (this.canvas.width / 2) - (stripeWidth / 2);
    this.ctx.fillRect(stripeX, 0, stripeWidth, this.canvas.height);

    // Draw game elements
    this.renderer.rect(this.paddleLeft.rect());
    this.renderer.rect(this.paddleRight.rect());
    this.renderer.circle(this.ball);
    
    // NEW: Draw power-up if active
    if (this.powerup.active) {
      this.renderer.rect(this.powerup.rect(), Config.powerup.color);
    }
    
    // Draw scores
    this.renderer.text(this.scores.p1, Config.canvas.width / 4, 50);
    this.renderer.text(this.scores.p2, 3 * Config.canvas.width / 4, 50);
    
    if (this.gameOver) {
      this.renderer.text("Game Over", Config.canvas.width / 2 - 80, Config.canvas.height / 2 - 20, Config.visuals.gameOver);
      const msg = this.scores.p1 >= Config.rules.winningScore ? "Player 1 Wins!" : "Player 2 Wins!";
      this.renderer.text(msg, Config.canvas.width / 2 - 120, Config.canvas.height / 2 + 20, Config.visuals.win);
    }
  }

  restart() {
    this.scores.p1 = 0;
    this.scores.p2 = 0;
    this.paddleLeft.position.y = (Config.canvas.height - this.paddleLeft.height) / 2;
    this.paddleRight.position.y = (Config.canvas.height - this.paddleRight.height) / 2;
    // NEW: Reset paddle sizes
    this.paddleLeft.resetSize();
    this.paddleRight.resetSize();
    // NEW: Clear power-up
    this.powerup.deactivate();
    if (this.powerupTimer) {
      clearTimeout(this.powerupTimer);
      this.powerupTimer = null;
    }
    this.ball.reset(true);
    this.gameOver = false;
    this.restartBtn.style.display = "none";
  }

  loop() {
    this.handleInput();
    this.update();
    this.draw();
    requestAnimationFrame(this.loop);
  }
}

// -------------------------------
// Bootstrapping
// -------------------------------
const canvas = document.getElementById('pongCanvas');
const restartBtn = document.getElementById('restartBtn');

canvas.width = Config.canvas.width;
canvas.height = Config.canvas.height;

const game = new Game(canvas, restartBtn);

// NEW: Difficulty button handlers
const difficultyButtons = document.querySelectorAll('.difficulty-btn');
difficultyButtons.forEach(btn => {
  btn.addEventListener('click', () => {
    // Remove active class from all buttons
    difficultyButtons.forEach(b => b.classList.remove('active'));
    // Add active class to clicked button
    btn.classList.add('active');
    // Update game difficulty
    const difficulty = btn.dataset.difficulty;
    game.setDifficulty(difficulty);
  });
});

game.loop();
  </script>

  <!-- Footer Section with Student Challenges -->
  <div style="max-width: 800px; margin: 30px auto; padding: 20px; background: #333; border-radius: 8px; color: #ddd;">
    <h3 style="color: #4caf50; margin-top: 0;">🎯 Student Challenges (TODO Checklist)</h3>
    <ol style="line-height: 1.8;">
      <li><strong>Make it YOUR game:</strong> Change colors in Config.visuals, dimensions/speeds in Config.</li>
      <li><strong>Center line + score SFX:</strong> Draw a dashed midline; play an audio on score.</li>
      <li><strong>Power-ups:</strong> Occasionally spawn a small rectangle; when ball hits it, apply effect (bigger paddle? faster ball?).</li>
      <li><strong>Pause/Resume:</strong> Map a key to toggle pause state in Game and skip updates when paused.</li>
      <li><strong>Win screen polish:</strong> Show a replay countdown, then auto-restart.</li>
      <li><strong>Advanced AI:</strong> Make the AI predict where the ball will land instead of just tracking it.</li>
    </ol>
    
    <h3 style="color: #4caf50; margin-top: 25px;">📝 Notes</h3>
    <p>This refactored version uses Object-Oriented Programming to make the game easy to modify and extend. Teachers: Encourage students to experiment with the Config values and complete the challenges above!</p>
    
    <p style="margin-top: 15px;"><strong>Controls:</strong></p>
    <ul style="line-height: 1.8;">
      <li>Player 1: W (up) / S (down)</li>
      <li>Player 2: AI controlled (adjustable difficulty)</li>
    </ul>
  </div>
</body>
</html>