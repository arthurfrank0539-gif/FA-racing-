<!DOCTYPE html>
<html>
<head>
    <title>Neon Rider - City Coupe Edition</title>
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
            background-color: #0c081d;
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
let obstacleWidth = 48;
let obstacleHeight = 82;
let obstacleX = 150 + Math.random() * (699 - 300 - obstacleWidth);
let obstacleY = -90;
let currentObstacleSpeed = baseSpeed;

// City Building Profiles
let leftBuildings = [
    { y: -100, width: 75, height: 140, color: "#140f30", lineNeon: "#ff00bb" },
    { y: 100, width: 85, height: 110, color: "#0d1126", lineNeon: "#00fff2" },
    { y: 280, width: 70, height: 160, color: "#1a102f", lineNeon: "#bc00ff" },
    { y: 480, width: 90, height: 130, color: "#0b1024", lineNeon: "#ff0055" }
];

let rightBuildings = [
    { y: -50, width: 80, height: 120, color: "#0d1126", lineNeon: "#00fff2" },
    { y: 140, width: 70, height: 150, color: "#140f30", lineNeon: "#ff00bb" },
    { y: 320, width: 90, height: 110, color: "#0b1024", lineNeon: "#00ff66" },
    { y: 490, width: 75, height: 140, color: "#1a102f", lineNeon: "#ff0055" }
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

// Mouse/Desktop support fallback to prevent locking
leftBtn.addEventListener("mousedown", () => touchLeft = true);
leftBtn.addEventListener("mouseup", () => touchLeft = false);
rightBtn.addEventListener("mousedown", () => touchRight = true);
rightBtn.addEventListener("mouseup", () => touchRight = false);
accelBtn.addEventListener("mousedown", () => touchAccel = true);
accelBtn.addEventListener("mouseup", () => touchAccel = false);
brakeBtn.addEventListener("mousedown", () => touchBrake = true);
brakeBtn.addEventListener("mouseup", () => touchBrake = false);

function checkCollision(r1x, r1y, r1w, r1h, r2x, r2y, r2w, r2h) {
    return r1x < r2x + r2w && r1x + r1w > r2x && r1y < r2y + r2h && r1y + r1h > r2y;
}

function resetGame() {
    score = 0;
    gameOver = false;
    carX = 325;
    carY = 400;
    obstacleX = 150 + Math.random() * (canvas.width - 300 - obstacleWidth);
    obstacleY = -90;
    baseSpeed = 5;
    restartBtn.style.display = "none";
    gameLoop();
}

function drawBuilding(x, y, width, height, bodyColor, trimNeon) {
    ctx.save();
    ctx.fillStyle = bodyColor;
    ctx.fillRect(x, y, width, height);
    
    ctx.fillStyle = trimNeon;
    ctx.shadowColor = trimNeon;
    ctx.shadowBlur = 8;
    ctx.fillRect(x, y, width, 4);

    ctx.shadowBlur = 0;
    ctx.fillStyle = "rgba(255, 255, 255, 0.12)";
    for (let wx = x + 10; wx < x + width - 10; wx += 14) {
        for (let wy = y + 15; wy < y + height - 15; wy += 22) {
            ctx.fillRect(wx, wy, 6, 10);
        }
    }
    ctx.restore();
}

function drawSleekCoupe(x, y, width, height, themeColor, isObstacle) {
    ctx.save();
    
    // Tires
    ctx.fillStyle = "#09090d";
    ctx.fillRect(x - 4, y + 14, 4, 18);
    ctx.fillRect(x + width, y + 14, 4, 18);
    ctx.fillRect(x - 4, y + height - 28, 4, 18);
    ctx.fillRect(x + width, y + height - 28, 4, 18);

    // Body
    ctx.fillStyle = themeColor;
    ctx.shadowColor = themeColor;
    ctx.shadowBlur = 14;
    
    ctx.beginPath();
    ctx.moveTo(x + width / 2, y);
    ctx.quadraticCurveTo(x + width, y + 5, x + width, y + 15);
    ctx.lineTo(x + width, y + height - 12);
    ctx.quadraticCurveTo(x + width, y + height, x + width - 6, y + height);
    ctx.lineTo(x + 6, y + height);
    ctx.quadraticCurveTo(x, y + height, x, y + height - 12);
    ctx.lineTo(x, y + 15);
    ctx.quadraticCurveTo(x, y + 5, x + width / 2, y);
    ctx.closePath();
    ctx.fill();
    ctx.shadowBlur = 0;

    // Windshield Cabin
    ctx.fillStyle = "#030308";
    ctx.fillRect(x + 6, y + 26, width - 12, 26);

    if (!isObstacle) {
        if (touchAccel) {
            ctx.shadowColor = "#ff7700";
            ctx.shadowBlur = 15;
            ctx.fillStyle = "#00ffff";
            ctx.fillRect(x + 8, y + height, 8, 20);
            ctx.fillRect(x + width - 16, y + height, 8, 20);
        }
        ctx.fillStyle = touchBrake ? "#ff4444" : "#990011";
        ctx.fillRect(x + 2, y + height - 4, 8, 4);
        ctx.fillRect(x + width - 10, y + height - 4, 8, 4);
    } else {
        ctx.fillStyle = "#ff0044";
        ctx.fillRect(x + 3, y + height - 4, 7, 4);
        ctx.fillRect(x + width - 10, y + height - 4, 7, 4);
    }
    ctx.restore();
}

function gameLoop() {
    if (gameOver)
