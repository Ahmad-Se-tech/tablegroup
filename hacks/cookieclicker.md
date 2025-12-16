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
}

.game {
    max-width: 600px;
    margin: 40px auto;
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
}

@keyframes floatUp {
    0% { transform: translateY(0); opacity:1; }
    100% { transform: translateY(-50px); opacity:0; }
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
#reset-high:hover {
    background: #c9302c;
}

/* Achievements smaller font */
.panel ul#achievements {
    font-size: 12px;
    margin: 0;
    padding-left: 20px;
    max-height: 150px;
    overflow-y: auto;
}

/* Falling cookies in background */
.fall-cookie {
    position: absolute;
    font-size: 30px;
    pointer-events: none;
    opacity: 0.8;
    animation-name: fall;
    animation-timing-function: linear;
}

@keyframes fall {
    0% { transform: translateY(-50px); opacity:0.8; }
    100% { transform: translateY(600px); opacity:0; }
}
</style>

<div class="game">
    <h2>🍪 Cookie Clicker</h2>
    <h3>
      Cookies: <span id="cookies">0</span> | 
      High Score: <span id="highScore">0</span>
      <button id="reset-high">Reset High Score</button>
    </h3>
    <p>Per Click: <span id="perClick">1</span> | Per Second: <span id="perSecond">0</span></p>

    <div id="cookie-container">
        <div id="cookie">🍪</div>
        <canvas id="cookie-crack"></canvas>
    </div>

    <!-- Side by side Shop and Achievements -->
    <div class="panel-container" style="display:flex; gap:15px; justify-content:center; flex-wrap: wrap;">
        <div class="panel" style="flex:1; min-width:200px;">
            <h3>🛒 Shop</h3>
            <div id="shop"></div>
        </div>

        <div class="panel" style="flex:1; min-width:200px;">
            <h3>🏆 Achievements</h3>
            <ul id="achievements"></ul>
        </div>
    </div>
</div>

<script>
let highScore = Number(localStorage.getItem("cookie_clicker_high")) || 0;
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

function resizeCanvas(){
    const rect = cookieEl.getBoundingClientRect();
    crackCanvas.width = rect.width;
    crackCanvas.height = rect.height;
}
window.addEventListener("resize", resizeCanvas);
resizeCanvas();

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
                clearCracks(); // reset cracks
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

cookieEl.onclick = () => {
    player.cookies += player.perClick;
    drawCracks();
    showFloating("+"+player.perClick);

    cookieEl.style.transform = "scale(0.9)";
    setTimeout(()=>cookieEl.style.transform="scale(1)",100);

    if(player.cookies > highScore){
        highScore = Math.floor(player.cookies);
        localStorage.setItem("cookie_clicker_high", highScore);
    }

    render();
};

function drawCracks(){
    ctx.clearRect(0,0,crackCanvas.width,crackCanvas.height);
    const radius = crackCanvas.width/2;
    const centerX = radius;
    const centerY = radius;
    const numCracks = 3 + Math.floor(Math.random()*3);

    for(let i=0;i<numCracks;i++){
        let angle = Math.random()*2*Math.PI;
        let r = Math.random()*radius*0.8;
        let x0 = centerX + r*Math.cos(angle);
        let y0 = centerY + r*Math.sin(angle);

        const steps = 3 + Math.floor(Math.random()*3);
        ctx.strokeStyle = "rgba(0,0,0,0.5)";
        ctx.lineWidth = 1.5;
        ctx.beginPath();
        ctx.moveTo(x0,y0);

        let x = x0, y = y0;
        for(let s=0; s<steps; s++){
            let stepAngle = Math.random()*2*Math.PI;
            let stepR = Math.random()*15;
            x += stepR*Math.cos(stepAngle);
            y += stepR*Math.sin(stepAngle);

            let dx = x-centerX;
            let dy = y-centerY;
            let dist = Math.sqrt(dx*dx+dy*dy);
            if(dist>radius) {
                x = centerX + dx/dist*radius;
                y = centerY + dy/dist*radius;
            }

            ctx.lineTo(x,y);
        }
        ctx.stroke();
    }
}

function clearCracks(){
    ctx.clearRect(0,0,crackCanvas.width,crackCanvas.height);
}

function showFloating(text){
    const span = document.createElement("span");
    span.textContent = text;
    span.className = "floating";
    const rect = cookieEl.getBoundingClientRect();
    span.style.left = (rect.left + rect.width/2 + (Math.random()*40-20)) + "px";
    span.style.top = (rect.top + window.scrollY - 20 + (Math.random()*10-5)) + "px";
    document.body.appendChild(span);
    setTimeout(()=>span.remove(),1000);
}

// Auto cookies
setInterval(()=>{
    if(player.perSecond>0){
        player.cookies += player.perSecond;
        if(player.cookies > highScore){
            highScore = Math.floor(player.cookies);
            localStorage.setItem("cookie_clicker_high", highScore);
        }
        render();
    }
},1000);

// Reset high score
resetHighBtn.onclick = ()=>{
    localStorage.setItem("cookie_clicker_high", 0);
    highScore = 0;
    highScoreEl.textContent = highScore;
    clearCracks();
};

// Falling cookies background
function createFallingCookie(){
    const cookie = document.createElement("div");
    cookie.className = "fall-cookie";
    cookie.textContent = "🍪";
    cookie.style.left = Math.random()*window.innerWidth + "px";
    cookie.style.animationDuration = (3 + Math.random()*3) + "s";
    cookie.style.fontSize = (15+Math.random()*25) + "px";
    document.body.appendChild(cookie);
    setTimeout(()=>cookie.remove(),6000);
}
setInterval(createFallingCookie, 500);

render();
</script>
