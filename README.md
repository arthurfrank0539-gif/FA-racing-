<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Neon Rider - Arcade Edition</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <style>
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
        h1 {
            margin: 5px 0;
            font-size: 1.5rem;
            font-weight: 700;
            text-transform: uppercase;
            letter-spacing: 5px;
            color: #ffffff;
            opacity: 0.95;
            text-align: center;
        }
        .container {
            position: relative;
            border-radius: 20px;
            overflow: hidden;
            width: 92%;
            max-width: 480px;
            box-shadow: 0 20px 50px rgba(0, 0, 0, 0.6);
            border: 1px solid rgba(255, 255, 255, 0.08);
        }
        canvas {
            display: block;
            width: 100%;
            height: auto;
            background: #090a14;
        }
        #audioOverlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(9, 10, 20, 0.95);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 20;
        }
        #audioBtn {
            padding: 16px 36px;
            font-size: 1.1rem;
            font-weight: 700;
            text-transform: uppercase;
            letter-spacing: 2px;
            background: #00fff2;
            color: #05060b;
            border: none;
            border-radius: 30px;
            cursor: pointer;
            box-shadow: 0 0 25px rgba(0, 255, 242, 0.4);
        }
        #restartBtn {
            display: none;
            position: absolute;
            top: 55%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 14px 32px;
            font-size: 1rem;
            font-weight: 600;
            letter-spacing: 1px;
            background: #ffffff;
            color: #000000;
            border: none;
            border-radius: 30px;
            cursor: pointer;
            box-shadow: 0 10px 25px rgba(255,255,255,0.3);
            z-index: 10;
        }
        .controls-pad {
            display: flex;
            justify-content: space-between;
            width: 92%;
            max-width: 480px;
            margin-top: 15px;
            gap: 20px;
        }
        .steering-group, .speed-group {
            display: flex;
            gap: 12px;
        }
        .btn {
            width: 60px;
            height: 60px;
            background: rgba(255, 255, 255, 0.05);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 50%;
            color: #ffffff;
            font-size: 1.3rem;
            display: flex;
            justify-content: center;
            align-items: center;
            touch-action: none;
            box-shadow: 0 4px 12px rgba(0,0,0,0.2);
        }
        .btn:active {
            background: rgba(255, 255, 255, 0.2);
        }
        .btn-action {
            color: #ff453a;
            border-color: rgba(255, 69, 58, 0.2);
        }
    </style>
</head>
<body>

<h1>NEON RIDER</h1>
<div class="container">
    <div id="audioOverlay">
        <button id="audioBtn">Start Engine</button>
    </div>
    <canvas id="gameCanvas" width="500" height="450"></canvas>
    <button id="restartBtn" onclick="resetGame()">Drive Again</button>
</div>

<div class="controls-pad">
    <div class="steering-group">
        <div class="btn" id="leftBtn">←</div>
        <div class="btn" id="rightBtn">→</div>
    </div>
    <div class="speed-group">
        <div class="btn btn-action" id="brakeBtn">↓</div>
        <div class="btn btn-action" id="accelBtn">↑</div>
    </div>
</div>

