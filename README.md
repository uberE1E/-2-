<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>雲端自主引擎控制台 (GitHub Web Dashboard)</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Fira+Code:wght@400;600&family=Inter:wght@400;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; }
        .terminal-font { font-family: 'Fira Code', monospace; }
    </style>
</head>
<body class="bg-slate-950 text-slate-100 min-h-screen flex flex-col items-center p-4 md:p-8">

    <!-- 主容器 -->
    <div class="w-full max-w-5xl bg-slate-900 border border-slate-800 rounded-2xl shadow-2xl overflow-hidden">
        
        <!-- 頂部標題列 -->
        <div class="bg-slate-850 px-6 py-4 border-b border-slate-800 flex flex-wrap justify-between items-center gap-4">
            <div>
                <h1 class="text-xl font-bold text-cyan-400 flex items-center gap-2">
                    <span>🚀</span> 跨平台自我複製引擎與控制台
                </h1>
                <p class="text-xs text-slate-400 mt-1">Target: [真複製] 筆記本底層邏輯實時監控與控制</p>
            </div>
            <div class="flex items-center gap-3">
                <span id="engine-status-badge" class="px-3 py-1 rounded-full text-xs font-semibold bg-amber-500/10 text-amber-400 border border-amber-500/30">
                    ● 已暫停 (Standby)
                </span>
            </div>
        </div>

        <!-- 控制面板區 -->
        <div class="p-6 grid grid-cols-1 md:grid-cols-3 gap-6 border-b border-slate-800 bg-slate-900/50">
            
            <!-- 按鈕控制組 -->
            <div class="flex flex-col justify-between gap-3">
                <label class="text-sm font-semibold text-slate-300">⚙️ 引擎狀態控制</label>
                <div class="grid grid-cols-3 gap-2">
                    <button onclick="startEngine()" class="bg-emerald-600 hover:bg-emerald-500 text-white py-2 px-3 rounded-lg text-sm font-semibold transition shadow-lg shadow-emerald-900/20">
                        ▶ 啟動
                    </button>
                    <button onclick="stopEngine()" class="bg-amber-600 hover:bg-amber-500 text-white py-2 px-3 rounded-lg text-sm font-semibold transition shadow-lg shadow-amber-900/20">
                        ⏸ 暫停
                    </button>
                    <button onclick="resetEngine()" class="bg-rose-600 hover:bg-rose-500 text-white py-2 px-3 rounded-lg text-sm font-semibold transition shadow-lg shadow-rose-900/20">
                        🔄 重設
                    </button>
                </div>
            </div>

            <!-- 數量調節桿 -->
            <div class="flex flex-col justify-between gap-2">
                <div class="flex justify-between items-center">
                    <label class="text-sm font-semibold text-slate-300">🎚️ 邊生邊刪最大檔數限制</label>
                    <span id="max-files-val" class="text-cyan-400 font-bold terminal-font text-base bg-slate-800 px-2 py-0.5 rounded">6</span>
                </div>
                <input id="max-files-slider" type="range" min="2" max="15" value="6" oninput="updateMaxFiles(this.value)"
                    class="w-full h-2 bg-slate-800 rounded-lg appearance-none cursor-pointer accent-cyan-400">
                <span class="text-xs text-slate-500">動態控制同時存在的最大子節點數量</span>
            </div>

            <!-- 數據統計 -->
            <div class="flex flex-col justify-between gap-2 bg-slate-950 p-4 rounded-xl border border-slate-800">
                <div class="flex justify-between text-xs text-slate-400">
                    <span>當前世代 (GEN):</span>
                    <span id="stat-gen" class="text-cyan-400 font-bold terminal-font">1</span>
                </div>
                <div class="flex justify-between text-xs text-slate-400">
                    <span>活躍 .py 檔案數:</span>
                    <span id="stat-py" class="text-emerald-400 font-bold terminal-font">0</span>
                </div>
                <div class="flex justify-between text-xs text-slate-400">
                    <span>活躍 Log 紀錄數:</span>
                    <span id="stat-log" class="text-amber-400 font-bold terminal-font">0</span>
                </div>
            </div>
        </div>

        <!-- 檔案變化觀察區 (Grid) -->
        <div class="p-6 grid grid-cols-1 md:grid-cols-2 gap-6 border-b border-slate-800">
            <!-- 模擬檔案列表 -->
            <div class="bg-slate-950 p-4 rounded-xl border border-slate-800 flex flex-col h-64">
                <h3 class="text-sm font-semibold text-slate-300 mb-3 flex items-center justify-between">
                    <span>📁 實時檔案變化觀察 (Files Monitor)</span>
                    <span class="text-xs text-slate-500">自動滾動更新</span>
                </h3>
                <div id="file-list-container" class="terminal-font text-xs overflow-y-auto flex-1 space-y-2 pr-1 text-slate-300">
                    <div class="text-slate-600 italic">（引擎尚未啟動，尚無動態檔案生成）</div>
                </div>
            </div>

            <!-- 終端機 Log 輸出 -->
            <div class="bg-slate-950 p-4 rounded-xl border border-slate-800 flex flex-col h-64">
                <h3 class="text-sm font-semibold text-slate-300 mb-3 flex items-center justify-between">
                    <span>💻 終端機執行日誌 (Console Log)</span>
                    <button onclick="clearLogs()" class="text-xs text-slate-500 hover:text-cyan-400 transition">清空</button>
                </h3>
                <div id="terminal-log" class="terminal-font text-xs overflow-y-auto flex-1 space-y-1 text-emerald-400 bg-slate-900/80 p-3 rounded-lg border border-slate-800/50">
                    <div>[System] 準備就緒。請點擊上方「啟動」按鈕開始運行引擎。</div>
                </div>
            </div>
        </div>

        <!-- 底部資訊 -->
        <div class="px-6 py-3 bg-slate-950 text-center text-xs text-slate-500 border-t border-slate-800">
            Hosted on GitHub Pages ｜ Cross-Platform Autonomous Agent Dashboard
        </div>

    </div>

    <!-- JavaScript 互動邏輯 -->
    <script>
        let isRunning = false;
        let maxFiles = 6;
        let currentGen = 1;
        let sessionId = Math.floor(Date.now() / 1000);
        let activeFiles = [];
        let activeLogs = [];
        let engineInterval = null;

        function updateMaxFiles(val) {
            maxFiles = parseInt(val);
            document.getElementById('max-files-val').innerText = maxFiles;
            appendLog(`[設定變更] 最大邊生邊刪檔數限制已調整為: ${maxFiles}`);
        }

        function startEngine() {
            if (isRunning) return;
            isRunning = true;
            document.getElementById('engine-status-badge').className = "px-3 py-1 rounded-full text-xs font-semibold bg-emerald-500/10 text-emerald-400 border border-emerald-500/30";
            document.getElementById('engine-status-badge').innerText = "● 運行中 (Online)";
            appendLog(`=== [引擎啟動] Session ID = ${sessionId} ===`);

            // 初始化根節點
            if (activeFiles.length === 0) {
                activeFiles.push({ name: `node_gen1_id1_t${sessionId}.py`, time: new Date().toLocaleTimeString(), type: 'py' });
            }

            engineInterval = setInterval(() => {
                if (!isRunning) return;
                
                currentGen++;
                document.getElementById('stat-gen').innerText = currentGen;

                // 模擬自我複製生成左右節點
                let newFile1 = { name: `node_gen${currentGen}_id${Math.floor(Math.random()*10)}_t${sessionId}.py`, time: new Date().toLocaleTimeString(), type: 'py' };
                let newFile2 = { name: `node_gen${currentGen}_id${Math.floor(Math.random()*10+1)}_t${sessionId}.py`, time: new Date().toLocaleTimeString(), type: 'py' };
                let newLog = { name: `log_gen${currentGen}.txt`, time: new Date().toLocaleTimeString(), type: 'log' };

                activeFiles.push(newFile1, newFile2);
                activeLogs.push(newLog);

                appendLog(`[世代 ${currentGen}] 成功繁衍子節點與建立 Log`);

                // 邊生邊刪機制控制
                while (activeFiles.filter(f => f.type === 'py').length > maxFiles) {
                    let removed = activeFiles.shift();
                    appendLog(`[邊生邊刪] 超過限制 (${maxFiles}檔)，已刪除舊檔案: ${removed.name}`, 'warning');
                }

                while (activeLogs.length > maxFiles) {
                    let removedLog = activeLogs.shift();
                    appendLog(`[邊生邊刪] Log 過多，已刪除舊記錄: ${removedLog.name}`, 'warning');
                }

                renderDashboard();
            }, 2500);
        }

        function stopEngine() {
            if (!isRunning) return;
            isRunning = false;
            clearInterval(engineInterval);
            document.getElementById('engine-status-badge').className = "px-3 py-1 rounded-full text-xs font-semibold bg-amber-500/10 text-amber-400 border border-amber-500/30";
            document.getElementById('engine-status-badge').innerText = "● 已暫停 (Standby)";
            appendLog(`[引擎暫停] 目前狀態已保留。`);
        }

        function resetEngine() {
            stopEngine();
            currentGen = 1;
            sessionId = Math.floor(Date.now() / 1000);
            activeFiles = [];
            activeLogs = [];
            document.getElementById('stat-gen').innerText = currentGen;
            renderDashboard();
            appendLog(`[系統重設] 引擎已歸零，準備就緒。`);
        }

        function appendLog(msg, type = 'normal') {
            const term = document.getElementById('terminal-log');
            const colorClass = type === 'warning' ? 'text-amber-400' : 'text-emerald-400';
            term.innerHTML += `<div class="${colorClass}">[${new Date().toLocaleTimeString()}] ${msg}</div>`;
            term.scrollTop = term.scrollHeight;
        }

        function clearLogs() {
            document.getElementById('terminal-log').innerHTML = '';
        }

        function renderDashboard() {
            const pyList = activeFiles.filter(f => f.type === 'py');
            document.getElementById('stat-py').innerText = pyList.length;
            document.getElementById('stat-log').innerText = activeLogs.length;

            const container = document.getElementById('file-list-container');
            if (activeFiles.length === 0 && activeLogs.length === 0) {
                container.innerHTML = `<div class="text-slate-600 italic">（尚無活動檔案）</div>`;
                return;
            }

            let html = '';
            activeFiles.forEach(f => {
                html += `<div class="flex justify-between items-center bg-slate-900 p-2 rounded border border-slate-800/80">
                    <span class="text-cyan-300">🐍 ${f.name}</span>
                    <span class="text-slate-500 text-[10px]">${f.time}</span>
                </div>`;
            });
            activeLogs.forEach(l => {
                html += `<div class="flex justify-between items-center bg-slate-900 p-2 rounded border border-slate-800/80">
                    <span class="text-amber-300">📄 ${l.name}</span>
                    <span class="text-slate-500 text-[10px]">${l.time}</span>
                </div>`;
            });
            container.innerHTML = html;
        }
    </script>
</body>
</html>
