<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cosmic Collector</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
    <style>
        /* สไตล์ของเกม */
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background-color: #00001a; /* Dark blue space background */
            font-family: 'Press Start 2P', cursive;
            color: #fff;
            overflow: hidden;
        }

        #game-container {
            position: relative;
            border: 3px solid #fff;
            border-radius: 10px;
            box-shadow: 0 0 20px #fff;
            background-color: #000;
        }

        canvas {
            display: block;
            background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="100%" height="100%"><defs><pattern id="stars" width="50" height="50" patternUnits="userSpaceOnUse"><circle fill="white" cx="10" cy="10" r="1"/><circle fill="white" cx="30" cy="40" r="0.5"/><circle fill="white" cx="45" cy="15" r="0.8"/></pattern></defs><rect width="100%" height="100%" fill="black"/><rect width="100%" height="100%" fill="url(%23stars)"/></svg>');
        }

        #game-over-screen {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.7);
            display: none; /* Initially hidden */
            flex-direction: column;
            justify-content: center;
            align-items: center;
            text-align: center;
        }

        #game-over-screen h2 {
            font-size: 2.5em;
            color: #ff4136; /* Red */
            margin-bottom: 20px;
        }

        #final-score {
            font-size: 1.5em;
            margin-bottom: 30px;
        }

        #restart-btn {
            padding: 15px 30px;
            font-size: 1em;
            font-family: 'Press Start 2P', cursive;
            color: #000;
            background-color: #f1c40f; /* Yellow */
            border: 2px solid #fff;
            border-radius: 5px;
            cursor: pointer;
            transition: transform 0.2s;
        }
        
        #restart-btn:hover {
            transform: scale(1.1);
        }
    </style>
