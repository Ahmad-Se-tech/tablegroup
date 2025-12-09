---
layout: opencs
title: Snake Game
permalink: /snake/
---

<style>
    body{}
    .wrap{
        margin-left: auto;
        margin-right: auto;
    }

    canvas{
        display: none;
        border-style: solid;
        border-width: 10px;
        border-color: rgba(21, 155, 0, 1);
    }
    canvas:focus{
        outline: none;
    }

    #gameover p, #setting p, #menu p{
        font-size: 20px;
    }

    #gameover a, #setting a, #menu a{
        font-size: 30px;
        display: block;
    }

    #gameover a:hover, #setting a:hover, #menu a:hover{
        cursor: pointer;
    }

    #gameover a:hover::before, #setting a:hover::before, #menu a:hover::before{
        content: ">";
        margin-right: 10px;
    }

    #menu{ display: block; }
    #gameover{ display: none; }
    #setting{ display: none; }
    #setting input{ display:none; }
    #setting label{ cursor: pointer; }
    #setting input:checked + label{
        background-color: #000000ff;
        color: #ffffffff;
    }
</style>

<h2>Snake</h2>
<div class="container">
    <p class="fs-4">Score: <span id="score_value">0</span></p>

    <div class="container bg-secondary" style="text-align:center;">
        <div id="menu" class="py-4 text-light">
            <p>Welcome to Snake, press <span style="background-color: #FFFFFF; color: #000000">space</span> to begin</p>
            <a id="new_game" class="link-alert">new game</a>
            <a id="setting_menu" class="link-alert">settings</a>
        </div>

        <div id="gameover" class="py-4 text-light">
            <p>You died, press <span style="background-color: #FFFFFF; color: #000000">space</span> to try again</p>
            <a id="new_game1" class="link-alert">new game</a>
            <a id="setting_menu1" class="link-alert">settings</a>
        </div>

        <canvas id="snake" class="wrap" width="320" height="320" tabindex="1"></canvas>

        <div id="setting" class="py-4 text-light">
            <p>Settings Screen, press <span style="background-color: #FFFFFF; color: #000000">space</span> to go back to playing</p>
            <a id="new_game2" class="link-alert">new game</a>
            <br>
            <p>Speed:
                <input id="speed1" type="radio" name="speed" value="200" checked/>
                <label for="speed1">Easy</label>
                <input id="speed2" type="radio" name="speed" value="175"/>
                <label for="speed2">Normal</label>
                <input id="speed3" type="radio" name="speed" value="150"/>
                <label for="speed3">Hard</label>
                <input id="speed4" type="radio" name="speed" value="50"/>
                <label for="speed3">Impossible</label>
            </p>
            <p>Wall:
                <input id="wallon" type="radio" name="wall" value="1" checked/>
                <label for="wallon">On</label>
                <input id="walloff" type="radio" name="wall" value="0"/>
                <label for="walloff">Off</label>
            </p>
        </div>
    </div>
</div>

 <script>
