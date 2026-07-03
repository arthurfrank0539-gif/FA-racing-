<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Neon GP - Time Sprint</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
        * {
            box-sizing: border-box;
        }
        body {
            margin: 0;
            padding: 0;
            background: #070810;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
            color: #ffffff;
            overflow: hidden;
            user-select: none;
            -webkit-user-select: none;
        }
        .header-title {
            margin: 8px 0;
            font-size: 1.4rem;
            font-weight: 700;
            text-transform: uppercase;
            letter-spacing: 4px;
            color: #00fff2;
            text-shadow: 0 0 10px rgba(0, 255, 242, 0.3);
            text-align: center;
        }
        .container {
            position: relative;
            border-radius: 16px;
            overflow: hidden;
            width: 92%;
            max-width: 420px;
            box-shadow: 0 20px 50px rgba(0, 0, 0, 0.7);
            border: 2px solid #00fff2;
            background: #090a14;
        }
        canvas {
            display: block;
            width: 100%;
            height: auto;
            background: #090a14;
        }
        #restartBtn {
            display: none;
            position: absolute;
            top: 60%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 14px 32px;
            font-size: 1rem;
            font-weight: 600;
            letter-spacing: 1px;
            background: #00fff2;
            color: #000000;
            border: none;
            border-radius: 30px;
            cursor: pointer;
            box-shadow: 0 0 20px rgba(0, 255, 242, 0.5);
            z-index: 20;
        }
        .controls-pad {
            display: flex;
            justify-content: space-between;
            width: 92%;
            max-width: 420px;
            margin-top: 15px;
            margin-bottom: 10px;
            padding: 0 5px;
        }
        .btn-group {
            display: flex;
            gap: 12px;
        }
        .btn {
            width: 60px;
            height: 60px;
            background: rgba(9, 10, 20, 0.8);
            border: 2px solid #00fff2;
            border-radius: 16px;
            color: #00fff2;
            font-size: 1.4rem;
            font-weight: bold;
            display: flex;
            justify-content: center;
            align-items: center;
            touch-action: none;
            box-shadow: 0 4px 10px rgba(0, 255, 242, 0.1);
        }
        .btn:active {
            background: #00fff2;
            color: #090a14;
            box-shadow: 0 0 15px rgba(0, 255, 242, 0.6);
        }
        .btn-action {
            color: #ff2d55;
            border-color: #ff2d55;
            box-shadow: 0 4px 10px rgba(255, 45, 85, 0.1);
        }
        .btn-action:active {
            background: #ff2d55;
            color: #090a14;
            box-shadow: 0 0 15px rgba(255, 45, 85, 0.6);
        }
    </style>
</head>
<body>

<div class="header-title">Neon Sprint</div>

<div class="container">
    <canvas id="gameCanvas" width="500" height="450"></canvas>
    <button id="restartBtn">Race Again</button>
</div>

<div class="controls-pad">
    <div class="btn-group">
        <div class="btn" id="leftBtn">←</div>
        <div class="btn" id="rightBtn">→</div>
    </div>
    <div class="btn-group">
        <div class="btn btn-action" id="brakeBtn">↓</div>
        <div class="btn btn-action" id="accelBtn">↑</div>
    </div>
</div>

