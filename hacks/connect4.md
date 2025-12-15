---
layout: opencs
title: Connect 4
permalink: /connect4/
---

<html lang="en">
<head>
<meta charset="UTF-8" />
<title>Connect 4</title>
<style>
  :root {
    --red: red;
    --yellow: yellow;
    --background: #ddd;
    --card: white;
    --board-border: #444;
  }

  body {
    background-color: var(--background);
    color: black;
    font-family: Arial, sans-serif;
  }

  #gameBoard {
    display: flex;
    flex-direction: column;
    width: max-content;
    border: 2px solid var(--board-border);
    padding: 10px;
    margin-top: 10px;
    background: var(--card);
  }

  .board-row {
    display: flex;
  }

  .cell {
    width: 50px;
    height: 50px;
    border: 2px solid #222;
    background: var(--background);
    border-radius: 50%;
    margin: 3px;
    cursor: default;
    outline: 1px solid black;
  }

  .cell.clickable {
    cursor: pointer;
  }

  .cell.clickable:hover {
    background-color: #aaa;
  }

  .red {
    background-color: var(--red);
  }

  .yellow {
    background-color: var(--yellow);
  }

  .preview-col {
    box-shadow: 0 0 10px 3px rgba(255, 255, 255, 0.7);
  }

  /* Buttons container */
  #themeButtons {
    margin-bottom: 10px;
  }
</style>
</head>
<body>

<h1>Connect 4</h1>

<!-- Theme Switcher Buttons -->
<div id="themeButtons">
  <button id="btnClassic">Classic Theme</button>
  <button id="btnMidnight">Midnight Theme</button>
</div>

<!-- Restart Button -->
<button id="btnRestart">Restart Game</button>

<!-- Game control buttons -->
<button onclick="startGame()">Start Game</button>
<button onclick="showRules()">Game Rules</button>

<!-- Game Board -->
<div id="gameBoard" style="display:none;"></div>

<!-- Output Message -->
<p id="output"></p>

<!-- Drop Sound -->
<audio id="dropSound" src="assets/audio/click.mp3" preload="auto"></audio>

