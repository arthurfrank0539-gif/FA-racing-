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
            background: #05060c;
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
            margin: 10px 0 5px 0;
            font-size: 1.4rem;
            font-weight: 800;
            text-transform: uppercase;
            letter-spacing: 4px;
            color: #00fff2;
            text-shadow: 0 0 15px rgba(0, 255, 242, 0.6);
            text-align: center;
        }
        .level-badge {
            font-size: 0.9rem;
            color: #ffb74d;
            font-weight: bold;
            margin-bottom: 8px;
            letter-spacing: 2px;
            text-transform: uppercase;
        }
        .container {
            position: relative;
            border-radius: 20px;
            overflow: hidden;
            width: 94%;
            max-width: 430px;
            box-shadow: 0 25px 60px rgba(0, 0, 0, 0.8), 0 0 30px rgba(0, 255, 242, 0.15);
            border: 2px solid #00fff2;
            background: #090a14;
        }
        canvas {
            display: block;
            width: 100%;
            height: auto;
            background: #090a14;
        }
        #nextLevelBtn {
            display: none;
            position: absolute;
            top: 65%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 16px 32px;
            font-size: 1.1rem;
            font-weight: 700;
            letter-spacing: 2px;
            text-transform: uppercase;
            background: #00fff2;
            color: #000000;
            border: none;
            border-radius: 35px;
            cursor: pointer;
            box-shadow: 0 0 25px #00fff2;
            z-index: 20;
        }
        /* Modernized Larger Control Pad */
        .controls-pad {
            display: flex;
            justify-content: space-between;
            align-items: center;
            width: 94%;
            max-width: 430px;
            margin-top: 16px;
            margin-bottom: 12px;
            padding: 12px 16px;
            background: rgba(10, 12, 26, 0.6);
            border: 1px solid rgba(255, 255, 255, 0.05);
            backdrop-filter: blur(10px);
            -webkit-backdrop-filter: blur(10px);
            border-radius: 30px;
        }
        .btn-group {
            display: flex;
            gap: 16px;
        }
        .btn {
            width: 76px;
            height: 76px;
            background: rgba(0, 255, 242, 0.04);
            border: 2px solid rgba(0, 255, 242, 0.4);
            border-radius: 24px;
            color: #00fff2;
            font-size: 1.8rem;
            font-weight: bold;
            display: flex;
            justify-content: center;
            align-items: center;
            touch-action: none;
            cursor: pointer;
            transition: background 0.1s, transform 0.1s, box-shadow 0.1s;
            box-shadow: inset 0 0 10px rgba(0, 255, 242, 0.1);
        }
        .btn:active {
            background: #00fff2;
            color: #090a14;
            transform: scale(0.95);
            box-shadow: 0 0 20px rgba(0, 255, 242, 0.6);
        }
        .btn-action {
            color: #ff2d55;
            background: rgba(ff, 45, 85, 0.04);
            border-color: rgba(2ff, 45, 85, 0.4);
            box-shadow: inset 0 0 10px rgba(255, 45, 85, 0.1);
        }
        .btn-action:active {
            background: #ff2d55;
            color: #090a14;
            box-shadow: 0 0 20px rgba(255, 45, 85, 0.6);
        }
    </style>
</head>
<body>

<div class="header-title">Neon Sprint GP</div>
<div class="level-badge" id="levelNameDisplay">Stage 1: Sector Zero</div>

