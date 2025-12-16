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
    position: relative;
    height: 100vh;
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

/* Reset button inline with High Score */
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

.falling-cookie {
    position: absolute;
    font-size: 20px;
    pointer-events: none;
    opacity: 0.7;
    animation-name: fallCookie;
    animation-timing-function: linear;
    animation-iteration-count: infinite;
}

@keyframes fallCookie {
    0% { transform: translateY(-30px); }
    100% { transform: translateY(100vh); }
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

// -------------------- CANVAS RESIZE --------------------
function resizeCanvas(){
    const rect = cookieEl.getBoundingClientRect();
    crackCanvas.width = rect.width;
    crackCanvas.height = rect.height;
}
window.addEventListener("resize", resizeCanvas);
resizeCanvas();

// -------------------- BACKGROUND FALLING COOKIES --------------------
const fallingCookies = [];
for(let i=0;i<20;i++){
    const el = document.createElement("div");
    el.className="falling-cookie";
    el.textContent = "🍪";
    el.style.left = Math.random()*window.innerWidth+"px";
    el.style.fontSize = (10+Math.random()*20)+"px";
    el.style.animationDuration = (5+Math.random()*5)+"s";
    el.style.animationDelay = (Math.random()*5)+"s";
    document.body.appendChild(el);
    fallingCookies.push(el);
}

// -------------------- RENDER --------------------
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
                clearCracks(); // reset cracks after purchase
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

// -------------------- COOKIE CLICK --------------------
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

// -------------------- CRACKS --------------------
function drawCracks(){
    clearCracks();
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

        // create falling crumb
        const crumb = document.createElement("div");
        crumb.textContent = "🍪";
        crumb.style.position="absolute";
        crumb.style.left=(cookieEl.offsetLeft + x - 10 + Math.random()*20 -10)+"px";
        crumb.style.top=(cookieEl.offsetTop + y - 10)+"px";
        crumb.style.fontSize=(5+Math.random()*10)+"px";
        crumb.style.pointerEvents="none";
        crumb.style.opacity=0.7;
        crumb.style.transition="transform 2s linear, opacity 2s";
        document.body.appendChild(crumb);
        setTimeout(()=>{
            crumb.style.transform="translateY(50px)";
            crumb.style.opacity=0;
        },50);
        setTimeout(()=>crumb.remove(),2000);
    }
}

function clearCracks(){
    ctx.clearRect(0,0,crackCanvas.width,crackCanvas.height);
}

// -------------------- FLOATING --------------------
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

// -------------------- AUTO CLICK --------------------
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

// -------------------- RESET HIGH SCORE --------------------
resetHighBtn.onclick = ()=>{
    localStorage.setItem("cookie_clicker_high", 0);
    highScore = 0;
    highScoreEl.textContent = highScore;
    clearCracks();
};

// -------------------- INITIALIZE --------------------
render();
</script>
