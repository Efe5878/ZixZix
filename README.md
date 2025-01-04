# ZixZix
Zıplama Oyunu
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ZixZix</title>
    <style>
        body {
            margin: 0;
            overflow: hidden;
            font-family: Arial, sans-serif;
        }
        canvas {
            display: block;
            background: linear-gradient(to bottom, #87CEEB, #ffffff), url('https://upload.wikimedia.org/wikipedia/commons/4/4d/Clouds_in_blue_sky.jpg') no-repeat center center;
            background-size: cover;
        }
        .scoreboard {
            position: absolute;
            top: 10px;
            left: 10px;
            font-size: 18px;
            color: black;
        }
        .stars {
            color: gold;
            font-size: 24px;
        }
        .game-title {
            position: absolute;
            top: 10px;
            left: 10px;
            font-size: 24px;
            font-weight: bold;
            color: white;
            z-index: 100;
        }
        .game-over {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 36px;
            font-weight: bold;
            color: red;
            text-align: center;
            z-index: 200;
        }
        .game-over span {
            display: block;
            margin-top: 20px;
            font-size: 24px;
            color: black;
        }
        .new-game-button {
            display: block;
            margin: 20px auto 0;
            padding: 10px 20px;
            font-size: 18px;
            font-weight: bold;
            color: white;
            background-color: blue;
            border: none;
            border-radius: 5px;
            cursor: pointer;
        }
        .new-game-button:hover {
            background-color: darkblue;
        }
    </style>
</head>
<body>
    <div class="game-title">ZixZix</div>
    <div class="scoreboard" style="top: 40px;">
        <p>Score: <span id="score">0</span></p>
        <p>High Score: <span id="highScore">0</span></p>
        <p>Lives: <span id="lives" class="stars">★★★</span></p>
    </div>
    <canvas id="gameCanvas"></canvas>
    <div id="gameOver" class="game-over" style="display: none;">
        <p>Game Over!</p>
        <span>Your Score: <span id="finalScore"></span></span>
        <button class="new-game-button" onclick="restartGame()">New Game</button>
    </div>
    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");

        canvas.width = window.innerWidth;
        canvas.height = window.innerHeight;

        const gameOverScreen = document.getElementById("gameOver");
        const finalScoreElement = document.getElementById("finalScore");

        // Player setup
        const player = {
            x: 50,
            y: canvas.height - 50,
            width: 50,
            height: 50,
            image: new Image(),
            dy: 0,
            gravity: 0.6,
            jumpPower: -15,
            isJumping: false,
        };
        player.image.src = "https://upload.wikimedia.org/wikipedia/en/0/03/Super_Mario_Artwork.png";

        // Obstacle setup
        let obstacles = [];
        const obstacleImage = new Image();
        obstacleImage.src = "https://upload.wikimedia.org/wikipedia/commons/e/e7/Stone_texture.jpg";

        // Game variables
        let score = 0;
        let highScore = localStorage.getItem("highScore") || 0;
        let lives = 3;
        let obstacleSpeed = -5;
        let obstacleSpawnInterval = 2000;
        let gameRunning = true;

        const scoreElement = document.getElementById("score");
        const highScoreElement = document.getElementById("highScore");
        const livesElement = document.getElementById("lives");

        highScoreElement.textContent = highScore;

        function updateLivesDisplay() {
            livesElement.textContent = "★".repeat(lives) + "☆".repeat(3 - lives);
        }

        function drawPlayer() {
            if (player.image.complete && player.image.naturalWidth !== 0) {
                ctx.drawImage(player.image, player.x, player.y, player.width, player.height);
            } else {
                ctx.fillStyle = "red";
                ctx.fillRect(player.x, player.y, player.width, player.height);
            }
        }

        function drawObstacles() {
            for (const obstacle of obstacles) {
                if (obstacleImage.complete && obstacleImage.naturalWidth !== 0) {
                    ctx.drawImage(obstacleImage, obstacle.x, obstacle.y, obstacle.width, obstacle.height);
                } else {
                    ctx.fillStyle = "gray";
                    ctx.fillRect(obstacle.x, obstacle.y, obstacle.width, obstacle.height);
                }
            }
        }

        function updatePlayer() {
            if (player.isJumping) {
                player.dy += player.gravity;
                player.y += player.dy;

                if (player.y >= canvas.height - player.height) {
                    player.y = canvas.height - player.height;
                    player.isJumping = false;
                }
            }
        }

        function updateObstacles() {
            for (let i = obstacles.length - 1; i >= 0; i--) {
                const obstacle = obstacles[i];
                obstacle.x += obstacle.dx;

                if (obstacle.x + obstacle.width < 0) {
                    obstacles.splice(i, 1);
                    score++;
                    scoreElement.textContent = score;

                    if (score % 5 === 0) {
                        obstacleSpeed -= 1;
                        obstacleSpawnInterval = Math.max(500, obstacleSpawnInterval - 200);
                        clearInterval(obstacleSpawner);
                        obstacleSpawner = setInterval(spawnObstacle, obstacleSpawnInterval);
                    }

                    if (score > highScore) {
                        highScore = score;
                        highScoreElement.textContent = highScore;
                        localStorage.setItem("highScore", highScore);
                    }
                }

                if (
                    player.x < obstacle.x + obstacle.width &&
                    player.x + player.width > obstacle.x &&
                    player.y < obstacle.y + obstacle.height &&
                    player.y + player.height > obstacle.y
                ) {
                    lives--;
                    updateLivesDisplay();
                    obstacles.splice(i, 1);
                    if (lives === 0) {
                        endGame();
                    }
                }
            }
        }

        function spawnObstacle() {
            const height = 50;
            const width = 50;
            const x = canvas.width;
            const y = canvas.height - height;
            const dx = obstacleSpeed;
            obstacles.push({ x, y, width, height, dx });
        }

        function gameLoop() {
            if (!gameRunning) return;

            ctx.clearRect(0, 0, canvas.width, canvas.height);

            drawPlayer();
            drawObstacles();

            updatePlayer();
            updateObstacles();

            requestAnimationFrame(gameLoop);
        }

        function endGame() {
            clearInterval(obstacleSpawner);
            gameRunning = false;
            gameOverScreen.style.display = "block";
            finalScoreElement.textContent = score;
        }

        function restartGame() {
            gameRunning = true;
            gameOverScreen.style.display = "none";
            score = 0;
            lives = 3;
            obstacleSpeed = -5;
            obstacleSpawnInterval = 2000;
            obstacles = [];
            scoreElement.textContent = score;
            updateLivesDisplay();
            obstacleSpawner = setInterval(spawnObstacle, obstacleSpawnInterval);
            gameLoop();
        }

        window.addEventListener("keydown", (e) => {
            if (e.code === "Space" && !player.isJumping) {
                player.isJumping = true;
                player.dy = player.jumpPower;
            }
        });

        canvas.addEventListener("touchstart", () => {
            if (!player.isJumping) {
                player.isJumping = true;
                player.dy = player.jumpPower;
            }
        });

        let obstacleSpawner = setInterval(spawnObstacle, obstacleSpawnInterval);
        gameLoop();
    </script>
</body>
</html>
