---
layout: base
title: Snakefail
permalink: /snakefail/
---

<style>

    body{
    }
    .wrap{
        margin-left: auto;
        margin-right: auto;
    }

    canvas{
        display: none;
        border-style: solid;
        border-width: 10px;
        border-color: #FFFFFF;
    }
    canvas:focus{
        outline: none;
    }

    /* All screens style */
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

    #menu{
        display: block;
    }

    #gameover{
        display: none;
    }

    #setting{
        display: none;
    }

    #setting input{
        display: inline-block;
    }

    #setting label{
        cursor: pointer;
    }

    #setting input:checked + label{
        background-color: #FFF;
        color: #000;
    }
</style>

<h2>Snake</h2>
<div class="container">
    <header class="pb-3 mb-4 border-bottom border-primary text-dark">
        <p class="fs-4">Score: <span id="score_value">0</span></p>
    </header>
    <div class="container bg-secondary" style="text-align:center;">
        <!-- Main Menu -->
        <div id="menu" class="py-4 text-light">
            <p>Welcome to Snake, press <span style="background-color: #FFFFFF; color: #000000">space</span> to begin</p>
            <a id="new_game" class="link-alert">new game</a>
            <a id="setting_menu" class="link-alert">settings</a>
        </div>
        <!-- Game Over -->
        <div id="gameover" class="py-4 text-light">
            <p>Game Over, press <span style="background-color: #FFFFFF; color: #000000">space</span> to try again</p>
            <a id="new_game1" class="link-alert">new game</a>
            <a id="setting_menu1" class="link-alert">settings</a>
        </div>
        <!-- Play Screen -->
        <canvas id="snake" class="wrap" width="320" height="320" tabindex="1"></canvas>
        <!-- Settings Screen -->
        <div id="setting" class="py-4 text-light">
            <p>Settings Screen, press <span style="background-color: #FFFFFF; color: #000000">space</span> to go back to playing</p>
            <a id="new_game2" class="link-alert">new game</a>
            <br>
            <p>Food:
                <input id="food_classic" type="radio" name="food" value="classic" checked/>
                <label for="food_classic">Classic◻️</label>
                <input id="food_noodles" type="radio" name="food" value="noodles"/>
                <label for="food_classic">Noodles🍜</label>
                <input id="food_apple" type="radio" name="food" value="apple"/>
                <label for="food_classic">Apple🍎</label>
                <input id="food_candy" type="radio" name="food" value="candy"/>
                <label for="food_classic">Candy🍬</label>
                <input id="food_potato" type="radio" name="food" value="potato"/>
                <label for="food_classic">Potato🥔</label>
                <!-- <input id="food_random" type="radio" name="food" value="random"/>
                <label for="food_classic">Random❓</label> -->
            </p>
            <p>Speed:
                <input id="speed1" type="radio" name="speed" value="120" checked/>
                <label for="speed1">Slow</label>
                <input id="speed2" type="radio" name="speed" value="75"/>
                <label for="speed2">Normal</label>
                <input id="speed3" type="radio" name="speed" value="35"/>
                <label for="speed3">Fast</label>
                <input id="speed4" type="radio" name="speed" value="0"/>
                <label for="speed4">ExtraFast</label>
            </p>
            <p>Wall:
                <input id="wallon" type="radio" name="wall" value="1" checked/>
                <label for="wallon">On</label>
                <input id="walloff" type="radio" name="wall" value="0"/>
                <label for="walloff">Off</label>
            </p>
            <p>Nearsightedness:
                <input id="bigscreen" type="radio" name="sight" value="1" checked/>
                <label for="bigscreen">On</label>
                <input id="smallscreen" type="radio" name="sight" value="0"/>
                <label for="smallscreen">Off</label>
            </p>
            <p>Camouflage:
                <input id="hidden" type="radio" name="camo" value="1" checked/>
                <label for="hidden">On</label>
                <input id="nothidden" type="radio" name="camo" value="0"/>
                <label for="nothidden">Off</label>
            </p>
        </div>
    </div>
</div>

