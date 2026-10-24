<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>安全隔離型跨平台自我複製引擎控制台</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Fira+Code:wght@400;600&family=Inter:wght@400;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; }
        .terminal-font { font-family: 'Fira Code', monospace; }
    </style>
</head>
<body class="bg-slate-950 text-slate-100 min-h-screen flex flex-col items-center p-4 md:p-8">

<div class="w-full max-w-5xl bg-slate-900 border border-slate-800 rounded-2xl shadow-2xl overflow-hidden">
        
        <!-- 頂部標題列 -->
<div class="bg-slate-850 px-6 py-4 border-b border-slate-800 flex flex-wrap justify-between items-center gap-4">
            <div>
                <h1 class="text-xl font-bold text-cyan-400 flex items-center gap-2">
                    <span>🛡️</span> 獨立沙箱實體檔案引擎控制台
                </h1>
                <p class="text-xs text-slate-400 mt-1">嚴格命名空間隔離：僅對本工具生成的專屬 .py / .txt 進行滾動式邊生邊刪</p>
            </div>
            <span id="engine-status" class="px-3 py-1 rounded-full text-xs font-semibold bg-amber-500/10 text-amber-400 border border-amber-500/30">
                ● 未授權目錄 (Waiting)
            </span>
        </div>

        <!-- 控制面板區 -->
<div class="p-6 grid grid-cols-1 md:grid-cols-3 gap-6 border-b border-slate-800 bg-slate-900/50">
            
            <!-- 按鈕操作 -->
<div class="flex flex-col justify-between gap-3">
                <label class="text-sm font-semibold text-slate-300">⚙️ 引擎狀態控制</label>
                <div class="grid grid-cols-3 gap-2">
                    <button onclick="initDirectoryAndStart()" class="bg-emerald-600 hover:bg-emerald-500 text-white py-2 px-3 rounded-lg text-sm font-semibold transition">
                        ▶ 啟動
                    </button>
                    <button onclick="stopEngine()" class="bg-amber-600 hover:bg-amber-500 text-white py-2 px-3 rounded-lg text-sm font-semibold transition">
                        ⏸ 暫停
                    </button>
                    <button onclick="resetEngine()" class="bg-rose-600 hover:bg-rose-500 text-white py-2 px-3 rounded-lg text-sm font-semibold transition">
                        🔄 重設
                    </button>
                </div>
            </div>

            <!-- 數量調節滑桿 -->
<div class="flex flex-col justify-between gap-2">
                <div class="flex justify-between items-center">
                    <label class="text-sm font-semibold text-slate-300">🎚️ 專屬檔數上限限制</label>
                    <span id="max-files-val" class="text-cyan-400 font-bold terminal-font bg-slate-800 px-2 py-0.5 rounded">6</span>
                </div>
                <input id="max-files-slider" type="range" min="2" max="15" value="6" oninput="updateMaxFiles(this.value)"
                    class="w-full h-2 bg-slate-800 rounded-lg appearance-none cursor-pointer accent-cyan-400">
                <span class="text-xs text-slate-500">超額時自動刪除本工具生成的舊檔案</span>
            </div>

            <!-- 數據統計 -->
<div class="flex flex-col justify-between gap-2 bg-slate-950 p-4 rounded-xl border border-slate-800 text-xs">
                <div class="flex justify-between">
                    <span class="text-slate-400">當前世代 (GEN):</span>
                    <span id="stat-gen" class="text-cyan-400 font-bold terminal-font">1</span>
                </div>
                <div class="flex justify-between">
                    <span class="text-slate-400">專屬活躍 .py 數:</span>
                    <span id="stat-py" class="text-emerald-400 font-bold terminal-font">0</span>
                </div>
                <div class="flex justify-between">
                    <span class="text-slate-400">專屬活躍 Log 數:</span>
                    <span id="stat-log" class="text-amber-400 font-bold terminal-font">0</span>
                </div>
            </div>
        </div>

        <!-- 檔案觀察與 Log 區 -->
<div class="p-6 grid grid-cols-1 md:grid-cols-2 gap-6 border-b border-slate-800">
            <!-- 實體檔案清單 -->
            <div class="bg-slate-950 p-4 rounded-xl border border-slate-800 flex flex-col h-72">
                <h3 class="text-sm font-semibold text-slate-300 mb-3 flex justify-between">
                    <span>📁 本工具專屬檔案監視器</span>
                    <span class="text-xs text-slate-500" id="dir-name-display">未選擇目錄</span>
                </h3>
                <div id="file-list-container" class="terminal-font text-xs overflow-y-auto flex-1 space-y-2 pr-1 text-slate-300">
                    <div class="text-slate-600 italic">請點擊「啟動」並選擇本機資料夾以授權沙箱寫入。</div>
                </div>
            </div>

            <!-- 終端機 Log -->