<script>
  // Theme data
  const themes = {
    classic: {
      '--red': 'red',
      '--yellow': 'yellow',
      '--background': '#ddd',
      '--card': 'white',
      '--board-border': '#444',
      'color': 'black'
    },
    midnight: {
      '--red': '#ff5555',
      '--yellow': '#ffdd55',
      '--background': '#222',
      '--card': '#333',
      '--board-border': '#666',
      'color': 'white'
    }
  };

  function applyTheme(theme) {
    const root = document.documentElement;
    for (const [key, value] of Object.entries(theme)) {
      root.style.setProperty(key, value);
    }
    document.body.style.color = theme.color;
  }

  document.getElementById('btnClassic').onclick = () => applyTheme(themes.classic);
  document.getElementById('btnMidnight').onclick = () => applyTheme(themes.midnight);

  // Set default theme on load
  applyTheme(themes.classic);

  // Player class
  class Player {
    constructor(name, color) {
      this.name = name;
      this.color = color;
      this.coins = 21;
    }
    useCoin() {
      if (this.coins > 0) {
        this.coins--;
        return true;
      }
      return false;
    }
  }

  // Board class
  class Board {
    constructor(rows = 6, cols = 7) {
      this.rows = rows;
      this.cols = cols;
      this.grid = Array.from({ length: rows }, () => Array(cols).fill(null));
    }
    dropCoin(col, player) {
      for (let r = this.rows - 1; r >= 0; r--) {
        if (!this.grid[r][col]) {
          this.grid[r][col] = player.color;
          return { row: r, col };
        }
      }
      return null; // Column full
    }
  }

  // Check Win function
  function checkWin(grid, row, col) {
    const color = grid[row][col];
    if (!color) return false;

    const directions = [
      { dr: 0, dc: 1 },  // horizontal
      { dr: 1, dc: 0 },  // vertical
      { dr: 1, dc: 1 },  // diagonal down-right
      { dr: 1, dc: -1 }  // diagonal down-left
    ];

    const rows = grid.length;
    const cols = grid[0].length;

    for (let {dr, dc} of directions) {
      let count = 1;

      // forward
      let r = row + dr;
      let c = col + dc;
      while (r >= 0 && r < rows && c >= 0 && c < cols && grid[r][c] === color) {
        count++;
        r += dr;
        c += dc;
      }

      // backward
      r = row - dr;
      c = col - dc;
      while (r >= 0 && r < rows && c >= 0 && c < cols && grid[r][c] === color) {
        count++;
        r -= dr;
        c -= dc;
      }

      if (count >= 4) {
        return true;
      }
    }

    return false;
  }

  // Game class
  class Game {
    constructor(player1, player2, board) {
      this.players = [player1, player2];
      this.board = board;
      this.current = 0;
      this.isOver = false;
    }
    get activePlayer() {
      return this.players[this.current];
    }
    switchPlayer() {
      this.current = 1 - this.current;
    }
    makeMove(col) {
      if (this.isOver) return "Game over";
      const player = this.activePlayer;
      if (!player.useCoin()) return "No coins left";

      const move = this.board.dropCoin(col, player);
      if (!move) return "Column full";

      // Play drop sound
      document.getElementById('dropSound').play().catch(() => {});

      // Check win
      if (checkWin(this.board.grid, move.row, move.col)) {
        this.isOver = true;
        return `${player.name} (${player.color}) wins!`;
      }

      this.switchPlayer();
      return null;
    }
  }

  let game = null;

  // Render the board
  function renderBoard(grid) {
    const container = document.getElementById("gameBoard");
    container.innerHTML = "";
    for (let r = 0; r < grid.length; r++) {
      const rowDiv = document.createElement("div");
      rowDiv.className = "board-row";

      for (let c = 0; c < grid[0].length; c++) {
        const cell = document.createElement("div");
        cell.className = "cell";

        if (grid[r][c]) {
          cell.classList.add(grid[r][c]);
        } else if (r === 0 && !game.isOver) {
          cell.classList.add("clickable");
          cell.dataset.col = c;
          
          // Highlight column on hover
          cell.onmouseenter = () => highlightColumn(c);
          cell.onmouseleave = () => removeHighlight();

          cell.onclick = () => handleColumnClick(c);
        }

        rowDiv.appendChild(cell);
      }
      container.appendChild(rowDiv);
    }
  }

  // Highlight entire column
  function highlightColumn(col) {
    const cells = document.querySelectorAll(`#gameBoard .cell`);
    cells.forEach(cell => {
      if (parseInt(cell.dataset.col) === col) {
        cell.classList.add('preview-col');
      }
    });
  }

  function removeHighlight() {
    document.querySelectorAll('.preview-col').forEach(cell => {
      cell.classList.remove('preview-col');
    });
  }

  // Start game
  function startGame() {
    const player1 = new Player("Player 1", "red");
    const player2 = new Player("Player 2", "yellow");
    const boardObj = new Board();
    game = new Game(player1, player2, boardObj);

    document.getElementById("gameBoard").style.display = "flex";
    renderBoard(game.board.grid);
    document.getElementById("output").textContent =
      `Game started! ${game.activePlayer.name} (${game.activePlayer.color}) goes first.`;
  }

  // Handle column click
  function handleColumnClick(col) {
    if (!game || game.isOver) return;

    const result = game.makeMove(col);
    renderBoard(game.board.grid);

    if (result) {
      showWinMessage(result);
    } else {
      document.getElementById("output").textContent =
        `Next turn: ${game.activePlayer.name} (${game.activePlayer.color})`;
    }
  }

  // Show rules
  function showRules() {
    document.getElementById("output").textContent =
      "Connect 4 rules: take turns dropping pieces into a 7x6 board. First to get 4 in a row wins! Click the top row to drop pieces.";
  }

  // Show win message with emoji and glow
  function showWinMessage(message) {
    const output = document.getElementById('output');
    let emoji = '🤝'; // default draw

    if (message.toLowerCase().includes('red')) emoji = '🔴';
    else if (message.toLowerCase().includes('yellow')) emoji = '🟡';

    output.innerHTML = `<span style="font-weight:bold; text-shadow: 0 0 5px #fff;">${emoji} ${message}</span>`;
  }

  // Restart confirmation
  document.getElementById('btnRestart').onclick = () => {
    if (confirm('Start a new game?')) {
      startGame();
    }
  };

</script>

</body>
</html>