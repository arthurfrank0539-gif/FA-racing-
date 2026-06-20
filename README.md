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

// City Building Arrays (Left & Right districts)
let leftBuildings = [
    { y: -100, width: 65, height: 110, color: "#16113a", lineNeon: "#ff00bb" },
    { y: 60, width: 55, height: 140, color: "#11162e", lineNeon: "#00fff2" },
    { y: 250, width: 70, height: 95, color: "#1d1233", lineNeon: "#bc00ff" },
    { y: 400, width: 60, height: 150, color: "#0f152b", lineNeon: "#ff0055" },
    { y: 580, width: 65, height: 120, color: "#16113a", lineNeon: "#00ff66" }
];

let rightBuildings = [
    { y: -60, width: 60, height: 130, color: "#11162e", lineNeon: "#00fff2" },
    { y: 110, width: 70, height: 100, color: "#16113a", lineNeon: "#ff00bb" },
    { y: 260, width: 55, height: 160, color: "#0f152b", lineNeon: "#00ff66" },
    { y: 460, width: 65, height: 115, color: "#1d1233", lineNeon: "#ff0055" },
    { y: 620, width: 60, height: 140, color: "#11162e", lineNeon: "#bc00ff" }
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
    obstacleX = 145 + Math.random() * (canvas.width - 290 - obstacleWidth);
    obstacleY = -90;
    baseSpeed = 5;
    restartBtn.style.display = "none";
    gameLoop();
}

// Function to draw realistic side cityscape structures
function drawBuilding(x, y, width, height, bodyColor, trimNeon, isRightSide) {
    ctx.save();
    
    // Draw building core silhouette
    ctx.fillStyle = bodyColor;
    ctx.fillRect(x, y, width, height);
    
    // Light accents on roofs
    ctx.fillStyle = trimNeon;
    ctx.shadowColor = trimNeon;
    ctx.shadowBlur = 8;
    ctx.fillRect(x, y, width, 4);

    // Glowing Matrix Windows inside buildings
    ctx.shadowBlur = 0;
    ctx.fillStyle = "rgba(255, 255, 255, 0.15)";
    for (let wx = x + 8; wx < x + width - 6; wx += 14) {
        for (let wy = y + 15; wy < y + height - 10; wy += 20) {
            // Randomize window sparkle states slightly
            if ((wx + wy) % 3 === 0) {
                ctx.fillStyle = trimNeon + "44"; // Matching neon translucency
            } else {
                ctx.fillStyle = "rgba(255, 255, 255, 0.1)";
            }
            ctx.fillRect(wx, wy, 6, 10);
        }
    }
    ctx.restore();
}

function drawSleekCoupe(x, y, width, height, themeColor, isObstacle) {
    ctx.save();
    ctx.fillStyle = "#09090d";
    ctx.fillRect(x - 4, y + 14, 4, 18);
    ctx.fillRect(x + width, y + 14, 4, 18);
    ctx.fillRect(x - 4, y + height - 28, 4, 18);
    ctx.fillRect(x + width, y + height - 28, 4, 18);

    ctx.fillStyle = themeColor;
    ctx.shadowColor = themeColor;
    ctx.shadowBlur = 18;
    
    ctx.beginPath();
    ctx.moveTo(x + width / 2, y);
    ctx.quadraticCurveTo(x + width, y + 5, x + width, y + 15);
    ctx.lineTo(x + width, y + height - 12);
    ctx.quadraticCurveTo(x + width, y + height, x + width - 6, y + height);
    ctx.lineTo(x + x + 6, y + height);
    ctx.quadraticCurveTo(x, y + height, x, y + height - 12);
    ctx.lineTo(x, y + 15);
    ctx.quadraticCurveTo(x, y + 5, x + width / 2, y);
    ctx.closePath();
    ctx.fill();
    ctx.shadowBlur = 0;

    ctx.fillStyle = "rgba(0, 0, 0, 0.3)";
    ctx.beginPath();
    ctx.moveTo(x + 10, y + 8); ctx.lineTo(x + width - 10, y + 8); ctx.lineTo(x + width - 6, y + 26); ctx.lineTo(x + 6, y + 26); ctx.closePath(); ctx.fill();

    ctx.fillStyle = "#030308";
    ctx.beginPath();
    ctx.moveTo(x + 8, y + 28); ctx.lineTo(x + width - 8, y + 28); ctx.lineTo(x + width - 4, y + 54); ctx.lineTo(x + 4, y + 54); ctx.closePath(); ctx.fill();

    ctx.fillStyle = isObstacle ? "rgba(255, 0, 85, 0.25)" : "rgba(0, 255, 242, 0.4)";
    ctx.beginPath();
    ctx.moveTo(x + 12, y + 28); ctx.lineTo(x + 18, y + 28); ctx.lineTo(x + 10, y + 54); ctx.lineTo(x + 4, y + 54); ctx.closePath(); ctx.fill();

    if (!isObstacle) {
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

        ctx.shadowColor = "#ff0044";
        ctx.shadowBlur = touchBrake ? 25 : 5;
        ctx.fillStyle = touchBrake ? "#ff4444" : "#990011";
        ctx.fillRect(x + 2, y + height - 4, 8, 4);
        ctx.fillRect(x + width - 10, y + height - 4, 8, 4);

        ctx.save();
        let beamGrad = ctx.createLinearGradient(0, y, 0, y - 120);
        beamGrad.addColorStop(0, "rgba(0, 255, 242, 0.45)");
        beamGrad.addColorStop(1, "rgba(0, 255, 242, 0)");
        ctx.fillStyle = beamGrad;
        ctx.beginPath(); ctx.moveTo(x + 4, y + 8); ctx.lineTo(x - 30, y - 120); ctx.lineTo(x + 25, y - 120); ctx.fill();
        ctx.beginPath(); ctx.moveTo(x + width - 4, y + 8); ctx.lineTo(x + width - 25, y - 120); ctx.lineTo(x + width + 30, y - 120); ctx.fill();
        ctx.restore();
    } else {
        ctx.fillStyle = "#ff0044";
        ctx.fillRect(x + 3, y + height - 4, 7, 4);
        ctx.fillRect(x + width - 10, y + height - 4, 7, 4);
    }
    ctx.restore();
}

