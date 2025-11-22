<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Airplane Runner</title>
<style>
    body {
        margin: 0;
        background: #f5f5f5;
        overflow: hidden;
    }
    canvas {
        display: block;
        margin: 0 auto;
        background: #ffffff;
    }
</style>
</head>
<body>

<canvas id="game" width="900" height="300"></canvas>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");

// Game variables
let planeY = 220;
let planeVY = 0;
const gravity = 0.6;
let obstacles = [];
let speed = 6;
let score = 0;
let gameOver = false;
let explosionFrame = 0;
let show911 = false;

// Draw ground
function drawGround() {
    ctx.strokeStyle = "#000";
    ctx.lineWidth = 2;
    ctx.beginPath();
    ctx.moveTo(0, 250);
    ctx.lineTo(canvas.width, 250);
    ctx.stroke();
}

// Draw clouds
function drawCloud(x, y) {
    ctx.fillStyle = "#aaa";
    ctx.beginPath();
    ctx.arc(x, y, 18, 0, Math.PI * 2);
    ctx.arc(x + 25, y + 5, 14, 0, Math.PI * 2);
    ctx.arc(x - 25, y + 5, 14, 0, Math.PI * 2);
    ctx.fill();
}

// Draw airplane
function drawPlane() {
    ctx.fillStyle = "#000";
    ctx.beginPath();

    ctx.rect(80, planeY, 80, 20);

    ctx.moveTo(80, planeY);
    ctx.lineTo(65, planeY - 15);
    ctx.lineTo(80, planeY + 5);

    ctx.moveTo(160, planeY);
    ctx.lineTo(180, planeY + 10);
    ctx.lineTo(160, planeY + 20);

    ctx.rect(110, planeY - 10, 30, 10);

    ctx.fill();
}

// Draw a single tower
function drawTower(t) {
    ctx.fillStyle = "#000";
    ctx.fillRect(t.x, t.y, t.w, t.h);

    ctx.fillStyle = "#888";
    for (let i = 0; i < 6; i++) {
        ctx.fillRect(t.x + 10, t.y + i * 30 + 10, 25, 20);
    }
}

// Explosion animation
function drawExplosion() {
    if (explosionFrame > 22) return;

    ctx.fillStyle = `rgba(255, 120, 0, ${1 - explosionFrame / 22})`;
    ctx.beginPath();
    ctx.arc(140, planeY + 10, 30 + explosionFrame * 2, 0, Math.PI * 2);
    ctx.fill();

    explosionFrame++;
    requestAnimationFrame(drawExplosion);
}

// Spawn TWIN TOWERS
function spawnObstacle() {
    const height1 = 100 + Math.random() * 60;
    const height2 = height1 + (Math.random() * 20 - 10);

    // Left tower
    obstacles.push({
        x: canvas.width,
        y: 250 - height1,
        w: 45,
        h: height1,
        twin: true,
        offset: 0
    });

    // Right tower (small gap)
    obstacles.push({
        x: canvas.width + 60,
        y: 250 - height2,
        w: 45,
        h: height2,
        twin: true,
        offset: 60
    });
}

setInterval(spawnObstacle, 2200);

// Jump
document.addEventListener("keydown", () => {
    if (!gameOver) planeVY = -10;
    else location.reload();
});

// Main loop
function loop() {
    if (gameOver) {
        if (show911) {
            ctx.fillStyle = "red";
            ctx.font = "bold 60px Arial";
            ctx.fillText("9/11", canvas.width / 2 - 70, canvas.height / 2);
        }
        return;
    }

    ctx.clearRect(0, 0, canvas.width, canvas.height);

    drawGround();
    drawCloud(200, 60);
    drawCloud(500, 80);
    drawPlane();

    // Physics
    planeVY += gravity;
    planeY += planeVY;
    if (planeY > 220) {
        planeY = 220;
        planeVY = 0;
    }

    // Move towers
    obstacles.forEach((t, i) => {
        t.x -= speed;
        drawTower(t);

        // Collision
        if (
            160 > t.x &&
            80 < t.x + t.w &&
            planeY + 20 > t.y
        ) {
            gameOver = true;
            show911 = true;
            drawExplosion();
            return;
        }

        // Remove off screen
        if (t.x + t.w < 0) {
            obstacles.splice(i, 1);
            score++;
        }
    });

    // Score
    ctx.fillStyle = "#000";
    ctx.font = "20px Arial";
    ctx.fillText("SCORE: " + score, 10, 25);

    requestAnimationFrame(loop);
}

loop();

</script>

</body>
</html>
