# money.github.io
四則運算大挑戰
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>48格大富翁尋寶遊戲 (閉環)</title>
    
    <style>
        /* 基本重置和字體設定 */
        body {
            font-family: 'Comic Sans MS', cursive, sans-serif;  
            background-color: #f5f0e8;  
            color: #4a4a4a;  
            margin: 0;
            padding: 10px;  
            text-align: center;
            min-height: 100vh;  
            display: flex;
            flex-direction: column;
        }

        header {
            background-color: #8b4513;  
            color: white;
            padding: 15px;  
            margin-bottom: 15px;
            border-radius: 8px;  
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
        }

        .controls {
            text-align: center;
            padding: 10px;
            margin-bottom: 15px;
            background-color: #fff;
            border-radius: 8px;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
        }
        
        /* 放大輸入框和開始按鈕 */
        .controls input[type="text"], .controls button {
            padding: 15px 20px;
            font-size: 1.2em;
            border-radius: 8px;
            margin: 5px;
        }

        .main-layout-container {
            display: flex;
            flex: 1;  
            gap: 15px;
            max-width: 1400px;  
            margin: 0 auto;
            width: 100%;
        }

        .side-panel {
            width: 280px; /* 略微放大側邊欄 */
            background-color: #fff;
            border: 3px solid #a07a4a;
            border-radius: 8px;
            padding: 10px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
            display: flex;
            flex-direction: column;
            overflow-y: auto;  
        }
        #players-list-container { order: 1; }
        #score-info-container { order: 3; }

        .center-map-area {
            flex: 1;  
            display: flex;
            justify-content: center;
            align-items: center;
            order: 2;  
            min-width: 600px;  
        }

        /* 地圖容器 - 8x6 網格佈局 🎯 修正處 */
        .map-container {
            display: grid;
            grid-template-columns: repeat(8, 1fr); /* 修正: 8 列 */
            grid-template-rows: repeat(6, minmax(50px, 1fr)); /* 修正: 6 行 */
            gap: 3px; /* 略微減少間距 */
            padding: 15px;
            background-color: #c9b79c;  
            width: 100%;  
            max-width: 900px; /* 增加最大寬度 */
            aspect-ratio: 8 / 6.5; /* 調整比例 */
            margin: 0;  
            position: relative;  
            border-radius: 10px;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.3);  
        }

        /* 地圖背景圖案 (保持不變) */
        .map-container::before {
            content: "";
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 0;
            pointer-events: none;  
            background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='100' height='100'%3E%3Ctext x='50%25' y='50%25' font-size='30' text-anchor='middle' alignment-baseline='middle'%3E%F0%9F%8C%B4%3C/text%3E%3C/svg%3E"),  
                                url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='100' height='100'%3E%3Ctext x='50%25' y='50%25' font-size='30' text-anchor='middle' alignment-baseline='middle'%3E%F0%9F%92%B0%3C/text%3E%3C/svg%3E"),
                                url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='100' height='100'%3E%3Ctext x='50%25' y='50%25' font-size='30' text-anchor='middle' alignment-baseline='middle'%3E%E2%98%A0%EF%B8%8F%3C/text%3E%3C/svg%3E");
            background-repeat: repeat;
            background-size: 100px 100px, 120px 120px, 80px 80px;
            background-position: 0 0, 50px 50px, 25px 25px;
            opacity: 0.15;
            filter: sepia(0.8) grayscale(0.5);  
        }

        /* 地圖格子樣式 🎯 修正處 */
        .cell {
            border: 2px solid #6b4c3b;  
            background-color: #f7e0b5;  
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-size: 1.5em; /* 縮小字體以適應更多格子 */
            font-weight: bold;
            position: relative;  
            padding: 5px;
            overflow: hidden;  
            border-radius: 50%;  
            aspect-ratio: 1 / 1;  
            z-index: 2;  
            box-shadow: 0 2px 5px rgba(0,0,0,0.2);
            color: #4a4a4a;  
            transition: all 0.2s;  
            min-height: 40px;  
            max-width: 60px; /* 減少最大寬度 */
            margin: auto;  
        }

        /* 點數顏色標記 */
        .cell-points-1 { background-color: #c8e6c9; }  
        .cell-points-2 { background-color: #fff9c4; }  
        .cell-points-3 { background-color: #ffccbc; }  
        .cell-points-4 { background-color: #fbd6ac; } /* 新增 4 點顏色 */
        .cell-points-5 { background-color: #ffe082; } /* 🎯 5 點 (大寶藏) 顏色 */

        /* 終點寶藏箱樣式 (Path ID 47) 🎯 修正處 */
        #cell-47 { 
            background-color: gold;  
            font-size: 2.2em;  
            box-shadow: 0 0 20px rgba(255, 215, 0, 0.8);  
            transform: none;  
        }
        #cell-47::before { 
            content: "💰";  
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 2em;
        }

        /* 路徑 SVG 樣式 - 確保直角 (保持不變) */
        .path-svg {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;  
            z-index: 1;  
            overflow: hidden;  
        }
        .path-svg path {
            fill: none;
            stroke: #6b4c3b;  
            stroke-width: 6px;  
            stroke-linecap: butt;  
            stroke-linejoin: miter;  
            filter: drop-shadow(1px 1px 2px rgba(0,0,0,0.5));  
        }

        /* 角色標記 (Token) 相關 (保持不變) */
        .token-container {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);  
            display: flex;  
            justify-content: center;
            align-items: center;
            gap: 1px;  
            z-index: 10;
            pointer-events: none;  
            flex-wrap: nowrap;
        }
        .token {
            position: static;  
            width: 25px;  
            height: 25px;
            line-height: 25px;
            border-radius: 50%;  
            font-size: 1em;  
            font-weight: bold;
            color: white;
            border: 2px solid white;
            z-index: 1;  
            transition: all 0.6s ease-in-out;  
            text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
            display: flex;  
            align-items: center;
            justify-content: center;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.5);  
            flex-shrink: 0;  
        }

        /* 玩家控制項放大 🎯 修正處 */
        .player-control {
            display: flex;
            flex-direction: column; /* 垂直堆疊 */
            align-items: flex-start;
            gap: 10px; /* 增加間距 */
            padding: 15px 10px;
            font-size: 1.2em; /* 放大整個控制區的基礎字體 */
            border-radius: 5px;
        }

        /* 放大玩家名稱 */
        .player-control strong {
            font-size: 1.5em; /* 讓玩家名稱更突出 */
            display: block;
            margin-bottom: 5px;
        }

        /* 骰骰子按鈕放大 🎯 修正處 */
        .player-control button {
            padding: 10px 15px; 
            font-size: 1em;     
            border-radius: 8px; 
            cursor: pointer;
            margin-top: 5px;
            transition: all 0.2s;
        }
        .player-control button[id^="roll-btn-"] {
            padding: 20px 30px; /* 進一步放大 */
            font-size: 1.5em; /* 進一步放大 */
            font-weight: bold;
            min-width: 150px; 
            background-color: #e74c3c;
            color: white;
            border: 3px solid #c0392b;
            box-shadow: 0 4px #c0392b;
            transform: translateY(0);
        }
        .player-control button[id^="roll-btn-"]:active {
            box-shadow: 0 2px #c0392b;
            transform: translateY(2px);
        }
        .player-control button[id^="roll-btn-"]:disabled {
            background-color: #bdc3c7;
            color: #7f8c8d;
            border: 3px solid #7f8c8d;
            box-shadow: none;
            cursor: not-allowed;
            transform: translateY(0);
        }
        /* 放大移除按鈕 */
        .player-control button.remove-player-btn {
            padding: 10px 15px;
            font-size: 1.1em;
            background-color: #f1f1f1;
            color: #7f8c8d;
            border: 1px solid #ccc;
        }
        
        /* 骰子/點數 滾動動畫疊層 (保持不變) */
        #overlay-layer {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.7);  
            display: none; /* 初始隱藏 */
            justify-content: center;
            align-items: center;
            z-index: 1000;
            backdrop-filter: blur(5px);  
        }
        #dice-display {
            width: 150px;
            height: 150px;
            background-color: white;
            border-radius: 25px;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 6em;  
            color: #8b4513;
            border: 5px solid gold;
            box-shadow: 0 0 30px rgba(255, 215, 0, 0.5);
            animation: bounce 0.5s infinite alternate;  
        }
        /* 點數提示框樣式 (保持不變) */
        #points-display {
            width: 280px;  
            height: 150px;
            background-color: white;
            border-radius: 25px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-size: 2.2em;  
            color: #8b4513;
            border: 5px solid #3498db;  
            box-shadow: 0 0 30px rgba(52, 152, 219, 0.5);
            animation: fanfare 0.3s forwards;  
            padding: 10px;
        }
        #points-display strong {
            font-size: 3em;  
            color: #e74c3c;
            white-space: nowrap;  
        }
        @keyframes bounce {
            from { transform: scale(1); }
            to { transform: scale(1.1); }
        }

        /* 獲勝動畫樣式 (修改為寶藏訊息) */
        #win-animation-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.8);
            display: none; /* 初始隱藏 */
            justify-content: center;
            align-items: center;
            z-index: 200;  
        }
        #win-message {
            background: linear-gradient(45deg, gold, yellow, #ffd700);
            color: #8b4513;
            padding: 40px 60px;
            border-radius: 20px;
            font-size: 3em;
            font-weight: bold;
            box-shadow: 0 0 50px gold, 0 0 20px rgba(255, 255, 0, 0.5);
            transform: scale(0.1);
            animation: fanfare 1s forwards cubic-bezier(0.68, -0.55, 0.27, 1.55);
            text-shadow: 2px 2px 5px rgba(0,0,0,0.3);
        }
        @keyframes fanfare {
            0% { transform: scale(0.1); opacity: 0; }
            80% { transform: scale(1.2); opacity: 1; }
            100% { transform: scale(1); opacity: 1; }
        }

        /* 分數表格樣式 */
        #individual-score-container {
            display: flex;
            flex-direction: column;
            gap: 15px;
            padding: 10px;
            /* 讓表格靠左對齊，保持整潔 */
            align-items: flex-start; 
        }
        #individual-score-container h3 {
             font-size: 1.5em; /* 放大標題 */
             padding: 10px 0 5px 0;
        }
        .player-score-table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 10px;
        }
        .player-score-table th, .player-score-table td {
            border: 1px solid #ccc;
            padding: 8px;
            text-align: center;
        }
        .player-score-table thead th {
            font-size: 1.1em;
        }

    </style>
