<!DOCTYPE html>
<html>
<head>
    <title>My Browser Car Game</title>
    <style>
        body {
            margin: 0;
            background: #222;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
        }
        canvas {
            background: #000;
            border: 4px solid #fff;
        }
    </style>
</head>
<body>

<canvas id="gameCanvas" width="699" height="400"></canvas>

<script>
const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

// Car properties
let carX = 250;
let carY = 100;
const carWidth = 50;
const carHeight = 30;
const speed = 5;

// Track keyboard inputs
const keys = {};
window.addEventListener("keydown", e => keys[e.key] = true);
window.addEventListener("keyup", e => keys[e.key] = false);

// Main Game Loop
function gameLoop() {
    // 1. Handle Movement
    if (keys["ArrowLeft"] || keys["a"]) carX -= speed;
    if (keys["ArrowRight"] || keys["d"]) carX += speed;
    if (keys["ArrowUp"] || keys["w"]) carY -= speed;
    if (keys["ArrowDown"] || keys["s"]) carY += speed;

    // 2. Clear Screen
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // 3. Draw Car (A sleek red rectangle for now)
    ctx.fillStyle = "red";
    ctx.fillRect(carX, carY, carWidth, carHeight);

    // Keep the loop running smoothly
    requestAnimationFrame(gameLoop);
}

// Start the game
gameLoop();
</script>

</body>
</html>
