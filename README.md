<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Neon Sprint GP</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }
        body {
            background-color: #0a0b10;
            color: #ffffff;
            font-family: sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            overflow: hidden;
            user-select: none;
            -webkit-user-select: none;
        }
        #stageTitle {
            font-size: 1.2rem;
            font-weight: bold;
            color: #00fff2;
            letter-spacing: 2px;
            margin-bottom: 5px;
            text-transform: uppercase;
        }
        .game-container {
            position: relative;
            width: 95%;
            max-width: 400px;
            background: #111;
            border: 2px solid #00fff2;
            border-radius: 12px;
            overflow: hidden;
        }
        canvas {
            display: block;
            width: 100%;
            height: auto;
            background: #0d0e15;
        }
        #nextBtn {
            display: none;
            position: absolute;
            top: 60%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 12px 24px;
            background: #00fff2;
            color: #000;
            border: none;
            font-weight: bold;
            border-radius: 20px;
            cursor: pointer;
            box-shadow: 0 0 15px #00fff2;
            z-index: 10;
        }
        .controls {
            display: flex;
            justify-content: space-between;
            width: 95%;
            max-width: 400px;
            margin-top: 15px;
            padding: 0 5px;
        }
        .btn-dir, .btn-speed {
            display: flex;
            gap: 10px;
        }
        .pad-btn {
            width: 65px;
            height: 65px;
            background: rgba(0, 255, 242, 0.1);
            border: 2px solid #00fff2;
            color: #00fff2;
            border-radius: 15px;
            font-size: 1.5rem;
            font-weight: bold;
            display: flex;
            align-items: center;
            justify-content: center;
            touch-action: none;
            cursor: pointer;
        }
        .pad-btn:active {
            background: #00fff2;
            color: #000;
        }
        .pad-btn.red {
            border-color: #ff2d55;
            color: #ff2d55;
            background: rgba(255, 45, 85, 0.1);
        }
        .pad-btn.red:active {
            background: #ff2d55;
            color: #000;
        }
    </style>
</head>
<body>

<div id="stageTitle">Stage 1: Sector Zero</div>

<div class="game-container">
    <canvas id="gameCanvas" width="400" height="450"></canvas>
    <button id="nextBtn" onclick="handleNextStage()">Next Stage</button>
</div>

<div class="controls">
    <div class="btn-dir">
        <div class="pad-btn" id="btnLeft">←</div>
        <div class="pad-btn" id="btnRight">→</div>
    </div>
    <div class="btn-speed">
        <div class="pad-btn red" id="btnBrake">↓</div>
        <div class="pad-btn red" id="btnAccel">↑</div>
    </div>
</div>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");
const nextBtn = document.getElementById("nextBtn");
const stageTitle = document.getElementById("stageTitle");

const configs = [
    { name: "Stage 1: Sector Zero", dist: 5000, speed: 5, color: "#00fff2" },
    { name: "Stage 2: Grid City", dist: 7000, speed: 6, color: "#ff00bb" },
    { name: "Stage 3: Cyber Basin", dist: 9000, speed: 7, color: "#bc00ff" }
];

let currentLevel = 0;
let isGameOver = false;
let statusText = "";
let score = 0;
let distanceTraveled = 0;
let trackOffset = 0;

let pX = 185;
const carW = 30;
const carH = 55;

let racers = [
    { name: "YOU", progress: 0, x: 185, isPlayer: true, c1: "#00c6ff", c2: "#0072ff" },
    { name: "VIOLET", progress: 140, x: 120, isPlayer: false, c1: "#7b1fa2", c2: "#e040fb" },
    { name: "ORANGE", progress: 70, x: 250, isPlayer: false, c1: "#f57c00", c2: "#ffb74d" }
];

let itemX = 200;
let itemY = -50;

// Controls tracking
let keys = { left: false, right: false, up: false, down: false };

function setupInput(elementId, keyProp) {
    const btn = document.getElementById(elementId);
    if (!btn) return;
    const start = (e) => { e.preventDefault(); keys[keyProp] = true; };
    const end = (e) => { e.preventDefault(); keys[keyProp] = false; };
    btn.addEventListener("touchstart", start);
    btn.addEventListener("touchend", end);
    btn.addEventListener("mousedown", start);
    btn.addEventListener("mouseup", end);
    btn.addEventListener("mouseleave", end);
}

setupInput("btnLeft", "left");
setupInput("btnRight", "right");
setupInput("btnAccel", "up");
setupInput("btnBrake", "down");

window.addEventListener("keydown", (e) => {
    if (e.key === "ArrowLeft" || e.key === "a") keys.left = true;
    if (e.key === "ArrowRight" || e.key === "d") keys.right = true;
    if (e.key === "ArrowUp" || e.key === "w") keys.up = true;
    if (e.key === "ArrowDown" || e.key === "s") keys.down = true;
});

window.addEventListener("keyup", (e) => {
    if (e.key === "ArrowLeft" || e.key === "a") keys.left = false;
    if (e.key === "ArrowRight" || e.key === "d") keys.right = false;
    if (e.key === "ArrowUp" || e.key === "w") keys.up = false;
    if (e.key === "ArrowDown" || e.key === "s") keys.down = false;
});

function initLevel(idx) {
    if (idx >= configs.length) {
        statusText = "GRAND GP CHAMPION!";
        isGameOver = true;
        nextBtn.style.display = "none";
        return;
    }
    currentLevel = idx;
    stageTitle.textContent = configs[currentLevel].name;
    isGameOver = false;
    statusText = "";
    distanceTraveled = 0;
    pX = 185;
    
    racers[0].progress = 0;
    racers[1].progress = 140;
    racers[2].progress = 70;
    
    itemY = -100;
    nextBtn.style.display = "none";
}

