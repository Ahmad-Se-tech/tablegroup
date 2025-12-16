---
title: Cookie Clicker
comments: true
hide: true
layout: opencs
description: Play Cookie Clicker when bored or for fun!
permalink: /cookie-clicker/
---

<style>
body {
    background: linear-gradient(180deg, #f3e2c7, #c9a66b);
    font-family: system-ui, sans-serif;
    text-align: center;
    overflow: hidden;
    margin: 0;
}

.game {
    max-width: 720px;
    margin: 28px auto;
    padding: 20px;
}

#cookie-container {
    position: relative;
    display: inline-block;
}

#cookie {
    font-size: 90px;
    cursor: pointer;
    user-select: none;
    transition: transform 0.1s;
    z-index: 1;
    position: relative;
}

#cookie:active {
    transform: scale(0.9);
}

#cookie-crack {
    position: absolute;
    top: 0;
    left: 0;
    pointer-events: none;
    z-index: 2;
}

.panel {
    background: rgba(255, 255, 255, 0.85);
    padding: 15px;
    border-radius: 12px;
    margin-top: 15px;
}

button {
    background: #8b5a2b;
    border: none;
    color: white;
    padding: 10px;
    margin: 5px;
    border-radius: 8px;
    cursor: pointer;
    transition: transform 0.1s, background 0.2s;
}