(function(){
    const canvas = document.getElementById("snake");
    const ctx = canvas.getContext("2d");

    const SCREEN_SNAKE = 0;
    const screen_snake = canvas;
    const ele_score = document.getElementById("score_value");
    const speed_setting = document.getElementsByName("speed");
    const wall_setting = document.getElementsByName("wall");

    const SCREEN_MENU = -1, SCREEN_GAME_OVER = 1, SCREEN_SETTING = 2;
    const screen_menu = document.getElementById("menu");
    const screen_game_over = document.getElementById("gameover");
    const screen_setting = document.getElementById("setting");

    const button_new_game = document.getElementById("new_game");
    const button_new_game1 = document.getElementById("new_game1");
    const button_new_game2 = document.getElementById("new_game2");
    const button_setting_menu = document.getElementById("setting_menu");
    const button_setting_menu1 = document.getElementById("setting_menu1");

    const BLOCK = 20;
    let SCREEN = SCREEN_MENU;
    let snake, snake_dir, snake_next_dir, snake_speed;
    let food = {x:0, y:0, type:"normal"}; // type: normal or gold
    let score, wall;

    let drawDot = function(x, y, color){
        ctx.fillStyle = color;
        ctx.fillRect(x * BLOCK, y * BLOCK, BLOCK, BLOCK);
    }

    let showScreen = function(screen_opt){
        SCREEN = screen_opt;
        screen_snake.style.display = screen_opt === SCREEN_SNAKE ? "block" : "none";
        screen_menu.style.display = screen_opt === SCREEN_MENU ? "block" : "none";
        screen_game_over.style.display = screen_opt === SCREEN_GAME_OVER ? "block" : "none";
        screen_setting.style.display = screen_opt === SCREEN_SETTING ? "block" : "none";
    }

    window.onload = function(){
        button_new_game.onclick = button_new_game1.onclick = button_new_game2.onclick = newGame;
        button_setting_menu.onclick = button_setting_menu1.onclick = ()=>showScreen(SCREEN_SETTING);

        setSnakeSpeed(200);
        for(let i=0;i<speed_setting.length;i++){
            speed_setting[i].addEventListener("click", function(){
                for(let j=0;j<speed_setting.length;j++){
                    if(speed_setting[j].checked){
                        setSnakeSpeed(speed_setting[j].value);
                    }
                }
            });
        }

        setWall(1);
        for(let i=0;i<wall_setting.length;i++){
            wall_setting[i].addEventListener("click", function(){
                for(let j=0;j<wall_setting.length;j++){
                    if(wall_setting[j].checked){
                        setWall(wall_setting[j].value);
                    }
                }
            });
        }

        window.addEventListener("keydown", function(evt){
            if(evt.code==="Space" && SCREEN!==SCREEN_SNAKE)
                newGame();
        }, true);
    }

    let mainLoop = function(){
        let _x = snake[0].x;
        let _y = snake[0].y;
        snake_dir = snake_next_dir;

    function drawSnakeHead(x, y, dir) {
    // head background
    drawDot(x, y, "#3d4bcbff");

    // draw eyes depending on direction
    ctx.fillStyle = "white";

    if (dir === 0) { 
        // UP
        ctx.fillRect(x*BLOCK + 4, y*BLOCK + 4, 4, 4);
        ctx.fillRect(x*BLOCK + 12, y*BLOCK + 4, 4, 4);
    } 
    else if (dir === 1) { 
        // RIGHT
        ctx.fillRect(x*BLOCK + 12, y*BLOCK + 4, 4, 4);
        ctx.fillRect(x*BLOCK + 12, y*BLOCK + 12, 4, 4);
    } 
    else if (dir === 2) {
        // DOWN
        ctx.fillRect(x*BLOCK + 4, y*BLOCK + 12, 4, 4);
        ctx.fillRect(x*BLOCK + 12, y*BLOCK + 12, 4, 4);
    } 
    else if (dir === 3) { 
        // LEFT
        ctx.fillRect(x*BLOCK + 4, y*BLOCK + 4, 4, 4);
        ctx.fillRect(x*BLOCK + 4, y*BLOCK + 12, 4, 4);
    }
}


        switch(snake_dir){
            case 0: _y--; break;
            case 1: _x++; break;
            case 2: _y++; break;
            case 3: _x--; break;
        }

        snake.pop();
        snake.unshift({x:_x, y:_y});

        if(wall===1){
            if(snake[0].x<0 || snake[0].x===canvas.width/BLOCK || snake[0].y<0 || snake[0].y===canvas.height/BLOCK){
                showScreen(SCREEN_GAME_OVER);
                return;
            }
        } else {
            for(let i=0;i<snake.length;i++){
                if(snake[i].x<0) snake[i].x += canvas.width/BLOCK;
                if(snake[i].x===canvas.width/BLOCK) snake[i].x -= canvas.width/BLOCK;
                if(snake[i].y<0) snake[i].y += canvas.height/BLOCK;
                if(snake[i].y===canvas.height/BLOCK) snake[i].y -= canvas.height/BLOCK;
            }
        }

        for(let i=1;i<snake.length;i++){
            if(snake[0].x===snake[i].x && snake[0].y===snake[i].y){
                showScreen(SCREEN_GAME_OVER);
                return;
            }
        }

        // Snake eats food
        if(checkBlock(snake[0].x, snake[0].y, food.x, food.y)){
            snake.push({x:snake[0].x, y:snake[0].y});
            score += food.type==="gold"?5:1; // gold gives 5 points
            altScore(score);
            addFood();
        }

        // Repaint canvas
        ctx.fillStyle = "#227e2bff";
        ctx.fillRect(0,0,canvas.width,canvas.height);

        // draw head
        drawSnakeHead(snake[0].x, snake[0].y, snake_dir);

        // draw body
        for(let i=1;i<snake.length;i++){
             drawDot(snake[i].x, snake[i].y, "#3d4bcbff");
}


        // Draw food
        if(food.type==="gold"){
            // blink effect
            let color = Math.floor(Date.now()/300)%2===0?"gold":"#ffff55";
            drawDot(food.x, food.y, color);
        } else {
            drawDot(food.x, food.y, "red");
        }

        setTimeout(mainLoop, snake_speed);
    }

    let newGame = function(){
        showScreen(SCREEN_SNAKE);
        screen_snake.focus();
        score=0; altScore(score);
        snake=[{x:0, y:15}];
        snake_next_dir=1;
        addFood();
        canvas.onkeydown=function(evt){ changeDir(evt.keyCode); }
        mainLoop();
    }

    let changeDir = function(key){
        switch(key){
            // Arrow keys
            case 37: if(snake_dir!==1) snake_next_dir=3; break; // Left
            case 38: if(snake_dir!==2) snake_next_dir=0; break; // Up
            case 39: if(snake_dir!==3) snake_next_dir=1; break; // Right
            case 40: if(snake_dir!==0) snake_next_dir=2; break; // Down
            // WASD keys
            case 65: if(snake_dir!==1) snake_next_dir=3; break; // A = Left
            case 87: if(snake_dir!==2) snake_next_dir=0; break; // W = Up
            case 68: if(snake_dir!==3) snake_next_dir=1; break; // D = Right
            case 83: if(snake_dir!==0) snake_next_dir=2; break; // S = Down
        }
    }

    let addFood = function(){
        food.x = Math.floor(Math.random()*(canvas.width/BLOCK-1));
        food.y = Math.floor(Math.random()*(canvas.height/BLOCK-1));

        // Decide if this is a gold apple (~20% chance)
        food.type = Math.random()<0.2 ? "gold":"normal";

        for(let i=0;i<snake.length;i++){
            if(checkBlock(food.x, food.y, snake[i].x, snake[i].y)){
                addFood();
                return;
            }
        }
    }

    let checkBlock = function(x,y,_x,_y){ return x===_x && y===_y; }
    let altScore = function(val){ ele_score.innerHTML=String(val); }
    let setSnakeSpeed=function(val){ snake_speed=val; }
    let setWall=function(val){
        wall=val;
        screen_snake.style.borderColor = wall===1 ? "#77fa3bff" : "#606060";
    }
})();
</script>


