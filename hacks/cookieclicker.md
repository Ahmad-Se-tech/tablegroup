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
}

.game {
    max-width: 600px;
    margin: 40px auto;
    padding: 20px;
}

#cookie {
    font-size: 90px;
    cursor: pointer;
    user-select: none;
    transition: transform 0.1s;
    display: block;
    margin: 20px auto;
}

#cookie:active {
    transform: scale(0.9);
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
</style>

<div class="game">
    <h2>🍪 Cookie Clicker</h2>
    <h3>Cookies: <span id="cookies">0</span> | High Score: <span id="highScore">0</span></h3>
    <p>Per Click: <span id="perClick">1</span> | Per Second: <span id="perSecond">0</span></p>

    <div id="cookie">🍪</div>

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
/* -------------------- GAME STATE -------------------- */
let player = JSON.parse(localStorage.getItem("cookie_clicker")) || {cookies:0, perClick:1, perSecond:0};
let highScore = Number(localStorage.getItem("cookie_clicker_high")) || 0;

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
const shopEl = document.getElementById("shop");
const achEl = document.getElementById("achievements");
const cookiesEl = document.getElementById("cookies");
const highScoreEl = document.getElementById("highScore");
const perClickEl = document.getElementById("perClick");
const perSecondEl = document.getElementById("perSecond");

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
                saveGame();
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

    // Random golden cookie: 20% chance for 5x cookies
    if(Math.random() < 0.2){
        gain *= 5;
        showFloating("+💛"+gain);
    } else {
        showFloating("+"+gain);
    }

    player.cookies += gain;
    cookieEl.style.transform = "scale(0.9)";
    setTimeout(()=>cookieEl.style.transform="scale(1)",100);

    if(player.cookies > highScore){
        highScore = Math.floor(player.cookies);
        localStorage.setItem("cookie_clicker_high", highScore);
    }

    saveGame();
    render();
};

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
        saveGame();
        render();
    }
},1000);

/* -------------------- SAVE -------------------- */
function saveGame(){
    localStorage.setItem("cookie_clicker", JSON.stringify(player));
}

/* -------------------- INITIALIZE -------------------- */
render();
</script>
