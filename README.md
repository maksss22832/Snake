<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Змейка</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            touch-action: none;
        }
        
        body {
            background-color: #f0f0f0;
            font-family: Arial, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            overflow: hidden;
        }
        
        #game-container {
            position: relative;
            width: 100%;
            max-width: 400px;
            max-height: 400px;
            aspect-ratio: 1/1;
            margin: 20px auto;
        }
        
        #game-canvas {
            background-color: #222;
            border: 2px solid #333;
            display: block;
            width: 100%;
            height: 100%;
        }
        
        #score {
            font-size: 24px;
            margin: 10px 0;
            color: #333;
        }
        
        .controls {
            display: flex;
            flex-direction: column;
            align-items: center;
            margin-top: 20px;
        }
        
        .mobile-controls {
            display: none;
            width: 150px;
            height: 150px;
            position: relative;
            margin-top: 20px;
        }
        
        .mobile-btn {
            position: absolute;
            width: 50px;
            height: 50px;
            background-color: rgba(0, 0, 0, 0.3);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            font-size: 24px;
            user-select: none;
        }
        
        #up-btn {
            top: 0;
            left: 50px;
        }
        
        #down-btn {
            bottom: 0;
            left: 50px;
        }
        
        #left-btn {
            top: 50px;
            left: 0;
        }
        
        #right-btn {
            top: 50px;
            right: 0;
        }
        
        .game-over {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.8);
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            color: white;
            z-index: 10;
        }
        
        .game-over h2 {
            font-size: 32px;
            margin-bottom: 20px;
        }
        
        .restart-btn {
            padding: 10px 20px;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            font-size: 18px;
            cursor: pointer;
        }
        
        @media (max-width: 768px) {
            .mobile-controls {
                display: block;
            }
        }
    </style>
