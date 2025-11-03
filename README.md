# money.github.io
四則運算大挑戰
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>48格大富翁尋寶遊戲 (不規則環形)</title>
    
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
            width: 280px; 
            background-color: #fff;
            border: 3px solid #a07a4a;
            border-radius: 8px;
            padding: 10px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
            display: flex;
            flex-direction: column;
            overflow-y: auto;  
        }

        .center-map-area {
            flex: 1;  
            display: flex;
            justify-content: center;
            align-items: center;
            order: 2;  
            min-width: 600px;  
        }

        /* 🎯 修正: 地圖容器 - 更大的網格和背景圖 */
        .map-container {
            display: grid;
            grid-template-columns: repeat(18, 1fr); /* 更大的網格 */
            grid-template-rows: repeat(18, 1fr);   /* 更大的網格 */
            gap: 1px; /* 減小間距，讓格子更緊密 */
            padding: 10px; /* 稍微減少內邊距 */
            background-color: #c9b79c;  
            width: 100%;  
            max-width: 900px; 
            aspect-ratio: 1 / 1; 
            margin: 0;  
            position: relative;  
            border-radius: 15px; /* 更圓潤的邊角 */
            box-shadow: 0 0 25px rgba(0, 0, 0, 0.4);  
            overflow: hidden; /* 防止背景圖溢出 */

            /* 藏寶圖背景圖片 (已加回) */
            background-image: url('https://i.imgur.com/G4P440E.png'); 
            background-size: cover; 
            background-position: center;
            background-repeat: no-repeat;
        }

        /* 🎯 修正: 地圖格子樣式 - 正方形，更有大富翁感覺 */
        .cell {
            border: 2px solid #6b4c3b;  
            background-color: rgba(247, 224, 181, 0.9); /* 半透明背景，能透出地圖 */
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-size: 1.1em; /* 稍微縮小字體 */
            font-weight: bold;
            position: relative;  
            padding: 2px;
            overflow: hidden;  
            border-radius: 4px; /* 方形格子，略帶圓角 */
            aspect-ratio: 1 / 1; /* 正方形 */
            z-index: 2;  
            box-shadow: 0 4px 8px rgba(0,0,0,0.3); /* 更明顯的陰影 */
            color: #4a4a4a;  
            transition: all 0.2s;  
            min-height: 30px; /* 確保最小尺寸 */
            max-width: 50px; /* 確保最大尺寸 */
            margin: 0; /* 移除自動外邊距 */

            /* 大富翁地皮效果 */
            background: linear-gradient(to bottom right, rgba(247, 224, 181, 0.95), rgba(230, 200, 160, 0.9));
            border-image: linear-gradient(45deg, #a07a4a, #6b4c3b) 1;
            border-width: 3px;
            border-style: solid;
            box-sizing: border-box; /* 確保 padding 和 border 在寬高內 */
        }

        /* 點數顏色標記 */
        .cell-points-1 { background-color: rgba(200, 230, 201, 0.9); }  
        .cell-points-2 { background-color: rgba(255, 249, 196, 0.9); }  
        .cell-points-3 { background-color: rgba(255, 204, 188, 0.9); }  
        .cell-points-4 { background-color: rgba(251, 214, 172, 0.9); } 
        .cell-points-5 { background-color: rgba(255, 224, 130, 0.9); } 

        /* 終點寶藏箱樣式 */
        #cell-47 { /* P47 仍然是大寶藏 */
            background-color: gold;  
            font-size: 2.2em;  
            box-shadow: 0 0 20px rgba(255, 215, 0, 0.8);  
            transform: none;  
            border-radius: 10px; /* 寶藏格子的圓角可以大一點 */
        }
        #cell-47::before { 
            content: "💰";  
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 1.5em;
        }

        /* 路徑 SVG 樣式 */
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
            stroke-width: 8px; /* 增加線條粗細 */
            stroke-linecap: round; /* 圓頭 */
            stroke-linejoin: round; /* 圓角 */
            filter: drop-shadow(1px 1px 3px rgba(0,0,0,0.6));  
        }

        /* 角色標記 (Token) 相關 */
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
            width: 25px; /* 稍微放大 token */
            height: 25px;
            line-height: 25px;
            border-radius: 50%;  
            font-size: 0.9em;  
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

        /* 玩家控制項放大 */
        .player-control {
            display: flex;
            flex-direction: column; 
            align-items: flex-start;
            gap: 10px; 
            padding: 15px 10px;
            font-size: 1.2em; 
            border-left: 5px solid;
            border-radius: 5px;
        }
        .player-control strong {
            font-size: 1.5em; 
            display: block;
            margin-bottom: 5px;
        }
        .player-control button[id^="roll-btn-"] {
            padding: 20px 30px; 
            font-size: 1.5em; 
            font-weight: bold;
            min-width: 150px; 
            background-color: #e74c3c;
            color: white;
            border: 3px solid #c0392b;
            box-shadow: 0 4px #c0392b;
            transform: translateY(0);
            border-radius: 8px;
            cursor: pointer;
            margin-top: 5px;
            transition: all 0.2s;
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
        .player-control button.remove-player-btn {
            padding: 10px 15px;
            font-size: 1.1em;
            background-color: #f1f1f1;
            color: #7f8c8d;
            border: 1px solid #ccc;
            border-radius: 8px;
            cursor: pointer;
            margin-top: 5px;
        }

        /* 骰子/點數 滾動動畫疊層 */
        #overlay-layer {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.7);  
            display: none; 
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

        /* 寶藏訊息樣式 */
        #win-animation-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.8);
            display: none; 
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
            align-items: flex-start; 
        }
        #individual-score-container h3 {
             font-size: 1.5em; 
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
        <h1>🎲 48格尋寶大富翁 (不規則環形模式)</h1>
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
        // ====== 1. 資料初始化：閉環、48格、5點寶藏 ======

        const TOTAL_CELLS = 48; 
        const TREASURE_PATH_ID = 47; // 大寶藏的 Path ID (最後一格)

        // 點數分配 (1, 2, 3, 4 點共 47 格)
        const pointsDistribution = [
            1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 
            2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 2, 
            3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 
            4, 4, 4, 4, 4, 4, 4, 4, 4, 4, 4     
        ];

        pointsDistribution.sort(() => Math.random() - 0.5);

        const mapPoints = new Array(TOTAL_CELLS).fill(0);
        for (let i = 0; i < TOTAL_CELLS - 1; i++) {
            mapPoints[i] = pointsDistribution[i];
        }
        mapPoints[TREASURE_PATH_ID] = 5; // 🎯 大寶藏 (Path ID 47) 設為 5 點

        const DICE_WAIT_MS = 600;  
        const MOVE_DELAY_MS = 250; 
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
         * 🎯 修正: GRID_MAPPING (48 格不規則大富翁環形路徑)
         * 使用 18 寬 x 18 高 的網格，手動設定 48 格在外圍，設計成不規則形狀。
         */
        const GRID_SIZE_FINAL = 18; 
        const GRID_MAPPING = [
            // 起點區 (左下角)
            { pathId: 0, row: 17, col: 1 }, // ⭐ 起點
            { pathId: 1, row: 17, col: 2 },
            { pathId: 2, row: 17, col: 3 },
            { pathId: 3, row: 17, col: 4 },
            { pathId: 4, row: 17, col: 5 },
            { pathId: 5, row: 16, col: 5 }, // 轉彎向上
            { pathId: 6, row: 15, col: 5 },
            { pathId: 7, row: 14, col: 5 },
            
            // 中段右側 (向右延伸)
            { pathId: 8, row: 14, col: 6 }, 
            { pathId: 9, row: 14, col: 7 },
            { pathId: 10, row: 14, col: 8 },
            { pathId: 11, row: 14, col: 9 },
            { pathId: 12, row: 14, col: 10 },
            { pathId: 13, row: 14, col: 11 },
            { pathId: 14, row: 14, col: 12 },
            { pathId: 15, row: 14, col: 13 },
            { pathId: 16, row: 14, col: 14 },
            { pathId: 17, row: 14, col: 15 },
            { pathId: 18, row: 13, col: 15 }, // 轉彎向上
            { pathId: 19, row: 12, col: 15 },
            { pathId: 20, row: 11, col: 15 },
            { pathId: 21, row: 10, col: 15 },
            { pathId: 22, row: 9, col: 15 },
            { pathId: 23, row: 8, col: 15 },
            
            // 上方路段 (向左延伸)
            { pathId: 24, row: 8, col: 14 }, // 轉彎向左
            { pathId: 25, row: 8, col: 13 },
            { pathId: 26, row: 8, col: 12 },
            { pathId: 27, row: 8, col: 11 },
            { pathId: 28, row: 7, col: 11 }, // 稍微向內縮
            { pathId: 29, row: 6, col: 11 },
            { pathId: 30, row: 6, col: 10 },
            { pathId: 31, row: 6, col: 9 },
            { pathId: 32, row: 6, col: 8 },
            { pathId: 33, row: 5, col: 8 }, // 再次向內縮
            { pathId: 34, row: 4, col: 8 },
            { pathId: 35, row: 4, col: 7 },
            { pathId: 36, row: 4, col: 6 },
            { pathId: 37, row: 4, col: 5 },
            
            // 左側路段 (向下延伸)
            { pathId: 38, row: 5, col: 5 }, // 轉彎向下
            { pathId: 39, row: 6, col: 5 },
            { pathId: 40, row: 7, col: 4 }, // 稍微向外延伸
            { pathId: 41, row: 8, col: 4 },
            { pathId: 42, row: 9, col: 4 },
            { pathId: 43, row: 10, col: 3 }, // 向內縮
            { pathId: 44, row: 11, col: 3 },
            { pathId: 45, row: 12, col: 2 }, // 向內縮
            { pathId: 46, row: 13, col: 2 },
            { pathId: 47, row: 14, col: 1 }  // 💰 寶藏 (與 Path ID 0 相連)
        ];


        // ====== 2. 遊戲初始化函數 ======

        function initializeDOMReferences() {
            mapContainer = document.getElementById('game-map');
            playersList = document.getElementById('players-list');
            individualScoreContainer = document.getElementById('score-info-container');
            playerNameInput = document.getElementById('playerNameInput');
            startGameButton = document.getElementById('startGameButton');
            winOverlay = document.getElementById('win-animation-overlay');  
            
            overlayLayer = document.getElementById('overlay-layer');
            displayContent = document.getElementById('display-content');

            window.addPlayer = addPlayer;
            window.startGame = startGame;
            window.playerRoll = playerRoll;
            window.removePlayer = removePlayer;  

            return mapContainer && playersList && individualScoreContainer && playerNameInput && startGameButton && winOverlay && overlayLayer && displayContent;
        }

        function renderMap() {
            if (!mapContainer) return;
            mapContainer.innerHTML = '';  
            
            GRID_MAPPING.forEach(mapping => {
                const { pathId, row, col } = mapping;
                const points = mapPoints[pathId];

                const cell = document.createElement('div');
                cell.className = 'cell';
                cell.id = `cell-${pathId}`;  
                
                // 網格定位：使用 GRID_MAPPING 定義的 Row/Col
                cell.style.gridRow = row;
                cell.style.gridColumn = col;
                
                let content = '';
                if (pathId === 0) {
                    content = '⭐'; // 起點
                } else if (pathId === TREASURE_PATH_ID) { 
                    content = ''; 
                    cell.classList.add('treasure-box');
                    cell.classList.add(`cell-points-5`); 
                } else {
                    content = points.toString();  
                    cell.classList.add(`cell-points-${points}`);  
                }
                
                cell.innerHTML = content;
                mapContainer.appendChild(cell);
            });
        }

        /**
         * 繪製路徑線條 (處理閉環)
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
            
            for (let i = 0; i < TOTAL_CELLS; i++) { 
                const currentCell = document.getElementById(`cell-${i}`);

                if (currentCell) {
                    const currentRect = currentCell.getBoundingClientRect();

                    // 計算中心點相對於 SVG 容器的座標
                    const currentX = (currentRect.left + currentRect.right) / 2 - mapRect.left;
                    const currentY = (currentRect.top + currentRect.bottom) / 2 - mapRect.top;

                    if (i === 0) {
                        dAttribute += `M ${currentX} ${currentY} `;
                    } else {
                        dAttribute += `L ${currentX} ${currentY} `;
                    }
                }
            }
            
            // 閉環處理：將 Path ID 47 連回 Path ID 0
            const startCell = document.getElementById(`cell-0`);
            if (startCell) {
                const startRect = startCell.getBoundingClientRect();
                const startX = (startRect.left + startRect.right) / 2 - mapRect.left;
                const startY = (startRect.top + startRect.bottom) / 2 - mapRect.top;
                dAttribute += `L ${startX} ${startY} `; 
            }

            path.setAttribute('d', dAttribute.trim());
            svg.appendChild(path);
        }

        function init() {
            if (initializeDOMReferences()) {
                // 修正 CSS 網格：由於 Grid Template 是 18x18，我們需要更新
                const mapContainerElement = document.getElementById('game-map');
                if(mapContainerElement) {
                    mapContainerElement.style.gridTemplateColumns = `repeat(${GRID_SIZE_FINAL}, 1fr)`;
                    mapContainerElement.style.gridTemplateRows = `repeat(${GRID_SIZE_FINAL}, 1fr)`;
                }

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

        // ====== 3. 玩家管理與流程控制 (與前次相同) ======

        function startGame() {
            if (players.length === 0) {
                alert("請至少加入一位玩家才能開始遊戲！");
                return;
            }
            gameStarted = true;
            startGameButton.disabled = true;  
            disableAllRollButtons(false);
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
            if (token) token.closest('.token-container').remove(); 

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

        // ====== 4. 遊戲核心邏輯：骰子與移動 (與前次相同) ======

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
            const contentDiv = displayContent.querySelector('#points-display');
            if (overlayLayer.style.display === 'flex' && isAnimationActive && contentDiv) {
                overlayLayer.style.display = 'none';
                displayContent.innerHTML = '';
                isAnimationActive = false;
                
                const currentPlayer = players.find(p => p.id === parseInt(contentDiv.dataset.playerId));
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

        function showPointsAnimation(player, points) { 
            let message = '';
            if (player.position === TREASURE_PATH_ID) {
                message = `🎉 找到寶藏！<br>最終點數：<strong>${points}</strong>`;
            } else {
                message = `最終點數：<br><strong>${points}</strong>`;
            }
            
            const pointsHtml = `<div id="points-display" data-player-id="${player.id}">${message}</div>`;
            showOverlay(pointsHtml, true); 
        }

        function showTreasureFound(player) {
            if (!winOverlay) return;

            const winMessage = document.getElementById('win-message');
            winMessage.innerHTML = `🎉 ${player.name} (${player.emoji}) 找到寶藏啦！獲得 ${mapPoints[TREASURE_PATH_ID]} 點！`; 

            winOverlay.style.display = 'flex';
            
            setTimeout(() => {
                winOverlay.style.display = 'none';
            }, 3000); 
        }

        function moveTokenSequentially(player, totalSteps, finalPosition, callback) {
            
            if (totalSteps <= 0) { 
                if (callback) callback();
                return;
            }
            
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
            
            const finalPosition = (player.position + steps) % TOTAL_CELLS;

            isAnimationActive = true;
            disableAllRollButtons(true); 

            showDiceAnimation(steps, () => {
                
                moveTokenSequentially(player, steps, finalPosition, () => {
                    
                    const newPosition = player.position;
                    const finalPoints = mapPoints[newPosition];
                    
                    player.currentTotalScore += finalPoints;
                    player.currentTotalSteps += steps;

                    recordPlayerTurn(player, steps, finalPoints);
                    updateUI(player, steps, finalPoints); 
                    
                    showPointsAnimation(player, finalPoints); 
                    
                    if (newPosition === TREASURE_PATH_ID) {
                        showTreasureFound(player); 
                    }
                }); 
            });
        }

        // ====== 5. 畫面更新與統計 (與前次相同) ======

        function updateUI(player, steps, points) { 
            renderPlayersList();
        }

        function placeToken(player) {
            const targetCell = document.getElementById(`cell-${player.position}`);
            if (!targetCell) return;
            
            const oldToken = document.querySelector(`.token[data-player-id="${player.id}"]`);
            if (oldToken) {
                const container = oldToken.closest('.token-container');
                oldToken.remove();
                if (container && container.children.length === 0) {
                    container.remove();
                }
            }

            let container = targetCell.querySelector('.token-container');
            if (!container) {
                container = document.createElement('div');
                container.className = 'token-container';
                targetCell.appendChild(container);
            }
            
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
                    <tr><th colspan="2" style="background-color: ${player.color}; color: white; text-align: center;">${player.name} (${player.emoji}) 的回合記錄 (總步數: ${player.currentTotalSteps}, 總點數: ${player.currentTotalScore})</th></tr>
                    <tr>
                        <th>步數</th>
                        <th>點數</th>
                    </tr>
                </thead>
                <tbody></tbody>
            `;

            tableContainer.appendChild(table); 
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
