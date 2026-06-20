<!DOCTYPE html>
<html>
<head>
    <title>Neon Rider - Car Game</title>
    <style>
        body {
            margin: 0;
            background: radial-gradient(circle, #1a1a2e 0%, #0f0f1b 100%);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            height: 100vh;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            color: #fff;
            overflow: hidden;
        }
        h1 {
            margin-bottom: 10px;
            font-size: 2.5rem;
            text-transform: uppercase;
            letter-spacing: 4px;
            text-shadow: 0 0 10px #00fff2, 0 0 20px #00fff2;
        }
        .container {
            position: relative;
            box-shadow: 0 0 30px rgba(0, 255, 242, 0.2);
            border-radius: 12px;
            overflow: hidden;
        }
        canvas {
            background: #111;
            display: block;
            border: 3px solid #00fff2;
        }
        #restartBtn {
            display: none;
            position: absolute;
            top: 65%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 12px 30px;
            font-size: 1.2rem;
            font-weight: bold;
            text-transform: uppercase;
            background: #ff0055;
            color: white;
            border: none;
            border-radius: 30px;
            cursor: pointer;
            box-shadow: 0 0 15px #ff0055;
            transition: 0.2s ease;
        }
        #restartBtn:hover {
            background: #ff3377;
            box-shadow: 0 0 25px #ff3377;
            transform: translate(-50%, -50%) scale(1.05);
        }
    </style>
</head>
<body>

<h1>Neon Rider</h1>
<div class="container">
    <canvas id="gameCanvas" width="699" height="500"></canvas>
    <button id="restartBtn" onclick="resetGame()">Drive Again</button>
</div>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");
const restartBtn = document.getElementById("restartBtn");

// Game State
let score = 0;
let gameOver = false;
let roadOffset = 0;

// Car properties
let carX = 325;
let carY = 380;
const carWidth = 45;
const carHeight = 75;
const speed = 7;

// Obstacle properties
let obstacleX = Math.random() * (canvas.width - 45);
let obstacleY = -80;
const obstacleWidth = 45;
const obstacleHeight = 75;
let obstacleSpeed = 5;

// Track keyboard inputs
const keys = {};
window.addEventListener("keydown", e => keys[e.key] = true);
window.addEventListener("keyup", e => keys[e.key] = false);

function checkCollision(rect1X, rect1Y, rect1W, rect1H, rect2X, rect2Y, rect2W, rect2H) {
    return rect1X < rect2X + rect2W &&
           rect1X + rect1W > rect2X &&
           rect1Y < rect2Y + rect2H &&
           rect1Y + rect1H > rect2Y;
}

function resetGame() {
    score = 0;
    gameOver = false;
    carX = 325;
    carY = 380;
    obstacleX = Math.random() * (canvas.width - 45);
    obstacleY = -80;
    obstacleSpeed = 5;
    restartBtn.style.display = "none";
    gameLoop();
}

// Main Game Loop
function gameLoop() {
    if (gameOver) {
        // Draw modern Game Over screen overlay
        ctx.fillStyle = "rgba(15, 15, 27, 0.85)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        ctx.fillStyle = "#ff0055";
        ctx.font = "bold 50px Arial";
        ctx.textAlign = "center";
        ctx.shadowColor = "#ff0055";
        ctx.shadowBlur = 15;
        ctx.fillText("CRASHED!", canvas.width / 2, canvas.height / 2 - 20);

        ctx.fillStyle = "#fff";
        ctx.font = "22px Arial";
        ctx.shadowBlur = 0;
        ctx.fillText("Final Score: " + score, canvas.width / 2, canvas.height / 2 + 30);
        
        restartBtn.style.display = "block";
        return;
    }

    // 1. Controls & Invisible Walls
    if ((keys["ArrowLeft"] || keys["a"]) && carX > 40) carX -= speed;
    if ((keys["ArrowRight"] || keys["d"]) && carX < canvas.width - carWidth - 40) carX += speed;
    if ((keys["ArrowUp"] || keys["w"]) && carY > 0) carY -= speed;
    if ((keys["ArrowDown"] || keys["s"]) && carY < canvas.height - carHeight) carY += speed;

    // 2. Road Environment Animation
    roadOffset += obstacleSpeed;
    if (roadOffset > 40) roadOffset = 0;

    obstacleY += obstacleSpeed;

    if (obstacleY > canvas.height) {
        obstacleY = -80;
        obstacleX = 40 + Math.random() * (canvas.width - obstacleWidth - 80);
        score += 1;
        obstacleSpeed += 0.4; 
    }

    // 3. Collision Logic
    if (checkCollision(carX, carY, carWidth, carHeight, obstacleX, obstacleY, obstacleWidth, obstacleHeight)) {
        gameOver = true;
    }

    // 4. Graphics Rendering
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.shadowBlur = 0; // Reset glows for track background

    // Draw the highway asphalt
    ctx.fillStyle = "#22222b";
    ctx.fillRect(40, 0, canvas.width - 80, canvas.height);

    // Side shoulders (Neon green guardrails)
    ctx.fillStyle = "#00ff66";
    ctx.fillRect(35, 0, 5, canvas.height);
    ctx.fillRect(canvas.width - 40, 0, 5, canvas.height);

    // Moving center road lines
    ctx.fillStyle = "#fff";
    for (let i = -40; i < canvas.height; i += 40) {
        ctx.fillRect(canvas.width / 2 - 3, i + roadOffset, 6, 20);
    }

    // 5. Draw Player Neon Car
    ctx.shadowColor = "#00fff2";
    ctx.shadowBlur = 12;
    ctx.fillStyle = "#00fff2";
    ctx.fillRect(carX, carY, carWidth, carHeight);
    
    // Windshield detail
    ctx.fillStyle = "#111";
    ctx.fillRect(carX + 5, carY + 15, carWidth - 10, 15);
    
    // Headlights
    ctx.fillStyle = "#fff";
    ctx.fillRect(carX + 4, carY, 6, 4);
    ctx.fillRect(carX + carWidth - 10, carY, 6, 4);

    // 6. Draw Obstacle Neon Car
    ctx.shadowColor = "#ff0055";
    ctx.shadowBlur = 12;
    ctx.fillStyle = "#ff0055";
    ctx.fillRect(obstacleX, obstacleY, obstacleWidth, obstacleHeight);
    
    // Obstacle Windshield detail
    ctx.fillStyle = "#111";
    ctx.fillRect(obstacleX + 5, obstacleY + 45, obstacleWidth - 10, 15);

    // 7. Render Score Interface
    ctx.shadowBlur = 0;
    ctx.fillStyle = "rgba(0, 0, 0, 0.5)";
    ctx.fillRect(20, 15, 140, 40);
    ctx.strokeStyle = "#00fff2";
    ctx.lineWidth = 1;
    ctx.strokeRect(20, 15, 140, 40);
    
    ctx.fillStyle = "#00fff2";
    ctx.font = "bold 18px Arial";
    ctx.textAlign = "left";
    ctx.fillText("SCORE: " + score, 35, 41);

    requestAnimationFrame(gameLoop);
}

// Kickstart deployment
gameLoop();
</script>

</body>
</html>