function gameLoop() {
    if (gameOver) {
        ctx.fillStyle = "rgba(5, 3, 10, 0.92)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);

        ctx.fillStyle = "#ff0055";
        ctx.font = "bold 44px 'Courier New'";
        ctx.textAlign = "center";
        ctx.shadowColor = "#ff0055";
        ctx.shadowBlur = 25;
        ctx.fillText("SYSTEM CRASHED", canvas.width / 2, canvas.height / 2 - 30);

        ctx.fillStyle = "#fff";
        ctx.font = "bold 22px 'Courier New'";
        ctx.shadowBlur = 0;
        ctx.fillText("TOTAL COUPE DRIFTS: " + score, canvas.width / 2, canvas.height / 2 + 25);
        
        restartBtn.style.display = "block";
        return;
    }

    // Steering restriction within adjusted road lines
    if (touchLeft && carX > 145) carX -= steerSpeed;
    if (touchRight && carX < canvas.width - carWidth - 145) carX += steerSpeed;

    if (touchAccel) {
        currentObstacleSpeed = baseSpeed * 1.9;
    } else if (touchBrake) {
        currentObstacleSpeed = baseSpeed * 0.4;
    } else {
        currentObstacleSpeed = baseSpeed;
    }

    // Advance Pavement Offset
    roadOffset += currentObstacleSpeed;
    if (roadOffset > 50) roadOffset = 0;

    obstacleY += currentObstacleSpeed;

    // Reset obstacle car on successful pass
    if (obstacleY > canvas.height) {
        obstacleY = -90;
        obstacleX = 145 + Math.random() * (canvas.width - obstacleWidth - 290);
        score += 1;
        baseSpeed += 0.3;
    }

    // Move Left City Buildings
    leftBuildings.forEach(b => {
        b.y += currentObstacleSpeed;
        if (b.y > canvas.height) {
            b.y = -b.height - (Math.random() * 40);
        }
    });

    // Move Right City Buildings
    rightBuildings.forEach(b => {
        b.y += currentObstacleSpeed;
        if (b.y > canvas.height) {
            b.y = -b.height - (Math.random() * 40);
        }
    });

    if (checkCollision(carX, carY, carWidth, carHeight, obstacleX, obstacleY, obstacleWidth, obstacleHeight)) {
        gameOver = true;
    }

    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // 1. RENDER BUILDINGS: Left side district
    leftBuildings.forEach(b => {
        drawBuilding(0, b.y, b.width, b.height, b.color, b.lineNeon, false);
    });

    // 2. RENDER BUILDINGS: Right side district
    rightBuildings.forEach(b => {
        drawBuilding(canvas.width - b.width, b.y, b.width, b.height, b.color, b.lineNeon, true);
    });

    // 3. ENVIRONMENT: Central Highway Track Asphalt
    ctx.fillStyle = "#12121a";
    ctx.fillRect(140, 0, canvas.width - 280, canvas.height);

    // 4. ENVIRONMENT: Dual Layer Glowing Guardrails
    ctx.shadowColor = "#ff00ff";
    ctx.shadowBlur = 10;
    ctx.fillStyle = "#ff00ff";
    ctx.fillRect(132, 0, 4, canvas.height);
    ctx.fillRect(canvas.width - 136, 0, 4, canvas.height);

    ctx.shadowColor = "#00ff66";
    ctx.shadowBlur = 10;
    ctx.fillStyle = "#00ff66";
    ctx.fillRect(136, 0, 4, canvas.height);
    ctx.fillRect(canvas.width - 140, 0, 4, canvas.height);
    ctx.shadowBlur = 0;

    // 5. ROAD LINES: Moving Center White Dashes
    ctx.fillStyle = "rgba(255, 255, 255, 0.65)";
    for (let i = -50; i < canvas.height; i += 60) {
        ctx.fillRect(canvas.width / 2 - 2, i + roadOffset, 4, 30);
    }

    // 6. CARS: Render Player and Rivals
    drawSleekCoupe(carX, carY, carWidth, carHeight, "#00fff2", false);
    drawSleekCoupe(obstacleX, obstacleY, obstacleWidth, obstacleHeight, "#ff0055", true);

    // 7. HUD Interface Panel
    ctx.fillStyle = "rgba(4, 2, 10, 0.85)";
    ctx.fillRect(25, 20, 160, 45);
    ctx.strokeStyle = "#00fff2";
    ctx.lineWidth = 2;
    ctx.strokeRect(25, 20, 160, 45);
    
    ctx.fillStyle = "#00fff2";
    ctx.font = "bold 15px 'Courier New'";
    ctx.textAlign = "left";
    ctx.fillText("SCORE: " + score, 42, 48);

    requestAnimationFrame(gameLoop);
}

gameLoop();
</script>

</body>
</html>
