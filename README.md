<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>跨平台自我複製引擎 - GitHub 控制台</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        darkbg: '#0f172a',
                        cardbg: '#1e293b',
                        neonblue: '#00f0ff',
                        neongreen: '#39ff14',
                        neonred: '#ff0055',
                        bordercolor: '#334155'
                    }
                }
            }
        }
    </script>
    <style>
        body {
            background-color: #0f172a;
            color: #f8fafc;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        .neon-border-blue {
            box-shadow: 0 0 10px rgba(0, 240, 255, 0.3);
            border: 1px solid rgba(0, 240, 255, 0.5);
        }
        .neon-border-green {
            box-shadow: 0 0 10px rgba(57, 255, 20, 0.3);
        }
        .console-scrollbar::-webkit-scrollbar {
            width: 6px;
        }
        .console-scrollbar::-webkit-scrollbar-track {
            background: #0f172a;
        }
        .console-scrollbar::-webkit-scrollbar-thumb {
            background: #334155;
            border-radius: 3px;
        }
        @keyframes pulse-slow {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.4; }
        }
        .pulse-active {
            animation: pulse-slow 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
        }
    </style>
</head>
<body class="min-h-screen p-4 md:p-8 flex flex-col justify-between">

    <!-- Header Section -->
    <header class="max-w-7xl mx-auto w-full mb-8">
        <div class="bg-cardbg rounded-2xl p-6 border border-bordercolor shadow-2xl flex flex-col md:flex-row justify-between items-center gap-4">
            <div class="flex items-center gap-4">
                <div class="p-3 bg-slate-800 rounded-xl border border-neonblue text-neonblue text-2xl">
                    <i class="fa-solid fa-network-wired"></i>
                </div>
                <div>
                    <h1 class="text-2xl font-bold tracking-wide text-white flex items-center gap-2">
                        跨平台複製引擎控制台
                        <span class="text-xs px-2 py-0.5 rounded bg-slate-800 text-neonblue border border-neonblue/40">v2.4 Live</span>
                    </h1>
                    <p class="text-slate-400 text-sm mt-1">二元樹雙重節點繁衍 & 滾動式「邊生邊刪」機制視覺化監控系統</p>
                </div>
            </div>
            <!-- Status Badge -->
            <div class="flex items-center gap-3 bg-slate-900/80 px-4 py-2 rounded-xl border border-bordercolor">
                <span id="status-indicator" class="w-3 h-3 rounded-full bg-slate-500"></span>
                <span id="status-text" class="text-sm font-semibold text-slate-400">已關閉 (OFFLINE)</span>
            </div>
        </div>
    </header>

    <!-- Main Content Area -->
    <main class="max-w-7xl mx-auto w-full flex-1 grid grid-cols-1 lg:grid-cols-3 gap-8">

        <!-- Left Panel: Controls & Metrics -->
        <div class="space-y-6">
            <!-- Control Buttons Card -->
            <div class="bg-cardbg rounded-2xl p-6 border border-bordercolor shadow-xl">
                <h2 class="text-lg font-semibold text-slate-200 mb-4 flex items-center gap-2">
                    <i class="fa-solid fa-gamepad text-neonblue"></i> 引擎操作指令
                </h2>
                <div class="space-y-3">
                    <button id="btn-start" onclick="startEngine()" class="w-full py-3 px-4 bg-emerald-600 hover:bg-emerald-500 text-white font-bold rounded-xl transition-all duration-200 flex items-center justify-center gap-2 shadow-lg shadow-emerald-900/30">
                        <i class="fa-solid fa-play"></i> 🚀 啟動引擎 (Start)
                    </button>
                    <button id="btn-stop" onclick="stopEngine()" disabled class="w-full py-3 px-4 bg-slate-700 text-slate-500 font-bold rounded-xl transition-all duration-200 flex items-center justify-center gap-2 cursor-not-allowed">
                        <i class="fa-solid fa-stop"></i> 🛑 關閉引擎 (Stop)
                    </button>
                    <button id="btn-restart" onclick="restartEngine()" disabled class="w-full py-3 px-4 bg-slate-700 text-slate-500 font-bold rounded-xl transition-all duration-200 flex items-center justify-center gap-2 cursor-not-allowed">
                        <i class="fa-solid fa-rotate"></i> 🔄 重新啟動 (Restart)
                    </button>
                </div>
            </div>

            <!-- Stats Grid -->
            <div class="grid grid-cols-2 gap-4">
                <div class="bg-cardbg p-4 rounded-xl border border-bordercolor">
                    <div class="text-xs text-slate-400 mb-1">當前世代 (GEN)</div>
                    <div id="stat-gen" class="text-2xl font-bold text-neonblue">0</div>
                </div>
                <div class="bg-cardbg p-4 rounded-xl border border-bordercolor">
                    <div class="text-xs text-slate-400 mb-1">活躍檔案佇列</div>
                    <div id="stat-files" class="text-2xl font-bold text-neongreen">0 / 6</div>
                </div>
                <div class="bg-cardbg p-4 rounded-xl border border-bordercolor">
                    <div class="text-xs text-slate-400 mb-1">總繁衍節點數</div>
                    <div id="stat-total" class="text-2xl font-bold text-slate-200">0</div>
                </div>
                <div class="bg-cardbg p-4 rounded-xl border border-bordercolor">
                    <div class="text-xs text-slate-400 mb-1">已觸發「邊生邊刪」</div>
                    <div id="stat-deleted" class="text-2xl font-bold text-neonred">0 次</div>
                </div>
            </div>

            <!-- System Info Card -->
            <div class="bg-cardbg rounded-2xl p-5 border border-bordercolor text-xs text-slate-400 space-y-2">
                <div class="flex justify-between border-b border-slate-700 pb-2">
                    <span>繁衍機制:</span>
                    <span class="text-slate-200 font-mono">二元樹分裂 (2x, 2x+1)</span>
                </div>
                <div class="flex justify-between border-b border-slate-700 pb-2">
                    <span>佇列容量:</span>
                    <span class="text-slate-200 font-mono">Max 6 Py / 6 Log 檔</span>
                </div>
                <div class="flex justify-between">
                    <span>相容平台:</span>
                    <span class="text-slate-200 font-mono">Windows / macOS / Linux</span>
                </div>
            </div>
        </div>

        <!-- Right Panel: File System Observer & Live Console -->
        <div class="lg:col-span-2 space-y-6">

            <!-- File Observer -->
            <div class="bg-cardbg rounded-2xl p-6 border border-bordercolor shadow-xl">
                <div class="flex justify-between items-center mb-4">
                    <h2 class="text-lg font-semibold text-slate-200 flex items-center gap-2">
                        <i class="fa-solid fa-folder-tree text-neongreen"></i> 動態檔案觀察器 (Max 6 檔佇列)
                    </h2>
                    <span class="text-xs text-slate-400 font-mono">自動移除過期舊檔</span>
                </div>

                <!-- Node Files Pool -->
                <div class="mb-4">
                    <div class="text-xs text-slate-400 mb-2 font-mono flex items-center gap-1">
                        <i class="fa-brands fa-python text-yellow-400"></i> Python 節點檔案 (.py)
                    </div>
                    <div id="py-file-container" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-3 min-h-[90px] p-3 bg-slate-900/60 rounded-xl border border-slate-800">
                        <div class="text-slate-600 text-xs italic col-span-full flex items-center justify-center h-16">系統未啟動</div>
                    </div>
                </div>

                <!-- Log Files Pool -->
                <div>
                    <div class="text-xs text-slate-400 mb-2 font-mono flex items-center gap-1">
                        <i class="fa-solid fa-file-lines text-blue-400"></i> 日誌檔案 (.txt)
                    </div>
                    <div id="log-file-container" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-3 min-h-[90px] p-3 bg-slate-900/60 rounded-xl border border-slate-800">
                        <div class="text-slate-600 text-xs italic col-span-full flex items-center justify-center h-16">系統未啟動</div>
                    </div>
                </div>
            </div>

            <!-- Console Output Card -->
            <div class="bg-cardbg rounded-2xl p-6 border border-bordercolor shadow-xl flex flex-col">
                <div class="flex justify-between items-center mb-3">
                    <h2 class="text-lg font-semibold text-slate-200 flex items-center gap-2">
                        <i class="fa-solid fa-terminal text-neonblue"></i> 實時 Console 記錄檔 (Live Output)
                    </h2>
                    <button onclick="clearConsole()" class="text-xs text-slate-400 hover:text-white transition-colors">
                        <i class="fa-solid fa-trash-can"></i> 清除紀錄
                    </button>
                </div>
                <div id="console-output" class="bg-slate-950 font-mono text-xs text-emerald-400 p-4 rounded-xl h-56 overflow-y-auto console-scrollbar border border-slate-800 space-y-1">
                    <div class="text-slate-500">[System] 靜態控制介面初始化完成。等待啟動指令...</div>
                </div>
            </div>

        </div>
    </main>

    <!-- Footer -->
    <footer class="max-w-7xl mx-auto w-full mt-8 text-center text-xs text-slate-500">
        <p>GitHub Pages Ready • Standalone HTML Console Environment</p>
    </footer>

    <!-- Logic Controller -->
    <script>
        let isRunning = false;
        let timer = null;
        let gen = 1;
        let nodeId = 1;
        let sessionId = 0;
        let totalCount = 0;
        let deletedCount = 0;

        let pyQueue = [];
        let logQueue = [];

        function updateUIState() {
            const btnStart = document.getElementById('btn-start');
            const btnStop = document.getElementById('btn-stop');
            const btnRestart = document.getElementById('btn-restart');
            const statusIndicator = document.getElementById('status-indicator');
            const statusText = document.getElementById('status-text');

            if (isRunning) {
                btnStart.disabled = true;
                btnStart.className = "w-full py-3 px-4 bg-slate-700 text-slate-500 font-bold rounded-xl cursor-not-allowed";
                
                btnStop.disabled = false;
                btnStop.className = "w-full py-3 px-4 bg-rose-600 hover:bg-rose-500 text-white font-bold rounded-xl transition-all duration-200 shadow-lg shadow-rose-900/30";
                
                btnRestart.disabled = false;
                btnRestart.className = "w-full py-3 px-4 bg-amber-600 hover:bg-amber-500 text-white font-bold rounded-xl transition-all duration-200 shadow-lg shadow-amber-900/30";

                statusIndicator.className = "w-3 h-3 rounded-full bg-neongreen pulse-active";
                statusText.className = "text-sm font-semibold text-neongreen";
                statusText.innerText = "全速運轉中 (RUNNING)";
            } else {
                btnStart.disabled = false;
                btnStart.className = "w-full py-3 px-4 bg-emerald-600 hover:bg-emerald-500 text-white font-bold rounded-xl transition-all duration-200 flex items-center justify-center gap-2 shadow-lg shadow-emerald-900/30";

                btnStop.disabled = true;
                btnStop.className = "w-full py-3 px-4 bg-slate-700 text-slate-500 font-bold rounded-xl cursor-not-allowed";

                btnRestart.disabled = true;
                btnRestart.className = "w-full py-3 px-4 bg-slate-700 text-slate-500 font-bold rounded-xl cursor-not-allowed";

                statusIndicator.className = "w-3 h-3 rounded-full bg-slate-500";
                statusText.className = "text-sm font-semibold text-slate-400";
                statusText.innerText = "已關閉 (OFFLINE)";
            }

            document.getElementById('stat-gen').innerText = isRunning ? gen : 0;
            document.getElementById('stat-files').innerText = `${pyQueue.length} / 6`;
            document.getElementById('stat-total').innerText = totalCount;
            document.getElementById('stat-deleted').innerText = `${deletedCount} 次`;
        }

        function logConsole(msg, type = 'info') {
            const consoleBox = document.getElementById('console-output');
            const entry = document.createElement('div');
            const timeStr = new Date().toLocaleTimeString();

            if (type === 'danger') {
                entry.className = "text-rose-400 font-semibold";
            } else if (type === 'warning') {
                entry.className = "text-yellow-400";
            } else if (type === 'success') {
                entry.className = "text-neongreen";
            } else {
                entry.className = "text-emerald-400";
            }

            entry.innerText = `[${timeStr}] ${msg}`;
            consoleBox.appendChild(entry);
            consoleBox.scrollTop = consoleBox.scrollHeight;
        }

        function clearConsole() {
            document.getElementById('console-output').innerHTML = '';
            logConsole("控制台紀錄已清除。");
        }

        function startEngine() {
            if (isRunning) return;
            isRunning = true;
            sessionId = Math.floor(Date.now() / 1000);
            gen = 1;
            nodeId = 1;
            totalCount = 0;
            deletedCount = 0;
            pyQueue = [];
            logQueue = [];

            logConsole(`=== 啟動跨平台複製引擎 (Session ID: ${sessionId}) ===`, 'success');
            logConsole("--- [系統] 正在清理殘留的舊檔案與記錄 ---", 'warning');
            
            updateUIState();
            stepEngine();

            timer = setInterval(stepEngine, 2200);
        }

        function stopEngine() {
            if (!isRunning) return;
            isRunning = false;
            clearInterval(timer);
            logConsole("🛑 引擎已停止運作。", 'danger');
            updateUIState();
        }

        function restartEngine() {
            stopEngine();
            setTimeout(() => {
                startEngine();
            }, 500);
        }

        function stepEngine() {
            if (!isRunning) return;

            const nextGen = gen + 1;
            const leftId = nodeId * 2;
            const rightId = nodeId * 2 + 1;

            const leftPy = `node_gen${nextGen}_id${leftId}_t${sessionId}.py`;
            const rightPy = `node_gen${nextGen}_id${rightId}_t${sessionId}.py`;

            const leftLog = `log_gen${nextGen}_id${leftId}.txt`;
            const rightLog = `log_gen${nextGen}_id${rightId}.txt`;

            // Push Left & Right Nodes
            addFileToQueue(pyQueue, leftPy, 'py');
            addFileToQueue(pyQueue, rightPy, 'py');
            addFileToQueue(logQueue, leftLog, 'log');
            addFileToQueue(logQueue, rightLog, 'log');

            totalCount += 2;
            logConsole(`[世代 ${gen}] 節點 ID ${nodeId} 已分裂生成: ${leftPy} & ${rightPy}`);

            gen = nextGen;
            nodeId = leftId; // Advance binary tree left branch

            renderFiles();
            updateUIState();
        }

        function addFileToQueue(queue, fileName, type) {
            queue.push(fileName);
            if (queue.length > 6) {
                const removed = queue.shift();
                deletedCount++;
                if (type === 'py') {
                    logConsole(`[邊生邊刪] 節點 .py 過多，已刪除最舊檔案: ${removed}`, 'danger');
                } else {
                    logConsole(`[邊生邊刪] Log 檔過多，已刪除最舊記錄: ${removed}`, 'warning');
                }
            }
        }

        function renderFiles() {
            const pyContainer = document.getElementById('py-file-container');
            const logContainer = document.getElementById('log-file-container');

            if (pyQueue.length === 0) {
                pyContainer.innerHTML = `<div class="text-slate-600 text-xs italic col-span-full flex items-center justify-center h-16">系統未啟動</div>`;
            } else {
                pyContainer.innerHTML = pyQueue.map((file, idx) => `
                    <div class="bg-slate-800 p-2.5 rounded-lg border border-slate-700 flex items-center justify-between text-xs animate-fade-in">
                        <span class="font-mono text-emerald-300 truncate"><i class="fa-brands fa-python mr-1.5 text-yellow-400"></i>${file}</span>
                        <span class="text-[10px] bg-slate-900 px-1.5 py-0.5 rounded text-slate-400">#${idx + 1}</span>
                    </div>
                `).join('');
            }

            if (logQueue.length === 0) {
                logContainer.innerHTML = `<div class="text-slate-600 text-xs italic col-span-full flex items-center justify-center h-16">系統未啟動</div>`;
            } else {
                logContainer.innerHTML = logQueue.map((file, idx) => `
                    <div class="bg-slate-800 p-2.5 rounded-lg border border-slate-700 flex items-center justify-between text-xs animate-fade-in">
                        <span class="font-mono text-blue-300 truncate"><i class="fa-solid fa-file-lines mr-1.5 text-blue-400"></i>${file}</span>
                        <span class="text-[10px] bg-slate-900 px-1.5 py-0.5 rounded text-slate-400">#${idx + 1}</span>
                    </div>
                `).join('');
            }
        }
    </script>
</body>
</html>
