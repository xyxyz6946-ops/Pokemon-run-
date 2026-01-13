<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Pikachu Run - Friends Mode</title>
    <style>
        body { margin: 0; overflow: hidden; background: #222; font-family: 'Arial', sans-serif; touch-action: manipulation; }
        #game-container { position: relative; width: 100vw; height: 100vh; background: linear-gradient(#87CEEB, #E0F6FF); display: flex; justify-content: center; align-items: center; }
        canvas { background: #f7f7f7; box-shadow: 0 0 20px rgba(0,0,0,0.5); border-bottom: 10px solid #4aaa4a; width: 100%; max-width: 800px; height: 400px; }
        #ui { position: absolute; top: 10px; width: 95%; display: flex; justify-content: space-between; color: #333; font-weight: bold; font-size: 18px; z-index: 5; }
        .btn { padding: 12px 20px; background: #FFCC00; border: 3px solid #333; border-radius: 10px; font-weight: bold; cursor: pointer; margin: 5px; }
        #menu { position: absolute; background: white; padding: 25px; border-radius: 20px; text-align: center; border: 5px solid #FFCC00; z-index: 10; width: 80%; max-width: 400px; }
        #controls { position: absolute; bottom: 20px; width: 100%; display: flex; justify-content: space-around; display: none; }
    </style>
</head>
<body>

<div id="game-container">
    <div id="ui">
        <div>High Score: <span id="highScore">0</span></div>
        <div>Current: <span id="score">0</span></div>
    </div>

    <canvas id="gameCanvas"></canvas>

    <div id="menu">
        <h1 style="color:#333;">PIKACHU & FRIENDS</h1>
        <p>High Score Saved Locally!</p>
        <button class="btn" onclick="startGame(1)">SINGLE PLAYER</button>
        <button class="btn" style="background:#44AAFF; color:white;" onclick="startGame(2)">PLAY WITH FRIEND (2P)</button>
    </div>

    <div id="controls">
        <button id="p1-jump" class="btn">P1 JUMP (Space)</button>
        <button id="p2-jump" class="btn" style="background:#FF8800; display:none;">P2 JUMP (Enter)</button>
    </div>
</div>

<script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const scoreEl = document.getElementById("score");
    const highScoreEl = document.getElementById("highScore");
    const menu = document.getElementById("menu");
    const ctrl = document.getElementById("controls");

    canvas.width = 800;
    canvas.height = 400;

    let gameActive = false;
    let mode = 1; // 1 = Single, 2 = Duo
    let score = 0;
    let frame = 0;
    let obstacles = [];
    let highScore = localStorage.getItem("pikaHighScore") || 0;
    highScoreEl.innerText = highScore;

    const p1 = { x: 80, y: 310, w: 40, h: 40, dy: 0, gravity: 0.8, jump: -15, color: "#FFCC00", alive: true };
    const p2 = { x: 40, y: 310, w: 40, h: 40, dy: 0, gravity: 0.8, jump: -15, color: "#FF8800", alive: true };

    function startGame(m) {
        mode = m;
        menu.style.display = "none";
        ctrl.style.display = "flex";
        if(mode === 2) document.getElementById("p2-jump").style.display = "block";
        gameActive = true;
        animate();
    }

    function updatePlayer(p) {
        p.dy += p.gravity;
        p.y += p.dy;
        if (p.y > 310) { p.y = 310; p.dy = 0; }
    }

    function animate() {
        if (!gameActive) return;
        ctx.clearRect(0, 0, canvas.width, canvas.height);
        frame++;

        // Draw Ground
        ctx.fillStyle = "#4aaa4a";
        ctx.fillRect(0, 350, canvas.width, 50);

        // Players
        updatePlayer(p1);
        ctx.fillStyle = p1.color;
        ctx.fillRect(p1.x, p1.y, p1.w, p1.h);

        if (mode === 2) {
            updatePlayer(p2);
            ctx.fillStyle = p2.color;
            ctx.fillRect(p2.x, p2.y, p2.w, p2.h);
        }

        // Obstacles (Enemies)
        if (frame % 90 === 0) obstacles.push({ x: canvas.width, y: 310, w: 30, h: 40 });

        obstacles.forEach((obs, i) => {
            obs.x -= (7 + score/20);
            ctx.fillStyle = "#7030A0"; // Gengar
            ctx.fillRect(obs.x, obs.y, obs.w, obs.h);
            
            // Collision Detection
            [p1, p2].forEach((p, idx) => {
                if (idx === 1 && mode === 1) return;
                if (p.x < obs.x + obs.w && p.x + p.w > obs.x && p.y < obs.y + obs.h && p.y + p.h > obs.y) {
                    gameOver();
                }
            });

            if (obs.x + obs.w < 0) {
                obstacles.splice(i, 1);
                score++;
                scoreEl.innerText = score;
            }
        });

        requestAnimationFrame(animate);
    }

    function gameOver() {
        gameActive = false;
        if (score > highScore) {
            localStorage.setItem("pikaHighScore", score);
        }
        alert("GAME OVER! Score: " + score);
        location.reload();
    }

    // Controls
    window.addEventListener("keydown", (e) => {
        if (e.code === "Space" && p1.y === 310) p1.dy = p1.jump;
        if (e.code === "Enter" && mode === 2 && p2.y === 310) p2.dy = p2.jump;
    });

    document.getElementById("p1-jump").onclick = () => { if(p1.y === 310) p1.dy = p1.jump; };
    document.getElementById("p2-jump").onclick = () => { if(p2.y === 310) p2.dy = p2.jump; };
</script>
</body>
</html>

