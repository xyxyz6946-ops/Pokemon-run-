# Pokemon-run-
Its a very interesting game 
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Pikachu World Adventure</title>
    <style>
        body { margin: 0; overflow: hidden; background: #222; font-family: 'Arial', sans-serif; touch-action: manipulation; }
        #game-container { position: relative; width: 100vw; height: 100vh; transition: background 1s; display: flex; justify-content: center; align-items: center; }
        canvas { background: rgba(255,255,255,0.1); box-shadow: 0 0 20px rgba(0,0,0,0.5); border-bottom: 10px solid #4aaa4a; width: 100%; max-width: 800px; height: 400px; }
        #ui { position: absolute; top: 10px; width: 95%; display: flex; justify-content: space-between; color: white; font-weight: bold; font-size: 18px; z-index: 5; text-shadow: 2px 2px #000; }
        .btn { padding: 12px 20px; background: #FFCC00; border: 3px solid #333; border-radius: 10px; font-weight: bold; cursor: pointer; margin: 5px; }
        #menu { position: absolute; background: white; padding: 25px; border-radius: 20px; text-align: center; border: 5px solid #FFCC00; z-index: 10; width: 80%; max-width: 400px; }
        #lvl-tag { position: absolute; top: 50px; left: 50%; transform: translateX(-50%); color: yellow; font-size: 24px; font-weight: bold; display: none; }
    </style>
</head>
<body>

<div id="game-container" style="background: #87CEEB;">
    <div id="ui">
        <div>Best: <span id="highScore">0</span></div>
        <div id="lvl-name">LEVEL: Forest</div>
        <div>Score: <span id="score">0</span></div>
    </div>

    <div id="lvl-tag">LEVEL UP!</div>

    <canvas id="gameCanvas"></canvas>

    <div id="menu">
        <h1 style="color:#333;">POKEMON ADVENTURE</h1>
        <p>Can you reach Level 3?</p>
        <button class="btn" onclick="startGame(1)">PLAY GAME</button>
    </div>
</div>

<script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const scoreEl = document.getElementById("score");
    const highScoreEl = document.getElementById("highScore");
    const lvlNameEl = document.getElementById("lvl-name");
    const container = document.getElementById("game-container");
    const lvlTag = document.getElementById("lvl-tag");

    canvas.width = 800;
    canvas.height = 400;

    // Images Loading
    const pikaImg = new Image();
    pikaImg.src = 'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/25.png';

    const enemy1 = new Image(); // Meowth
    enemy1.src = 'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/52.png';

    const enemy2 = new Image(); // Geodude
    enemy2.src = 'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/74.png';

    const enemy3 = new Image(); // Gastly
    enemy3.src = 'https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/92.png';

    let gameActive = false;
    let score = 0;
    let level = 1;
    let frame = 0;
    let obstacles = [];
    let speed = 7;
    let highScore = localStorage.getItem("pikaHigh") || 0;
    highScoreEl.innerText = highScore;

    const p1 = { x: 80, y: 300, w: 60, h: 60, dy: 0, gravity: 0.8, jump: -16 };

    function startGame(m) {
        document.getElementById("menu").style.display = "none";
        gameActive = true;
        animate();
    }

    function checkLevel() {
        if (score > 10 && level === 1) {
            level = 2;
            speed = 10;
            container.style.background = "#2c3e50"; 
            showLevelUp("LEVEL 2: DARK CAVE");
        } else if (score > 25 && level === 2) {
            level = 3;
            speed = 13;
            container.style.background = "#4B0082"; 
            showLevelUp("LEVEL 3: GHOST CITY");
        }
    }

    function showLevelUp(txt) {
        lvlNameEl.innerText = txt;
        lvlTag.innerText = "LEVEL UP!";
        lvlTag.style.display = "block";
        setTimeout(() => { lvlTag.style.display = "none"; }, 2000);
    }

    function animate() {
        if (!gameActive) return;
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        frame++;

        // Physics
        p1.dy += p1.gravity;
        p1.y += p1.dy;
        if (p1.y > 290) { p1.y = 290; p1.dy = 0; }

        // Draw Pikachu Image
        ctx.drawImage(pikaImg, p1.x, p1.y, p1.w, p1.h);

        // Obstacles
        if (frame % (Math.max(40, 90 - score)) === 0) {
            let img = enemy1;
            if (level === 2) img = enemy2;
            if (level === 3) img = enemy3;
            obstacles.push({ x: canvas.width, y: 300, w: 50, h: 50, img: img });
        }

        obstacles.forEach((obs, i) => {
            obs.x -= speed;
            
            // Draw Enemy Image
            ctx.drawImage(obs.img, obs.x, obs.y, obs.w, obs.h);
            
            // Collision Detection
            if (p1.x < obs.x + obs.w - 10 && p1.x + p1.w - 10 > obs.x && p1.y < obs.y + obs.h - 10 && p1.y + p1.h - 10 > obs.y) {
                gameOver();
            }

            if (obs.x + obs.w < 0) {
                obstacles.splice(i, 1);
                score++;
                scoreEl.innerText = score;
                checkLevel();
            }
        });

        requestAnimationFrame(animate);
    }

    function gameOver() {
        gameActive = false;
        if (score > highScore) localStorage.setItem("pikaHigh", score);
        alert("GAME OVER! Score: " + score);
        location.reload();
    }

    window.addEventListener("keydown", (e) => {
        if (e.code === "Space" && p1.y >= 290) p1.dy = p1.jump;
    });
    canvas.addEventListener("touchstart", (e) => {
        e.preventDefault();
        if (p1.y >= 290) p1.dy = p1.jump;
    });
</script>
</body>
</html>
