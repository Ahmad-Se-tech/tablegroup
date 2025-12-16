---
title: Cookie Clicker
comments: true
hide: true
layout: opencs
description: play cookie clicker when you are bored or for fun!!
permalink: /cookie-clicker/
---

<style>
body {
    background: linear-gradient(180deg, #f3e2c7, #c9a66b);
    font-family: system-ui, sans-serif;
    text-align: center;
    overflow: hidden;
}

.game {
    max-width: 600px;
    margin: 40px auto;
    padding: 20px;
    position: relative;
    z-index: 2;
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
}

#cookie:active {
    transform: scale(0.9);
}

#cookie-crack {
    position: absolute;
    top: 0;
    left: 0;
    pointer-events: none;
}

.panel {
    background: rgba(255, 255, 255, 0.85);
    padding: 15px;
    border-radius: 12px;
    margin-top: 15px;
    position: relative;
    z-index: 2;
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

button:hover:not(:disabled) {
    background: #a77244;
    transform: scale(1.05);
}

button:disabled {
    background: #aaa;
    cursor: not-allowed;
}

.floating {
    position: absolute;
    font-weight: bold;
    color: gold;
    pointer-events: none;
    animation: floatUp 1s forwards;
    z-index: 3;
}

@keyframes floatUp {
    0% { transform: translateY(0); opacity:1; }
    100% { transform: translateY(-50px); opacity:0; }
}

.cookie-bg {
    position: absolute;
    font-size: 30px;
    pointer-events: none;
    animation: moveDown linear infinite;
    opacity: 0.6;
    z-index: 1;
}

@keyframes moveDown {
    0% { transform: translateY(-50px); }
    100% { transform: translateY(100vh); }
}
</style>

<div class="game">
    <h2>🍪 Cookie Clicker</h2>
    <h3>Cookies: <span id="cookies">0</span> | High Score: <span id="highScore">0</span></h3>
    <p>Per Click: <span id="perClick">1</span> | Per Second: <span id="perSecond">0</span></p>

    <div id="cookie-container">
        <div id="cookie">🍪</div>
        <canvas id="cookie-crack" width="90" height="90"></canvas>
    </div>

    <div class="panel">
        <h3>🛒 Shop</h3>
        <div id="shop"></div>
        <button id="reset-high">Reset High Score</button>
    </div>

    <div class="panel">
        <h3>🏆 Achievements</h3>
        <ul id="achievements"></ul>
    </div>
</div>

<script>
/* -------------------- GAME STATE -------------------- */
let highScore = Number(localStorage.getItem("cookie_clicker_high")) || 0;

// Reset session cookies and upgrades every load
let player = {cookies:0, perClick:1, perSecond:0};
let upgrades = [
    {name:"🖱 Cursor", cost:10, effect:1, type:"click", owned:0},
    {name:"👵 Grandma", cost:25, effect:1, type:"auto", owned:0},
    {name:"🏭 Factory", cost:100, effect:5, type:"auto", owned:0}
];
let achievements = [
    {name:"First Cookie", condition: p => p.cookies >= 1, unlocked:false},
    {name:"100 Cookies", condition: p => p.cookies >= 100, unlocked:false},
    {name:"Cookie Factory", condition: p => p.perSecond >= 10, unlocked:false}
];

/* -------------------- ELEMENTS -------------------- */
const cookieEl = document.getElementById("cookie");
const crackCanvas = document.getElementById("cookie-crack");
const ctx = crackCanvas.getContext("2d");
const shopEl = document.getElementById("shop");
const achEl = document.getElementById("achievements");
const cookiesEl = document.getElementById("cookies");
const highScoreEl = document.getElementById("highScore");
const perClickEl = document.getElementById("perClick");
const perSecondEl = document.getElementById("perSecond");
const resetHighBtn = document.getElementById("reset-high");

/* -------------------- RENDER -------------------- */
function render() {
    cookiesEl.textContent = Math.floor(player.cookies);
    perClickEl.textContent = player.perClick;
    perSecondEl.textContent = player.perSecond;
    highScoreEl.textContent = highScore;

    // Shop
    shopEl.innerHTML = "";
    upgrades.forEach(u=>{
        const btn = document.createElement("button");
        btn.textContent = `${u.name} (${u.cost}) [Owned: ${u.owned}]`;
        btn.disabled = player.cookies < u.cost;
        btn.onclick = ()=>{
            if(player.cookies >= u.cost){
                player.cookies -= u.cost;
                u.owned++;
                u.cost = Math.floor(u.cost * 1.5);
                if(u.type==="click") player.perClick += u.effect;
                else player.perSecond += u.effect;
                clearCracks(); // Reset cracks after purchase
                render();
            }
        };
        shopEl.appendChild(btn);
    });

    // Achievements
    achEl.innerHTML = "";
    achievements.forEach(a=>{
        if(!a.unlocked && a.condition(player)) a.unlocked = true;
        const li = document.createElement("li");
        li.textContent = a.unlocked ? `✅ ${a.name}` : `🔒 ${a.name}`;
        achEl.appendChild(li);
    });
}

/* -------------------- COOKIE CLICK -------------------- */
cookieEl.onclick = () => {
    let gain = player.perClick;

    if(Math.random() < 0.2){
        gain *= 5;
        showFloating("+💛"+gain);
    } else {
        showFloating("+"+gain);
    }

    player.cookies += gain;
    drawCracks();
    cookieEl.style.transform = "scale(0.9)";
    setTimeout(()=>cookieEl.style.transform="scale(1)",100);

    if(player.cookies > highScore){
        highScore = Math.floor(player.cookies);
        localStorage.setItem("cookie_clicker_high", highScore);
    }

    render();
};

/* -------------------- DRAW CRACKS -------------------- */
function drawCracks(){
    const numCracks = 3 + Math.floor(Math.random()*3);
    for(let i=0;i<numCracks;i++){
        const x1 = Math.random()*crackCanvas.width;
        const y1 = Math.random()*crackCanvas.height;
        const x2 = Math.random()*crackCanvas.width;
        const y2 = Math.random()*crackCanvas.height;
        ctx.strokeStyle = "rgba(0,0,0,0.5)";
        ctx.lineWidth = 1.5;
        ctx.beginPath();
        ctx.moveTo(x1,y1);
        ctx.lineTo(x2,y2);
        ctx.stroke();
    }
}

function clearCracks(){
    ctx.clearRect(0,0,crackCanvas.width,crackCanvas.height);
}

/* -------------------- FLOATING NUMBERS -------------------- */
function showFloating(text){
    const span = document.createElement("span");
    span.textContent = text;
    span.className = "floating";
    span.style.left = (cookieEl.offsetLeft + cookieEl.offsetWidth/2 + (Math.random()*40-20)) + "px";
    span.style.top = (cookieEl.offsetTop - 20 + (Math.random()*10-5)) + "px";
    document.body.appendChild(span);
    setTimeout(()=>span.remove(),1000);
}

/* -------------------- AUTO-CLICKER -------------------- */
setInterval(()=>{
    if(player.perSecond>0){
        player.cookies += player.perSecond;
        showFloating("+"+player.perSecond);
        if(player.cookies > highScore){
            highScore = Math.floor(player.cookies);
            localStorage.setItem("cookie_clicker_high", highScore);
        }
        render();
    }
},1000);

/* -------------------- BACKGROUND COOKIES -------------------- */
function spawnCookieBg(){
    const span = document.createElement("span");
    span.className = "cookie-bg";
    span.textContent = "🍪";
    span.style.left = Math.random()*window.innerWidth + "px";
    span.style.fontSize = (20 + Math.random()*20) + "px";
    span.style.animationDuration = (5 + Math.random()*5) + "s";
    document.body.appendChild(span);
    setTimeout(()=>span.remove(),10000);
}
setInterval(spawnCookieBg, 500);

/* -------------------- RESET HIGH SCORE -------------------- */
resetHighBtn.onclick = () => {
    localStorage.setItem("cookie_clicker_high", 0);
    highScore = 0;
    render();
};

/* -------------------- INITIALIZE -------------------- */
render();
</script>
