---
layout: opencs
title: Tic Tac Toe
permalink: /tictactoe/
---

<style>
    body { font-family: sans-serif; text-align: center; background-color: #f0f0f0; }
    .container { margin: auto; width: 320px; padding: 20px; }
    #board { 
        display: grid; 
        grid-template-columns: repeat(3, 100px); 
        grid-template-rows: repeat(3, 100px); 
        gap: 5px; 
        margin: auto; 
    }
    .cell { 
        background-color: #fff; 
        border: 2px solid #333; 
        font-size: 48px; 
        display: flex; 
        justify-content: center; 
        align-items: center; 
        cursor: pointer; 
        transition: background 0.2s;
    }
    .cell:hover {
        background-color: #e6e6e6;
    }
    .highlight {
        background-color: #ffeb3b !important;
    }
    #status { margin-top: 20px; font-size: 20px; }
    button { 
        margin-top: 10px; 
        padding: 10px 20px; 
        font-size: 16px; 
        cursor: pointer; 
        border: none;
        background: #333;
        color: white;
        border-radius: 5px;
    }
    button:hover {
        background: #000;
    }
</style><h2>Tic Tac Toe</h2>

<div class="container">
    <div id="board"></div>

    <div id="status">Click a cell to start!</div>

    <button id="restart">Restart Game</button>
</div>

<script>
class TicTacToe {
    constructor() {
        this.board = Array(9).fill("");
        this.currentPlayer = "X";
        this.gameOver = false;



