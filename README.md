<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Neon Pro GP</title>
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
            margin: 15px 0 10px 0;
            font-size: 1.6rem;
            font-weight: 800;
            text-transform: uppercase;
            letter-spacing: 6px;
            color: #00fff2;
            text-shadow: 0 0 15px rgba(0, 255, 242, 0.6);
            text-align: center;
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
        #restartBtn {
            display: none;
            position: absolute;
            top: 60%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 16px 36px;
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
        .controls-pad {
            display: flex;
            justify-content: space-between;
            align-items: center;
            width: 94%;
            max-width: 430px;
            margin-top: 15px;
            margin-bottom: 15px;
            padding: 12px;
            background: rgba(13, 15, 30, 0.9);
            border: 1px solid rgba(0, 255, 242, 0.2);
            border-radius: 24px;
            box-shadow: inset 0 0 15px rgba(0, 255, 242, 0.05);
        }
        .btn-group {
            display: flex;
            gap: 16px;
        }
        .btn {
            width: 68px;
            height: 68px;
            background: linear-gradient(135deg, #12152a 0%, #0a0c18 100%);
            border: 2px solid #00fff2;
            border-radius: 20px;
            color: #00fff2;
            font-size: 1.6rem;
            font-weight: bold;
            display: flex;
            justify-content: center;
            align-items: center;
            touch-action: none;
            box-shadow: 0 6px 12px rgba(0, 255, 242, 0.15);
            transition: transform 0.05s;
        }
        .btn:active {
            background: #00fff2;
            color: #090a14;
            box-shadow: 0 0 25px #00fff2;
            transform: scale(0.95);
        }
        .btn-action {
            color: #ff2d55;
            border-color: #ff2d55;
            box-shadow: 0 6px 12px rgba(255, 45, 85, 0.15);
        }
        .btn-action:active {
            background: #ff2d55;
            color: #090a14;
            box-shadow: 0 0 25px #ff2d55;
        }
    </style>
</head>
<body>

<div class="header-title">Neon Pro GP</div>

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

    const TOTAL_DISTANCE = 12000; 
    let score = 0;
    let gameOver = false;
    let finishState = ""; 
    let roadOffset = 0;
    let baseSpeed = 5;
    let currentSpeed = baseSpeed;
    let playerProgress = 0;

    // Player Configuration
    let carX = 232;
    let carY = 320; // Moved up slightly to allow trailing cars to stay on screen
    const carW = 34;
    const carH = 62;

    // Competitors Dynamic Data Layout (Tightly packed around starting line)
    let racers = [
        { id: "Player", name: "YOU", progress: 0, speedModifier: 1.0, isPlayer: true, x: 232, color1: '#2f80ed', color2: '#00c6ff' },
        { id: "AI1", name: "VIOLET", progress: 80, speedModifier: 0.99, x: 155, color1: '#7b1fa2', color2: '#e040fb' },
        { id: "AI2", name: "ORANGE", progress: 40, speedModifier: 1.01, x: 310, color1: '#f57c00', color2: '#ffb74d' },
        { id: "AI3", name: "GREEN", progress: 120, speedModifier: 0.97, x: 232, color1: '#388e3c', color2: '#69f0ae' }
    ];

    let coinX = 145 + Math.random() * (210 - 18);
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

        racers[0].progress = 0;
        racers[1].progress = 60 + Math.random() * 40;
        racers[2].progress = 30 + Math.random() * 40;
        racers[3].progress = 90 + Math.random() * 40;

        coinY = -150;
        if (restartBtn) restartBtn.style.display = "none";
    }

    if (restartBtn) {
        restartBtn.addEventListener("click", resetGame);
        restartBtn.addEventListener("touchstart", (e) => { e.preventDefault(); resetGame(); });
    }

    function gameLoop() {
        try {
            if (!gameOver) {
                if (touchAccel) currentSpeed = baseSpeed * 1.9;
                else if (touchBrake) currentSpeed = baseSpeed * 0.4;
                else currentSpeed = baseSpeed;

                if (touchLeft && carX > 140) carX -= 5;
                if (touchRight && carX < canvas.width - 140 - carW) carX += 5;

                playerProgress += currentSpeed;
                racers[0].progress = playerProgress;
                racers[0].x = carX;

                roadOffset += currentSpeed;
                if (roadOffset > 60) roadOffset = 0;

                // Move Competitor Positions
                for (let i = 1; i < racers.length; i++) {
                    let ai = racers[i];
                    // Dynamic rubber-banding logic keeps them close enough to be seen on track
                    let aiSpeed = (baseSpeed + (Math.sin(playerProgress * 0.002 + i) * 1.2)) * ai.speedModifier;
                    ai.progress += aiSpeed;
                    
                    // Lane weaving behavior
                    if(i === 1) ai.x = 155 + Math.sin(playerProgress * 0.001) * 8;
                    if(i === 2) ai.x = 310 + Math.cos(playerProgress * 0.001) * 8;
                }

                if (playerProgress >= TOTAL_DISTANCE) {
                    gameOver = true;
                    let standingsCheck = [...racers].sort((a, b) => b.progress - a.progress);
                    let finalRank = standingsCheck.findIndex(r => r.isPlayer) + 1;
                    let suffixes = ["ST", "ND", "RD", "TH"];
                    finishState = "FINISHED: " + finalRank + suffixes[finalRank - 1] + " PLACE!";
                }

                coinY += currentSpeed;
                if (coinY > canvas.height) {
                    coinY = -150 - Math.random() * 250;
                    coinX = 145 + Math.random() * (210 - coinSize);
                }
                if (carX < coinX + coinSize && carX + carW > coinX && carY < coinY + coinSize && carY + carH > coinY) {
                    score++;
                    coinY = -200; 
                    coinX = 145 + Math.random() * (210 - coinSize);
                }

                buildings.forEach(b => {
                    b.y += currentSpeed * 0.4;
                    if (b.y > canvas.height) b.y = -b.h;
                });
            }

            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Sidewalk Buildings
            buildings.forEach(b => {
                let drawX = b.leftSide ? b.xOffset : canvas.width - b.w - b.xOffset;
                ctx.fillStyle = "#111222";
                ctx.fillRect(drawX, b.y, b.w, b.h);
                ctx.fillStyle = b.accentColor;
                ctx.fillRect(drawX, b.y, b.w, 3);
            });

            // Roadways
            let roadGrad = ctx.createLinearGradient(135, 0, canvas.width - 135, 0);
            roadGrad.addColorStop(0, '#07080e');
            roadGrad.addColorStop(0.5, '#111324');
            roadGrad.addColorStop(1, '#07080e');
            ctx.fillStyle = roadGrad;
            ctx.fillRect(135, 0, canvas.width - 270, canvas.height);

            // Neon Rails
            ctx.fillStyle = "rgba(0, 255, 242, 0.4)";
            ctx.fillRect(135, 0, 2, canvas.height);
            ctx.fillRect(canvas.width - 137, 0, 2, canvas.height);

            // Track center lanes
            ctx.fillStyle = "rgba(255, 255, 255, 0.12)";
            for (let i = -60; i < canvas.height; i += 60) {
                ctx.fillRect(canvas.width / 2 - 1, i + roadOffset, 2, 30);
            }

            // Target Finish line drawing
            let remainingDist = TOTAL_DISTANCE - playerProgress;
            if (remainingDist < canvas.height) {
                let finishY = carY - remainingDist;
                ctx.fillStyle = "#00fff2";
                ctx.fillRect(135, finishY, canvas.width - 270, 12);
            }

            // Energy Crystal Orb
            ctx.fillStyle = "#ffcc00";
            ctx.beginPath();
            ctx.arc(coinX + coinSize/2, coinY + coinSize/2, coinSize/2, 0, Math.PI * 2);
            ctx.fill();

            // --- RENDER RACERS WITH UPDATED POSITION CAMERA ---
            racers.forEach(r => {
                // Calculate position relative to player anchor view
                // This ensures trailing cars stay visible behind the player!
                let renderY = carY - (r.progress - playerProgress);
                
                if (renderY > -70 && renderY < canvas.height + 100) {
                    let cg = ctx.createLinearGradient(r.x, renderY, r.x + carW, renderY);
                    cg.addColorStop(0, r.color1);
                    cg.addColorStop(1, r.color2);
                    ctx.fillStyle = cg;
                    ctx.fillRect(r.x, renderY, carW, carH);

                    // Windshield cockpit
                    ctx.fillStyle = "#090a12";
                    ctx.fillRect(r.x + 4, renderY + 12, carW - 8, 14);
                    
                    // Headlights
                    ctx.fillStyle = "#ffffff";
                    ctx.fillRect(r.x + 3, renderY, 5, 2);
                    ctx.fillRect(r.x + carW - 8, renderY, 5, 2);

                    // Name Tag for AI opponents
                    if (!r.isPlayer) {
                        ctx.fillStyle = "rgba(255, 255, 255, 0.85)";
                        ctx.font = "bold 10px sans-serif";
                        ctx.fillText(r.name, r.x, renderY - 8);
                    }
                }
            });

            // Calculate live standing rank
            let standings = [...racers].sort((a, b) => b.progress - a.progress);
            let playerRank = standings.findIndex(r => r.isPlayer) + 1;
            let suffixes = ["ST", "ND", "RD", "TH"];
            let rankStr = playerRank + suffixes[playerRank - 1];

            // Primary Telemetry Panel
            ctx.fillStyle = "rgba(10, 12, 26, 0.88)";
            ctx.fillRect(15, 15, 130, 62);
            ctx.strokeStyle = "rgba(0, 255, 242, 0.25)";
            ctx.strokeRect(15, 15, 130, 62);
            
            ctx.fillStyle = "#ffffff";
            ctx.font = "bold 11px sans-serif";
            let displayDist = Math.max(0, Math.floor(remainingDist / 10));
            ctx.fillText("DIST TO GO: " + displayDist + " M", 22, 32);
            ctx.fillText("POSITION: " + rankStr, 22, 48);
            ctx.fillStyle = "#ffcc00";
            ctx.fillText("CRYSTALS: " + score, 22, 64);

            // Leaderboard Side HUD Box
            ctx.fillStyle = "rgba(10, 12, 26, 0.88)";
            ctx.fillRect(canvas.width - 115, 15, 100, 75);
            ctx.strokeRect(canvas.width - 115, 15, 100, 75);

            ctx.font = "9px sans-serif";
            standings.forEach((st, idx) => {
                ctx.fillStyle = st.isPlayer ? "#00fff2" : "#ffffff";
                ctx.fillText((idx + 1) + ". " + st.name, canvas.width - 105, 31 + (idx * 14));
            });

            if (gameOver) {
                ctx.fillStyle = "rgba(8, 9, 18, 0.97)";
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                
                ctx.fillStyle = "#00fff2";
                ctx.font = "bold 26px sans-serif";
                ctx.textAlign = "center";
                ctx.fillText(finishState, canvas.width / 2, canvas.height / 2 - 20);
                
                ctx.fillStyle = "rgba(255,255,255,0.8)";
                ctx.font = "16px sans-serif";
                ctx.fillText("Final Crystals: " + score, canvas.width / 2, canvas.height / 2 + 15);
                
                if (restartBtn) restartBtn.style.display = "block";
            }

        } catch(err) {}

        requestAnimationFrame(gameLoop);
    }

    gameLoop();
}

if (document.readyState === "complete" || document.readyState === "interactive") {
    startNeonRider();
} else {
    window.addEventListener("DOMContentLoaded", startNeonRider);
}
</script>

</body>
</html>