button:hover:not(:disabled) { background: #a77244; transform: scale(1.05); }
button:disabled { background: #aaa; cursor: not-allowed; }

.floating {
    position: absolute;
    font-weight: bold;
    color: gold;
    pointer-events: none;
    animation: floatUp 1s forwards;
    z-index: 10000;
}

@keyframes floatUp {
    0% { transform: translateY(0); opacity:1; }
    100% { transform: translateY(-50px); opacity:0; }
}

.crumb {
    position: absolute;
    pointer-events: none;
    color: saddlebrown;
    z-index: 9999;
    will-change: transform, opacity;
}

#reset-high {
    display: inline-block;
    background: #d9534f;
    color: white;
    padding: 6px 12px;
    font-size: 14px;
    border-radius: 8px;
    font-weight: bold;
    margin-left: 10px;
    cursor: pointer;
    vertical-align: middle;
    border: none;
    box-shadow: 1px 1px 4px rgba(0,0,0,0.3);
    transition: background 0.2s, transform 0.1s;
}
#reset-high:hover { background: #c9302c; }
</style>

<div class="game">
    <h2>🍪 Cookie Clicker</h2>
    <h3>
      Cookies: <span id="cookies">0</span> |
      High Score: <span id="highScore">0</span>
      <button id="reset-high">Reset High Score</button>
    </h3>
    <p>Per Click: <span id="perClick">1</span> | Per Second: <span id="perSecond">0</span></p>

    <div id="cookie-container" aria-hidden="false">
        <div id="cookie" role="button" aria-label="Cookie button">🍪</div>
        <canvas id="cookie-crack"></canvas>
    </div>

    <div class="panel">
        <h3>🛒 Shop</h3>
        <div id="shop"></div>
    </div>

    <div class="panel">
        <h3>🏆 Achievements</h3>
        <ul id="achievements"></ul>
    </div>
</div>

<script>
// -------------------- STATE --------------------
let highScore = Number(localStorage.getItem("cookie_clicker_high")) || 0;
let player = { cookies: 0, perClick: 1, perSecond: 0 };

let upgrades = [
  { name: "🖱 Cursor", cost: 10, effect: 1, type: "click", owned: 0 },
  { name: "👵 Grandma", cost: 25, effect: 1, type: "auto", owned: 0 },
  { name: "🏭 Factory", cost: 100, effect: 5, type: "auto", owned: 0 }
];

let achievements = [
  { name: "First Cookie", condition: p => p.cookies >= 1, unlocked: false },
  { name: "100 Cookies", condition: p => p.cookies >= 100, unlocked: false },
  { name: "Cookie Factory", condition: p => p.perSecond >= 10, unlocked: false }
];

// -------------------- ELEMENTS --------------------
const cookieEl = document.getElementById("cookie");
const container = document.getElementById("cookie-container");
const crackCanvas = document.getElementById("cookie-crack");
const ctx = crackCanvas.getContext("2d");
const shopEl = document.getElementById("shop");
const achEl = document.getElementById("achievements");
const cookiesEl = document.getElementById("cookies");
const highScoreEl = document.getElementById("highScore");
const perClickEl = document.getElementById("perClick");
const perSecondEl = document.getElementById("perSecond");
const resetHighBtn = document.getElementById("reset-high");

// crumb layers (DOM container to hold crumb spans)
const crumbLayer = document.createElement("div");
document.body.appendChild(crumbLayer);

// -------------------- CANVAS RESIZE --------------------
function resizeCanvas() {
  const rect = cookieEl.getBoundingClientRect();
  // make canvas match the displayed size and handle DPR
  const dpr = window.devicePixelRatio || 1;
  crackCanvas.style.width = rect.width + "px";
  crackCanvas.style.height = rect.height + "px";
  crackCanvas.width = Math.max(1, Math.floor(rect.width * dpr));
  crackCanvas.height = Math.max(1, Math.floor(rect.height * dpr));
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
}
window.addEventListener("resize", resizeCanvas);
resizeCanvas();

// -------------------- CRACKS (no initial cracks) --------------------
// We accumulate cracks on the canvas; clearCracks() empties them.
function drawCracks() {
  // draw several branched, curved cracks starting near center
  const radius = crackCanvas.width / (2 * (window.devicePixelRatio || 1));
  const centerX = radius;
  const centerY = radius;
  const numCracks = 3 + Math.floor(Math.random() * 3);

  for (let i = 0; i < numCracks; i++) {
    // start near center (random small offset)
    let angle = Math.random() * 2 * Math.PI;
    let r = Math.random() * radius * 0.28;
    let x0 = centerX + r * Math.cos(angle);
    let y0 = centerY + r * Math.sin(angle);

    const steps = 4 + Math.floor(Math.random() * 4);
    ctx.strokeStyle = "rgba(25,25,25,0.65)";
    ctx.lineWidth = 1.2;
    ctx.beginPath();
    ctx.moveTo(x0, y0);

    let x = x0, y = y0;
    for (let s = 0; s < steps; s++) {
      // random curved step
      const stepAngle = (Math.random() - 0.5) * Math.PI / 2;
      const stepR = 4 + Math.random() * 12;
      x += stepR * Math.cos(stepAngle);
      y += stepR * Math.sin(stepAngle);

      // project back into circle if outside
      const dx = x - centerX, dy = y - centerY;
      const dist = Math.sqrt(dx * dx + dy * dy);
      if (dist > radius) {
        const ratio = (radius * 0.95) / dist;
        x = centerX + dx * ratio;
        y = centerY + dy * ratio;
      }

      ctx.lineTo(x, y);

      // small branches
      if (Math.random() < 0.25) {
        const bx = x, by = y;
        ctx.moveTo(bx, by);
        const bSteps = 1 + Math.floor(Math.random() * 2);
        for (let b = 0; b < bSteps; b++) {
          let bax = bx + (Math.random() - 0.5) * 10;
          let bay = by + (Math.random() - 0.5) * 10;
          const bdx = bax - centerX, bdy = bay - centerY;
          const bdist = Math.sqrt(bdx * bdx + bdy * bdy);
          if (bdist > radius) {
            const br = (radius * 0.95) / bdist;
            bax = centerX + bdx * br;
            bay = centerY + bdy * br;
          }
          ctx.lineTo(bax, bay);
        }
        ctx.moveTo(x, y); // return path
      }
    }
    ctx.stroke();
  }
}

function clearCracks() {
  ctx.clearRect(0, 0, crackCanvas.width, crackCanvas.height);
}

// -------------------- CRUMBS (big -> tiny) --------------------
let crumbs = [];      // big crumbs
let tinyCrumbs = [];  // tiny broken pieces

function spawnCrumbs() {
  const rect = cookieEl.getBoundingClientRect();
  const centerX = rect.left + rect.width / 2;
  const centerY = rect.top + rect.height / 2;
  const n = 3 + Math.floor(Math.random() * 3);
  for (let i = 0; i < n; i++) {
    crumbs.push({
      x: centerX + (Math.random() * 40 - 20),
      y: centerY + (Math.random() * 14 - 7),
      r: 4 + Math.random() * 4,
      vx: (Math.random() * 2 - 1),
      vy: -1 - Math.random() * 2,
      alpha: 1
    });
  }
}

function renderCrumbs() {
  // clear layer
  crumbLayer.innerHTML = "";
  const toTiny = [];

  // big crumbs
  crumbs.forEach(c => {
    const el = document.createElement("span");
    el.className = "crumb";
    el.textContent = "•";
    el.style.left = (c.x) + "px";
    el.style.top = (c.y) + "px";
    el.style.fontSize = Math.max(2, c.r) + "px";
    el.style.opacity = Math.max(0, c.alpha);
    crumbLayer.appendChild(el);

    // physics
    c.x += c.vx;
    c.y += c.vy;
    c.vy += 0.06; // gravity
    c.alpha -= 0.015;

    // occasionally break into tiny crumbs
    if (Math.random() < 0.03) {
      toTiny.push({
        x: c.x,
        y: c.y,
        r: Math.max(1, c.r * 0.5),
        vx: (Math.random() * 2 - 1) * 1.6,
        vy: -1 - Math.random() * 1.8,
        alpha: c.alpha
      });
    }
  });

  // remove faded big crumbs and append the new tiny
  crumbs = crumbs.filter(c => c.alpha > 0);
  tinyCrumbs = tinyCrumbs.concat(toTiny);

  // tiny crumbs
  tinyCrumbs.forEach(tc => {
    const el = document.createElement("span");
    el.className = "crumb";
    el.textContent = "•";
    el.style.left = (tc.x) + "px";
    el.style.top = (tc.y) + "px";
    el.style.fontSize = Math.max(1, tc.r) + "px";
    el.style.opacity = Math.max(0, tc.alpha);
    crumbLayer.appendChild(el);

    tc.x += tc.vx;
    tc.y += tc.vy;
    tc.vy += 0.06;
    tc.alpha -= 0.02;
  });

  tinyCrumbs = tinyCrumbs.filter(tc => tc.alpha > 0);
}

// run crumb rendering loop
setInterval(renderCrumbs, 30);

// -------------------- FLOATING POINTS & COOKIE ICON --------------------
function showFloating(text) {
  const rect = cookieEl.getBoundingClientRect();
  const baseX = rect.left + rect.width / 2 + (Math.random() * 40 - 20);
  const baseY = rect.top + window.scrollY - 20 + (Math.random() * 10 - 5);

  // +points
  const span = document.createElement("span");
  span.className = "floating";
  span.textContent = text;
  span.style.left = baseX + "px";
  span.style.top = baseY + "px";
  document.body.appendChild(span);
  setTimeout(() => span.remove(), 1000);

  // small cookie icon
  const cookieSpan = document.createElement("span");
  cookieSpan.className = "floating";
  cookieSpan.textContent = "🍪";
  cookieSpan.style.fontSize = "22px";
  cookieSpan.style.left = (baseX + (Math.random() * 20 - 10)) + "px";
  cookieSpan.style.top = (baseY + 12) + "px";
  document.body.appendChild(cookieSpan);
  setTimeout(() => cookieSpan.remove(), 1200);
}

// -------------------- RENDER UI & SHOP & ACHIEVEMENTS --------------------
function render() {
  cookiesEl.textContent = Math.floor(player.cookies);
  perClickEl.textContent = player.perClick;
  perSecondEl.textContent = player.perSecond;
  highScoreEl.textContent = highScore;

  // shop
  shopEl.innerHTML = "";
  upgrades.forEach((u, idx) => {
    const btn = document.createElement("button");
    btn.textContent = `${u.name} (${u.cost}) [Owned: ${u.owned}]`;
    btn.disabled = player.cookies < u.cost;
    btn.onclick = () => {
      if (player.cookies >= u.cost) {
        player.cookies -= u.cost;
        u.owned++;
        u.cost = Math.floor(u.cost * 1.5);
        if (u.type === "click") player.perClick += u.effect;
        else player.perSecond += u.effect;

        // reset visual damage
        crumbs = [];
        tinyCrumbs = [];
        clearCracks();

        render();
      }
    };
    shopEl.appendChild(btn);
  });

  // achievements
  achEl.innerHTML = "";
  achievements.forEach(a => {
    if (!a.unlocked && a.condition(player)) a.unlocked = true;
    const li = document.createElement("li");
    li.textContent = a.unlocked ? `✅ ${a.name}` : `🔒 ${a.name}`;
    achEl.appendChild(li);
  });
}

// -------------------- CLICK HANDLER --------------------
cookieEl.addEventListener("click", () => {
  // add cookies
  player.cookies += player.perClick;

  // spawn cracks and crumbs
  drawCracks();
  spawnCrumbs();

  // floating feedback
  showFloating("+" + player.perClick);

  // quick click animation
  cookieEl.style.transform = "scale(0.92)";
  setTimeout(() => cookieEl.style.transform = "scale(1)", 100);

  // update high score
  if (player.cookies > highScore) {
    highScore = Math.floor(player.cookies);
    localStorage.setItem("cookie_clicker_high", highScore);
  }

  render();
});

// -------------------- AUTO CLICK LOOP --------------------
setInterval(() => {
  if (player.perSecond > 0) {
    player.cookies += player.perSecond;
    // show small floating occasionally (every second)
    showFloating("+" + player.perSecond);
    if (player.cookies > highScore) {
      highScore = Math.floor(player.cookies);
      localStorage.setItem("cookie_clicker_high", highScore);
    }
    render();
  }
}, 1000);

// -------------------- RESET HIGH SCORE --------------------
resetHighBtn.addEventListener("click", () => {
  localStorage.setItem("cookie_clicker_high", 0);
  highScore = 0;
  highScoreEl.textContent = highScore;
  // clear visual debris
  crumbs = [];
  tinyCrumbs = [];
  clearCracks();
  render();
});

// -------------------- INITIALIZE --------------------
render();
</script>