<script>
window.addEventListener('load', function() {
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");
    const restartBtn = document.getElementById("restartBtn");
    const audioOverlay = document.getElementById("audioOverlay");
    const audioBtn = document.getElementById("audioBtn");

    let score = 0;
    let highScore = localStorage.getItem("neonRider_highScore") ? parseInt(localStorage.getItem("neonRider_highScore")) : 0;
    let gameOver = false;
    let gameStarted = false;
    let roadOffset = 0;
    let baseSpeed = 4;
    let currentSpeed = baseSpeed;

    let carX = 230;
    let carY = 340;
    const carW = 36;
    const carH = 66;

    let obsW = 36;
    let obsH = 66;
    let obsX = 140 + Math.random() * (220 - obsW);
    let obsY = -100;
    let obsSpeedModifier = 1;
    let obsDirection = 1;

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

    // --- STRONGER AUDIO INITIALIZATION SYSTEM ---
    let audioCtx = null;

    function initAudio() {
        if (!audioCtx) {
            audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        }
        if (audioCtx && audioCtx.state === 'suspended') {
            audioCtx.resume();
        }
    }

    // Force unlock audio context on initial menu click interaction
    audioBtn.addEventListener("click", function() {
        initAudio();
        audioOverlay.style.display = "none";
        gameStarted = true;
        // Play an initial starter chime confirmation tone
        playSound('coin'); 
    });
    audioBtn.addEventListener("touchstart", function(e) {
        e.preventDefault();
        initAudio();
        audioOverlay.style.display = "none";
        gameStarted = true;
        playSound('coin');
    });

    function playSound(type) {
        if (!audioCtx || audioCtx.state === 'suspended') return;
        try {
            let osc = audioCtx.createOscillator();
            let gain = audioCtx.createGain();
            osc.connect(gain);
            gain.connect(audioCtx.destination);

            if (type === 'coin') {
                osc.type = 'sine';
                osc.frequency.setValueAtTime(587.33, audioCtx.currentTime); // D5
                osc.frequency.setValueAtTime(880, audioCtx.currentTime + 0.08); // A5
                gain.gain.setValueAtTime(0.15, audioCtx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.25);
                osc.start();
                osc.stop(audioCtx.currentTime + 0.25);
            } else if (type === 'crash') {
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(140, audioCtx.currentTime);
                osc.frequency.linearRampToValueAtTime(30, audioCtx.currentTime + 0.45);
                gain.gain.setValueAtTime(0.4, audioCtx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.5);
                osc.start();
                osc.stop(audioCtx.currentTime + 0.5);
            }
        } catch(e) { console.log(e); }
    }

    function addEvent(id, startEvt, endEvt, setter) {
        const el = document.getElementById(id);
        if (!el) return;
        el.addEventListener(startEvt, (e) => { 
            e.preventDefault(); 
            if (audioCtx && audioCtx.state === 'suspended') audioCtx.resume();
            setter(true); 
        });
        el.addEventListener(endEvt, (e) => { 
            e.preventDefault(); 
            setter(false); 
        });
    }

    addEvent("leftBtn", "touchstart", "touchend", (v) => touchLeft = v);
    addEvent("rightBtn", "touchstart", "touchend", (v) => touchRight = v);
    addEvent("accelBtn", "touchstart", "touchend", (v) => touchAccel = v);
    addEvent("brakeBtn", "touchstart", "touchend", (v) => touchBrake = v);

    addEvent("leftBtn", "mousedown", "mouseup", (v) => touchLeft = v);
    addEvent("rightBtn", "mousedown", "mouseup", (v) => touchRight = v);
    addEvent("accelBtn", "mousedown", "mouseup", (v) => touchAccel = v);
    addEvent("brakeBtn", "mousedown", "mouseup", (v) => touchBrake = v);

    window.resetGame = function() {
        if (audioCtx && audioCtx.state === 'suspended') audioCtx.resume();
        score = 0;
        gameOver = false;
        carX = 230;
        obsY = -100;
        obsX = 140 + Math.random() * (220 - obsW);
        obsSpeedModifier = 1;
        coinY = -300;
        coinX = 140 + Math.random() * (220 - coinSize);
        baseSpeed = 4;
        restartBtn.style.display = "none";
        gameLoop();
    };

    function gameLoop() {
        if (!gameStarted) {
            requestAnimationFrame(gameLoop);
            return;
        }

        if (gameOver) {
            ctx.fillStyle = "rgba(10, 11, 21, 0.95)";
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            ctx.fillStyle = "#ffffff";
            ctx.font = "600 26px sans-serif";
            ctx.textAlign = "center";
            ctx.fillText("CRASH DETECTED", canvas.width / 2, canvas.height / 2 - 40);
            
            ctx.fillStyle = "rgba(255,255,255,0.7)";
            ctx.font = "16px sans-serif";
            ctx.fillText("Score: " + score, canvas.width / 2, canvas.height / 2);
            
            ctx.fillStyle = "#00fff2";
            ctx.font = "bold 16px sans-serif";
            ctx.fillText("BEST RUN: " + highScore, canvas.width / 2, canvas.height / 2 + 30);
            
            restartBtn.style.display = "block";
            return;
        }

        if (touchAccel) currentSpeed = baseSpeed * 1.8;
        else if (touchBrake) currentSpeed = baseSpeed * 0.4;
        else currentSpeed = baseSpeed;

        if (touchLeft && carX > 140) carX -= 5.5;
        if (touchRight && carX < canvas.width - 140 - carW) carX += 5.5;

        roadOffset += currentSpeed;
        if (roadOffset > 60) roadOffset = 0;

        // Enemy Speed Variance & Movement Path Tracking
        obsY += currentSpeed * obsSpeedModifier;
        obsX += obsDirection * 0.8;
        if (obsX < 140 || obsX > canvas.width - 140 - obsW) {
            obsDirection *= -1;
        }

        if (obsY > canvas.height) {
            obsY = -100;
            obsX = 140 + Math.random() * (220 - obsW);
            obsSpeedModifier = 0.8 + Math.random() * 0.7; 
            baseSpeed += 0.15;
        }

        coinY += currentSpeed;
        if (coinY > canvas.height) {
            coinY = -150 - Math.random() * 250;
            coinX = 145 + Math.random() * (210 - coinSize);
        }

        buildings.forEach(b => {
            b.y += currentSpeed * 0.4;
            if (b.y > canvas.height) b.y = -b.h;
        });

        if (carX < obsX + obsW && carX + carW > obsX && carY < obsY + obsH && carY + carH > obsY) {
            gameOver = true;
            if (score > highScore) {
                highScore = score;
                localStorage.setItem("neonRider_highScore", highScore);
            }
            playSound('crash');
        }

        if (carX < coinX + coinSize && carX + carW > coinX && carY < coinY + coinSize && carY + carH > coinY) {
            score++;
            playSound('coin');
            coinY = -150 - Math.random() * 250; 
            coinX = 145 + Math.random() * (210 - coinSize);
        }

        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // --- SCENERY BUILDINGS ---
        buildings.forEach(b => {
            let drawX = b.leftSide ? b.xOffset : canvas.width - b.w - b.xOffset;
            ctx.fillStyle = "#16182c";
            ctx.fillRect(drawX, b.y, b.w, b.h);

            for (let wx = drawX + 8; wx < drawX + b.w - 8; wx += 16) {
                for (let wy = b.y + 12; wy < b.y + b.h - 12; wy += 22) {
                    if ((Math.floor(wx + wy)) % 6 !== 0) {
                        ctx.fillStyle = "rgba(255, 230, 120, 0.28)";
                    } else {
                        ctx.fillStyle = "rgba(255, 255, 255, 0.05)";
                    }
                    ctx.fillRect(wx, wy, 6, 10);
                }
            }
            ctx.fillStyle = b.accentColor;
            ctx.fillRect(drawX, b.y, b.w, 4);

            ctx.strokeStyle = "rgba(255, 255, 255, 0.25)";
            ctx.lineWidth = 1.5;
            ctx.beginPath();
            ctx.moveTo(drawX + b.w / 2, b.y);
            ctx.lineTo(drawX + b.w / 2, b.y - 14);
            ctx.stroke();

            ctx.fillStyle = (Math.floor(Date.now() / 300) % 2 === 0) ? "#ff3b30" : "rgba(255,255,255,0.05)";
            ctx.beginPath();
            ctx.arc(drawX + b.w / 2, b.y - 14, 3, 0, Math.PI * 2);
            ctx.fill();
        });

        // --- HIGHWAY CORRIDOR ---
        let roadGrad = ctx.createLinearGradient(135, 0, canvas.width - 135, 0);
        roadGrad.addColorStop(0, '#0c0e18');
        roadGrad.addColorStop(0.5, '#16192e');
        roadGrad.addColorStop(1, '#0c0e18');
        ctx.fillStyle = roadGrad;
        ctx.fillRect(135, 0, canvas.width - 270, canvas.height);

        ctx.fillStyle = "rgba(255, 255, 255, 0.08)";
        ctx.fillRect(135, 0, 3, canvas.height);
        ctx.fillRect(canvas.width - 138, 0, 3, canvas.height);

        ctx.fillStyle = "rgba(255, 255, 255, 0.18)";
        for (let i = -60; i < canvas.height; i += 60) {
            ctx.fillRect(canvas.width / 2 - 1.5, i + roadOffset, 3, 30);
        }

        // --- ENERGY COIN ---
        ctx.fillStyle = "#ffcc00";
        ctx.beginPath();
        ctx.arc(coinX + coinSize/2, coinY + coinSize/2, coinSize/2, 0, Math.PI * 2);
        ctx.fill();

        // --- PLAYER SPORTS CAR ---
        if (touchAccel) {
            ctx.fillStyle = "rgba(0, 160, 255, 0.4)";
            ctx.fillRect(carX + 5, carY + carH, 4, 12);
            ctx.fillRect(carX + carW - 9, carY + carH, 4, 12);
        }

        let carGrad = ctx.createLinearGradient(carX, carY, carX + carW, carY);
        carGrad.addColorStop(0, '#2f80ed');
        carGrad.addColorStop(1, '#00c6ff');
        ctx.fillStyle = carGrad;
        ctx.beginPath();
        ctx.moveTo(carX + 8, carY); 
        ctx.lineTo(carX + carW - 8, carY);
        ctx.lineTo(carX + carW, carY + 14); 
        ctx.lineTo(carX + carW - 1, carY + carH - 6); 
        ctx.lineTo(carX + carW - 4, carY + carH);
        ctx.lineTo(carX + 4, carY + carH);
        ctx.lineTo(carX + 1, carY + carH - 6);
        ctx.lineTo(carX, carY + 14);
        ctx.closePath();
        ctx.fill();

        ctx.fillStyle = "#090a12";
        ctx.beginPath();
        ctx.moveTo(carX + 8, carY + 16);
        ctx.lineTo(carX + carW - 8, carY + 16);
        ctx.lineTo(carX + carW - 5, carY + 38);
        ctx.lineTo(carX + 5, carY + 38);
        ctx.closePath();
        ctx.fill();

        ctx.fillStyle = "#ffffff";
        ctx.fillRect(carX + 5, carY + 1, 4, 2);
        ctx.fillRect(carX + carW - 9, carY + 1, 4, 2);

        ctx.fillStyle = touchBrake ? "#ff3b30" : "#b2131b";
        ctx.fillRect(carX + 5, carY + carH - 3, carW - 10, 2);

        // --- RIVAL WEAVING SEDAN ---
        ctx.fillStyle = "#272a3f";
        ctx.fillRect(obsX, obsY, obsW, obsH);

        ctx.fillStyle = "#0c0d15";
        ctx.fillRect(obsX + 4, obsY + 20, obsW - 8, 22);

        ctx.fillStyle = "#ff2d55";
        ctx.fillRect(obsX + 3, obsY + obsH - 3, 5, 2);
        ctx.fillRect(obsX + obsW - 8, obsY + obsH - 3, 5, 2);

        // --- HUD OVERLAY ---
        ctx.fillStyle = "rgba(10, 12, 26, 0.75)";
        ctx.fillRect(15, 15, 125, 45);
        ctx.strokeStyle = "rgba(255, 255, 255, 0.1)";
        ctx.strokeRect(15, 15, 125, 45);
        
        ctx.fillStyle = "#ffffff";
        ctx.font = "bold 11px sans-serif";
        ctx.textAlign = "left";
        ctx.fillText("SCORE: " + score, 25, 32);
        
        ctx.fillStyle = "#00fff2";
        ctx.fillText("HI-SCORE: " + highScore, 25, 48);

        requestAnimationFrame(gameLoop);
    }

    gameLoop();
});
</script>

</body>
</html>
