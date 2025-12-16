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