function handleNextStage() {
    if (statusText.includes("CLEAR")) {
        initLevel(currentLevel + 1);
    } else {
        initLevel(currentLevel);
    }
}

function update() {
    if (isGameOver) return;

    let cfg = configs[currentLevel];
    let currentSpeed = cfg.speed;

    if (keys.up) currentSpeed *= 1.5;
    if (keys.down) currentSpeed *= 0.4;

    if (keys.left && pX > 105) pX -= 4;
    if (keys.right && pX < canvas.width - 105 - carW) pX += 4;

    distanceTraveled += currentSpeed;
    racers[0].progress = distanceTraveled;
    racers[0].x = pX;

    // AI movement behavior
    for (let i = 1; i < racers.length; i++) {
        let ai = racers[i];
        let diff = distanceTraveled - ai.progress;
        let aiSpeed = cfg.speed * (i === 1 ? 0.96 : 1.02);
        
        if (diff > 100) aiSpeed += 1.2;
        if (diff < -150) aiSpeed -= 0.8;
        
        ai.progress += aiSpeed;
    }

    trackOffset = (trackOffset + currentSpeed) % 40;

    // Crystal Item loop logic
    itemY += currentSpeed;
    let pRenderY = canvas.height - 90;
    if (itemY > canvas.height) {
        itemY = -60;
        itemX = 110 + Math.random() * (canvas.width - 220);
    }

    if (pX < itemX + 16 && pX + carW > itemX && pRenderY < itemY + 16 && pRenderY + carH > itemY) {
        score++;
        itemY = -200;
        itemX = 110 + Math.random() * (canvas.width - 220);
    }

    // Goal calculation
    if (distanceTraveled >= cfg.dist) {
        isGameOver = true;
        let finalStandings = [...racers].sort((a, b) => b.progress - a.progress);
        let rank = finalStandings.findIndex(r => r.isPlayer) + 1;
        
        if (rank <= 2) {
            statusText = "STAGE CLEAR! RANK: " + rank;
            nextBtn.textContent = "Next Stage";
        } else {
            statusText = "FAILED! RANK: " + rank;
            nextBtn.textContent = "Retry";
        }
        nextBtn.style.display = "block";
    }
}

function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    let cfg = configs[currentLevel];

    // Track rendering 
    ctx.fillStyle = "#161824";
    ctx.fillRect(100, 0, canvas.width - 200, canvas.height);

    ctx.fillStyle = cfg.color;
    ctx.fillRect(98, 0, 3, canvas.height);
    ctx.fillRect(canvas.width - 101, 0, 3, canvas.height);

    // Center divider lines
    ctx.fillStyle = "rgba(255,255,255,0.2)";
    for (let y = -40; y < canvas.height; y += 40) {
        ctx.fillRect(canvas.width / 2 - 2, y + trackOffset, 4, 20);
    }

    // Finish line overlay
    let finishY = canvas.height - 90 - (cfg.dist - distanceTraveled);
    if (finishY > -20 && finishY < canvas.height) {
        ctx.fillStyle = "#ffcc00";
        ctx.fillRect(100, finishY, canvas.width - 200, 12);
    }

    // Draw Crystal
    if (itemY > -20 && itemY < canvas.height) {
        ctx.fillStyle = "#ffff00";
        ctx.beginPath();
        ctx.arc(itemX + 8, itemY + 8, 8, 0, Math.PI * 2);
        ctx.fill();
    }

    // Render Vehicles
    racers.forEach(r => {
        let renderY = canvas.height - 90 - (r.progress - distanceTraveled);
        if (renderY > -60 && renderY < canvas.height + 60) {
            ctx.fillStyle = r.c1;
            ctx.fillRect(r.x, renderY, carW, carH);
            
            ctx.fillStyle = "#0a0a0f";
            ctx.fillRect(r.x + 3, renderY + 10, carW - 6, 12);

            ctx.fillStyle = "#ffffff";
            ctx.font = "bold 9px sans-serif";
            ctx.fillText(r.name, r.x, renderY - 6);
        }
    });

    // Score overlay panel
    ctx.fillStyle = "rgba(10,12,20,0.85)";
    ctx.fillRect(10, 10, 110, 55);
    ctx.strokeStyle = cfg.color;
    ctx.strokeRect(10, 10, 110, 55);

    ctx.fillStyle = "#fff";
    ctx.font = "11px sans-serif";
    let rem = Math.max(0, Math.floor((cfg.dist - distanceTraveled) / 10));
    ctx.fillText("GOAL: " + rem + "m", 16, 26);
    ctx.fillStyle = "#ffcc00";
    ctx.fillText("CRYSTALS: " + score, 16, 46);

    if (isGameOver) {
        ctx.fillStyle = "rgba(0, 0, 0, 0.85)";
        ctx.fillRect(0, 0, canvas.width, canvas.height);
        ctx.fillStyle = cfg.color;
        ctx.font = "bold 20px sans-serif";
        ctx.textAlign = "center";
        ctx.fillText(statusText, canvas.width / 2, canvas.height / 2 - 10);
        ctx.textAlign = "left";
    }
}

function loop() {
    update();
    draw();
    requestAnimationFrame(loop);
}

initLevel(0);
loop();
</script>

</body>
</html>
