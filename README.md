<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>テトリス風ゲーム</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&family=Noto+Sans+JP:wght@400;700&display=swap');
        
        body {
            font-family: 'Noto Sans JP', sans-serif;
            background-color: #111827;
            color: white;
            touch-action: none;
        }

        .game-title {
            font-family: 'Press Start 2P', 'Noto Sans JP', cursive;
            text-shadow: 2px 2px 0px #ef4444, -2px -2px 0px #3b82f6;
        }

        canvas {
            display: block;
            background-color: #000;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);
            border: 2px solid #374151;
            border-radius: 4px;
        }

        .control-btn {
            user-select: none;
            -webkit-user-select: none;
            touch-action: manipulation;
        }
        
        .control-btn:active {
            transform: scale(0.95);
            background-color: #4b5563;
        }
    </style>
</head>
<body class="min-h-screen flex flex-col items-center justify-center p-4">

    <div class="max-w-5xl w-full flex flex-col items-center gap-4 md:gap-6 relative">
        
        <!-- ヘッダー -->
        <div class="text-center">
            <h1 class="text-3xl md:text-4xl font-bold game-title mb-2 tracking-wider">TETRIS CLONE</h1>
            <p class="text-gray-400 text-sm hidden md:block">矢印移動、↑/Xで右回転、Zで左回転、Spaceで一気に落下、Shift/Cでホールド</p>
        </div>

        <!-- スマホ専用情報パネル (Hold, Score, Next) -->
        <div class="md:hidden flex justify-between items-stretch w-full max-w-[340px] gap-2">
            <div class="bg-gray-800 p-2 rounded border border-gray-700 flex flex-col items-center justify-center w-16">
                <span class="text-gray-400 text-[10px] font-bold">HOLD</span>
                <canvas id="hold-sp" width="60" height="60" class="w-[50px] h-[50px] mt-1"></canvas>
            </div>
            <div class="bg-gray-800 p-2 rounded border border-gray-700 flex-1 flex flex-col items-center justify-center">
                <span class="text-gray-400 text-[10px] font-bold">SCORE</span>
                <span id="score-sp" class="text-yellow-400 font-bold text-lg leading-tight">0</span>
                <div class="text-[10px] text-gray-300 mt-1">LV:<span id="level-sp">1</span> L:<span id="lines-sp">0</span></div>
            </div>
            <div class="bg-gray-800 p-2 rounded border border-gray-700 flex flex-col items-center w-16">
                <span class="text-gray-400 text-[10px] font-bold">NEXT</span>
                <canvas id="next-sp" width="60" height="180" class="w-[45px] h-[135px] mt-1"></canvas>
            </div>
        </div>

        <!-- メインゲームエリア -->
        <div class="flex flex-col md:flex-row gap-6 items-start justify-center w-full">
            
            <!-- PC用 左サイド (Hold) -->
            <div class="hidden md:flex flex-col gap-4 w-32">
                <div class="bg-gray-800 p-4 rounded-xl border border-gray-700 flex flex-col items-center">
                    <h3 class="text-gray-400 text-sm font-bold mb-2 uppercase tracking-wide">Hold</h3>
                    <canvas id="hold-pc" width="100" height="100" class="w-20 h-20 bg-transparent"></canvas>
                </div>
            </div>

            <!-- キャンバス（ゲーム画面） -->
            <div class="relative group max-w-[300px] w-full mx-auto md:mx-0">
                <canvas id="tetris" width="300" height="600" class="w-full h-auto aspect-[1/2]"></canvas>
                
                <!-- T-SPIN インジケーター (表示用) -->
                <div id="t-spin-indicator" class="absolute top-1/3 left-1/2 transform -translate-x-1/2 -translate-y-1/2 text-2xl font-black text-fuchsia-400 italic drop-shadow-[0_2px_4px_rgba(0,0,0,0.8)] opacity-0 pointer-events-none transition-all duration-300 z-20 whitespace-nowrap">
                    T-SPIN!
                </div>

                <!-- オーバーレイ -->
                <div id="game-overlay" class="absolute inset-0 bg-black/80 flex flex-col items-center justify-center rounded backdrop-blur-sm z-10 transition-opacity">
                    <h2 id="overlay-title" class="text-2xl font-bold mb-4 text-white">準備完了</h2>
                    <p id="overlay-score" class="text-lg text-yellow-400 mb-6 hidden">最終スコア: <span id="final-score">0</span></p>
                    <button id="start-btn" class="bg-blue-600 hover:bg-blue-500 text-white font-bold py-3 px-8 rounded-full shadow-lg shadow-blue-500/50 transition-all transform hover:scale-105 active:scale-95">
                        スタート
                    </button>
                </div>
            </div>

            <!-- PC用 右サイド (Next, Score, Ranking) -->
            <div class="hidden md:flex flex-col gap-4 w-48">
                <!-- 次のブロック表示 -->
                <div class="bg-gray-800 p-4 rounded-xl border border-gray-700 flex flex-col items-center">
                    <h3 class="text-gray-400 text-sm font-bold mb-2 uppercase tracking-wide">Next</h3>
                    <canvas id="next-pc" width="100" height="300" class="bg-transparent w-20 h-60"></canvas>
                </div>

                <!-- スコアボード & ランキング -->
                <div class="bg-gray-800 p-4 rounded-xl border border-gray-700 w-full flex-1 flex flex-col gap-2">
                    <div class="flex flex-col gap-2 w-full text-left">
                        <div>
                            <h3 class="text-gray-400 text-xs font-bold uppercase tracking-wide">Score</h3>
                            <p id="score" class="text-2xl font-mono font-bold text-yellow-400">0</p>
                        </div>
                        <div class="flex justify-between">
                            <div>
                                <h3 class="text-gray-400 text-xs font-bold uppercase tracking-wide">Level</h3>
                                <p id="level" class="text-xl font-mono font-bold text-white">1</p>
                            </div>
                            <div>
                                <h3 class="text-gray-400 text-xs font-bold uppercase tracking-wide">Lines</h3>
                                <p id="lines" class="text-xl font-mono font-bold text-white">0</p>
                            </div>
                        </div>
                    </div>

                    <!-- ランキングエリア -->
                    <div class="mt-2 pt-2 border-t border-gray-700 flex-1">
                        <h3 class="text-gray-400 text-xs font-bold uppercase tracking-wide mb-2 text-left flex justify-between items-center">
                            Ranking <span id="sync-status" class="w-2 h-2 rounded-full bg-red-500 inline-block"></span>
                        </h3>
                        <ul id="leaderboard-list" class="flex flex-col gap-1 w-full min-h-[80px]">
                            <li class="text-gray-500 text-xs text-center mt-2">Loading...</li>
                        </ul>
                    </div>
                </div>
                
                <!-- コントロールボタン -->
                <button id="pause-btn" class="bg-gray-700 hover:bg-gray-600 text-white font-bold py-2 px-4 rounded transition-colors w-full">
                    一時停止 (P)
                </button>
            </div>
        </div>

        <!-- モバイル用コントロールパネル -->
        <div class="md:hidden grid grid-cols-6 gap-2 w-full max-w-[340px] mt-2 pb-6">
            <!-- 1行目: 移動系 -->
            <button id="btn-left" class="control-btn bg-gray-700 p-4 rounded-lg flex items-center justify-center active:bg-gray-600 col-span-2">
                <svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18"></path></svg>
            </button>
            <button id="btn-down" class="control-btn bg-gray-700 p-4 rounded-lg flex items-center justify-center active:bg-gray-600 col-span-2">
                <svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 14l-7 7m0 0l-7-7m7 7V3"></path></svg>
            </button>
            <button id="btn-right" class="control-btn bg-gray-700 p-4 rounded-lg flex items-center justify-center active:bg-gray-600 col-span-2">
                <svg class="w-6 h-6 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14 5l7 7m0 0l-7 7m7-7H3"></path></svg>
            </button>
            
            <!-- 2行目: 回転系 (左回転、右回転) -->
            <button id="btn-rotate-left" class="control-btn bg-indigo-600 p-3 rounded-lg flex flex-col items-center justify-center active:bg-indigo-500 col-span-3 text-xs font-bold shadow-md">
                <svg class="w-5 h-5 text-white mb-1" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20 4v5h-.582m-15.356 2A8.001 8.001 0 0119.418 9H15"></path>
                </svg>
                左回転 (Z)
            </button>
            <button id="btn-rotate-right" class="control-btn bg-blue-600 p-3 rounded-lg flex flex-col items-center justify-center active:bg-blue-500 col-span-3 text-xs font-bold shadow-md">
                <svg class="w-5 h-5 text-white mb-1" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9"></path>
                </svg>
                右回転 (X/↑)
            </button>
            
            <!-- 3行目: アクション -->
            <button id="btn-hold" class="control-btn bg-yellow-600 p-3 rounded-lg flex items-center justify-center active:bg-yellow-500 col-span-3 text-sm font-bold shadow-md">
                HOLD
            </button>
            <button id="btn-drop" class="control-btn bg-red-600 p-3 rounded-lg flex items-center justify-center active:bg-red-500 col-span-3 text-sm font-bold shadow-md">
                HARD DROP
            </button>
        </div>

    </div>

    <!-- Firebase SDK と メインロジック -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, doc, setDoc, getDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // --- Firebase 設定と初期化 ---
        let firebaseConfig = null;
        try {
            if (typeof __firebase_config !== 'undefined') {
                firebaseConfig = JSON.parse(__firebase_config);
            }
        } catch (e) {
            console.error("Firebase config parse error", e);
        }
        
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'tetris-clone-default';

        let auth = null;
        let db = null;
        let currentUser = null;
        let unsubscribeScores = null;
        const syncStatus = document.getElementById('sync-status');

        if (firebaseConfig) {
            const app = initializeApp(firebaseConfig);
            auth = getAuth(app);
            db = getFirestore(app);

            const initAuth = async () => {
                try {
                    if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                        await signInWithCustomToken(auth, __initial_auth_token);
                    } else {
                        await signInAnonymously(auth);
                    }
                } catch(e) {
                    console.error("Auth Error", e);
                }
            };
            initAuth();

            onAuthStateChanged(auth, (user) => {
                currentUser = user;
                if (user) {
                    syncStatus.classList.replace('bg-red-500', 'bg-green-500');
                    setupLeaderboard();
                } else {
                    syncStatus.classList.replace('bg-green-500', 'bg-red-500');
                    if(unsubscribeScores) unsubscribeScores();
                }
            });
        } else {
            document.getElementById('leaderboard-list').innerHTML = '<li class="text-gray-500 text-xs text-center mt-2">保存機能オフライン</li>';
        }

        // --- ランキングデータの取得 ---
        function setupLeaderboard() {
            if (!currentUser || !db) return;
            const scoresRef = collection(db, 'artifacts', appId, 'public', 'data', 'scores');
            
            unsubscribeScores = onSnapshot(scoresRef, (snapshot) => {
                const scores = [];
                snapshot.forEach((doc) => {
                    scores.push({ id: doc.id, ...doc.data() });
                });
                
                scores.sort((a, b) => b.score - a.score);
                const topScores = scores.slice(0, 5);
                renderLeaderboard(topScores);
            }, (error) => {
                console.error("Error fetching scores:", error);
            });
        }

        // --- スコアの保存 ---
        async function saveScoreToFirebase(finalScore) {
            if (!currentUser || !db || finalScore <= 0) return;
            
            const userScoreRef = doc(db, 'artifacts', appId, 'public', 'data', 'scores', currentUser.uid);
            try {
                const docSnap = await getDoc(userScoreRef);
                if (docSnap.exists()) {
                    const currentData = docSnap.data();
                    if (finalScore > currentData.score) {
                        await setDoc(userScoreRef, { score: finalScore, userId: currentUser.uid, timestamp: Date.now() });
                    }
                } else {
                    await setDoc(userScoreRef, { score: finalScore, userId: currentUser.uid, timestamp: Date.now() });
                }
            } catch (e) {
                console.error("Error saving score:", e);
            }
        }

        function renderLeaderboard(scores) {
            const listEl = document.getElementById('leaderboard-list');
            if (!listEl) return;
            listEl.innerHTML = '';
            
            if (scores.length === 0) {
                listEl.innerHTML = '<li class="text-gray-500 text-xs text-center mt-2">まだスコアがありません</li>';
                return;
            }

            scores.forEach((item, index) => {
                const li = document.createElement('li');
                li.className = "flex justify-between items-center text-xs p-1 rounded bg-gray-700/30";
                const isMe = currentUser && item.userId === currentUser.uid;
                
                let rankColor = "text-gray-400";
                if (index === 0) rankColor = "text-yellow-400 font-bold";
                else if (index === 1) rankColor = "text-gray-300";
                else if (index === 2) rankColor = "text-orange-400";

                li.innerHTML = `
                    <span class="${rankColor}">
                        #${index + 1} 
                        ${isMe ? '<span class="text-[10px] text-green-400 ml-1">(You)</span>' : ''}
                    </span>
                    <span class="font-mono ${isMe ? 'text-green-400 font-bold' : 'text-white'}">${item.score}</span>
                `;
                listEl.appendChild(li);
            });
        }


        // ==========================================
        // ゲームロジック
        // ==========================================

        const COLS = 10;
        const ROWS = 20;
        const BLOCK_SIZE = 30;

        const canvas = document.getElementById('tetris');
        const ctx = canvas.getContext('2d');
        const nextCtxPC = document.getElementById('next-pc')?.getContext('2d');
        const nextCtxSP = document.getElementById('next-sp')?.getContext('2d');
        const holdCtxPC = document.getElementById('hold-pc')?.getContext('2d');
        const holdCtxSP = document.getElementById('hold-sp')?.getContext('2d');

        const COLORS = [
            null, '#06b6d4', '#3b82f6', '#f97316', '#eab308', '#22c55e', '#a855f7', '#ef4444'
        ];

        const SHAPES = [
            [],
            [[0, 0, 0, 0], [1, 1, 1, 1], [0, 0, 0, 0], [0, 0, 0, 0]], // I (type 1)
            [[2, 0, 0], [2, 2, 2], [0, 0, 0]], // J (type 2)
            [[0, 0, 3], [3, 3, 3], [0, 0, 0]], // L (type 3)
            [[4, 4], [4, 4]], // O (type 4)
            [[0, 5, 5], [5, 5, 0], [0, 0, 0]], // S (type 5)
            [[0, 6, 0], [6, 6, 6], [0, 0, 0]], // T (type 6)
            [[7, 7, 0], [0, 7, 7], [0, 0, 0]]  // Z (type 7)
        ];

        let board = [];
        let score = 0;
        let lines = 0;
        let level = 1;
        let isGameOver = false;
        let isPaused = false;
        let isPlaying = false;
        let animationId = null;

        let dropCounter = 0;
        let dropInterval = 1000;
        let lastTime = 0;

        const NEXT_COUNT = 3;
        let pieceBag = []; // 7種一巡用のバッグ
        let nextPieces = [];
        let holdPiece = null; 
        let canHold = true;
        let lastAction = null; // 'move', 'rotate', 'drop'
        
        let lockDelay = 500; 
        let lockTimer = 0;
        let isLocking = false;

        const keys = { Left: false, Right: false, Down: false };
        const DAS = 150; 
        const ARR = 50;  
        let dasTimer = 0;
        let arrTimer = 0;
        let currentDir = 0; 

        let player = {
            pos: {x: 0, y: 0},
            matrix: null,
            type: 0 
        };

        const overlay = document.getElementById('game-overlay');
        const overlayTitle = document.getElementById('overlay-title');
        const startBtn = document.getElementById('start-btn');
        const overlayScore = document.getElementById('overlay-score');
        const finalScoreSpan = document.getElementById('final-score');
        const scoreEl = document.getElementById('score');
        const levelEl = document.getElementById('level');
        const linesEl = document.getElementById('lines');
        const pauseBtn = document.getElementById('pause-btn');
        const tSpinIndicator = document.getElementById('t-spin-indicator');

        function createMatrix(w, h) {
            const matrix = [];
            while (h--) {
                matrix.push(new Array(w).fill(0));
            }
            return matrix;
        }

        function getPieceMatrix(type) {
            return SHAPES[type];
        }

        // --- 7種一巡 (7-bag) 生成 ---
        function getNextPieceType() {
            if (pieceBag.length === 0) {
                pieceBag = [1, 2, 3, 4, 5, 6, 7];
                // Fisher-Yates シャッフル
                for (let i = pieceBag.length - 1; i > 0; i--) {
                    const j = Math.floor(Math.random() * (i + 1));
                    [pieceBag[i], pieceBag[j]] = [pieceBag[j], pieceBag[i]];
                }
            }
            return pieceBag.shift();
        }

        function collide(board, player) {
            const m = player.matrix;
            const o = player.pos;
            for (let y = 0; y < m.length; ++y) {
                for (let x = 0; x < m[y].length; ++x) {
                    if (m[y][x] !== 0 &&
                       (board[y + o.y] && board[y + o.y][x + o.x]) !== 0) {
                        return true;
                    }
                }
            }
            return false;
        }

        function merge(board, player) {
            player.matrix.forEach((row, y) => {
                row.forEach((value, x) => {
                    if (value !== 0) {
                        board[y + player.pos.y][x + player.pos.x] = value;
                    }
                });
            });
        }

        // --- T-Spin 判定 ---
        function checkTSpin() {
            if (player.type !== 6) return false; // Tミノ以外は除外
            
            let cornerCount = 0;
            // Tミノの3x3ボックスの四隅
            const corners = [
                {x: 0, y: 0}, {x: 2, y: 0},
                {x: 0, y: 2}, {x: 2, y: 2}
            ];

            corners.forEach(c => {
                const checkX = player.pos.x + c.x;
                const checkY = player.pos.y + c.y;
                
                // 盤面外（壁・床）またはブロックがあるか
                if (checkX < 0 || checkX >= COLS || checkY >= ROWS || (board[checkY] && board[checkY][checkX] !== 0)) {
                    cornerCount++;
                }
            });

            return cornerCount >= 3;
        }

        function showTSpinIndicator(linesCleared) {
            if (linesCleared === 0) tSpinIndicator.innerText = "T-SPIN!";
            else if (linesCleared === 1) tSpinIndicator.innerText = "T-SPIN SINGLE!";
            else if (linesCleared === 2) tSpinIndicator.innerText = "T-SPIN DOUBLE!!";
            else if (linesCleared >= 3) tSpinIndicator.innerText = "T-SPIN TRIPLE!!!";
            
            tSpinIndicator.style.opacity = 1;
            tSpinIndicator.style.transform = "translate(-50%, -50%) scale(1.2)";
            setTimeout(() => {
                tSpinIndicator.style.opacity = 0;
                tSpinIndicator.style.transform = "translate(-50%, -50%) scale(1)";
            }, 1200);
        }

        function sweep(isTSpin) {
            let rowCount = 0;
            outer: for (let y = board.length - 1; y >= 0; --y) {
                for (let x = 0; x < board[y].length; ++x) {
                    if (board[y][x] === 0) continue outer;
                }
                const row = board.splice(y, 1)[0].fill(0);
                board.unshift(row);
                ++y;
                rowCount++;
            }

            if (rowCount > 0 || isTSpin) {
                let baseScore = 0;
                
                if (isTSpin) {
                    showTSpinIndicator(rowCount);
                    // T-Spin ボーナススコア
                    if (rowCount === 0) baseScore = 400;
                    else if (rowCount === 1) baseScore = 800;
                    else if (rowCount === 2) baseScore = 1200;
                    else if (rowCount >= 3) baseScore = 1600;
                } else {
                    // 通常スコア (ガイドライン準拠風)
                    const baseScores = [0, 100, 300, 500, 800];
                    baseScore = baseScores[rowCount] || 0;
                }

                if (baseScore > 0) {
                    score += baseScore * level;
                }
                
                if (rowCount > 0) {
                    lines += rowCount;
                    level = Math.floor(lines / 10) + 1;
                    dropInterval = Math.max(50, 1000 - (level - 1) * 100); 
                }
                updateScoreBoard();
            }
        }

        function playerDrop() {
            player.pos.y++;
            if (collide(board, player)) {
                player.pos.y--;
                if (!isLocking) {
                    isLocking = true;
                    lockTimer = 0; 
                }
            } else {
                isLocking = false;
                lockTimer = 0;
                dropCounter = 0;
                lastAction = 'drop'; // 下に移動できたらdrop扱い
            }
        }

        function playerHardDrop() {
            let dropped = false;
            while (!collide(board, player)) {
                player.pos.y++;
                dropped = true;
            }
            player.pos.y--;
            if (dropped) lastAction = 'drop';
            
            // 固定直前にT-Spin判定
            let isTSpin = false;
            if (lastAction === 'rotate' && checkTSpin()) {
                isTSpin = true;
            }
            
            merge(board, player);
            playerReset();
            sweep(isTSpin);
            dropCounter = 0;
            isLocking = false;
        }

        function playerMove(dir) {
            player.pos.x += dir;
            if (collide(board, player)) {
                player.pos.x -= dir;
            } else {
                if (isLocking) lockTimer = 0; 
                lastAction = 'move';
            }
        }

        function rotateMatrix(matrix, dir) {
            for (let y = 0; y < matrix.length; ++y) {
                for (let x = 0; x < y; ++x) {
                    [matrix[x][y], matrix[y][x]] = [matrix[y][x], matrix[x][y]];
                }
            }
            if (dir > 0) {
                matrix.forEach(row => row.reverse());
            } else {
                matrix.reverse();
            }
        }

        function playerRotate(dir) {
            const pos = { ...player.pos };
            rotateMatrix(player.matrix, dir);
            
            // T-Spinや狭い隙間に入れるための簡易ウォールキックデータ
            // 順に: そのまま, 右, 左, 上, 右上, 左上, さらに上
            const kicks = [
                {x: 0, y: 0},
                {x: 1, y: 0}, {x: -1, y: 0},
                {x: 0, y: -1},
                {x: 1, y: -1}, {x: -1, y: -1},
                {x: 0, y: -2}, {x: 1, y: -2}, {x: -1, y: -2}
            ];

            for (let i = 0; i < kicks.length; i++) {
                player.pos.x = pos.x + kicks[i].x;
                player.pos.y = pos.y + kicks[i].y;
                
                if (!collide(board, player)) {
                    // キック成功
                    if (isLocking) lockTimer = 0;
                    lastAction = 'rotate';
                    return;
                }
            }
            
            // 全てダメなら回転をキャンセル
            rotateMatrix(player.matrix, -dir);
            player.pos.x = pos.x;
            player.pos.y = pos.y;
        }

        function playerHold() {
            if (!canHold || !isPlaying || isPaused || isGameOver) return;
            
            if (holdPiece === null) {
                holdPiece = player.type;
                const type = nextPieces.shift();
                nextPieces.push(getNextPieceType());
                player.type = type;
                player.matrix = getPieceMatrix(type);
            } else {
                const temp = player.type;
                player.type = holdPiece;
                holdPiece = temp;
                player.matrix = getPieceMatrix(player.type); // 初期状態の向きに戻す
            }
            
            player.pos.y = 0;
            player.pos.x = Math.floor(COLS / 2) - Math.floor(player.matrix[0].length / 2);
            canHold = false;
            dropCounter = 0;
            isLocking = false;
            lastAction = null;
            
            drawHoldPiece();
            drawNextPieces();
        }

        function playerReset() {
            if (nextPieces.length === 0) {
                while(nextPieces.length < NEXT_COUNT) nextPieces.push(getNextPieceType());
            }
            const type = nextPieces.shift();
            nextPieces.push(getNextPieceType());
            
            player.type = type;
            player.matrix = getPieceMatrix(type);
            player.pos.y = 0;
            player.pos.x = Math.floor(COLS / 2) - Math.floor(player.matrix[0].length / 2);

            canHold = true;
            dropCounter = 0;
            isLocking = false;
            lastAction = null;

            if (collide(board, player)) {
                gameOver();
            }
            
            drawNextPieces();
            drawHoldPiece();
        }

        function drawMatrix(matrix, offset, ctxToDraw, blockSize, isGhost = false, isDark = false) {
            matrix.forEach((row, y) => {
                row.forEach((value, x) => {
                    if (value !== 0) {
                        const color = COLORS[value];
                        
                        if (isGhost) {
                            ctxToDraw.fillStyle = color + '40';
                            ctxToDraw.strokeStyle = color + '80';
                        } else if (isDark) {
                            ctxToDraw.fillStyle = '#6b7280';
                            ctxToDraw.strokeStyle = '#374151';
                        } else {
                            ctxToDraw.fillStyle = color;
                            ctxToDraw.strokeStyle = '#000';
                        }

                        const bx = (x + offset.x) * blockSize;
                        const by = (y + offset.y) * blockSize;
                        
                        ctxToDraw.fillRect(bx, by, blockSize, blockSize);
                        
                        if (!isGhost && !isDark) {
                            ctxToDraw.fillStyle = 'rgba(255,255,255,0.3)';
                            ctxToDraw.fillRect(bx, by, blockSize, 4);
                            ctxToDraw.fillRect(bx, by, 4, blockSize);
                            ctxToDraw.fillStyle = 'rgba(0,0,0,0.3)';
                            ctxToDraw.fillRect(bx, by + blockSize - 4, blockSize, 4);
                            ctxToDraw.fillRect(bx + blockSize - 4, by, 4, blockSize);
                        }
                        ctxToDraw.strokeRect(bx, by, blockSize, blockSize);
                    }
                });
            });
        }

        function drawNextPieces() {
            [nextCtxPC, nextCtxSP].forEach(ctx => {
                if (!ctx) return;
                ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);
                const isSP = ctx.canvas.width < 80;
                const blockSize = isSP ? 12 : 20; 
                
                nextPieces.forEach((type, index) => {
                    const matrix = getPieceMatrix(type);
                    const offsetX = (ctx.canvas.width / blockSize - matrix[0].length) / 2;
                    const offsetY = index * 3.5 + 0.5; 
                    drawMatrix(matrix, {x: offsetX, y: offsetY}, ctx, blockSize);
                });
            });
        }

        function drawHoldPiece() {
            [holdCtxPC, holdCtxSP].forEach(ctx => {
                if (!ctx) return;
                ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);
                if (holdPiece === null) return;

                const isSP = ctx.canvas.width < 80;
                const blockSize = isSP ? 12 : 20;
                const matrix = getPieceMatrix(holdPiece);
                const offsetX = (ctx.canvas.width / blockSize - matrix[0].length) / 2;
                const offsetY = (ctx.canvas.height / blockSize - matrix.length) / 2;
                
                drawMatrix(matrix, {x: offsetX, y: offsetY}, ctx, blockSize, false, !canHold);
            });
        }

        function draw() {
            ctx.fillStyle = '#000';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            ctx.strokeStyle = '#222';
            ctx.lineWidth = 1;
            for (let x = 0; x <= COLS; x++) {
                ctx.beginPath();
                ctx.moveTo(x * BLOCK_SIZE, 0);
                ctx.lineTo(x * BLOCK_SIZE, canvas.height);
                ctx.stroke();
            }
            for (let y = 0; y <= ROWS; y++) {
                ctx.beginPath();
                ctx.moveTo(0, y * BLOCK_SIZE);
                ctx.lineTo(canvas.width, y * BLOCK_SIZE);
                ctx.stroke();
            }

            drawMatrix(board, {x: 0, y: 0}, ctx, BLOCK_SIZE);

            if (isPlaying && !isGameOver) {
                const ghost = {
                    matrix: player.matrix,
                    pos: { x: player.pos.x, y: player.pos.y }
                };
                while (!collide(board, ghost)) {
                    ghost.pos.y++;
                }
                ghost.pos.y--;
                drawMatrix(ghost.matrix, ghost.pos, ctx, BLOCK_SIZE, true);
                drawMatrix(player.matrix, player.pos, ctx, BLOCK_SIZE);
            }
        }

        function updateScoreBoard() {
            scoreEl.innerText = score;
            levelEl.innerText = level;
            linesEl.innerText = lines;
            
            document.getElementById('score-sp').innerText = score;
            document.getElementById('level-sp').innerText = level;
            document.getElementById('lines-sp').innerText = lines;
        }

        function handleInput(deltaTime) {
            if (currentDir !== 0) {
                dasTimer += deltaTime;
                if (dasTimer >= DAS) {
                    arrTimer += deltaTime;
                    if (arrTimer >= ARR) {
                        playerMove(currentDir);
                        arrTimer = 0;
                    }
                }
            }
            if (keys.Down) {
                dropCounter += deltaTime * 20;
            }
        }

        function update(time = 0) {
            if (isPaused || isGameOver || !isPlaying) return;

            const deltaTime = time - lastTime;
            lastTime = time;

            handleInput(deltaTime);

            dropCounter += deltaTime;
            if (dropCounter > dropInterval) {
                playerDrop();
            }

            if (isLocking) {
                lockTimer += deltaTime;
                if (lockTimer >= lockDelay) {
                    let isTSpin = false;
                    if (lastAction === 'rotate' && checkTSpin()) {
                        isTSpin = true;
                    }

                    merge(board, player);
                    playerReset();
                    sweep(isTSpin);
                    isLocking = false;
                    dropCounter = 0;
                }
            }

            draw();
            animationId = requestAnimationFrame(update);
        }

        function startGame() {
            board = createMatrix(COLS, ROWS);
            score = 0;
            lines = 0;
            level = 1;
            dropInterval = 1000;
            isGameOver = false;
            isPaused = false;
            isPlaying = true;
            
            pieceBag = [];
            nextPieces = [];
            holdPiece = null;
            lastAction = null;
            
            updateScoreBoard();
            playerReset();
            
            overlay.classList.add('hidden');
            pauseBtn.innerText = "一時停止 (P)";
            
            lastTime = performance.now();
            if (animationId) cancelAnimationFrame(animationId);
            update(lastTime);
        }

        function gameOver() {
            isGameOver = true;
            isPlaying = false;
            if (animationId) cancelAnimationFrame(animationId);
            
            overlayTitle.innerText = "ゲームオーバー";
            overlayTitle.classList.replace('text-white', 'text-red-500');
            finalScoreSpan.innerText = score;
            overlayScore.classList.remove('hidden');
            startBtn.innerText = "もう一度プレイ";
            overlay.classList.remove('hidden');

            saveScoreToFirebase(score);
        }

        function togglePause() {
            if (!isPlaying || isGameOver) return;
            
            isPaused = !isPaused;
            if (isPaused) {
                if (animationId) cancelAnimationFrame(animationId);
                overlayTitle.innerText = "一時停止中";
                overlayTitle.classList.replace('text-red-500', 'text-white');
                overlayScore.classList.add('hidden');
                startBtn.innerText = "再開する";
                overlay.classList.remove('hidden');
                pauseBtn.innerText = "再開 (P)";
            } else {
                overlay.classList.add('hidden');
                pauseBtn.innerText = "一時停止 (P)";
                lastTime = performance.now();
                update(lastTime);
            }
        }

        // --- イベントリスナー ---

        document.addEventListener('keydown', event => {
            if (!isPlaying || isGameOver) {
                if (event.key === 'Enter' || event.code === 'Space') {
                    if (isPaused) togglePause();
                    else if (overlay.classList.contains('hidden') === false) startGame();
                }
                return;
            }

            switch (event.code) {
                case 'ArrowLeft':
                    if (!keys.Left) { playerMove(-1); dasTimer = 0; currentDir = -1; }
                    keys.Left = true;
                    break;
                case 'ArrowRight':
                    if (!keys.Right) { playerMove(1); dasTimer = 0; currentDir = 1; }
                    keys.Right = true;
                    break;
                case 'ArrowDown':
                    keys.Down = true;
                    break;
                case 'ArrowUp':
                case 'KeyX':
                    // 右回転 (時計回り)
                    if (!isPaused) playerRotate(1);
                    break;
                case 'KeyZ':
                case 'ControlLeft':
                case 'ControlRight':
                    // 左回転 (反時計回り)
                    if (!isPaused) playerRotate(-1);
                    break;
                case 'Space':
                    if (!isPaused) playerHardDrop();
                    event.preventDefault();
                    break;
                case 'KeyC':
                case 'ShiftLeft':
                case 'ShiftRight':
                    if (!isPaused) playerHold();
                    break;
                case 'KeyP':
                case 'Escape':
                    togglePause();
                    break;
            }
            if (!isPaused) draw();
        });

        document.addEventListener('keyup', event => {
            switch (event.code) {
                case 'ArrowLeft':
                    keys.Left = false;
                    if (currentDir === -1) currentDir = keys.Right ? 1 : 0;
                    break;
                case 'ArrowRight':
                    keys.Right = false;
                    if (currentDir === 1) currentDir = keys.Left ? -1 : 0;
                    break;
                case 'ArrowDown':
                    keys.Down = false;
                    break;
            }
        });

        startBtn.addEventListener('click', () => {
            if (isPaused) togglePause();
            else startGame();
        });
        pauseBtn.addEventListener('click', togglePause);

        function addBtnListener(id, actionStart, actionEnd) {
            const btn = document.getElementById(id);
            if (!btn) return;

            const start = (e) => {
                e.preventDefault();
                if (isPaused || isGameOver || !isPlaying) return;
                actionStart();
                draw();
            };
            const end = (e) => {
                e.preventDefault();
                actionEnd();
            };

            btn.addEventListener('touchstart', start, {passive: false});
            btn.addEventListener('touchend', end, {passive: false});
            btn.addEventListener('touchcancel', end, {passive: false});
            
            btn.addEventListener('mousedown', start);
            btn.addEventListener('mouseup', end);
            btn.addEventListener('mouseleave', end);
        }

        addBtnListener('btn-left', 
            () => { if (!keys.Left) { playerMove(-1); dasTimer = 0; currentDir = -1; } keys.Left = true; },
            () => { keys.Left = false; if (currentDir === -1) currentDir = keys.Right ? 1 : 0; }
        );
        addBtnListener('btn-right', 
            () => { if (!keys.Right) { playerMove(1); dasTimer = 0; currentDir = 1; } keys.Right = true; },
            () => { keys.Right = false; if (currentDir === 1) currentDir = keys.Left ? -1 : 0; }
        );
        addBtnListener('btn-down', 
            () => { keys.Down = true; },
            () => { keys.Down = false; }
        );
        addBtnListener('btn-rotate-left', 
            () => { if(!isPaused) playerRotate(-1); },
            () => {}
        );
        addBtnListener('btn-rotate-right', 
            () => { if(!isPaused) playerRotate(1); },
            () => {}
        );
        addBtnListener('btn-hold', 
            () => { if(!isPaused) playerHold(); },
            () => {}
        );
        addBtnListener('btn-drop', 
            () => { if(!isPaused) playerHardDrop(); },
            () => {}
        );

        draw();

    </script>
</body>
</html>
