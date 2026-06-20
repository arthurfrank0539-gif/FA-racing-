<!DOCTYPE html>
<html>
<head>
    <title>Neon Rider - Coupe Edition</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
        body {
            margin: 0;
            background: radial-gradient(circle at center, #1b113a 0%, #080511 100%);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            font-family: 'Courier New', Courier, monospace;
            color: #fff;
            overflow: hidden;
            user-select: none;
            -webkit-user-select: none;
        }
        h1 {
            margin: 8px 0;
            font-size: 2.2rem;
            text-transform: uppercase;
            letter-spacing: 8px;
            text-shadow: 0 0 10px #00fff2, 0 0 20px #00fff2;
            font-weight: bold;
        }
        .container {
            position: relative;
            box-shadow: 0 0 45px rgba(0, 255, 242, 0.5);
            border-radius: 16px;
            overflow: hidden;
            width: 92%;
            max-width: 699px;
            background: #000;
        }
        canvas {
            display: block;
            border: 4px solid #00fff2;
            width: 100%;
            height: auto;
        }
        #restartBtn {
            display: none;
            position: absolute;
            top: 55%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 14px 35px;
            font-family: 'Courier New', Courier, monospace;
            font-size: 1.3rem;
            font-weight: bold;
            text-transform: uppercase;
            background: #ff0055;
            color: white;
            border: 3px solid #fff;
            border-radius: 8px;
            cursor: pointer;
            box-shadow: 0 0 20px #ff0055;
            z-index: 10;
        }
        .controls-pad {
            display: flex;
            justify-content: space-between;
            width: 92%;
            max-width: 699px;
            margin-top: 15px;
            margin-bottom: 15px;
            gap: 15px;
        }
        .steering-group, .speed-group {
            display: flex;
            gap: 15px;
        }
        .btn {
            width: 80px;
            height: 80px;
            background: rgba(0, 255, 242, 0.05);
            border: 3px solid #00fff2;
            border-radius: 20px;
            color: #00fff2;
            font-size: 2.2rem;
            font-weight: bold;
            display: flex;
            justify-content: center;
            align-items: center;
            box-shadow: 0 0 15px rgba(0, 255, 242, 0.2);
            touch-action: none;
        }
        .btn-action {
            border-color: #ff0055;
            color: #ff0055;
            box-shadow: 0 0 15px rgba(255, 0, 85, 0.2);
        }
        .btn:active {
            background: rgba(0, 255, 242, 0.4);
            box-shadow: 0 0 30px #00fff2;
            color: #fff;
        }
        .btn-action:active {
            background: rgba(255, 0, 85, 0.4);
            box-shadow: 0 0 30px #ff0055;
            color: #fff;
        }
    </style>
</head>
<body>

<h1>NEON RIDER</h1>
<div class="container">
    <canvas id="gameCanvas" width="699" height="520"></canvas>
    <button id="restartBtn" onclick="resetGame()">REIGNITE ENGINE</button>
</div>

<div class="controls-pad">
    <div class="steering-group">
        <div class="btn" id="leftBtn">◀</div>
        <div class="btn" id="rightBtn">▶</div>
    </div>
    <div class="speed-group">
        <div class="btn btn-action" id="brakeBtn">B</div>
        <div class="btn btn-action" id="accelBtn">A</div>
    </div>
</div>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");
const restartBtn = document.getElementById("restartBtn");

let score = 0;
let gameOver = false;
let roadOffset = 0;
let baseSpeed = 5;

// Player Setup
let carX = 325;
let carY = 400;
const carWidth = 48;
const carHeight = 82;
const steerSpeed = 8.5;

// Obstacle Setup
let obstacleX = Math.random() * (canvas.width - 48);
let obstacleY = -90;
const obstacleWidth = 48;
const obstacleHeight = 82;
let currentObstacleSpeed = baseSpeed;

// Environment Setup
let streetlights = [
    { y: 0 }, { y: 130 }, { y: 260 }, { y: 390 }, { y: 520 }
];

let touchLeft = false;
let touchRight = false;
let touchAccel = false;
let touchBrake = false;

const leftBtn = document.getElementById("leftBtn");
const rightBtn = document.getElementById("rightBtn");
const accelBtn = document.getElementById("accelBtn");
const brakeBtn = document.getElementById("brakeBtn");

leftBtn.addEventListener("touchstart", (e) => { e.preventDefault(); touchLeft = true; });
leftBtn.addEventListener("touchend", () => touchLeft = false);
rightBtn.addEventListener("touchstart", (e) => { e.preventDefault(); touchRight = true; });
rightBtn.addEventListener("touchend", () => touchRight = false);
accelBtn.addEventListener("touchstart", (e) => { e.preventDefault(); touchAccel = true; });
accelBtn.addEventListener("touchend", () => touchAccel = false);
brakeBtn.addEventListener("touchstart", (e) => { e.preventDefault(); touchBrake = true; });
brakeBtn.addEventListener("touchend", () => touchBrake = false);