</head>
<body>
    <h1>Змейка</h1>
    <div id="score">Счет: 0</div>
    
    <div id="game-container">
        <canvas id="game-canvas"></canvas>
        
        <div class="game-over" id="game-over">
            <h2>Игра окончена!</h2>
            <p id="final-score">Ваш счет: 0</p>
            <button class="restart-btn" id="restart-btn">Играть снова</button>
        </div>
    </div>
    
    <div class="mobile-controls">
        <div class="mobile-btn" id="up-btn">↑</div>
        <div class="mobile-btn" id="down-btn">↓</div>
        <div class="mobile-btn" id="left-btn">←</div>
        <div class="mobile-btn" id="right-btn">→</div>
    </div>

    <script>
        // Инициализация canvas
        const canvas = document.getElementById('game-canvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const gameOverElement = document.getElementById('game-over');
        const finalScoreElement = document.getElementById('final-score');
        const restartBtn = document.getElementById('restart-btn');
        
        // Размеры игрового поля
        const gridSize = 20;
        let tileSize;
        let canvasSize;
        
        // Игровые переменные
        let snake = [];
        let food = {};
        let direction = 'right';
        let nextDirection = 'right';
        let score = 0;
        let gameSpeed = 150;
        let gameLoopId;
        let isGameOver = false;
        
        // Мобильные кнопки
        const upBtn = document.getElementById('up-btn');
        const downBtn = document.getElementById('down-btn');
        const leftBtn = document.getElementById('left-btn');
        const rightBtn = document.getElementById('right-btn');
        
        // Инициализация игры
        function initGame() {
            // Установка размеров canvas
            resizeCanvas();
            
            // Создание змейки
            snake = [
                {x: 5, y: 10},
                {x: 4, y: 10},
                {x: 3, y: 10}
            ];
            
            // Создание еды
            generateFood();
            
            // Сброс направления и счета
            direction = 'right';
            nextDirection = 'right';
            score = 0;
            scoreElement.textContent = `Счет: ${score}`;
            isGameOver = false;
            gameOverElement.style.display = 'none';
            
            // Запуск игрового цикла
            if (gameLoopId) clearInterval(gameLoopId);
            gameLoopId = setInterval(gameLoop, gameSpeed);
        }
        
        // Установка размеров canvas
        function resizeCanvas() {
            const container = document.getElementById('game-container');
            canvasSize = Math.min(container.clientWidth, container.clientHeight);
            canvas.width = canvasSize;
            canvas.height = canvasSize;
            tileSize = canvasSize / gridSize;
        }
        
        // Генерация еды
        function generateFood() {
            const maxPos = gridSize - 1;
            
            // Проверяем, чтобы еда не появилась на змейке
            let validPosition = false;
            let foodX, foodY;
            
            while (!validPosition) {
                foodX = Math.floor(Math.random() * maxPos);
                foodY = Math.floor(Math.random() * maxPos);
                
                validPosition = true;
                for (let segment of snake) {
                    if (segment.x === foodX && segment.y === foodY) {
                        validPosition = false;
                        break;
                    }
                }
            }
            
            food = {x: foodX, y: foodY};
        }
        
        // Основной игровой цикл
        function gameLoop() {
            if (isGameOver) return;
            
            // Обновление направления
            direction = nextDirection;
            
            // Перемещение змейки
            const head = {...snake[0]};
            
            switch (direction) {
                case 'up':
                    head.y -= 1;
                    break;
                case 'down':
                    head.y += 1;
                    break;
                case 'left':
                    head.x -= 1;
                    break;
                case 'right':
                    head.x += 1;
                    break;
            }
            
            // Проверка столкновений
            if (
                head.x < 0 || head.x >= gridSize ||
                head.y < 0 || head.y >= gridSize ||
                checkCollision(head)
            ) {
                gameOver();
                return;
            }
            
            // Добавление новой головы
            snake.unshift(head);
            
            // Проверка, съела ли змейка еду
            if (head.x === food.x && head.y === food.y) {
                score += 10;
                scoreElement.textContent = `Счет: ${score}`;
                
                // Увеличение скорости каждые 50 очков
                if (score % 50 === 0 && gameSpeed > 50) {
                    gameSpeed -= 10;
                    clearInterval(gameLoopId);
                    gameLoopId = setInterval(gameLoop, gameSpeed);
                }
                
                generateFood();
            } else {
                // Удаление хвоста, если еда не съедена
                snake.pop();
            }
            
            // Отрисовка игры
            drawGame();
        }
        
        // Проверка столкновений
        function checkCollision(head) {
            for (let i = 1; i < snake.length; i++) {
                if (head.x === snake[i].x && head.y === snake[i].y) {
                    return true;
                }
            }
            return false;
        }
        
        // Отрисовка игры
        function drawGame() {
            // Очистка canvas
            ctx.fillStyle = '#222';
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            
            // Отрисовка змейки
            for (let i = 0; i < snake.length; i++) {
                const segment = snake[i];
                
                // Голова змейки
                if (i === 0) {
                    ctx.fillStyle = '#4CAF50';
                } else {
                    ctx.fillStyle = '#8BC34A';
                }
                
                ctx.fillRect(
                    segment.x * tileSize,
                    segment.y * tileSize,
                    tileSize - 1,
                    tileSize - 1
                );
            }
            
            // Отрисовка еды
            ctx.fillStyle = '#FF5252';
            ctx.beginPath();
            ctx.arc(
                food.x * tileSize + tileSize / 2,
                food.y * tileSize + tileSize / 2,
                tileSize / 2 - 1,
                0,
                Math.PI * 2
            );
            ctx.fill();
        }
        
        // Конец игры
        function gameOver() {
            isGameOver = true;
            clearInterval(gameLoopId);
            finalScoreElement.textContent = `Ваш счет: ${score}`;
            gameOverElement.style.display = 'flex';
        }
        
        // Обработка клавиатуры
        document.addEventListener('keydown', (e) => {
            switch (e.key) {
                case 'ArrowUp':
                    if (direction !== 'down') nextDirection = 'up';
                    break;
                case 'ArrowDown':
                    if (direction !== 'up') nextDirection = 'down';
                    break;
                case 'ArrowLeft':
                    if (direction !== 'right') nextDirection = 'left';
                    break;
                case 'ArrowRight':
                    if (direction !== 'left') nextDirection = 'right';
                    break;
            }
        });
        
        // Мобильное управление
        upBtn.addEventListener('touchstart', () => {
            if (direction !== 'down') nextDirection = 'up';
        });
        
        downBtn.addEventListener('touchstart', () => {
            if (direction !== 'up') nextDirection = 'down';
        });
        
        leftBtn.addEventListener('touchstart', () => {
            if (direction !== 'right') nextDirection = 'left';
        });
        
        rightBtn.addEventListener('touchstart', () => {
            if (direction !== 'left') nextDirection = 'right';
        });
        
        // Кнопка перезапуска
        restartBtn.addEventListener('click', initGame);
        
        // Обработка изменения размера окна
        window.addEventListener('resize', () => {
            resizeCanvas();
            drawGame();
        });
        
        // Запуск игры
        initGame();
    </script>
</body>
</html>