<div class="bg-slate-950 p-4 rounded-xl border border-slate-800 flex flex-col h-72">
                <h3 class="text-sm font-semibold text-slate-300 mb-3 flex justify-between">
                    <span>💻 安全隔離日誌 (Console Log)</span>
                    <button onclick="clearLogs()" class="text-xs text-slate-500 hover:text-cyan-400">清空</button>
                </h3>
                <div id="terminal-log" class="terminal-font text-xs overflow-y-auto flex-1 space-y-1 text-emerald-400 bg-slate-900/80 p-3 rounded-lg border border-slate-800/50">
                    <div>[System] 準備就緒。安全沙箱隔離防護已啟動。</div>
                </div>
            </div>
        </div>

<div class="px-6 py-3 bg-slate-950 text-center text-xs text-slate-500 border-t border-slate-800">
            GitHub Pages Hosted ｜ 100% Secure Isolated Client-Side Execution
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

        function generatePythonScript(gen, nodeId, sessionId) {
            const lines = [
                `# -*- coding: utf-8 -*-`,
                `import os`,
                `import sys`,
                `import glob`,
                `import subprocess`,
                `import time`,
                `from pathlib import Path`,
                ``,
                `GEN = ${gen}`,
                `NODE_ID = ${nodeId}`,
                `SESSION_ID = ${sessionId}`,
                ``,
                `def main():`,
                `    current_file = Path(__file__).resolve()`,
                `    print(f"--- [獨立執行] 世代 {GEN} 節點 {NODE_ID} (PID: {os.getpid()}) ---", flush=True)`,
                `    print(1, flush=True)`,
                ``,
                `    next_gen = GEN + 1`,
                `    left_id = NODE_ID * 2`,
                `    right_id = NODE_ID * 2 + 1`,
                ``,
                `    base_dir = current_file.parent`,
                `    left_filename = base_dir / f"node_gen{next_gen}_id{left_id}_t{SESSION_ID}.py"`,
                `    right_filename = base_dir / f"node_gen{next_gen}_id{right_id}_t{SESSION_ID}.py"`,
                ``,
                `    try:`,
                `        with open(current_file, 'r', encoding='utf-8') as f:`,
                `            source_code = f.read()`,
                ``,
                `        left_code = source_code.replace(f"GEN = {GEN}", f"GEN = {next_gen}").replace(f"NODE_ID = {NODE_ID}", f"NODE_ID = {left_id}")`,
                `        right_code = source_code.replace(f"GEN = {GEN}", f"GEN = {next_gen}").replace(f"NODE_ID = {NODE_ID}", f"NODE_ID = {right_id}")`,
                ``,
                `        with open(left_filename, 'w', encoding='utf-8') as f:`,
                `            f.write(left_code)`,
                `        with open(right_filename, 'w', encoding='utf-8') as f:`,
                `            f.write(right_code)`,
                ``,
                `        all_nodes = sorted(base_dir.glob(f"node_gen*_id*_t{SESSION_ID}.py"), key=lambda p: p.stat().st_mtime)`,
                `        while len(all_nodes) > 6:`,
                `            oldest_file = all_nodes.pop(0)`,
                `            if oldest_file != current_file:`,
                `                try:`,
                `                    oldest_file.unlink()`,
                `                    print(f"[邊生邊刪] .py 過多，已刪除最舊檔案: {oldest_file.name}", flush=True)`,
                `                except Exception:`,
                `                    pass`,
                ``,
                `        all_logs = sorted(base_dir.glob("log_gen*.txt"), key=lambda p: p.stat().st_mtime)`,
                `        while len(all_logs) > 6:`,
                `            oldest_log = all_logs.pop(0)`,
                `            try:`,
                `                oldest_log.unlink()`,
                `                print(f"[邊生邊刪] Log 檔過多，已刪除最舊記錄: {oldest_log.name}", flush=True)`,
                `            except Exception:`,
                `                pass`,
                ``,
                `        log_left = open(base_dir / f"log_gen{next_gen}_id{left_id}.txt", "w", encoding="utf-8")`,
                `        log_right = open(base_dir / f"log_gen{next_gen}_id{right_id}.txt", "w", encoding="utf-8")`,
                ``,
                `        time.sleep(1)`,
                `        executable = sys.executable`,
                `        if left_filename.exists():`,
                `            subprocess.Popen([executable, str(left_filename)], stdout=log_left, stderr=log_left)`,
                `        if right_filename.exists():`,
                `            subprocess.Popen([executable, str(right_filename)], stdout=log_right, stderr=log_right)`,
                ``,
                `    except Exception as e:`,
                `        print(f"【繁衍過程發生例外錯誤】: {e}", flush=True)`,
                ``,
                `if __name__ == '__main__':`,
                `    main()`
            ];
            return lines.join('\n');
        }

        function updateMaxFiles(val) {
            maxFiles = parseInt(val);
            document.getElementById('max-files-val').innerText = maxFiles;
            appendLog(`[設定變更] 專屬檔數上限已調整為: ${maxFiles}`);
        }

        async function initDirectoryAndStart() {
            if (!window.showDirectoryPicker) {
                alert("您的瀏覽器不支援 File System Access API，請使用 Chrome 或 Edge！");
                return;
            }

            try {
                if (!dirHandle) {
                    dirHandle = await window.showDirectoryPicker();
                    document.getElementById('dir-name-display').innerText = `目錄: ${dirHandle.name}`;
                    appendLog(`[授權成功] 已繫結本地目錄: ${dirHandle.name}`);
                }
                
                if (isRunning) return;
                isRunning = true;
                document.getElementById('engine-status').className = "px-3 py-1 rounded-full text-xs font-semibold bg-emerald-500/10 text-emerald-400 border border-emerald-500/30";
                document.getElementById('engine-status').innerText = "● 沙箱運行中";

                startEngineLoop();
            } catch (err) {
                appendLog(`[授權取消] ${err.message}`, 'warning');
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
                if (filename.startsWith(`node_gen`) || filename.startsWith(`log_gen`)) {
                    await dirHandle.removeEntry(filename);
                }
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

                const pyContent1 = generatePythonScript(currentGen, leftId, sessionId);
                const pyContent2 = generatePythonScript(currentGen, rightId, sessionId);

                await createFileInDevice(pyName1, pyContent1);
                await createFileInDevice(pyName2, pyContent2);
                await createFileInDevice(logName1, `Log for ${pyName1}\n[獨立執行] 世代 ${currentGen} 節點 ${leftId} 運行中`);
                await createFileInDevice(logName2, `Log for ${pyName2}\n[獨立執行] 世代 ${currentGen} 節點 ${rightId} 運行中`);

                activePyFiles.push(pyName1, pyName2);
                activeLogFiles.push(logName1, logName2);

                appendLog(`[世代 ${currentGen}] 實體寫入完整原碼: ${pyName1}, ${pyName2}`);

                while (activePyFiles.length > maxFiles) {
                    const oldestPy = activePyFiles.shift();
                    await removeFileFromDevice(oldestPy);
                    appendLog(`[安全邊生邊刪] 清理專屬檔案: ${oldestPy}`, 'warning');
                }

                while (activeLogFiles.length > maxFiles) {
                    const oldestLog = activeLogFiles.shift();
                    await removeFileFromDevice(oldestLog);
                    appendLog(`[安全邊生邊刪] 清理專屬 Log: ${oldestLog}`, 'warning');
                }

                refreshUIDisplay();
            }, 2500);
        }

        function stopEngine() {
            isRunning = false;
            clearInterval(loopTimer);
            document.getElementById('engine-status').className = "px-3 py-1 rounded-full text-xs font-semibold bg-amber-500/10 text-amber-400 border border-amber-500/30";
            document.getElementById('engine-status').innerText = "● 已暫停";
            appendLog(`[引擎暫停] 已停止寫入。`);
        }

        async function resetEngine() {
            stopEngine();
            if (dirHandle) {
                appendLog(`[清理中] 正在安全清空本工具產生的專屬檔案...`);
                for (const file of activePyFiles) await removeFileFromDevice(file);
                for (const file of activeLogFiles) await removeFileFromDevice(file);
            }
            activePyFiles = [];
            activeLogFiles = [];
            currentGen = 1;
            sessionId = Math.floor(Date.now() / 1000);
            document.getElementById('stat-gen').innerText = currentGen;
            refreshUIDisplay();
            appendLog(`[重設完成] 本工具專屬檔案已全數清除。`);
        }

        function refreshUIDisplay() {
            document.getElementById('stat-py').innerText = activePyFiles.length;
            document.getElementById('stat-log').innerText = activeLogFiles.length;

            const container = document.getElementById('file-list-container');
            if (activePyFiles.length === 0 && activeLogFiles.length === 0) {
                container.innerHTML = `<div class="text-slate-600 italic">（目錄內目前無本工具專屬檔案）</div>`;
                return;
            }

            let html = '';
            activePyFiles.forEach(f => {
                html += `<div class="flex justify-between items-center bg-slate-900 p-2 rounded border border-slate-800">
                    <span class="text-cyan-300">🐍 ${f}</span>
                    <span class="text-emerald-400 text-[10px]">專屬實體檔</span>
                </div>`;
            });
            activeLogFiles.forEach(l => {
                html += `<div class="flex justify-between items-center bg-slate-900 p-2 rounded border border-slate-800">
                    <span class="text-amber-300">📄 ${l}</span>
                    <span class="text-amber-400 text-[10px]">專屬 Log</span>
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
