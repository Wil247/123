<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Simple Horizontal Ping Pong Game</title>
<style>
    body {
        margin: 0;
        padding: 0;
        font-family: 'Roboto', sans-serif;
        background-color: #f9f9f9;
    }
    .header {
        height: auto;
        text-align: center;
        padding: 8px;
        background-color: #333;
        color: #fff;
    }

    .container {
        width: 100%;
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        padding: 20px;
    }

    canvas {
        border: 1px solid #000;
        width: 70vw; /* 90% del ancho de la ventana */
        max-width: auto; /* Máximo de 800px de ancho */
        height: auto; /* Altura automática para mantener la relación de aspecto */
    }

    .button-container {
        display: flex;
        justify-content: center;
        gap: 10px;
        margin-top: 5px;
    }
    .button-container button {
        padding: 10px 20px;
        font-size: 1rem;
    }
    .score-container {
        margin-top: 2px;
        text-align: center;
    }
</style>
</head>
<body onload="adjustCellSize()">


<div class="header">
    <h1>Creado por: Wilfran Luis Decena</h1>
    <h2>Estudiante de Informática en la UASD Centro San Pedro de Macorís</h2>
    <p>Creador del juego Ping Pong</p>
</div>

<div class="container">
    <div class="score-container">
        <p>Puntos: <span id="score">0</span> acumulado de 100 para ser el vencedor.</p>
    </div>
    <canvas id="pingPongCanvas"></canvas>
    <div class="button-container">
        <button class="button" id="startButton" onclick="startStop()">Iniciar</button>
        <button class="button" id="stopButton" onclick="stopGame()" disabled>Parar</button>
        <button class="button" id="restartButton" onclick="restart()">Cargar juego</button>
    </div>
</div>

<script>
const canvas = document.getElementById("pingPongCanvas");
const ctx = canvas.getContext("2d");

const paddleWidth = canvas.width * 0.2; // Ancho de la paleta como el 20% del ancho del canvas
const paddleHeight = canvas.height * 0.03; // Altura de la paleta como el 3% del alto del canvas
const paddleSpeed = canvas.width * 0.01; // Velocidad de la paleta como el 1% del ancho del canvas
let playerX = canvas.width / 2 - paddleWidth / 2;
let ballX = canvas.width / 2;
let ballY = canvas.height / 2;
let ballRadius = Math.min(canvas.width, canvas.height) * 0.03; // Diámetro de la pelota como el 3% del menor lado del canvas
let ballSpeedX = canvas.width * 0.003; // Velocidad inicial en x como el 0.3% del ancho del canvas
let ballSpeedY = canvas.width * 0.003; // Velocidad inicial en y como el 0.3% del ancho del canvas
let gameOver = false;
let gameRunning = false;
let score = 0;
let level = 1;
let collisions = 0;

function draw() {
    // Clear canvas
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    // Draw paddle
    ctx.fillStyle = "#000";
    ctx.fillRect(playerX, canvas.height - paddleHeight, paddleWidth, paddleHeight);

    // Draw ball
    ctx.beginPath();
    ctx.arc(ballX, ballY, ballRadius, 0, Math.PI * 2);
    ctx.fillStyle = "rgba(255, 255, 255, 0.5)"; // Relleno transparente
    ctx.strokeStyle = "black"; // Borde sólido
    ctx.lineWidth = 2; // Grosor del borde
    ctx.fill();
    ctx.stroke(); // Dibujar borde
    ctx.closePath();

    // Move ball
    if (gameRunning && !gameOver) {
        ballX += ballSpeedX;
        ballY += ballSpeedY;
    }

    // Ball collision with walls
    if (ballX + ballRadius > canvas.width || ballX - ballRadius < 0) {
        ballSpeedX = -ballSpeedX;
    }
    if (ballY + ballRadius > canvas.height || ballY - ballRadius < 0) {
        ballSpeedY = -ballSpeedY;
    }

    // Ball collision with paddle
    if (
        ballY + ballRadius > canvas.height - paddleHeight &&
        ballX > playerX &&
        ballX < playerX + paddleWidth
    ) {
        ballSpeedY = -ballSpeedY;
        // Increase ball speed every 5 collisions
        score++;
        collisions++;
        if (collisions === 5) {
            ballSpeedX *= 0.5;
            ballSpeedY *= 0.5;
            collisions = 0;
        }
        document.getElementById("score").textContent = score;
        if (score % 100 === 0) {
            level++;
            alert("¡Has alcanzado el nivel " + level + "!");
        }
    }

    // Check if ball falls off the paddle
    if (ballY + ballRadius > canvas.height) {
        gameOver = true;
        ballY = canvas.height - paddleHeight - ballRadius - 1;
        alert("¡Perdiste!");
        resetGame();
    }

    requestAnimationFrame(draw);
}

// Mouse control for paddle
canvas.addEventListener("mousemove", function (e) {
    let mouseX = e.clientX - canvas.getBoundingClientRect().left;
    playerX = mouseX - paddleWidth / 2;
    if (playerX < 0) playerX = 0;
    if (playerX > canvas.width - paddleWidth) playerX = canvas.width - paddleWidth;
});

function startStop() {
    gameRunning = !gameRunning;
    if (gameRunning) {
        document.getElementById("startButton").disabled = true;
        document.getElementById("stopButton").disabled = false;
        document.getElementById("restartButton").disabled = true; // Deshabilitar reinicio durante el juego
        draw();
    } else {
        document.getElementById("startButton").disabled = false;
        document.getElementById("stopButton").disabled = true;
        document.getElementById("restartButton").disabled = false; // Habilitar reinicio al detener el juego
    }
}

function stopGame() {
    gameRunning = false;
    document.getElementById("startButton").disabled = false;
    document.getElementById("stopButton").disabled = true;
    document.getElementById("restartButton").disabled = false; // Habilitar reinicio al detener el juego
}

function resetGame() {
    playerX = canvas.width / 2 - paddleWidth / 2;
    ballX = canvas.width / 2;
    ballY = canvas.height / 2;
    ballSpeedX = canvas.width * 0.001;
    ballSpeedY = canvas.width * 0.001;
    gameOver = false;
    score = 0;
    level = 1;
    collisions = 0;
    document.getElementById("score").textContent = score;
    stopGame(); // Detener el juego automáticamente después de reiniciar
}

function restart() {
    resetGame();
    window.location.reload(); // Recargar la página
}
</script>
</body>
</html>