</head>
<body>

    <div id="game-container">
        <canvas id="gameCanvas"></canvas>
        <div id="game-over-screen">
            <h2>GAME OVER</h2>
            <p id="final-score">SCORE: 0</p>
            <button id="restart-btn">RESTART</button>
        </div>
    </div>

    <script>
        // โค้ด JavaScript สำหรับควบคุมเกม

        // --- การตั้งค่าพื้นฐาน ---
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const gameOverScreen = document.getElementById('game-over-screen');
        const finalScoreEl = document.getElementById('final-score');
        const restartBtn = document.getElementById('restart-btn');

        let canvasWidth = window.innerWidth * 0.8;
        let canvasHeight = window.innerHeight * 0.8;
        if (canvasWidth > 800) canvasWidth = 800;
        if (canvasHeight > 600) canvasHeight = 600;

        canvas.width = canvasWidth;
        canvas.height = canvasHeight;

        // --- สถานะของเกม ---
        let score = 0;
        let gameOver = false;
        let gameObjects = []; // Array to hold stars and asteroids
        let spawnTimer = 0;
        const SPAWN_RATE = 75; // Lower is faster

        // --- ผู้เล่น ---
        const player = {
            x: canvas.width / 2 - 25,
            y: canvas.height - 60,
            width: 50,
            height: 30,
            draw: function() {
                // วาดรูปยานอวกาศด้วย SVG path
                ctx.fillStyle = '#3498db'; // Blue color
                ctx.beginPath();
                ctx.moveTo(this.x + this.width / 2, this.y); // Top point
                ctx.lineTo(this.x + this.width, this.y + this.height); // Bottom right
                ctx.lineTo(this.x, this.y + this.height); // Bottom left
                ctx.closePath();
                ctx.fill();
                // วาดส่วนไฟท้าย
                ctx.fillStyle = '#f39c12'; // Orange
                ctx.fillRect(this.x + this.width / 2 - 5, this.y + this.height, 10, 10);
            }
        };

        // --- คลาสสำหรับวัตถุที่ตกลงมา ---
        class FallingObject {
            constructor(x, y, size, speed, type) {
                this.x = x;
                this.y = y;
                this.size = size;
                this.speed = speed;
                this.type = type; // 'star' or 'asteroid'
            }

            update() {
                this.y += this.speed;
            }

            draw() {
                if (this.type === 'star') {
                    // วาดดาว
                    ctx.fillStyle = '#f1c40f'; // Yellow
                    ctx.beginPath();
                    for (let i = 0; i < 5; i++) {
                        ctx.lineTo(
                            this.x + this.size * Math.cos(i * 2 * Math.PI / 5 * 2 + Math.PI / 2),
                            this.y + this.size * Math.sin(i * 2 * Math.PI / 5 * 2 + Math.PI / 2)
                        );
                        ctx.lineTo(
                            this.x + (this.size / 2) * Math.cos((i * 2 + 1) * Math.PI / 5 * 2 + Math.PI / 2),
                            this.y + (this.size / 2) * Math.sin((i * 2 + 1) * Math.PI / 5 * 2 + Math.PI / 2)
                        );
                    }
                    ctx.closePath();
                    ctx.fill();
                } else {
                    // วาดอุกกาบาต
                    ctx.fillStyle = '#95a5a6'; // Gray
                    ctx.beginPath();
                    ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
                    ctx.fill();
                }
            }
        }

        // --- ฟังก์ชันสร้างวัตถุ ---
        function spawnObject() {
            const size = Math.random() * 20 + 10; // 10 to 30
            const x = Math.random() * (canvas.width - size);
            const speed = Math.random() * 2 + 1; // 1 to 3
            const type = Math.random() > 0.3 ? 'star' : 'asteroid'; // 70% chance for star
            gameObjects.push(new FallingObject(x, -size, size, speed, type));
        }

        // --- ฟังก์ชันตรวจจับการชน ---
        function checkCollision(obj) {
            return (
                player.x < obj.x + obj.size &&
                player.x + player.width > obj.x - obj.size &&
                player.y < obj.y + obj.size &&
                player.y + player.height > obj.y - obj.size
            );
        }
        
        // --- ฟังก์ชันวาดคะแนน ---
        function drawScore() {
            ctx.fillStyle = 'white';
            ctx.font = '20px "Press Start 2P"';
            ctx.fillText('SCORE: ' + score, 10, 30);
        }

        // --- ฟังก์ชันเริ่มต้นเกมใหม่ ---
        function resetGame() {
            score = 0;
            gameOver = false;
            gameObjects = [];
            player.x = canvas.width / 2 - 25;
            gameOverScreen.style.display = 'none';
            gameLoop();
        }

        // --- Game Loop (หัวใจของเกม) ---
        function gameLoop() {
            if (gameOver) {
                // แสดงหน้าจอ Game Over
                finalScoreEl.textContent = 'SCORE: ' + score;
                gameOverScreen.style.display = 'flex';
                return;
            }

            // 1. Clear Canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // 2. Update and Draw
            // Player
            player.draw();

            // Falling Objects
            spawnTimer++;
            if (spawnTimer > SPAWN_RATE) {
                spawnObject();
                spawnTimer = 0;
            }

            for (let i = gameObjects.length - 1; i >= 0; i--) {
                const obj = gameObjects[i];
                obj.update();
                obj.draw();

                // Check for collision
                if (checkCollision(obj)) {
                    if (obj.type === 'star') {
                        score += 10;
                        gameObjects.splice(i, 1); // Remove the star
                    } else {
                        gameOver = true;
                    }
                }

                // Remove objects that are off-screen
                if (obj.y > canvas.height + obj.size) {
                    gameObjects.splice(i, 1);
                }
            }
            
            // Draw Score
            drawScore();

            // 3. Request next frame
            requestAnimationFrame(gameLoop);
        }

        // --- Event Listeners ---
        canvas.addEventListener('mousemove', (e) => {
            const rect = canvas.getBoundingClientRect();
            let mouseX = e.clientX - rect.left;
            
            // ให้ยานอยู่ตรงกลางเมาส์
            player.x = mouseX - player.width / 2;

            // ไม่ให้ยานตกขอบ
            if (player.x < 0) {
                player.x = 0;
            }
            if (player.x + player.width > canvas.width) {
                player.x = canvas.width - player.width;
            }
        });

        restartBtn.addEventListener('click', resetGame);

        // --- เริ่มเกม ---
        gameLoop();
    </script>
</body>
</html>
