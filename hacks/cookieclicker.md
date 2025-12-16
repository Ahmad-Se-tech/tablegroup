---
title: Cookie Clicker
comments: true
hide: true
layout: opencs
description: play cookie clicker when you are bored or for funnn!!
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
#cookie:active { transform: scale(0.9); }

.panel {
    background: rgba(255,255,255,0.8);
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
}
button:disabled {
    background: #aaa;
    cursor: not-allowed;
}
</style>

<div class="game">
    <h2>🍪 Cookie Clicker</h2>
    <h3>Cookies: <span id="cookies">0</span></h3>
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
/* ---------- SIMPLE GAME STATE ---------- */
let player = JSON.parse(localStorage.getItem("cookie_clicker")) || {cookies:0, perClick:1, perSecond:0};
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

/* ---------- ELEMENTS ---------- */
const cookieEl = document.getElementById("cookie");
const shopEl = document.getElementById("shop");
const achEl = document.getElementById("achievements");
const cookiesEl = document.getElementById("cookies");
const perClickEl = document.getElementById("perClick");
const perSecondEl = document.getElementById("perSecond");

/* ---------- RENDER ---------- */
function render() {
    cookiesEl.textContent = Math.floor(player.cookies);
    perClickEl.textContent = player.perClick;
    perSecondEl.textContent = player.perSecond;

    // Shop buttons
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

/* ---------- COOKIE CLICK ---------- */
cookieEl.onclick = ()=>{
    player.cookies += player.perClick;
    cookieEl.style.transform = "scale(0.9)";
    setTimeout(()=>cookieEl.style.transform="scale(1)",100);
    saveGame();
    render();
};

/* ---------- AUTO-CLICKER ---------- */
setInterval(()=>{
    player.cookies += player.perSecond;
    saveGame();
    render();
},1000);

/* ---------- SAVE & LOAD ---------- */
function saveGame(){ localStorage.setItem("cookie_clicker", JSON.stringify(player)); }

/* ---------- INITIALIZE ---------- */
render();
</script>