function checkCollision(r1x, r1y, r1w, r1h, r2x, r2y, r2w, r2h) {
    return r1x < r2x + r2w && r1x + r1w > r2x && r1y < r2y + r2h && r1y + r1h > r2y;
}

function resetGame() {
    score = 0;
    gameOver = false;
    carX = 325;
    carY = 400;
    obstacleX = 90 + Math.random() * (canvas.width - 180 - obstacleWidth);
    obstacleY = -90;
    baseSpeed = 5;
    restartBtn.style.display = "none";
    gameLoop();
}

// Drawing a sleek, curved sports coupe
function drawSleekCoupe(x, y, width, height, themeColor, isObstacle) {
    ctx.save();
    
    // 1. Wide Performance Tires
    ctx.fillStyle = "#09090d";
    ctx.fillRect(x - 4, y + 14, 4, 18);
    ctx.fillRect(x + width, y + 14, 4, 18);
    ctx.fillRect(x - 4, y + height - 28, 4, 18);
    ctx.fillRect(x + width, y + height - 28, 4, 18);

    // 2. Main Aerodynamic Coupe Chassis (Curved Edges)
    ctx.fillStyle = themeColor;
    ctx.shadowColor = themeColor;
    ctx.shadowBlur = 18;
    
    ctx.beginPath();
    ctx.moveTo(x + width / 2, y); // Front Center Nose
    ctx.quadraticCurveTo(x + width, y + 5, x + width, y + 15); // Front Right Fender
    ctx.lineTo(x + width, y + height - 12); // Right Side Flank
    ctx.quadraticCurveTo(x + width, y + height, x + width - 6, y + height); // Rear Right
    ctx.lineTo(x + x + 6, y + height); // Rear Trunk Base
    ctx.quadraticCurveTo(x, y + height, x, y + height - 12); // Rear Left
    ctx.lineTo(x, y + 15); // Left Side Flank
    ctx.quadraticCurveTo(x, y + 5, x + width / 2, y); // Front Left Fender
    ctx.closePath();
    ctx.fill();
    ctx.shadowBlur = 0;

    // 3. Carbon Fiber Hood / Trim Details
    ctx.fillStyle = "rgba(0, 0, 0, 0.3)";
    ctx.beginPath();
    ctx.moveTo(x + 10, y + 8);
    ctx.lineTo(x + width - 10, y + 8);
    ctx.lineTo(x + width - 6, y + 26);
    ctx.lineTo(x + 6, y + 26);
    ctx.closePath();
    ctx.fill();

    // 4. Sloped Windshield & Coupe Cabin Glass
    ctx.fillStyle = "#030308";
    ctx.beginPath();
    ctx.moveTo(x + 8, y + 28);
    ctx.lineTo(x + width - 8, y + 28);
    ctx.lineTo(x + width - 4, y + 54);
    ctx.lineTo(x + 4, y + 54);
    ctx.closePath();
    ctx.fill();

    // Solar Reflection Sheen Across Glass
    ctx.fillStyle = isObstacle ? "rgba(255, 0, 85, 0.25)" : "rgba(0, 255, 242, 0.4)";
    ctx.beginPath();
    ctx.moveTo(x + 12, y + 28);
    ctx.lineTo(x + 18, y + 28);
    ctx.lineTo(x + 10, y + 54);
    ctx.lineTo(x + 4, y + 54);
    ctx.closePath();
    ctx.fill();

    // 5. Quad Rear Exhaust Outlets
    ctx.fillStyle = "#444";
    ctx.fillRect(x + 6, y + height, 6, 3);
    ctx.fillRect(x + 14, y + height, 6, 3);
    ctx.fillRect(x + width - 12, y + height, 6, 3);
    ctx.fillRect(x + width - 20, y + height, 6, 3);

    if (!isObstacle) {
        // Dynamic Exhaust Flame on Gas (A Button)
        if (touchAccel) {
            ctx.shadowColor = "#ff7700";
            ctx.shadowBlur = 20;
            let flameGrad = ctx.createLinearGradient(0, y + height, 0, y + height + 28);
            flameGrad.addColorStop(0, "#ffffff");
            flameGrad.addColorStop(0.2, "#00ffff");
            flameGrad.addColorStop(1, "rgba(0, 150, 255, 0)");
            ctx.fillStyle = flameGrad;
            ctx.fillRect(x + 8, y + height + 2, 10, 25);
            ctx.fillRect(x + width - 18, y + height + 2, 10, 25);
        }

        // Active Rear Brake Lights Glow (B Button)
        ctx.shadowColor = "#ff0044";
        ctx.shadowBlur = touchBrake ? 25 : 5;
        ctx.fillStyle = touchBrake ? "#ff4444" : "#990011";
        ctx.fillRect(x + 2, y + height - 4, 8, 4);
        ctx.fillRect(x + width - 10, y + height - 4, 8, 4);

        // Headlight Beams Projecting Forward
        ctx.save();
        let beamGrad = ctx.createLinearGradient(0, y,
