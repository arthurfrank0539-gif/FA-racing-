<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Neon Rider - Street Race</title>
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

<div class="header-title">Neon GP</div>

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

    let score = 0;
    let highScore = 0;
    try {
        highScore = localStorage.getItem("neonRider_highScore") ? parseInt(localStorage.getItem("neonRider_highScore")) : 0;
    } catch(e) {}

    let gameOver = false;
    let finishState = ""; // "CRASHED" or placement
    let roadOffset = 0;
    let baseSpeed = 5;
    let currentSpeed = baseSpeed;
    let playerProgress = 0;

    // Player Car Setup
    let carX = 232;
    let carY = 340;
    const carW = 34;
    const carH = 62;

    // AI Racers Setup
    let racers = [
        { id: "Player", name: "YOU", x: 232, y: 340, color1: "#2f80ed", color2: "#00c6ff", speedOffset: 0, progress: 0, active: true, isPlayer: true },
        { id: "AI1", name: "VIOLET", x: 160, y: 150, color1: "#8e2de2", color2: "#4a00e0", speedOffset: 0.2, progress: 120, active: true, targetX: 160 },
        { id: "AI2", name: "ORANGE", x: 250, y: -50, color1: "#f2994a", color2: "#f2c94c", speedOffset: -0.1, progress: 300, active: true, targetX: 250 },
        { id: "AI3", name: "GREEN", x: 310, y: -250, color1: "#11998e", color2: "#38ef7d", speedOffset: 0.4, progress: 450, active: true, targetX: 310 }
    ];

    // Standard road obstacles (Neutral traffic blocks)
    let obstacles = [
        { x: 200, y: -150, w: 34, h: 62, speed: 2 },
        { x: 290, y: -450, w: 34, h: 62, speed: 1.5 }
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
        racers[0].x = 232;
        
        racers[1].x = 160; racers[1].y = 100; racers[1].progress = 150;
        racers[2].x = 250; racers[2].y = -100; racers[2].progress = 320;
        racers[3].x = 310; racers[3].y = -300; racers[3].progress = 480;

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
                
                ctx.fillStyle = finishState === "CRASHED" ? "#ff2d55" : "#00fff2";
                ctx.font = "bold 28px sans-serif";
                ctx.textAlign = "center";
                ctx.fillText(finishState, canvas.width / 2, canvas.height / 2 - 30);
                
                ctx.fillStyle = "rgba(255,255,255,0.8)";
                ctx.font = "16px sans-serif";
                ctx.fillText("Energy Collected: " + score, canvas.width / 2, canvas.height / 2 + 10);
                
                if (restartBtn) restartBtn.style.display = "block";
                return;
            }

            // Player Mechanics
            if (touchAccel) currentSpeed = baseSpeed * 1.7;
            else if (touchBrake) currentSpeed = baseSpeed * 0.4;
            else currentSpeed = baseSpeed;

            if (touchLeft && carX > 140) carX -= 5;
            if (touchRight && carX < canvas.width - 140 - carW) carX += 5;

            playerProgress += currentSpeed;
            racers[0].x = carX;
            racers[0].progress = playerProgress;

            roadOffset += currentSpeed;
            if (roadOffset > 60) roadOffset = 0;

            // Neutral Obstacles Management
            obstacles.forEach(obs => {
                obs.y += currentSpeed - obs.speed;
                if (obs.y > canvas.height) {
                    obs.y = -100 - Math.random() * 200;
                    obs.x = 145 + Math.random() * (210 - obs.w);
                }
                // Collision check with player
                if (carX < obs.x + obs.w && carX + carW > obs.x && carY < obs.y + obs.h && carY + carH > obs.y) {
                    gameOver = true;
                    finishState = "CRASHED";
                    playSound('crash');
                }
            });

            // AI Logic & Progression
            for (let i = 1; i < racers.length; i++) {
                let ai = racers[i];
                
                // Keep AI moving at baseline dynamic speeds
                let aiDesiredSpeed = baseSpeed + ai.speedOffset + (Math.sin(playerProgress * 0.005 + i) * 0.5);
                ai.progress += aiDesiredSpeed;

                // Project physical screen positions relative to player progress
                ai.y = carY - (ai.progress - playerProgress);

                // Simple AI steering adjustment system (Avoid obstacles and borders)
                if (Math.random() < 0.02) {
                    let lanes = [160, 232, 310];
                    ai.targetX = lanes[Math.floor(Math.random() * lanes.length)];
                }

                // Smooth steer transition
                if (ai.x < ai.targetX) ai.x += 2;
                if (ai.x > ai.targetX) ai.x -= 2;

                // Out-of-bounds reset logic to keep racers circling nearby
                if (ai.y > canvas.height + 200) {
                    ai.progress = playerProgress - 300 - Math.random() * 100;
                    ai.x = 145 + Math.random() * 200;
                } else if (ai.y < -600) {
                    ai.progress = playerProgress + 500 + Math.random() * 100;
                }

                // Crash verification with Player
                if (ai.y > 0 && ai.y < canvas.height) {
                    if (carX < ai.x + carW && carX + carW > ai.x && carY < ai.y + carH && carY + carH > ai.y) {
                        gameOver = true;
                        finishState = "CRASHED";
                        playSound('crash');
                    }
                }
            }

            // Coin Mechanics
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

            // Scenery Background Handling
            buildings.forEach(b => {
                b.y += currentSpeed * 0.4;
                if (b.y > canvas.height) b.y = -b.h;
            });

            // Calculate Leaderboard rankings
            let standings = [...racers].sort((a, b) => b.progress - a.progress);
            let playerRank = standings.findIndex(r => r.isPlayer) + 1;
            let suffixes = ["ST", "ND", "RD", "TH"];
            let rankStr = playerRank + suffixes[playerRank - 1];

            // RENDER SYSTEM
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Draw Background Buildings
            buildings.forEach(b => {
                let drawX = b.leftSide ? b.xOffset : canvas.width - b.w - b.xOffset;
                ctx.fillStyle = "#141526";
                ctx.fillRect(drawX, b.y, b.w, b.h);
                ctx.fillStyle = b.accentColor;
                ctx.fillRect(drawX, b.y, b.w, 3);
            });

            // Draw Highway Roadbed
            let roadGrad = ctx.createLinearGradient(135, 0, canvas.width - 135, 0);
            roadGrad.addColorStop(0, '#0a0b14');
            roadGrad.addColorStop(0.5, '#131526');
            roadGrad.addColorStop(1, '#0a0b14');
            ctx.fillStyle = roadGrad;
            ctx.fillRect(135, 0, canvas.width - 270, canvas.height);

            // Bounds markings
            ctx.fillStyle = "rgba(0, 255, 242, 0.4)";
            ctx.fillRect(135, 0, 2, canvas.height);
            ctx.fillRect(canvas.width - 137, 0, 2, canvas.height);

            // Center Lanes
            ctx.fillStyle = "rgba(255, 255, 255, 0.15)";
            for (let i = -60; i < canvas.height; i += 60) {
                ctx.fillRect(canvas.width / 2 - 1, i + roadOffset, 2, 30);
            }

            // Render Energy Tokens
            ctx.fillStyle = "#ffcc00";
            ctx.shadowColor = "#ffcc00";
            ctx.shadowBlur = 8;
            ctx.beginPath();
            ctx.arc(coinX + coinSize/2, coinY + coinSize/2, coinSize/2, 0, Math.PI * 2);
            ctx.fill();
            ctx.shadowBlur = 0; // reset

            // Render Obstacle Traffic Blocks
            obstacles.forEach(obs => {
                ctx.fillStyle = "#3a3d52";
                ctx.fillRect(obs.x, obs.y, obs.w, obs.h);
                ctx.fillStyle = "#ff3b30";
                ctx.fillRect(obs.x + 4, obs.y + 2, 6, 2);
                ctx.fillRect(obs.x + obs.w - 10, obs.y + 2, 6, 2);
            });

            // Render All Active Racers
            racers.forEach(r => {
                if (r.y < -100 || r.y > canvas.height + 100) return;

                let cg = ctx.createLinearGradient(r.x, r.y, r.x + carW, r.y);
                cg.addColorStop(0, r.color1);
                cg.addColorStop(1, r.color2);
                ctx.fillStyle = cg;
                
                // Clean rectangular racing profiles
                ctx.fillRect(r.x, r.y, carW, carH);

                // Windshield window grids
                ctx.fillStyle = "#090a12";
                ctx.fillRect(r.x + 4, r.y + 12, carW - 8, 14);

                // Front Headlights
                ctx.fillStyle = "#ffffff";
                ctx.fillRect(r.x + 3, r.y, 5, 2);
                ctx.fillRect(r.x + carW - 8, r.y, 5, 2);
            });

            // Heads Up Display HUD Interface Overlay
            ctx.fillStyle = "rgba(10, 12, 26, 0.85)";
            ctx.fillRect(15, 15, 120, 45);
            ctx.strokeStyle = "rgba(0, 255, 242, 0.2)";
            ctx.strokeRect(15, 15, 120, 45);
            
            ctx.fillStyle = "#ffffff";
            ctx.font = "bold 11px sans-serif";
            ctx.fillText("ENERGY: " + score, 22, 32);
            ctx.fillStyle = "#00fff2";
            ctx.fillText("POSITION: " + rankStr, 22, 48);

            // Mini Standing Leaderboard
            ctx.fillStyle = "rgba(10, 12, 26, 0.85)";
            ctx.fillRect(canvas.width - 115, 15, 100, 75);
            ctx.strokeRect(canvas.width - 115, 15, 100, 75);

            ctx.font = "9px sans-serif";
            standings.forEach((st, idx) => {
                ctx.fillStyle = st.isPlayer ? "#00fff2" : "#ffffff";
                ctx.fillText((idx + 1) + ". " + st.name, canvas.width - 105, 30 + (idx * 14));
            });

        } catch(gameErr) {}

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
