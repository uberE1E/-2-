<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>純 HTML 跨平台自我複製引擎控制台</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Fira+Code:wght@400;600&family=Inter:wght@400;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; }
        .terminal-font { font-family: 'Fira Code', monospace; }
    </style>
</head>
<body class="bg-slate-950 text-slate-100 min-h-screen flex flex-col items-center p-4 md:p-8">

    <div class="w-full max-w-5xl bg-slate-900 border border-slate-800 rounded-2xl shadow-2xl overflow-hidden">
        
        <!-- 標題列 -->
        <div class="bg-slate-850 px-6 py-4 border-b border-slate-800 flex flex-wrap justify-between items-center gap-4">
            <div>
                <h1 class="text-xl font-bold text-cyan-400 flex items-center gap-2">
                    <span>⚡</span> 實體檔案生成引擎 (Pure HTML Client Drive)
                </h1>
                <p class="text-xs text-slate-400 mt-1">直接於使用者裝置硬碟寫入、複製與滾動刪除 .py / .txt 檔案</p>
            </div>
            <span id="engine-status" class="px-3 py-1 rounded-full text-xs font-semibold bg-amber-500/10 text-amber-400 border border-amber-500/30">
                ● 未授權目錄 (Waiting)
            </span>
        </div>

        <!-- 控制區 -->
        <div class="p-6 grid grid-cols-1 md:grid-cols-3 gap-6 border-b border-slate-800 bg-slate-900/50">
            
            <!-- 按鈕操作 -->
            <div class="flex flex-col justify-between gap-3">
                <label class="text-sm font-semibold text-slate-300">⚙️ 引擎控制 (實體檔案動作)</label>
                <div class="grid grid-cols-3 gap-2">
                    <button onclick="initDirectoryAndStart()" class="bg-emerald-600 hover:bg-emerald-500 text-white py-2 px-3 rounded-lg text-sm font-semibold transition">
                        ▶ 啟動面板
                    </button>
                    <button onclick="stopEngine()" class="bg-amber-600 hover:bg-amber-500 text-white py-2 px-3 rounded-lg text-sm font-semibold transition">
                        ⏸ 暫停
                    </button>
                    <button onclick="resetEngine()" class="bg-rose-600 hover:bg-rose-500 text-white py-2 px-3 rounded-lg text-sm font-semibold transition">
                        🔄 清理重設
                    </button>
                </div>
            </div>

            <!-- 檔案限制滑桿 -->
            <div class="flex flex-col justify-between gap-2">
                <div class="flex justify-between items-center">
                    <label class="text-sm font-semibold text-slate-300">🎚️ 滾動刪除最大檔數限制</label>
                    <span id="max-files-val" class="text-cyan-400 font-bold terminal-font bg-slate-800 px-2 py-0.5 rounded">6</span>
                </div>
                <input id="max-files-slider" type="range" min="2" max="15" value="6" oninput="updateMaxFiles(this.value)"
                    class="w-full h-2 bg-slate-800 rounded-lg appearance-none cursor-pointer accent-cyan-400">
                <span class="text-xs text-slate-500">超額時自動實體刪除最舊的 .py 與 .txt</span>
            </div>

            <!-- 數據狀態 -->
            <div class="flex flex-col justify-between gap-2 bg-slate-950 p-4 rounded-xl border border-slate-800 text-xs">
                <div class="flex justify-between">
                    <span class="text-slate-400">當前世代 (GEN):</span>
                    <span id="stat-gen" class="text-cyan-400 font-bold terminal-font">1</span>
                </div>
                <div class="flex justify-between">
                    <span class="text-slate-400">裝置實體 .py 數:</span>
                    <span id="stat-py" class="text-emerald-400 font-bold terminal-font">0</span>
                </div>
                <div class="flex justify-between">
                    <span class="text-slate-400">裝置實體 .txt 數:</span>
                    <span id="stat-log" class="text-amber-400 font-bold terminal-font">0</span>
                </div>
            </div>
        </div>

        <!-- 監視器與日誌 -->
        <div class="p-6 grid grid-cols-1 md:grid-cols-2 gap-6 border-b border-slate-800">
            <!-- 實體檔案目錄狀態 -->
            <div class="bg-slate-950 p-4 rounded-xl border border-slate-800 flex flex-col h-72">
                <h3 class="text-sm font-semibold text-slate-300 mb-3 flex justify-between">
                    <span>📂 使用者裝置實體目錄狀態</span>
                    <span class="text-xs text-slate-500" id="dir-name-display">未選擇目錄</span>
                </h3>
                <div id="file-list-container" class="terminal-font text-xs overflow-y-auto flex-1 space-y-2 pr-1 text-slate-300">
                    <div class="text-slate-600 italic">請點擊「啟動面板」並選擇存放實體檔案的本地資料夾。</div>
                </div>
            </div>

            <!-- 實時 Console Log -->
            <div class="bg-slate-950 p-4 rounded-xl border border-slate-800 flex flex-col h-72">
                <h3 class="text-sm font-semibold text-slate-300 mb-3 flex justify-between">
                    <span>💻 執行與檔案操作日誌 (Console Log)</span>
                    <button onclick="clearLogs()" class="text-xs text-slate-500 hover:text-cyan-400">清空</button>
                </h3>
                <div id="terminal-log" class="terminal-font text-xs overflow-y-auto flex-1 space-y-1 text-emerald-400 bg-slate-900/80 p-3 rounded-lg border border-slate-800/50">
                    <div>[System] 等待使用者授權本地裝置資料夾權限...</div>
                </div>
            </div>
        </div>

        <div class="px-6 py-3 bg-slate-950 text-center text-xs text-slate-500 border-t border-slate-800">
            Powered by Web File System Access API ｜ 完全純 HTML / JS 客戶端運作
        </div>
    </div>

    <script>
        let dirHandle = null;
        let isRunning = false;
        let maxFiles = 6;
        let currentGen = 1;
        let sessionId = Math.floor(Date.now() / 1000);
        let activePyFiles = [];
        let activeLogFiles = [];
        let loopTimer = null;

        const NODE_SCRIPT_TEMPLATE = `# -*- coding: utf-8 -*-
import os, sys, glob, subprocess, time
from pathlib import Path

GEN = {gen}
NODE_ID = {node_id}
SESSION_ID = {session_id}

def main():
    current_file = Path(__file__).resolve()
    print(f"--- [獨立執行] 世代 {GEN} 節點 {NODE_ID} (PID: {os.getpid()}) ---", flush=True)

if __name__ == '__main__':
    main()`;

        function updateMaxFiles(val) {
            maxFiles = parseInt(val);
            document.getElementById('max-files-val').innerText = maxFiles;
            appendLog(`[設定變更] 最大邊生邊刪檔數限制調整為: ${maxFiles}`);
        }

        async function initDirectoryAndStart() {
            if (!window.showDirectoryPicker) {
                alert("您的瀏覽器不支援 File System Access API，請使用 Chrome 或 Edge 瀏覽器！");
                return;
            }

            try {
                if (!dirHandle) {
                    dirHandle = await window.showDirectoryPicker();
                    document.getElementById('dir-name-display').innerText = `目錄: ${dirHandle.name}`;
                    appendLog(`[授權成功] 已取得裝置目錄授權: ${dirHandle.name}`);
                }
                
                if (isRunning) return;
                isRunning = true;
                document.getElementById('engine-status').className = "px-3 py-1 rounded-full text-xs font-semibold bg-emerald-500/10 text-emerald-400 border border-emerald-500/30";
                document.getElementById('engine-status').innerText = "● 實體檔案引擎運行中";

                // 開始遞迴繁衍與建立實體檔案
                startEngineLoop();
            } catch (err) {
                appendLog(`[授權取消/錯誤] ${err.message}`, 'warning');
            }
        }

        async function createFileInDevice(filename, content) {
            if (!dirHandle) return;
            const fileHandle = await dirHandle.getFileHandle(filename, { create: true });
            const writable = await fileHandle.createWritable();
            await writable.write(content);
            await writable.close();
        }

        async function removeFileFromDevice(filename) {
            if (!dirHandle) return;
            try {
                await dirHandle.removeEntry(filename);
            } catch (e) {
                console.error(e);
            }
        }

        function startEngineLoop() {
            loopTimer = setInterval(async () => {
                if (!isRunning || !dirHandle) return;

                currentGen++;
                document.getElementById('stat-gen').innerText = currentGen;

                const leftId = currentGen * 2;
                const rightId = currentGen * 2 + 1;

                const pyName1 = `node_gen${currentGen}_id${leftId}_t${sessionId}.py`;
                const pyName2 = `node_gen${currentGen}_id${rightId}_t${sessionId}.py`;
                const logName1 = `log_gen${currentGen}_id${leftId}.txt`;
                const logName2 = `log_gen${currentGen}_id${rightId}.txt`;

                // 1. 於使用者實體裝置寫入 .py 檔案（包含模板 code）
                const pyContent1 = NODE_SCRIPT_TEMPLATE.replace('{gen}', currentGen).replace('{node_id}', leftId).replace('{session_id}', sessionId);
                const pyContent2 = NODE_SCRIPT_TEMPLATE.replace('{gen}', currentGen).replace('{node_id}', rightId).replace('{session_id}', sessionId);

                await createFileInDevice(pyName1, pyContent1);
                await createFileInDevice(pyName2, pyContent2);

                // 2. 於使用者實體裝置寫入 .txt Log 檔案
                const logContent = `--- [世代 ${currentGen}] Log 紀錄產生時間: ${new Date().toLocaleString()} ---`;
                await createFileInDevice(logName1, logContent);
                await createFileInDevice(logName2, logContent);

                activePyFiles.push(pyName1, pyName2);
                activeLogFiles.push(logName1, logName2);

                appendLog(`[世代 ${currentGen}] 實體寫入: ${pyName1}, ${pyName2}`);

                // 3. 邊生邊刪機制：超過限制時，從使用者實體電腦上刪除檔案
                while (activePyFiles.length > maxFiles) {
                    const oldestPy = activePyFiles.shift();
                    await removeFileFromDevice(oldestPy);
                    appendLog(`[邊生邊刪] 刪除實體檔案: ${oldestPy}`, 'warning');
                }

                while (activeLogFiles.length > maxFiles) {
                    const oldestLog = activeLogFiles.shift();
                    await removeFileFromDevice(oldestLog);
                    appendLog(`[邊生邊刪] 刪除實體 Log: ${oldestLog}`, 'warning');
                }

                refreshUIDisplay();
            }, 2000);
        }

        function stopEngine() {
            isRunning = false;
            clearInterval(loopTimer);
            document.getElementById('engine-status').className = "px-3 py-1 rounded-full text-xs font-semibold bg-amber-500/10 text-amber-400 border border-amber-500/30";
            document.getElementById('engine-status').innerText = "● 已暫停";
            appendLog(`[引擎暫停] 已停止寫入實體檔案。`);
        }

        async function resetEngine() {
            stopEngine();
            if (dirHandle) {
                appendLog(`[清理中] 正在清理本地實體檔案...`);
                for (const file of activePyFiles) await removeFileFromDevice(file);
                for (const file of activeLogFiles) await removeFileFromDevice(file);
            }
            activePyFiles = [];
            activeLogFiles = [];
            currentGen = 1;
            sessionId = Math.floor(Date.now() / 1000);
            document.getElementById('stat-gen').innerText = currentGen;
            refreshUIDisplay();
            appendLog(`[重設完成] 本地實體檔案已完全清空。`);
        }

        function refreshUIDisplay() {
            document.getElementById('stat-py').innerText = activePyFiles.length;
            document.getElementById('stat-log').innerText = activeLogFiles.length;

            const container = document.getElementById('file-list-container');
            if (activePyFiles.length === 0 && activeLogFiles.length === 0) {
                container.innerHTML = `<div class="text-slate-600 italic">（目錄內目前無活動檔案）</div>`;
                return;
            }

            let html = '';
            activePyFiles.forEach(f => {
                html += `<div class="flex justify-between items-center bg-slate-900 p-2 rounded border border-slate-800">
                    <span class="text-cyan-300">🐍 ${f}</span>
                    <span class="text-emerald-400 text-[10px]">實體檔案</span>
                </div>`;
            });
            activeLogFiles.forEach(l => {
                html += `<div class="flex justify-between items-center bg-slate-900 p-2 rounded border border-slate-800">
                    <span class="text-amber-300">📄 ${l}</span>
                    <span class="text-amber-400 text-[10px]">實體 Log</span>
                </div>`;
            });
            container.innerHTML = html;
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
    </script>
</body>
</html>
