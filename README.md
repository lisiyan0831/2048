<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>2048游戏</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.7.2/css/all.min.css" rel="stylesheet">
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#faa31b',
                        secondary: '#8f7a66',
                        background: '#faf8ef',
                        cellEmpty: '#cdc1b4',
                        textDark: '#776e65',
                        textLight: '#f9f6f2'
                    },
                    fontFamily: {
                        inter: ['Inter', 'sans-serif'],
                    },
                }
            }
        }
    </script>
    <style type="text/tailwindcss">
        @layer utilities {
            .content-auto {
                content-visibility: auto;
            }
            .tile-2 { background-color: #eee4da; color: #776e65; }
            .tile-4 { background-color: #ede0c8; color: #776e65; }
            .tile-8 { background-color: #f2b179; color: #f9f6f2; }
            .tile-16 { background-color: #f59563; color: #f9f6f2; }
            .tile-32 { background-color: #f67c5f; color: #f9f6f2; }
            .tile-64 { background-color: #f65e3b; color: #f9f6f2; }
            .tile-128 { background-color: #edcf72; color: #f9f6f2; }
            .tile-256 { background-color: #edcc61; color: #f9f6f2; }
            .tile-512 { background-color: #edc850; color: #f9f6f2; }
            .tile-1024 { background-color: #edc53f; color: #f9f6f2; }
            .tile-2048 { background-color: #edc22e; color: #f9f6f2; }
            .tile-super { background-color: #3c3a32; color: #f9f6f2; }
            .tile-appear { animation: appear 0.3s ease-out; }
            .tile-merge { animation: merge 0.3s ease-out; }
            .grid-container { background-color: #bbada0; }
            .grid-cell { background-color: rgba(238, 228, 218, 0.35); }
            .game-message { animation: fade-in 0.5s ease-out; }
        }

        @keyframes appear {
            0% { transform: scale(0); opacity: 0; }
            70% { transform: scale(1.1); }
            100% { transform: scale(1); opacity: 1; }
        }

        @keyframes merge {
            0% { transform: scale(1); }
            50% { transform: scale(1.2); }
            100% { transform: scale(1); }
        }

        @keyframes fade-in {
            0% { opacity: 0; }
            100% { opacity: 1; }
        }
    </style>
</head>
<body class="bg-background font-inter text-textDark min-h-screen flex flex-col items-center justify-center p-4">
    <div class="max-w-md w-full mx-auto">
        <!-- 游戏标题和分数区域 -->
        <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-4">
            <div>
                <h1 class="text-[clamp(2.5rem,5vw,3.5rem)] font-bold text-secondary mb-1">2048</h1>
                <p class="text-sm text-textDark/70 max-w-xs">通过方向键或滑动来合并相同数字的方块，尝试得到2048分！</p>
            </div>
            <div class="flex gap-2 mt-4 sm:mt-0">
                <div class="bg-secondary rounded-md text-center px-3 py-2">
                    <div class="text-xs text-textLight/70">得分</div>
                    <div id="score" class="text-xl font-bold text-textLight">0</div>
                </div>
                <div class="bg-secondary rounded-md text-center px-3 py-2">
                    <div class="text-xs text-textLight/70">最高分</div>
                    <div id="best-score" class="text-xl font-bold text-textLight">0</div>
                </div>
            </div>
        </div>

        <!-- 游戏控制按钮 -->
        <div class="flex justify-between items-center mb-4">
            <button id="new-game" class="bg-primary hover:bg-primary/90 text-white font-bold py-2 px-4 rounded-md transition-all duration-200 transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-primary/50">
                <i class="fa-solid fa-refresh mr-1"></i> 新游戏
            </button>
            <div class="text-sm text-textDark/70">
                <i class="fa-solid fa-star text-primary"></i> 合并相同数字的方块
            </div>
        </div>

        <!-- 游戏网格容器 -->
        <div class="relative">
            <div id="game-container" class="grid-container rounded-lg p-2 sm:p-3">
                <!-- 网格单元格将由JS动态生成 -->
            </div>

            <!-- 游戏结束/胜利遮罩 -->
            <div id="game-message" class="game-message absolute inset-0 bg-black/70 rounded-lg flex flex-col items-center justify-center hidden z-10">
                <p id="game-message-text" class="text-[clamp(1.5rem,3vw,2rem)] font-bold text-white mb-4"></p>
                <button id="game-message-button" class="bg-primary hover:bg-primary/90 text-white font-bold py-2 px-4 rounded-md transition-all duration-200 transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-primary/50">
                    再来一局
                </button>
            </div>
        </div>

        <!-- 游戏说明 -->
        <div class="mt-6 text-sm text-textDark/70">
            <h3 class="font-bold mb-2">游戏说明:</h3>
            <ul class="list-disc pl-5 space-y-1">
                <li>使用 <span class="font-bold">方向键</span> 移动方块</li>
                <li>在移动过程中，相同数字的方块会合并成它们的和</li>
                <li>每次移动后会随机生成一个新的2或4</li>
                <li>当无法移动时游戏结束</li>
            </ul>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            // 游戏配置
            const config = {
                gridSize: 4,
                startTiles: 2,
                winningValue: 2048
            };

            // 游戏状态
            let grid = [];
            let score = 0;
            let bestScore = localStorage.getItem('2048-best-score') || 0;
            let gameOver = false;
            let gameWon = false;
            let canMove = true;

            // DOM元素
            const gameContainer = document.getElementById('game-container');
            const scoreDisplay = document.getElementById('score');
            const bestScoreDisplay = document.getElementById('best-score');
            const newGameButton = document.getElementById('new-game');
            const gameMessage = document.getElementById('game-message');
            const gameMessageText = document.getElementById('game-message-text');
            const gameMessageButton = document.getElementById('game-message-button');

            // 初始化游戏
            function initGame() {
                // 重置游戏状态
                grid = createEmptyGrid();
                score = 0;
                gameOver = false;
                gameWon = false;
                canMove = true;

                // 更新UI
                updateScore();
                updateBestScore();
                renderGrid();
                addStartTiles();
                hideGameMessage();
            }

            // 创建空网格
            function createEmptyGrid() {
                const grid = [];
                for (let y = 0; y < config.gridSize; y++) {
                    grid[y] = [];
                    for (let x = 0; x < config.gridSize; x++) {
                        grid[y][x] = 0;
                    }
                }
                return grid;
            }

            // 渲染网格
            function renderGrid() {
                // 清空容器
                gameContainer.innerHTML = '';

                // 创建网格单元格
                for (let y = 0; y < config.gridSize; y++) {
                    for (let x = 0; x < config.gridSize; x++) {
                        // 创建网格背景单元格
                        const gridCell = document.createElement('div');
                        gridCell.classList.add('grid-cell', 'rounded-md');
                        gridCell.style.width = `calc(25% - 4px)`;
                        gridCell.style.height = `calc(25% - 4px)`;
                        gridCell.style.margin = '2px';
                        gameContainer.appendChild(gridCell);
                    }
                }

                // 设置容器为网格布局
                gameContainer.style.display = 'grid';
                gameContainer.style.gridTemplateColumns = `repeat(${config.gridSize}, 1fr)`;
                gameContainer.style.gap = '4px';
                gameContainer.style.width = '100%';
                gameContainer.style.aspectRatio = '1/1';

                // 添加方块
                addTiles();
            }

            // 添加初始方块
            function addStartTiles() {
                for (let i = 0; i < config.startTiles; i++) {
                    addRandomTile();
                }
            }

            // 添加随机方块
            function addRandomTile() {
                if (hasEmptyCell()) {
                    // 随机选择位置
                    let emptyCells = [];
                    for (let y = 0; y < config.gridSize; y++) {
                        for (let x = 0; x < config.gridSize; x++) {
                            if (grid[y][x] === 0) {
                                emptyCells.push({ x, y });
                            }
                        }
                    }

                    // 随机选择一个空位置
                    const position = emptyCells[Math.floor(Math.random() * emptyCells.length)];
                    
                    // 90%概率生成2，10%概率生成4
                    const value = Math.random() < 0.9 ? 2 : 4;
                    
                    // 设置方块值
                    grid[position.y][position.x] = value;
                    
                    // 更新UI
                    renderGrid();
                }
            }

            // 添加方块到UI
            function addTiles() {
                // 清空现有方块
                const existingTiles = gameContainer.querySelectorAll('.tile');
                existingTiles.forEach(tile => tile.remove());

                // 添加新方块
                for (let y = 0; y < config.gridSize; y++) {
                    for (let x = 0; x < config.gridSize; x++) {
                        const value = grid[y][x];
                        if (value > 0) {
                            addTile(x, y, value);
                        }
                    }
                }
            }

            // 添加单个方块到UI
            function addTile(x, y, value) {
                const tile = document.createElement('div');
                const tileValue = document.createElement('div');
                
                // 设置方块样式
                tile.classList.add('tile', `tile-${value}`, 'rounded-md', 'flex', 'items-center', 'justify-center', 'font-bold', 'tile-appear');
                tile.style.position = 'absolute';
                tile.style.transition = 'transform 0.2s ease-in-out';
                
                // 计算方块位置和大小
                const cellSize = `calc((100% - 16px) / ${config.gridSize})`;
                const margin = '2px';
                const tileSize = `calc(${cellSize} - ${margin} * 2)`;
                
                // 设置方块位置和大小
                tile.style.width = tileSize;
                tile.style.height = tileSize;
                tile.style.left = `calc(${x} * ${cellSize} + ${margin} * (${x} + 1))`;
                tile.style.top = `calc(${y} * ${cellSize} + ${margin} * (${y} + 1))`;
                
                // 设置字体大小
                let fontSize;
                if (value < 100) {
                    fontSize = 'clamp(1.5rem, 5vw, 2.5rem)';
                } else if (value < 1000) {
                    fontSize = 'clamp(1.2rem, 4vw, 2rem)';
                } else {
                    fontSize = 'clamp(1rem, 3vw, 1.5rem)';
                }
                
                // 设置方块值
                tileValue.textContent = value;
                tileValue.style.fontSize = fontSize;
                
                tile.appendChild(tileValue);
                gameContainer.appendChild(tile);
            }

            // 检查是否有空单元格
            function hasEmptyCell() {
                for (let y = 0; y < config.gridSize; y++) {
                    for (let x = 0; x < config.gridSize; x++) {
                        if (grid[y][x] === 0) {
                            return true;
                        }
                    }
                }
                return false;
            }

            // 移动方块
            function move(direction) {
                if (!canMove || gameOver || gameWon) return;
                
                let moved = false;
                let merged = Array(config.gridSize).fill().map(() => Array(config.gridSize).fill(false));
                
                // 根据方向处理移动
                switch (direction) {
                    case 'up':
                        for (let x = 0; x < config.gridSize; x++) {
                            for (let y = 1; y < config.gridSize; y++) {
                                if (grid[y][x] !== 0) {
                                    let newY = y;
                                    // 移动方块
                                    while (newY > 0 && grid[newY - 1][x] === 0) {
                                        grid[newY - 1][x] = grid[newY][x];
                                        grid[newY][x] = 0;
                                        newY--;
                                        moved = true;
                                    }
                                    // 合并方块
                                    if (newY > 0 && grid[newY - 1][x] === grid[newY][x] && !merged[newY - 1][x]) {
                                        grid[newY - 1][x] *= 2;
                                        score += grid[newY - 1][x];
                                        grid[newY][x] = 0;
                                        merged[newY - 1][x] = true;
                                        moved = true;
                                    }
                                }
                            }
                        }
                        break;
                    
                    case 'down':
                        for (let x = 0; x < config.gridSize; x++) {
                            for (let y = config.gridSize - 2; y >= 0; y--) {
                                if (grid[y][x] !== 0) {
                                    let newY = y;
                                    // 移动方块
                                    while (newY < config.gridSize - 1 && grid[newY + 1][x] === 0) {
                                        grid[newY + 1][x] = grid[newY][x];
                                        grid[newY][x] = 0;
                                        newY++;
                                        moved = true;
                                    }
                                    // 合并方块
                                    if (newY < config.gridSize - 1 && grid[newY + 1][x] === grid[newY][x] && !merged[newY + 1][x]) {
                                        grid[newY + 1][x] *= 2;
                                        score += grid[newY + 1][x];
                                        grid[newY][x] = 0;
                                        merged[newY + 1][x] = true;
                                        moved = true;
                                    }
                                }
                            }
                        }
                        break;
                    
                    case 'left':
                        for (let y = 0; y < config.gridSize; y++) {
                            for (let x = 1; x < config.gridSize; x++) {
                                if (grid[y][x] !== 0) {
                                    let newX = x;
                                    // 移动方块
                                    while (newX > 0 && grid[y][newX - 1] === 0) {
                                        grid[y][newX - 1] = grid[y][newX];
                                        grid[y][newX] = 0;
                                        newX--;
                                        moved = true;
                                    }
                                    // 合并方块
                                    if (newX > 0 && grid[y][newX - 1] === grid[y][newX] && !merged[y][newX - 1]) {
                                        grid[y][newX - 1] *= 2;
                                        score += grid[y][newX - 1];
                                        grid[y][newX] = 0;
                                        merged[y][newX - 1] = true;
                                        moved = true;
                                    }
                                }
                            }
                        }
                        break;
                    
                    case 'right':
                        for (let y = 0; y < config.gridSize; y++) {
                            for (let x = config.gridSize - 2; x >= 0; x--) {
                                if (grid[y][x] !== 0) {
                                    let newX = x;
                                    // 移动方块
                                    while (newX < config.gridSize - 1 && grid[y][newX + 1] === 0) {
                                        grid[y][newX + 1] = grid[y][newX];
                                        grid[y][newX] = 0;
                                        newX++;
                                        moved = true;
                                    }
                                    // 合并方块
                                    if (newX < config.gridSize - 1 && grid[y][newX + 1] === grid[y][newX] && !merged[y][newX + 1]) {
                                        grid[y][newX + 1] *= 2;
                                        score += grid[y][newX + 1];
                                        grid[y][newX] = 0;
                                        merged[y][newX + 1] = true;
                                        moved = true;
                                    }
                                }
                            }
                        }
                        break;
                }
                
                // 如果有移动，更新UI并添加新方块
                if (moved) {
                    updateScore();
                    addRandomTile();
                    
                    // 检查游戏状态
                    if (checkWin()) {
                        gameWon = true;
                        showGameMessage('恭喜你赢了！');
                    } else if (checkGameOver()) {
                        gameOver = true;
                        showGameMessage('游戏结束！');
                    }
                }
            }

            // 检查是否获胜
            function checkWin() {
                for (let y = 0; y < config.gridSize; y++) {
                    for (let x = 0; x < config.gridSize; x++) {
                        if (grid[y][x] >= config.winningValue) {
                            return true;
                        }
                    }
                }
                return false;
            }

            // 检查游戏是否结束
            function checkGameOver() {
                // 检查是否有空单元格
                if (hasEmptyCell()) {
                    return false;
                }
                
                // 检查是否可以合并
                for (let y = 0; y < config.gridSize; y++) {
                    for (let x = 0; x < config.gridSize; x++) {
                        const value = grid[y][x];
                        
                        // 检查右侧
                        if (x < config.gridSize - 1 && grid[y][x + 1] === value) {
                            return false;
                        }
                        
                        // 检查下方
                        if (y < config.gridSize - 1 && grid[y + 1][x] === value) {
                            return false;
                        }
                    }
                }
                
                return true;
            }

            // 更新分数
            function updateScore() {
                scoreDisplay.textContent = score;
                
                // 更新最高分
                if (score > bestScore) {
                    bestScore = score;
                    localStorage.setItem('2048-best-score', bestScore);
                    updateBestScore();
                }
            }

            // 更新最高分显示
            function updateBestScore() {
                bestScoreDisplay.textContent = bestScore;
            }

            // 显示游戏消息
            function showGameMessage(message) {
                gameMessageText.textContent = message;
                gameMessage.classList.remove('hidden');
                
                // 为再来一局按钮添加事件
                gameMessageButton.addEventListener('click', initGame, { once: true });
            }

            // 隐藏游戏消息
            function hideGameMessage() {
                gameMessage.classList.add('hidden');
            }

            // 键盘控制
            document.addEventListener('keydown', (e) => {
                if (gameOver || gameWon) return;
                
                switch (e.key) {
                    case 'ArrowUp':
                        e.preventDefault();
                        move('up');
                        break;
                    case 'ArrowDown':
                        e.preventDefault();
                        move('down');
                        break;
                    case 'ArrowLeft':
                        e.preventDefault();
                        move('left');
                        break;
                    case 'ArrowRight':
                        e.preventDefault();
                        move('right');
                        break;
                }
            });

            // 触摸控制 - 移动端支持
            let touchStartX = 0;
            let touchStartY = 0;
            let touchEndX = 0;
            let touchEndY = 0;

            gameContainer.addEventListener('touchstart', (e) => {
                touchStartX = e.touches[0].clientX;
                touchStartY = e.touches[0].clientY;
            }, { passive: true });

            gameContainer.addEventListener('touchend', (e) => {
                touchEndX = e.changedTouches[0].clientX;
                touchEndY = e.changedTouches[0].clientY;
                handleSwipe();
            }, { passive: true });

            function handleSwipe() {
                const diffX = touchEndX - touchStartX;
                const diffY = touchEndY - touchStartY;
                
                // 忽略微小移动
                if (Math.abs(diffX) < 20 && Math.abs(diffY) < 20) {
                    return;
                }
                
                // 判断方向
                if (Math.abs(diffX) > Math.abs(diffY)) {
                    // 水平滑动
                    if (diffX > 0) {
                        move('right');
                    } else {
                        move('left');
                    }
                } else {
                    // 垂直滑动
                    if (diffY > 0) {
                        move('down');
                    } else {
                        move('up');
                    }
                }
            }

            // 新游戏按钮事件
            newGameButton.addEventListener('click', initGame);

            // 初始化游戏
            initGame();
        });
    </script>
</body>
</html>
    