<div class="container">
    <canvas id="gameCanvas" width="500" height="450"></canvas>
    <button id="nextLevelBtn">Next Stage</button>
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
window.addEventListener("load", function() {
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const nextLevelBtn = document.getElementById("nextLevelBtn");
    const levelNameDisplay = document.getElementById("levelNameDisplay");

    const levelsConfig = [
        { name: "Stage 1: Sector Zero", distance: 6000, baseSpeed: 5.0, aiAggression: 0.02, theme: "#00fff2" },
        { name: "Stage 2: Grid City", distance: 7500, baseSpeed: 5.5, aiAggression: 0.03, theme: "#ff00bb" },
        { name: "Stage 3: Cyber Basin", distance: 9000, baseSpeed: 6.0, aiAggression: 0.04, theme: "#bc00ff" },
        { name: "Stage 4: Laser Highway", distance: 10500, baseSpeed: 6.5, aiAggression: 0.05, theme: "#00ff66" },
        { name: "Stage 5: Tokyo Grid", distance: 12000, baseSpeed: 7.0, aiAggression: 0.05, theme: "#ffff00" },
        { name: "Stage 6: Synth Outpost", distance: 13000, baseSpeed: 7.5, aiAggression: 0.06, theme: "#ff5500" },
        { name: "Stage 7: Overdrive Pass", distance: 14000, baseSpeed: 8.0, aiAggression: 0.06, theme: "#00ffff" },
        { name: "Stage 8: Matrix Hub", distance: 15000, baseSpeed: 8.5, aiAggression: 0.07, theme: "#ff0055" },
        { name: "Stage 9: Vector Drift", distance: 16000, baseSpeed: 9.0, aiAggression: 0.07, theme: "#aa00ff" },
        { name: "Stage 10: Neon Core", distance: 17000, baseSpeed: 9.5, aiAggression: 0.08, theme: "#33ff00" },
        { name: "Stage 11: Zenith Route", distance: 18000, baseSpeed: 10.0, aiAggression: 0.08, theme: "#0088ff" },
        { name: "Stage 12: Velocity Valley", distance: 19000, baseSpeed: 10.5, aiAggression: 0.09, theme: "#ff00aa" },
        { name: "Stage 13: Cosmic Loop", distance: 20000, baseSpeed: 11.0, aiAggression: 0.09, theme: "#ccff00" },
        { name: "Stage 14: Quantum Slipway", distance: 22000, baseSpeed: 11.5, aiAggression: 0.10, theme: "#ff3300" },
        { name: "Stage 15: Grand Finale GP", distance: 25000, baseSpeed: 12.5, aiAggression: 0.12, theme: "#ffffff" }
    ];

    let currentLevelIdx = 0;
    let score = 0;
    let gameOver = false;
    let finishState = ""; 
    let roadOffset = 0;
    let playerProgress = 0;
    let cameraProgress = 0;

    let carX = 232;
    const carW = 34;
    const carH = 62;

    let racers = [
        { id: "Player", name: "YOU", progress: 0, speedModifier: 1.0, isPlayer: true, x: 232, color1: '#00c6ff', color2: '#0072ff' },
        { id: "AI1", name: "VIOLET", progress: 120, speedModifier: 0.97, x: 160, color1: '#7b1fa2', color2: '#e040fb' },
        { id: "AI2", name: "ORANGE", progress: 60, speedModifier: 1.01, x: 305, color1: '#f57c00', color2: '#ffb74d' },
        { id: "AI3", name: "GREEN", progress: 180, speedModifier: 0.95, x: 232, color1: '#388e3c', color2: '#69f0ae' }
    ];

    let coinX = 145 + Math.random() * (210 - 18);
    let coinY = -150;
    const coinSize = 18;

    let buildings = [
        { leftSide: true, xOffset: 8, y: 0, w: 85, h: 200 },
        { leftSide: true, xOffset: 18, y: 250, w: 75, h: 150 },
        { leftSide: false, xOffset: 8, y: 50, w: 85, h: 220 },
        { leftSide: false, xOffset: 20, y: 300, w: 70, h: 160 }
    ];

    let touchLeft = false, touchRight = false, touchAccel = false, touchBrake = false;

    function addEvent(id, startEvt, endEvt, setter) {
        const el = document.getElementById(id);
        if (!el) return;
        el.addEventListener(startEvt, (e) => { e.preventDefault(); setter(true); });
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

    function loadLevel(idx) {
        if(idx >= levelsConfig.length) {
            finishState = "CHAMPIONSHIP COMPLETE!";
            gameOver = true;
            return;
        }
        currentLevelIdx = idx;
        let cfg = levelsConfig[currentLevelIdx];
        levelNameDisplay.textContent = cfg.name;

        gameOver = false;
        finishState = "";
        playerProgress = 0;
        cameraProgress = 0;
        carX = 232;

        racers[0].progress = 0;
        racers[1].progress = 120;
        racers[2].progress = 60;
        racers[3].progress = 180;

        coinY = -150;
        nextLevelBtn.style.display = "none";
    }

    nextLevelBtn.addEventListener("click", () => {
        if (finishState.includes("CLEAR") || finishState.includes("COMPLETED")) {
            loadLevel(currentLevelIdx + 1);
        } else {
            loadLevel(currentLevelIdx);
        }
    });

    function gameLoop() {
        let cfg = levelsConfig[currentLevelIdx];
        let runningBaseSpeed = cfg.baseSpeed;

        if (!gameOver) {
            let currentSpeed = runningBaseSpeed;
            if (touchAccel) currentSpeed = runningBaseSpeed * 1.6;
            else if (touchBrake) currentSpeed = runningBaseSpeed * 0.3;

            if (touchLeft && carX > 140) carX -= 5;
            if (touchRight && carX < canvas.width - 140 - carW) carX += 5;

            playerProgress += currentSpeed;
            racers[0].progress = playerProgress;
            racers[0].x = carX;

            for (let i = 1; i < racers.length; i++) {
                let ai = racers[i];
                let distanceBehind = playerProgress - ai.progress;
                
                let dynamicModifier = ai.speedModifier;
                if (distanceBehind > 200) dynamicModifier += (cfg.aiAggression + 0.04); 
                if (distanceBehind < -200) dynamicModifier -= (cfg.aiAggression);

                let aiSpeed = (runningBaseSpeed + (Math.sin(playerProgress * 0.003 + i) * 1.0)) * dynamicModifier;
                ai.progress += aiSpeed;
                
                if(i === 1) ai.x = 160 + Math.sin(playerProgress * 0.002) * 12;
                if(i === 2) ai.x = 305 + Math.cos(playerProgress * 0.002) * 12;
            }

            cameraProgress = playerProgress - 150;

            roadOffset += currentSpeed;
            if (roadOffset > 60) roadOffset = 0;

            if (playerProgress >= cfg.