</head>
<body>

    <header>
        <h1>🎲 48格尋寶大富翁 (閉環模式)</h1>
    </header>

    <div class="controls">
        <input type="text" id="playerNameInput" placeholder="輸入玩家名稱">
        <button onclick="addPlayer()">加入玩家</button>
        <button id="startGameButton" onclick="startGame()" disabled>開始遊戲</button>
    </div>

    <div class="main-layout-container">
        
        <div class="side-panel" id="players-list-container">
            <h3>玩家控制區</h3>
            <div id="players-list">
                <p class="hint">請先加入玩家</p>
            </div>
        </div>

        <div class="center-map-area">
            <div class="map-container" id="game-map">
                </div>
        </div>

        <div class="side-panel" id="score-info-container">
            <h3>回合與點數記錄</h3>
            <div id="individual-score-container">
                <p class="hint">開始遊戲後，玩家記錄將顯示在此處。</p>
            </div>
        </div>

    </div>

    <div id="overlay-layer">
        <div id="display-content"></div>
    </div>
    
    <div id="win-animation-overlay">
        <div id="win-message"></div>
    </div>


    <script>
        // ====== 1. 資料初始化 🎯 核心修正處 ======

        const TOTAL_CELLS = 48; 
        const TREASURE_PATH_ID = 47; // 大寶藏的 Path ID
        
        // 點數分配 (1, 2, 3, 4 點共 47 格)
        const pointsDistribution = [
            1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, // 1點 x 12
            2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, // 2點 x 12
            3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, // 3點 x 12
            4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4     // 4點 x 11
        ];

        // 隨機洗牌點數
        pointsDistribution.sort(() => Math.random() - 0.5);

        // 最終 mapPoints 陣列 (總長度 48)
        const mapPoints = new Array(TOTAL_CELLS).fill(0);
        for (let i = 0; i < TOTAL_CELLS - 1; i++) {
            mapPoints[i] = pointsDistribution[i];
        }
        mapPoints[TREASURE_PATH_ID] = 5; // 🎯 大寶藏 (Path ID 47) 設為 5 點

        const DICE_WAIT_MS = 600;  
        const MOVE_DELAY_MS = 650;  
        const DICE_ANIMATION_MS = 1500;

        let players = [];  
        let nextPlayerId = 1;  
        let gameStarted = false;  
        let isAnimationActive = false;  

        let mapContainer, playersList, individualScoreContainer, playerNameInput, startGameButton, winOverlay;
        let overlayLayer, displayContent;  

        const playerEmojis = ['😀', '😎', '🤩', '🥳', '🤓', '😇', '🤠', '🤖', '👻', '👽', '🐶', '🐱'];
        const playerColors = ['#e74c3c', '#3498db', '#2ecc71', '#f39c12', '#9b59b6', '#1abc9c', '#e67e22', '#2c3e50', '#7f8c8d', '#c0392b', '#16a085', '#d35400'];  

        /**
         * 🎯 修正: GRID_ORDER 陣列長度為 48，採用 8x6 閉環螺旋路徑 (回字形)，
         * 確保 Path ID 47 (大寶藏) 位於 Grid ID 28，路徑無交叉。
         * 路徑設計：R1->, C8 Down, R6 <-, C1 Up, R2 ->, C7 Down, R5 <-, C2 Up, R3 ->, R4 <- (終點)
         */
        const GRID_ORDER = [
            // 外環 (Path 0 - 23)
            0, 1, 2, 3, 4, 5, 6, 7,        // R1 -> (P0-7)
            15, 23, 31, 39, 47,            // C8 Down (P8-12)
            46, 45, 44, 43, 42, 41, 40,    // R6 <- (P13-19)
            32, 24, 16, 8,                 // C1 Up (P20-23)
            
            // 內環 (Path 24 - 47)
            9, 10, 11, 12, 13, 14,         // R2 -> (P24-29)
            22, 30, 38,                    // C7 Down (P30-32)
            37, 36, 35, 34,                // R5 <- (P33-36)
            26, 18,                        // C2 Up (P37-38)
            17, 19, 20, 21,                // R3 -> (P39-42)
            29, 30, 31, 32, 33, 27,        // R4 <- (P43-46) 修正後的閉環路徑
            
            28                             // 🎯 P47 (大寶藏) 位於 Grid ID 28
        ];


        // ====== 2. 遊戲初始化函數 ======

        function initializeDOMReferences() {
            mapContainer = document.getElementById('game-map');
            playersList = document.getElementById('players-list');
            individualScoreContainer = document.getElementById('individual-score-container');
            playerNameInput = document.getElementById('playerNameInput');
            startGameButton = document.getElementById('startGameButton');
            winOverlay = document.getElementById('win-animation-overlay');  
            
            overlayLayer = document.getElementById('overlay-layer');
            displayContent = document.getElementById('display-content');

            // 設置全局函數 (確保按鈕點擊事件有效)
            window.addPlayer = addPlayer;
            window.startGame = startGame;
            window.playerRoll = playerRoll;
            window.removePlayer = removePlayer;  

            return mapContainer && playersList && individualScoreContainer && playerNameInput && startGameButton && winOverlay && overlayLayer && displayContent;
        }

        function renderMap() {
            if (!mapContainer) return;
            mapContainer.innerHTML = '';  
            
            mapPoints.forEach((points, pathId) => {
                const gridId = GRID_ORDER[pathId];  

                const cell = document.createElement('div');
                cell.className = 'cell';
                cell.id = `cell-${pathId}`;  
                
                // 網格定位：(R: 1-6, C: 1-8)
                const COLUMNS = 8;
                const row = Math.floor(gridId / COLUMNS);
                const col = gridId % COLUMNS;
                
                cell.style.gridRow = row + 1;
                cell.style.gridColumn = col + 1;
                
                let content = '';
                if (pathId === 0) {
                    content = '⭐'; // 起點
                } else if (pathId === TREASURE_PATH_ID) { // 🎯 修正: 檢查大寶藏 ID
                    content = ''; // 終點 (Path ID 47)
                    cell.classList.add('treasure-box');
                    cell.classList.add(`cell-points-5`); // 特殊顏色標記
                } else {
                    content = points.toString();  
                    cell.classList.add(`cell-points-${points}`);  
                }
                
                cell.innerHTML = content;
                mapContainer.appendChild(cell);
            });
        }

        /**
         * 繪製路徑線條 (保持不變)
         */
        function drawPathLines() {
            if (!mapContainer) return;

            document.querySelectorAll('.path-svg').forEach(svg => svg.remove());

            const mapRect = mapContainer.getBoundingClientRect();
            
            const svg = document.createElementNS("http://www.w3.org/2000/svg", "svg");
            svg.setAttribute('class', 'path-svg');
            svg.setAttribute('width', mapRect.width);
            svg.setAttribute('height', mapRect.height);
            svg.setAttribute('viewBox', `0 0 ${mapRect.width} ${mapRect.height}`);
            mapContainer.appendChild(svg);

            const path = document.createElementNS("http://www.w3.org/2000/svg", "path");
            let dAttribute = '';
            
            for (let i = 0; i < TOTAL_CELLS; i++) { // 🎯 修正: 遍歷所有 48 格
                const currentCell = document.getElementById(`cell-${i}`);

                if (currentCell) {
                    const currentRect = currentCell.getBoundingClientRect();

                    const currentX = (currentRect.left + currentRect.right) / 2 - mapRect.left;
                    const currentY = (currentRect.top + currentRect.bottom) / 2 - mapRect.top;

                    if (i === 0) {
                        dAttribute += `M ${currentX} ${currentY} `;
                    } else {
                        dAttribute += `L ${currentX} ${currentY} `;
                    }
                }
            }
            
            // 🎯 閉環處理：將 P47 連回 P0
            const startCell = document.getElementById(`cell-0`);
            if (startCell) {
                const startRect = startCell.getBoundingClientRect();
                const startX = (startRect.left + startRect.right) / 2 - mapRect.left;
                const startY = (startRect.top + startRect.bottom) / 2 - mapRect.top;
                dAttribute += `L ${startX} ${startY} `; // 連接終點到起點
            }


            path.setAttribute('d', dAttribute.trim());
            svg.appendChild(path);
        }

        function init() {
            if (initializeDOMReferences()) {
                renderMap();  
                
                window.addEventListener('load', () => {
                    setTimeout(() => {
                        drawPathLines();
                        players.forEach(player => placeToken(player));  
                    }, 500);  
                });

                overlayLayer.addEventListener('click', hideOverlay);
            } else {
                console.error("初始化失敗：部分 HTML 元素未載入。");
            }
        }

        document.addEventListener('DOMContentLoaded', init);

        window.addEventListener('resize', () => {
            setTimeout(() => {
                if (mapContainer && mapContainer.children.length > 0) {
                    drawPathLines();
                    players.forEach(player => placeToken(player));
                }
            }, 100);
        });

        // ====== 3. 玩家管理與流程控制 ======

        function startGame() {
            if (players.length === 0) {
                alert("請至少加入一位玩家才能開始遊戲！");
                return;
            }
            gameStarted = true;
            startGameButton.disabled = true;  
            disableAllRollButtons(false);
            // 確保第一個玩家可以擲骰子
            if (players.length > 0) {
                document.getElementById(`roll-btn-${players[0].id}`).disabled = false;
            }
        }

        function addPlayer() {
            if (!playerNameInput) return;

            const name = playerNameInput.value.trim();
            if (name === "" || players.length >= playerEmojis.length) { 
                alert("請輸入有效名字或玩家數量已達上限！");
                return;
            }
            // 確保玩家名稱不重複
            if (players.some(p => p.name === name)) {
                alert("玩家名稱已存在，請使用不同名稱！");
                return;
            }

            const usedEmojis = players.map(p => p.emoji);
            const availableEmojis = playerEmojis.filter(emoji => !usedEmojis.includes(emoji));
            
            const usedColors = players.map(p => p.color);
            const availableColors = playerColors.filter(color => !usedColors.includes(color));

            if (availableEmojis.length === 0 || availableColors.length === 0) {
                alert("已無可用的表情符號或顏色。");
                return;
            }

            const randomEmoji = availableEmojis[Math.floor(Math.random() * availableEmojis.length)];
            const randomColor = availableColors[Math.floor(Math.random() * availableColors.length)];

            const newPlayer = {
                id: nextPlayerId++,
                name: name,
                position: 0, 
                currentTotalScore: 0, 
                currentTotalSteps: 0, 
                emoji: randomEmoji, 
                color: randomColor
            };

            players.push(newPlayer);
            playerNameInput.value = '';

            renderPlayersList(); 
            createPlayerScoreTable(newPlayer); 
            placeToken(newPlayer); 
            
            startGameButton.disabled = false;
        }

        function removePlayer(playerId) {
            const playerIndex = players.findIndex(p => p.id === playerId);
            if (playerIndex === -1) return;

            const removedPlayer = players[playerIndex];
            players.splice(playerIndex, 1);

            const scoreBox = document.getElementById(`score-box-player-${playerId}`);
            if (scoreBox) scoreBox.remove();
            
            const token = document.querySelector(`.token[data-player-id="${playerId}"]`);
            if (token) token.closest('.token-container').remove(); // 移除整個容器

            renderPlayersList();
            if (players.length === 0) {
                startGameButton.disabled = true;
            }

            alert(`玩家 ${removedPlayer.name} (${removedPlayer.emoji}) 已被移除。`);
        }

        function renderPlayersList() {
            if (players.length === 0) {
                playersList.innerHTML = '<p class="hint">請先加入玩家</p>';
                return;
            }
            
            let html = '';
            players.forEach(player => {
                const isDisabled = !gameStarted || isAnimationActive; 
                
                html += `
                    <div class="player-control" data-player-id="${player.id}" style="border-left: 5px solid ${player.color};">
                        <strong>${player.name} (${player.emoji})</strong>  
                        <button id="roll-btn-${player.id}" onclick="playerRoll(${player.id})" ${isDisabled ? 'disabled' : ''}>🎲 骰骰子</button>
                        <button class="remove-player-btn" onclick="removePlayer(${player.id})">移除</button>
                    </div>
                `;
            });
            playersList.innerHTML = html;
        }

        function disableAllRollButtons(disable) {
            players.forEach(player => {
                const button = document.getElementById(`roll-btn-${player.id}`);
                if (button) button.disabled = disable;
            });
        }

        // ====== 4. 遊戲核心邏輯：骰子與移動 ======

        function getNextPlayer(currentPlayerId) {
            const currentPlayerIndex = players.findIndex(p => p.id === currentPlayerId);
            if (currentPlayerIndex === -1) return null;
            
            const nextIndex = (currentPlayerIndex + 1) % players.length;
            return players[nextIndex];
        }

        function showOverlay(contentHtml, isPermanent = false, callback) {
            isAnimationActive = true;
            disableAllRollButtons(true);
            
            displayContent.innerHTML = contentHtml;
            overlayLayer.style.display = 'flex';
            
            if (!isPermanent && callback) {
                setTimeout(callback, 500); 
            }
        }

        function hideOverlay() {
            // 只有在顯示點數提示時才允許手動關閉
            const contentDiv = displayContent.querySelector('#points-display');
            if (overlayLayer.style.display === 'flex' && isAnimationActive && contentDiv) {
                overlayLayer.style.display = 'none';
                displayContent.innerHTML = '';
                isAnimationActive = false;
                
                // 輪到下一位玩家
                const currentPlayer = players.find(p => p.id === contentDiv.dataset.playerId);
                if (currentPlayer) {
                    const nextPlayer = getNextPlayer(currentPlayer.id);
                    if (nextPlayer) {
                        document.getElementById(`roll-btn-${nextPlayer.id}`).disabled = false;
                    }
                }
                
                renderPlayersList();
            }
        }

        function showDiceAnimation(steps, callback) {
            const diceHtml = `<div id="dice-display">${Math.floor(Math.random() * 6) + 1}</div>`;
            showOverlay(diceHtml, false);

            const interval = setInterval(() => {
                document.getElementById('dice-display').textContent = Math.floor(Math.random() * 6) + 1; 
            }, 100);

            setTimeout(() => {
                clearInterval(interval);
                document.getElementById('dice-display').textContent = steps;
                
                setTimeout(() => {
                    overlayLayer.style.display = 'none';
                    displayContent.innerHTML = '';
                    if (callback) callback();
                }, DICE_WAIT_MS); 
            }, DICE_ANIMATION_MS); 
        }

        function showPointsAnimation(player, points) { // 🎯 傳入 player 參數
            let message = '';
            if (player.position === TREASURE_PATH_ID) {
                 message = `🎉 找到寶藏！<br>最終點數：<strong>${points}</strong>`;
            } else {
                 message = `最終點數：<br><strong>${points}</strong>`;
            }
            
            const pointsHtml = `<div id="points-display" data-player-id="${player.id}">${message}</div>`;
            showOverlay(pointsHtml, true); // true 意味著需要手動點擊或定時器關閉
        }
        
        // 🎯 新增: 顯示找到寶藏的訊息 (取代 showWinAnimation 的功能)
        function showTreasureFound(player) {
            if (!winOverlay) return;

            const winMessage = document.getElementById('win-message');
            winMessage.innerHTML = `🎉 ${player.name} (${player.emoji}) 找到寶藏啦！獲得 ${mapPoints[TREASURE_PATH_ID]} 點！`; 

            winOverlay.style.display = 'flex';
            
            setTimeout(() => {
                winOverlay.style.display = 'none';
                // 3秒後自動關閉寶藏訊息，但保留點數提示（需要手動點擊）
            }, 3000); 
        }

        function moveTokenSequentially(player, totalSteps, finalPosition, callback) {
            
            if (totalSteps <= 0) { 
                if (callback) callback();
                return;
            }
            
            // 🎯 修正: 閉環移動
            let nextPosition = (player.position + 1) % TOTAL_CELLS; 
            
            player.position = nextPosition;
            placeToken(player); 
            
            setTimeout(() => {
                moveTokenSequentially(player, totalSteps - 1, finalPosition, callback);
            }, MOVE_DELAY_MS); 
        }

        function playerRoll(playerId) {
            if (!gameStarted || isAnimationActive) return; 

            const player = players.find(p => p.id === playerId);
            if (!player) return;

            const steps = Math.floor(Math.random() * 6) + 1; 
            
            // 🎯 修正: 閉環最終位置
            const finalPosition = (player.position + steps) % TOTAL_CELLS;

            isAnimationActive = true;
            disableAllRollButtons(true); 

            // 1. 顯示骰子動畫
            showDiceAnimation(steps, () => {
                
                // 2. 開始逐步移動
                moveTokenSequentially(player, steps, finalPosition, () => {
                    
                    // 3. 移動結束後，計算分數並更新 UI
                    const newPosition = player.position;
                    const finalPoints = mapPoints[newPosition];
                    
                    player.currentTotalScore += finalPoints;
                    player.currentTotalSteps += steps;

                    recordPlayerTurn(player, steps, finalPoints);
                    // 🎯 修正: 不再傳遞 isFinished
                    updateUI(player, steps, finalPoints); 
                    
                    // 4. 顯示點數提示 (需手動關閉)
                    showPointsAnimation(player, finalPoints); // 傳入 player
                    
                    // 5. 🎯 新增: 處理大寶藏到達事件
                    if (newPosition === TREASURE_PATH_ID) {
                        showTreasureFound(player); 
                    }
                    
                    // 注意：hideOverlay 中將處理輪換到下一位玩家的邏輯
                }); 
            });
        }

        // ====== 5. 畫面更新與統計 ======

        function updateUI(player, steps, points) { 
            // 🎯 移除所有終點相關的 UI 邏輯
            renderPlayersList();
        }

        function placeToken(player) {
            const targetCell = document.getElementById(`cell-${player.position}`);
            if (!targetCell) return;
            
            // 1. 移除舊的 token
            const oldToken = document.querySelector(`.token[data-player-id="${player.id}"]`);
            if (oldToken) {
                const container = oldToken.closest('.token-container');
                oldToken.remove();
                if (container && container.children.length === 0) {
                    container.remove();
                }
            }

            // 2. 尋找或創建 token-container
            let container = targetCell.querySelector('.token-container');
            if (!container) {
                container = document.createElement('div');
                container.className = 'token-container';
                targetCell.appendChild(container);
            }
            
            // 3. 創建並放置新的 token
            const token = document.createElement('div');
            token.className = 'token';
            token.setAttribute('data-player-id', player.id);
            token.textContent = player.emoji;  
            token.style.backgroundColor = player.color;
            
            container.appendChild(token);
        }

        function createPlayerScoreTable(player) {
            const hint = individualScoreContainer.querySelector('.hint');
            if (hint) hint.remove();

            const tableContainer = document.createElement('div');
            tableContainer.style.border = `2px solid ${player.color}`;
            tableContainer.style.borderRadius = '5px';
            tableContainer.style.padding = '10px';
            tableContainer.id = `score-box-player-${player.id}`;

            const table = document.createElement('table');
            table.className = 'player-score-table';
            table.id = `score-table-player-${player.id}`;
            
            table.innerHTML = `
                <thead>
                    <tr><th colspan="2" style="background-color: ${player.color}; color: white; text-align: center;">${player.name} (${player.emoji}) 的回合記錄 (總步數: 0, 總點數: 0)</th></tr>
                    <tr>
                        <th>步數</th>
                        <th>點數</th>
                    </tr>
                </thead>
                <tbody></tbody>
            `;

            tableContainer.appendChild(table); // 🎯 修正: 將 table 加入 tableContainer
            individualScoreContainer.appendChild(tableContainer);
        }

        function recordPlayerTurn(player, steps, points) {
            const table = document.getElementById(`score-table-player-${player.id}`);
            if (!table) return;

            const tBody = table.querySelector('tbody');
            const tHeadRow = table.querySelector('thead tr:first-child th');
            
            tHeadRow.innerHTML = `${player.name} (${player.emoji}) 的回合記錄 (總步數: ${player.currentTotalSteps}, 總點數: ${player.currentTotalScore})`;

            const newRow = tBody.insertRow(0);  
            
            newRow.insertCell().textContent = steps;  
            newRow.insertCell().textContent = points;  
        }

    </script>
</body>
</html>
