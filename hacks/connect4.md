---
layout: opencs
title: Connect 4
permalink: /connect4/
---

<style>
    body{
        font-family: Arial, sans-serif;
        text-align: center;
        margin: 0;
        padding: 0;
    }
    .wrap{
        width: 95%;
        margin-left: auto;
        margin-right: auto;
        padding top: 40px;
    }
    .menu-btn {
        padding: 12px 20px;
        margin: 10px;
        background: #0074d9;
        color: white;
        border: none;
        border-radius: 8px;
        font-size: 18px;
        cursor: pointer;
    }

    .menu-btn:hover {
        background: #005fa3;
    }
</style>

<body>
    <div class="wrap">
        <h1>Connect 4</h1>

        <button class="menu-btn" onclick="startGame()">Start Game</button>
        <button class="menu-btn" onclick="showRules()">Game Rules</button>

        <p id="output"></p>
    </div>

    <script>
        // Class for each player class Player
        class Player {
            constructor(name, color, coins = 21, time = 300) {
                this.name = name;
                this.color = color;
                this.coins = coins;
                this.time = time;
            }

            useCoin() {
                if (this.coins > 0) {
                    this.coins--;
                    return true;
                }
                return false;
            }
        }

        // Class for the board class Board
        class Board {
            constructor(rows = 6, cols = 7) {
                this.rows = rows;
                this.cols = cols;
                this.grid = Array.from({ length: rows }, () =>
                    Array(cols).fill(null)
                );
            }

            dropCoin(col, player) {
                for (let r = this.rows - 1; r >= 0; r--) {
                    if (!this.grid[r][col]) {
                        this.grid[r][col] = player.color;
                        return { row: r, col };
                    }
                }
                return null; // column full
            }
        }

        // Class for the game controller class Game 
        class Game {
            constructor(player1, player2, board) {
                this.players = [player1, player2];
                this.board = board;
                this.current = 0;
            }

            get activePlayer() {
                return this.players[this.current];
            }

            switchTurn() {
                this.current = 1 - this.current;
            }

            playMove(col) {
                const player = this.activePlayer;

                if (!player.useCoin())
                    return "No coins left!";

                const move = this.board.dropCoin(col, player);
                if (!move)
                    return "Column full!";

                this.switchTurn();
                return `Placed in column ${col}.`;
            }
        }

        //Menu
        function startGame() {
            document.getElementById("output").textContent =
                "Starting game... (you can add your board UI next!)";
        }

        function showRules() {
            document.getElementById("output").textContent =
                "Connect 4 rules: take turns dropping pieces into a 7x6 board. First to get 4 in a row wins!";
        }
    </script>
</body>