<script>
function startNeonRider() {
    const canvas = document.getElementById("gameCanvas");
    if (!canvas) return;
    const ctx = canvas.getContext("2d");
    const restartBtn = document.getElementById("restartBtn");

    const TOTAL_DISTANCE = 10000; 
    let score = 0;
    let gameOver = false;
    let finishState = ""; 
    let roadOffset = 0;
    let baseSpeed = 5;
    let currentSpeed = baseSpeed;
    let playerProgress = 0;

    // Player Car Setup
    let carX = 232;
    let carY = 340;
    const carW = 34;
    const carH = 62;

    // Background Simulating Racers (Hidden from screen rendering)
    let racers = [
        { id: "Player", name: "YOU", progress: 0, speedModifier: 1.0, isPlayer: true },
        { id: "AI1", name: "VIOLET", progress: 40, speedModifier: 0.98 },
        { id: "AI2", name: "ORANGE", progress: 20, speedModifier: 1.01 },
        { id: "AI3", name: "GREEN", progress: 60, speedModifier: 0.95 }
    ];

    // Standard road obstacles (Neutral traffic blocks)
    let obstacles = [
        { x: 190, y: -150, w: 34, h: 62, speed: 2 },
        { x: 280, y: -450, w: 34, h: 62, speed: 1.5 }
    ];

    let coinX = 140 + Math.random() * (220 - 18);
    let coinY = -300;
    const coinSize = 18;

    let buildings = [
        { leftSide: true, xOffset: 8, y: 0, w: 85, h: 200, accentColor: "#00fff2" },
        { leftSide: true, xOffset: 18, y: 250, w: 75, h: 150, accentColor: "#ff00bb" },
        { leftSide: false, xOffset: 8, y: 50, w: 85, h: 220, accentColor: "#bc00ff" },
        { leftSide: false, xOffset: 20, y: 300, w: 70, h: 160, accentColor: "#00ff66" }
    ];

    let touchLeft = false;
    let touchRight = false;
    let touchAccel = false;
    let touchBrake = false;
    let audioCtx = null;

    function initAudio() {
        try {
            if (!audioCtx) {
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            }
            if (audioCtx && audioCtx.state === 'suspended') {
                audioCtx.resume();
            }
        } catch(e) {}
    }

    function playSound(type) {
        if (!audioCtx || audioCtx.state === 'suspended') return;
        try {
            let osc = audioCtx.createOscillator();
            let gain = audioCtx.createGain();
            osc.connect(gain);
            gain.connect(audioCtx.destination);

            if (type === 'coin') {
                osc.type = 'sine';
                osc.frequency.setValueAtTime(587.33, audioCtx.currentTime); 
                osc.frequency.setValueAtTime(880, audioCtx.currentTime + 0.08); 
                gain.gain.setValueAtTime(0.12, audioCtx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.2);
                osc.start();
                osc.stop(audioCtx.currentTime + 0.2);
            } else if (type === 'crash') {
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(130, audioCtx.currentTime);
                osc.frequency.linearRampToValueAtTime(25, audioCtx.currentTime + 0.4);
                gain.gain.setValueAtTime(0.3, audioCtx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.45);
                osc.start();
                osc.stop(audioCtx.currentTime + 0.45);
            }
        } catch(e) {}
    }

    function addEvent(id, startEvt, endEvt, setter) {
        const el = document.getElementById(id);
        if (!el) return;
        el.addEventListener(startEvt, (e) => { e.preventDefault(); initAudio(); setter(true); });
        el.addEventListener(endEvt, (e) => { e.preventDefault(); setter(false); });
    }

    addEvent("leftBtn", "touchstart", "touchend", (v) => touchLeft = v);
    addEvent("rightBtn", "touchstart", "touchend", (v) => touchRight = v);
    addEvent("accelBtn", "touchstart", "touchend", (v) => touchAccel = v);
    addEvent("brakeBtn", "touchstart", "touchend", (v) => touchBrake = v);
    addEvent("leftBtn", "mousedown", "mouseup", (v) => touchLeft = v);
    addEvent("rightBtn", "mousedown", "mouseup", (v) => touchRight = v);
    addEvent("accelBtn", "mousedown", "mouseup", (v) => touchAccel = v);
    addEvent("brakeBtn", "mousedown", "mouseup", (v) => touchBrake = v);

    function resetGame() {
        initAudio();
        score = 0;
        gameOver = false;
        finishState = "";
        baseSpeed = 5;
        playerProgress = 0;
        
        carX = 232;
        carY = 340;

        racers[0].progress = 0;
        racers[1].progress = 80 + Math.random() * 40;
        racers[2].progress = 40 + Math.random() * 50;
        racers[3].progress = 100 + Math.random() * 30;

        obstacles[0].y = -200; obstacles[0].x = 180;
        obstacles[1].y = -500; obstacles[1].x = 280;

        coinY = -150;
        if (restartBtn) restartBtn.style.display = "none";
        gameLoop();
    }

    if (restartBtn) {
        restartBtn.addEventListener("click", resetGame);
        restartBtn.addEventListener("touchstart", (e) => { e.preventDefault(); resetGame(); });
    }

    function gameLoop() {
        try {
            if (gameOver) {
                ctx.fillStyle = "rgba(10, 11, 21, 0.96)";
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                
                ctx.fillStyle = finishState.includes("CRASHED") ? "#ff2d55" : "#00fff2";
                ctx.font = "bold 26px sans-serif";
                ctx.textAlign = "center";
                ctx.fillText(finishState, canvas.width / 2, canvas.height / 2 - 30);
                
                ctx.fillStyle = "rgba(255,255,255,0.8)";
                ctx.font = "16px sans-serif";
                ctx.fillText("Energy Crystals: " + score, canvas.width / 2, canvas.height / 2 + 10);
                
                if (restartBtn) restartBtn.style.display = "block";
                return;
            }

            // Player Mechanics
            if (touchAccel) currentSpeed = baseSpeed * 1.8;
            else if (touchBrake) currentSpeed = baseSpeed * 0.4;
            else currentSpeed = baseSpeed;

            if (touchLeft && carX > 140) carX -= 5;
            if (touchRight && carX < canvas.width - 140 - carW) carX += 5;

            playerProgress += currentSpeed;
            racers[0].progress = playerProgress;

            roadOffset += currentSpeed;
            if (roadOffset > 60) roadOffset = 0;

            // Background AI Progress Logic
            for (let i = 1; i < racers.length; i++) {
                let ai = racers[i];
                // AI speeds fluctuate slightly around base speed
                let aiSpeed = (baseSpeed + (Math.sin(playerProgress * 0.002 + i) * 0.8)) * ai.speedModifier;
                ai.progress += aiSpeed;
            }

            // Standings & Leaderboard
            let standings = [...racers].sort((a, b) => b.progress - a.progress);
            let playerRank = standings.findIndex(r => r.isPlayer) + 1;
            let suffixes = ["ST", "ND", "RD", "TH"];
            let rankStr = playerRank + suffixes[playerRank - 1];

            // Check if anyone hit the finish line point
            if (playerProgress >= TOTAL_DISTANCE) {
                gameOver = true;
                finishState = "RACE FINISHED: " + rankStr + " PLACE!";
            } else {
                // If any AI crossed the absolute finish line point significantly ahead
                standings.forEach(r => {
                    if (r.progress >= TOTAL_DISTANCE && !r.isPlayer && playerProgress < TOTAL_DISTANCE) {
                        // Keep game going until player crosses, but ranking locks
                    }
                });
            }

            // Traffic Obstacles Handling
            obstacles.forEach(obs => {
                obs.y += currentSpeed - obs.speed;
                if (obs.y > canvas.height) {
                    obs.y = -100 - Math.random() * 200;
                    obs.x = 145 + Math.random() * (210 - obs.w);
                }
                if (carX < obs.x + obs.w && carX + carW > obs.x && carY < obs.y + obs.h && carY + carH > obs.y) {
                    gameOver = true;
                    finishState = "CRASHED";
                    playSound('crash');
                }
            });

            // Coin Collection
            coinY += currentSpeed;
            if (coinY > canvas.height) {
                coinY = -150 - Math.random() * 250;
                coinX = 145 + Math.random() * (210 - coinSize);
            }
            if (carX < coinX + coinSize && carX + carW > coinX && carY < coinY + coinSize && carY + carH > coinY) {
                score++;
                playSound('coin');
                coinY = -200; 
                coinX = 145 + Math.random() * (210 - coinSize);
            }

            buildings.forEach(b => {
                b.y += currentSpeed * 0.4;
                if (b.y > canvas.height) b.y = -b.h;
            });

            // RENDER GRAPHICS
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Buildings Background
            buildings.forEach(b => {
                let drawX = b.leftSide ? b.xOffset : canvas.width - b.w - b.xOffset;
                ctx.fillStyle = "#141526";
                ctx.fillRect(drawX, b.y, b.w, b.h);
                ctx.fillStyle = b.accentColor;
                ctx.fillRect