<script>
    (function(){
        /* Attributes of Game */
        /////////////////////////////////////////////////////////////
        // Canvas & Context
        const canvas = document.getElementById("snake");
        const ctx = canvas.getContext("2d");
        // HTML Game IDs
        const SCREEN_SNAKE = 0;
        const screen_snake = document.getElementById("snake");
        const ele_score = document.getElementById("score_value");
        const speed_setting = document.getElementsByName("speed");
        const wall_setting = document.getElementsByName("wall");
        const sight_setting = document.getElementsByName("sight");
        const camo_setting = document.getElementsByName("camo");
        const food_setting = document.getElementsByName("food");
        // HTML Screen IDs (div)
        const SCREEN_MENU = -1, SCREEN_GAME_OVER=1, SCREEN_SETTING=2;
        const screen_menu = document.getElementById("menu");
        const screen_game_over = document.getElementById("gameover");
        const screen_setting = document.getElementById("setting");
        // HTML Event IDs (a tags)
        const button_new_game = document.getElementById("new_game");
        const button_new_game1 = document.getElementById("new_game1");
        const button_new_game2 = document.getElementById("new_game2");
        const button_setting_menu = document.getElementById("setting_menu");
        const button_setting_menu1 = document.getElementById("setting_menu1");
        // Game Control
        let BLOCK = 10;   // size of block rendering
        let SCREEN = SCREEN_MENU;
        let foodStyle = "classic";
        let snake;
        let snake_dir;
        let snake_next_dir;
        let snake_speed;
        let food = {x: 0, y: 0};
        let score;
        let wall;
        let sight;
        let camo;
        let foodClassic = "classic";
        let foodNoodles = "🍜";
        let foodApple = "🍎";
        let foodCandy = "🍬";
        let foodPotato = "🥔";
        let snakeColor = "white";

        /* Display Control */
        /////////////////////////////////////////////////////////////
        // 0 for the game
        // 1 for the main menu
        // 2 for the settings screen
        // 3 for the game over screen
        let showScreen = function(screen_opt){
            SCREEN = screen_opt;
            switch(screen_opt){
                case SCREEN_SNAKE:
                    screen_snake.style.display = "block";
                    screen_menu.style.display = "none";
                    screen_setting.style.display = "none";
                    screen_game_over.style.display = "none";
                    break;
                case SCREEN_GAME_OVER:
                    screen_snake.style.display = "block";
                    screen_menu.style.display = "none";
                    screen_setting.style.display = "none";
                    screen_game_over.style.display = "block";
                    break;
                case SCREEN_SETTING:
                    screen_snake.style.display = "none";
                    screen_menu.style.display = "none";
                    screen_setting.style.display = "block";
                    screen_game_over.style.display = "none";
                    break;
            }
        };
        /* Actions and Events  */
        /////////////////////////////////////////////////////////////
        window.onload = function(){
            
            // HTML Events to Functions
            button_new_game.onclick = function(){newGame();};
            button_new_game1.onclick = function(){newGame();};
            button_new_game2.onclick = function(){newGame();};
            button_setting_menu.onclick = function(){showScreen(SCREEN_SETTING);};
            button_setting_menu1.onclick = function(){showScreen(SCREEN_SETTING);};
            //food setting
            setFood("classic");
            for (let i = 0; i < food_setting.length; i++) {
                food_setting[i].addEventListener("click", function () {
                    for (let i = 0; i < food_setting.length; i++) {
                        if (food_setting[i].checked) {
                            foodStyle = food_setting[i].value;
                        }
                    }
                });
            }
            food_setting.forEach(foodOption => {
                foodOption.addEventListener("change", function() {
                    foodStyle = this.value; // Update food style
                });
            });
            
            // speed
            setSnakeSpeed(150);
            document.getElementById(120).checked = true;
            for(let i = 0; i < speed_setting.length; i++){
                speed_setting[i].addEventListener("click", function(){
                    for(let i = 0; i < speed_setting.length; i++){
                        if(speed_setting[i].checked){
                            setSnakeSpeed(speed_setting[i].value);
                        }
                    }
                });
            }
            // wall setting
            setWall(1);
            document.getElementById("wallon").checked = true;
            for(let i = 0; i < wall_setting.length; i++){
                wall_setting[i].addEventListener("click", function(){
                    for(let i = 0; i < wall_setting.length; i++){
                        if(wall_setting[i].checked){
                            setWall(wall_setting[i].value);
                        }
                    }
                });
            }
            //sight setting
            setSight(0);
            document.getElementById("smallscreen").checked = true;
            for (let i = 0; i < sight_setting.length; i++) {
                sight_setting[i].addEventListener("click", function () {
                    for (let i = 0; i < sight_setting.length; i++) {
                        if (sight_setting[i].checked) {
                            setSight(parseInt(sight_setting[i].value));
                        }
                    }
                });
            }
            setCamo(0);
            document.getElementById("nothidden").checked = true;
            for (let i = 0; i < camo_setting.length; i++) {
                camo_setting[i].addEventListener("click", function () {
                    for (let i = 0; i < camo_setting.length; i++) {
                        if (camo_setting[i].checked) {
                            setCamo(parseInt(camo_setting[i].value));
                        }
                    }
                });
            }
            
            screen_snake.focus();

            // activate window events
            window.addEventListener("keydown", function(evt) {
                if (SCREEN === SCREEN_SNAKE) {
                    evt.preventDefault();
                
                // spacebar detected
                if(evt.code === "Space" && SCREEN !== SCREEN_SNAKE)
                    newGame();
                }

                switch (evt.key) {
                    case "ArrowUp":
                        if (snake_dir !== 2) snake_next_dir = 0;
                        break;
                    case "ArrowRight":
                        if (snake_dir !== 3) snake_next_dir = 1;
                        break;
                    case "ArrowDown":
                        if (snake_dir !== 0) snake_next_dir = 2;
                        break;
                    case "ArrowLeft":
                        if (snake_dir !== 1) snake_next_dir = 3;
                        break;
                    case "Space":
                        if(SCREEN !== SCREEN_SNAKE) newGame();
                        break;
                }
            }, true);
            // if (["ArrowUp", "ArrowDown", "ArrowLeft", "ArrowRight", "Space"].includes(evt.code)) {
            //         evt.preventDefault();
            //     }

            //     if (SCREEN === SCREEN_SNAKE) {
            //         switch (evt.code) {
            //             case "ArrowUp":
            //                 if (snake_dir !== 2) snake_next_dir = 0;
            //                 break;
            //             case "ArrowRight":
            //                 if (snake_dir !== 3) snake_next_dir = 1;
            //                 break;
            //             case "ArrowDown":
            //                 if (snake_dir !== 0) snake_next_dir = 2;
            //                 break;
            //             case "ArrowLeft":
            //                 if (snake_dir !== 1) snake_next_dir = 3;
            //                 break;
            //         }
            //     } else if (evt.code === "Space") {
            //         // Space starts the game or restarts from other screens
            //         newGame();
            //     }
            // });
        }
        /* Snake is on the Go (Driver Function)  */
        /////////////////////////////////////////////////////////////
        let mainLoop = function(){
            console.log({snake, snake_dir, SCREEN});
            let _x = snake[0].x;
            let _y = snake[0].y;
            snake_dir = snake_next_dir;   // read async event key
            // Direction 0 - Up, 1 - Right, 2 - Down, 3 - Left
            switch(snake_dir){
                case 0: _y--; break;
                case 1: _x++; break;
                case 2: _y++; break;
                case 3: _x--; break;
            }
            snake.pop(); // tail is removed
            snake.unshift({x: _x, y: _y}); // head is new in new position/orientation
            // Wall Checker
            if(wall === 1){
                // Wall on, Game over test
                if (snake[0].x < 0 || snake[0].x === canvas.width / BLOCK || snake[0].y < 0 || snake[0].y === canvas.height / BLOCK){
                    showScreen(SCREEN_GAME_OVER);
                    return;
                }
            }else{
                // Wall Off, Circle around
                for(let i = 0, x = snake.length; i < x; i++){
                    if(snake[i].x < 0){
                        snake[i].x = snake[i].x + (canvas.width / BLOCK);
                    }
                    if(snake[i].x === canvas.width / BLOCK){
                        snake[i].x = snake[i].x - (canvas.width / BLOCK);
                    }
                    if(snake[i].y < 0){
                        snake[i].y = snake[i].y + (canvas.height / BLOCK);
                    }
                    if(snake[i].y === canvas.height / BLOCK){
                        snake[i].y = snake[i].y - (canvas.height / BLOCK);
                    }
                }
            }
            //food
            for (let i = 0; i < food_setting.length; i++) {
                food_setting[i].addEventListener("change", function () {
                    foodStyle = this.value; // Updates foodStyle based on the selected input
                });
            }
            //Trying to make a canvas bigger for nearsighted people if they press on and want to play without their glasses for some reason
            for (let i = 0; i < sight_setting.length; i++) {
                sight_setting[i].addEventListener("click", function () {
                    for (let i = 0; i < sight_setting.length; i++) {
                        if (sight_setting[i].checked) {
                            setSight(parseInt(sight_setting[i].value));
                        }
                    }
                });
            }
            //camo?
            for (let i = 0; i < camo_setting.length; i++) {
                camo_setting[i].addEventListener("click", function () {
                    for (let i = 0; i < camo_setting.length; i++) {
                        if (camo_setting[i].checked) {
                            setCamo(parseInt(camo_setting[i].value));
                        }
                    }
                });
            }
            // Snake vs Snake checker
            for(let i = 1; i < snake.length; i++){
                // Game over test
                if (snake[0].x === snake[i].x && snake[0].y === snake[i].y){
                    showScreen(SCREEN_GAME_OVER);
                    return;
                }
            }
            // Snake eats food checker
            if(checkBlock(snake[0].x, snake[0].y, food.x, food.y)){
                snake[snake.length] = {x: snake[0].x, y: snake[0].y};
                altScore(++score);
                addFood();
                activeDot(food.x, food.y, true);
            }
            // Repaint canvas
            ctx.beginPath();
            ctx.fillStyle = "royalblue";
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            // Paint snake
            for(let i = 0; i < snake.length; i++){
                activeDot(snake[i].x, snake[i].y);
            }
            // Paint food
            activeDot(food.x, food.y);
            // Debug
            //document.getElementById("debug").innerHTML = snake_dir + " " + snake_next_dir + " " + snake[0].x + " " + snake[0].y;
            // Recursive call after speed delay, déjà vu
            setTimeout(mainLoop, snake_speed);
        }
        /* New Game setup */
        /////////////////////////////////////////////////////////////
        let newGame = function(){
            // snake game screen
            showScreen(SCREEN_SNAKE);
            screen_snake.focus();
            // game score to zero
            score = 0;
            altScore(score);
            // initial snake
            snake = [{x: 0, y: 15}];
            snake_dir = 1;
            snake_next_dir = 1;
            // food on canvas
            addFood();
            // setSnakeSpeed(150);
            // setWall(1);
            // activate canvas event
            canvas.onkeydown = function(evt) {
                evt.preventDefault();
                changeDir(evt.keyCode);
                // screen_snake_focus();
                // canvas.focus();
            }
            mainLoop();
        }
        /* Key Inputs and Actions */
        /////////////////////////////////////////////////////////////
        let changeDir = function(key){
            // test key and switch direction
            switch(key) {
                case 37:    // left arrow
                    if (snake_dir !== 1)    // not right
                        snake_next_dir = 3; // then switch left
                    break;
                case 38:    // up arrow
                    if (snake_dir !== 2)    // not down
                        snake_next_dir = 0; // then switch up
                    break;
                case 39:    // right arrow
                    if (snake_dir !== 3)    // not left
                        snake_next_dir = 1; // then switch right
                    break;
                case 40:    // down arrow
                    if (snake_dir !== 0)    // not up
                        snake_next_dir = 2; // then switch down
                    break;
            }
        }
        /* Dot for Food or Snake part */
        /////////////////////////////////////////////////////////////
        let activeDot = function(x, y, isFood = false) {
            const pixelX = x * BLOCK;
            const pixelY = y * BLOCK;

            if (isFood) {
                switch (foodStyle) {
                    case "classic":
                        ctx.fillStyle = snakeColor;
                        ctx.fillRect(x * BLOCK, y * BLOCK, BLOCK, BLOCK);
                        break;

                    case "noodles":
                        ctx.font = `${BLOCK}px Arial`;
                        ctx.textAlign = "center";
                        ctx.textBaseline = "middle";
                        ctx.fillText(foodNoodles, pixelX + BLOCK / 2, pixelY + BLOCK / 2);
                        break;

                    case "apple":
                        ctx.font = `${BLOCK}px Arial`;
                        ctx.textAlign = "center";
                        ctx.textBaseline = "middle";
                        ctx.fillText(foodApple, pixelX + BLOCK / 2, pixelY + BLOCK / 2);
                        break;

                    case "candy":
                        ctx.font = `${BLOCK}px Arial`;
                        ctx.textAlign = "center";
                        ctx.textBaseline = "middle";
                        ctx.fillText(foodCandy, pixelX + BLOCK / 2, pixelY + BLOCK / 2);
                        break;

                    case "potato":
                        ctx.font = `${BLOCK}px Arial`;
                        ctx.textAlign = "center";
                        ctx.textBaseline = "middle";
                        ctx.fillText(foodPotato, pixelX + BLOCK / 2, pixelY + BLOCK / 2);
                        break;
                } 
            } else {
                    ctx.fillStyle = snakeColor;
                    ctx.fillRect(pixelX, pixelY, BLOCK, BLOCK);
                }
            };
        
        /* Random food placement */
        /////////////////////////////////////////////////////////////
        let addFood = function(){
            food.x = Math.floor(Math.random() * ((canvas.width / BLOCK) - 1));
            food.y = Math.floor(Math.random() * ((canvas.height / BLOCK) - 1));
            for(let i = 0; i < snake.length; i++){
                if(checkBlock(food.x, food.y, snake[i].x, snake[i].y)){
                    addFood();
                }
            }
        };
        /* Collision Detection */
        /////////////////////////////////////////////////////////////
        let checkBlock = function(x, y, _x, _y){
            return (x === _x && y === _y);
        };
        /* Update Score */
        /////////////////////////////////////////////////////////////
        let altScore = function(score_val){
            ele_score.innerHTML = String(score_val);
        }
        /////////////////////////////////////////////////////////////
        // Change the snake speed...
        // 150 = slow
        // 100 = normal
        // 50 = fast
        let setSnakeSpeed = function(speed_value){
            snake_speed = speed_value;
        }
        /////////////////////////////////////////////////////////////
        let setWall = function(wall_value){
            wall = parseInt(wall_value);
            if(wall === 0){
                screen_snake.style.borderColor = "#FFFFFF";
            }
            if(wall === 1){
                screen_snake.style.borderColor = "#212121";//#606060
            }
        }
    //sight stuff
    let setSight = function (sight_value) {
        sight = sight_value;
        if (sight === 1) { // Nearsighted (big canvas)
            canvas.width = 800;
            canvas.height = 800;
            BLOCK = 25;
        } else { // Normal
            canvas.width = 320;
            canvas.height = 320;
            BLOCK = 10;
        }
        // Reset game elements to fit the new canvas size
        resetGameElements();
    };
    //camo stuff
    let setCamo = function (camo_value) {
        camo = camo_value;
        if (camo === 1) {
            snakeColor = "#4161e1";
        } else {
            snakeColor = "#FFFFFF";
        }
    };
    let resetGameElements = function () {
        // Reinitialize the snake to prevent issues with the new canvas size
        snake = [{ x: Math.floor(canvas.width / (2 * BLOCK)), y: Math.floor(canvas.height / (2 * BLOCK)) }];
        addFood(); // Reposition food within the new canvas size
        altScore(0); // Reset score display
    };        
    })();
</script>