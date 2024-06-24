# small-games
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Volleyball Game</title>
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: #87CEEB;
            font-family: Arial, sans-serif;
        }
        #gameContainer {
            position: relative;
        }
        canvas {
            border: 2px solid black;
        }
        #startButton, #restartButton {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            padding: 10px 20px;
            font-size: 20px;
            cursor: pointer;
        }
        #gameOverScreen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.7);
            display: none;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            color: white;
            font-size: 24px;
        }
    </style>
</head>
<body>
    <div id="gameContainer">
        <canvas id="gameCanvas" width="800" height="400"></canvas>
        <button id="startButton">Start Game</button>
        <div id="gameOverScreen">
            <p id="winnerText"></p>
            <button id="restartButton">Restart Game</button>
        </div>
    </div>
    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const startButton = document.getElementById('startButton');
        const restartButton = document.getElementById('restartButton');
        const gameOverScreen = document.getElementById('gameOverScreen');
        const winnerText = document.getElementById('winnerText');

        let gameRunning = false;

        const ball = {
            x: canvas.width / 2,
            y: canvas.height / 2,
            radius: 10,
            dx: 3,
            dy: -3,
            speed: 5
        };

        const player1 = {
            x: 50,
            y: canvas.height - 50,
            width: 40,
            height: 80,
            dy: 0,
            score: 0
        };

        const player2 = {
            x: canvas.width - 90,
            y: canvas.height - 50,
            width: 40,
            height: 80,
            dy: 0,
            score: 0
        };

        const net = {
            x: canvas.width / 2 - 2,
            y: canvas.height - 100,
            width: 4,
            height: 100
        };

        const keys = {};

        document.addEventListener('keydown', (e) => {
            keys[e.code] = true;
        });

        document.addEventListener('keyup', (e) => {
            keys[e.code] = false;
        });

        startButton.addEventListener('click', startGame);
        restartButton.addEventListener('click', restartGame);

        function drawRect(x, y, width, height, color) {
            ctx.fillStyle = color;
            ctx.fillRect(x, y, width, height);
        }

        function drawCircle(x, y, radius, color) {
            ctx.fillStyle = color;
            ctx.beginPath();
            ctx.arc(x, y, radius, 0, Math.PI * 2, false);
            ctx.closePath();
            ctx.fill();
        }

        function drawText(text, x, y, color) {
            ctx.fillStyle = color;
            ctx.font = '30px Arial';
            ctx.fillText(text, x, y);
        }

        function drawGame() {
            drawRect(0, 0, canvas.width, canvas.height, "#87CEEB");
            drawRect(0, canvas.height - 10, canvas.width, 10, "#8B4513");
            drawRect(net.x, net.y, net.width, net.height, "white");
            drawRect(player1.x, player1.y, player1.width, player1.height, "yellow");
            drawRect(player2.x, player2.y, player2.width, player2.height, "red");
            drawCircle(ball.x, ball.y, ball.radius, "white");
            drawText(player1.score, canvas.width / 4, 50, "yellow");
            drawText(player2.score, 3 * canvas.width / 4, 50, "red");
        }

        function movePlayers() {
            if (keys['KeyW'] && player1.y > 0) {
                player1.dy = -5;
            } else if (keys['KeyS'] && player1.y < canvas.height - player1.height - 10) {
                player1.dy = 5;
            } else {
                player1.dy = 0;
            }

            if (keys['ArrowUp'] && player2.y > 0) {
                player2.dy = -5;
            } else if (keys['ArrowDown'] && player2.y < canvas.height - player2.height - 10) {
                player2.dy = 5;
            } else {
                player2.dy = 0;
            }

            player1.y += player1.dy;
            player2.y += player2.dy;
        }

        function moveBall() {
            ball.x += ball.dx;
            ball.y += ball.dy;

            if (ball.y - ball.radius < 0 || ball.y + ball.radius > canvas.height - 10) {
                ball.dy = -ball.dy;
            }

            let player = (ball.x < canvas.width / 2) ? player1 : player2;
            if (
                ball.x - ball.radius < player.x + player.width &&
                ball.x + ball.radius > player.x &&
                ball.y - ball.radius < player.y + player.height &&
                ball.y + ball.radius > player.y
            ) {
                let collidePoint = (ball.y - (player.y + player.height / 2)) / (player.height / 2);
                let angleRad = (Math.PI / 4) * collidePoint;
                let direction = (ball.x < canvas.width / 2) ? 1 : -1;
                
                ball.dx = direction * Math.cos(angleRad) * ball.speed;
                ball.dy = Math.sin(angleRad) * ball.speed;

                if (direction === 1) {
                    ball.x = player.x + player.width + ball.radius;
                } else {
                    ball.x = player.x - ball.radius;
                }
            }

            if (
                ball.x + ball.radius > net.x &&
                ball.x - ball.radius < net.x + net.width &&
                ball.y + ball.radius > net.y
            ) {
                ball.dx = -ball.dx;
                ball.x = ball.x < net.x ? net.x - ball.radius : net.x + net.width + ball.radius;
            }

            if (ball.x - ball.radius < 0) {
                player2.score++;
                checkGameOver();
                resetBall();
            } else if (ball.x + ball.radius > canvas.width) {
                player1.score++;
                checkGameOver();
                resetBall();
            }
        }

        function resetBall() {
            ball.x = canvas.width / 2;
            ball.y = canvas.height / 2;
            ball.dx = (Math.random() > 0.5 ? 1 : -1) * 3;
            ball.dy = -3;
        }

        function checkGameOver() {
            if (player1.score >= 6 || player2.score >= 6) {
                gameRunning = false;
                let winner = player1.score >= 6 ? "Player 1" : "Player 2";
                winnerText.textContent = `${winner} wins!`;
                gameOverScreen.style.display = 'flex';
            }
        }

        function gameLoop() {
            if (gameRunning) {
                movePlayers();
                moveBall();
                drawGame();
                requestAnimationFrame(gameLoop);
            }
        }

        function startGame() {
            gameRunning = true;
            startButton.style.display = 'none';
            gameLoop();
        }

        function restartGame() {
            player1.score = 0;
            player2.score = 0;
            resetBall();
            gameOverScreen.style.display = 'none';
            startGame();
        }

        drawGame();
    </script>
</body>
</html>
