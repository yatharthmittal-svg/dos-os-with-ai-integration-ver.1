<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MS-DOS v6.22 Emulator / WebDOS Operating System</title>
    <!-- Tailwind CSS for layout structure -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome for CRT/Retro style icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=VT323&family=Share+Tech+Mono&display=swap');

        :root {
            --bg-color: #000000;
            --text-color: #ffffff;
            --dim-color: #aaaaaa;
            --highlight-color: #ffffff;
            --font-family: 'VT323', monospace;
        }

        * {
            box-sizing: border-box;
            user-select: none;
        }

        html, body {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            background-color: #000000;
            font-family: var(--font-family);
            color: #ffffff;
            overflow: hidden;
        }

        /* CRT Screen Wrapper & Scanlines */
        .crt-container {
            position: relative;
            width: 100vw;
            height: 100vh;
            background-color: #000000;
            display: flex;
            flex-direction: column;
            padding: 16px;
            overflow: hidden;
            box-shadow: inset 0 0 100px rgba(0,0,0,0.8);
        }

        .crt-container::before {
            content: " ";
            display: block;
            position: absolute;
            top: 0; left: 0; bottom: 0; right: 0;
            background: linear-gradient(rgba(18, 16, 16, 0) 50%, rgba(0, 0, 0, 0.3) 50%), linear-gradient(90deg, rgba(255, 0, 0, 0.03), rgba(0, 255, 0, 0.01), rgba(0, 0, 255, 0.03));
            z-index: 50;
            background-size: 100% 4px, 6px 100%;
            pointer-events: none;
            opacity: 0.15;
        }

        .crt-flicker {
            animation: flicker 0.15s infinite;
            pointer-events: none;
            position: absolute;
            inset: 0;
            background: rgba(18, 16, 16, 0.02);
            z-index: 51;
        }

        @keyframes flicker {
            0% { opacity: 0.97; }
            50% { opacity: 1; }
            100% { opacity: 0.98; }
        }

        /* Terminal Window */
        #terminal-container {
            flex: 1;
            overflow-y: auto;
            font-size: 1.35rem;
            line-height: 1.25;
            padding-bottom: 20px;
            scroll-behavior: smooth;
        }

        #terminal-container::-webkit-scrollbar {
            width: 8px;
        }
        #terminal-container::-webkit-scrollbar-thumb {
            background: #ffffff;
        }

        .line {
            white-space: pre-wrap;
            word-break: break-all;
            margin-bottom: 4px;
        }

        /* ASCII Retro Box Formatting */
        .dos-box {
            border: 2px solid #ffffff;
            padding: 8px 12px;
            margin: 8px 0;
            background-color: #000000;
            box-shadow: 0 0 5px rgba(255, 255, 255, 0.3);
            white-space: pre-wrap;
            word-break: break-all;
        }

        .dos-box-title {
            display: inline-block;
            background-color: #ffffff;
            color: #000000;
            padding: 0 6px;
            font-weight: bold;
            margin-bottom: 6px;
        }

        .input-line {
            display: flex;
            align-items: center;
        }

        .prompt {
            color: #ffffff;
            margin-right: 8px;
            font-weight: bold;
        }

        #cmd-input {
            background: transparent;
            border: none;
            outline: none;
            color: #ffffff;
            font-family: var(--font-family);
            font-size: 1.35rem;
            flex: 1;
            caret-color: transparent;
        }

        .cursor {
            display: inline-block;
            width: 10px;
            height: 1.2rem;
            background-color: #ffffff;
            vertical-align: middle;
            animation: blink 0.8s infinite;
        }

        @keyframes blink {
            0%, 49% { opacity: 1; }
            50%, 100% { opacity: 0; }
        }

        /* Status Bar */
        .status-bar {
            height: 28px;
            border-top: 1px dashed #ffffff;
            display: flex;
            justify-content: space-between;
            align-items: center;
            font-size: 1rem;
            color: #ffffff;
            padding-top: 4px;
            z-index: 10;
        }

        /* AI Thinking Indicator */
        .ai-indicator {
            color: #ffffff;
            font-style: italic;
            animation: pulse 1s infinite alternate;
        }

        @keyframes pulse {
            0% { opacity: 0.4; }
            100% { opacity: 1; }
        }

        /* Modal Desktop Applications */
        .app-modal {
            position: absolute;
            top: 20px;
            left: 20px;
            right: 20px;
            bottom: 45px;
            background-color: #000000;
            border: 2px solid #ffffff;
            z-index: 100;
            display: flex;
            flex-direction: column;
            padding: 10px;
            box-shadow: 0 0 20px rgba(255, 255, 255, 0.4);
        }

        .app-header {
            background: #ffffff;
            color: #000000;
            padding: 4px 8px;
            font-weight: bold;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .app-body {
            flex: 1;
            overflow: auto;
            padding: 8px 0;
        }

        textarea.dos-editor {
            width: 100%;
            height: 100%;
            background: #000000;
            color: #ffffff;
            font-family: var(--font-family);
            font-size: 1.2rem;
            border: none;
            outline: none;
            resize: none;
        }

        .hidden {
            display: none !important;
        }

        /* Retro App Table Styling for Calc/Office */
        .dos-table {
            border-collapse: collapse;
            width: 100%;
        }

        .dos-table th, .dos-table td {
            border: 1px solid #ffffff;
            padding: 4px;
            text-align: left;
        }

        .dos-table input {
            background: black;
            color: white;
            border: none;
            width: 100%;
            font-family: var(--font-family);
            outline: none;
        }
    </style>
</head>
<body class="bg-black text-white">

    <div class="crt-container">
        <div class="crt-flicker"></div>

        <!-- Terminal Output -->
        <div id="terminal-container" onclick="focusInput()">
            <div id="terminal-output"></div>
            
            <div class="input-line" id="active-input-line">
                <span class="prompt" id="prompt-text">C:\></span>
                <span id="cmd-prefix" class="text-white"></span>
                <input type="text" id="cmd-input" autocomplete="off" spellcheck="false" autofocus />
                <span class="cursor" id="custom-cursor"></span>
            </div>
        </div>

        <!-- System App Modal (NET, EDIT, OFFICE, PAINT, CALC, GAMES) -->
        <div id="app-modal" class="app-modal hidden">
            <div class="app-header">
                <span id="app-title">APPLICATION</span>
                <button onclick="closeAppModal()" class="bg-black text-white px-2 border border-black hover:bg-white hover:text-black">ESC / CLOSE</button>
            </div>
            <div class="app-body" id="app-body">
                <!-- Dynamic Application Content -->
            </div>
        </div>

        <!-- System Bar -->
        <div class="status-bar">
            <span>WebDOS v3.0 [Internet & Offline Apps Active]</span>
            <span id="clock-display">12:00 PM</span>
            <span>Type HELP, NET, OFFICE, PAINT, CALC, SNAKE, MINES</span>
        </div>
    </div>

    <script>
        // System File System
        const initialFS = {
            "C:": {
                type: "DIR",
                children: {
                    "SYSTEM": {
                        type: "DIR",
                        children: {
                            "COMMAND.COM": { type: "FILE", content: "[Main System Executable]" },
                            "NET.EXE": { type: "FILE", content: "[Web Browser & YouTube Player]" },
                            "EDIT.COM": { type: "FILE", content: "[Text Editor]" }
                        }
                    },
                    "APPS": {
                        type: "DIR",
                        children: {
                            "OFFICE.EXE": { type: "FILE", content: "[LibreOffice Calc & Writer Offline Suite]" },
                            "PAINT.EXE": { type: "FILE", content: "[Retro Paint & Sketch Tool]" },
                            "CALC.EXE": { type: "FILE", content: "[Calculator Utility]" }
                        }
                    },
                    "GAMES": {
                        type: "DIR",
                        children: {
                            "SNAKE.EXE": { type: "FILE", content: "[Snake Game]" },
                            "MINES.EXE": { type: "FILE", content: "[Minesweeper Game]" }
                        }
                    },
                    "README.TXT": { 
                        type: "FILE", 
                        content: "Welcome to WebDOS v3.0!\n\nFeatured Offline & Online Apps:\n- NET       : Full Web Browser & YouTube Video Player\n- OFFICE    : Offline Spreadsheet & Word Processor\n- PAINT     : Pixel Drawing & Canvas Tool\n- CALC      : Arithmetic Calculator\n- SNAKE     : Arcade Snake Game\n- MINES     : Minesweeper Game\n- EDIT      : Plain Text Editor\n\nYou can also type simple English like 'create file mydoc' or 'open youtube dinosaurs'!" 
                    }
                }
            }
        };

        let fileSystem = JSON.parse(localStorage.getItem('webdos_fs')) || initialFS;
        let currentPath = ["C:"];
        let commandHistory = [];
        let historyIndex = -1;
        let audioCtx = null;
        let activeApp = null;
        let isAiProcessing = false;

        function initAudio() {
            if (!audioCtx) {
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            }
        }

        function playBeep(freq = 800, duration = 0.08, type = "square") {
            try {
                initAudio();
                if (audioCtx.state === 'suspended') {
                    audioCtx.resume();
                }
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                osc.type = type;
                osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
                gain.gain.setValueAtTime(0.1, audioCtx.currentTime);
                osc.connect(gain);
                gain.connect(audioCtx.destination);
                osc.start();
                osc.stop(audioCtx.currentTime + duration);
            } catch (e) { }
        }

        const terminalOutput = document.getElementById('terminal-output');
        const cmdInput = document.getElementById('cmd-input');
        const promptText = document.getElementById('prompt-text');
        const terminalContainer = document.getElementById('terminal-container');
        const appModal = document.getElementById('app-modal');
        const appTitle = document.getElementById('app-title');
        const appBody = document.getElementById('app-body');
        const clockDisplay = document.getElementById('clock-display');

        function updateClock() {
            const now = new Date();
            clockDisplay.innerText = now.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit', second: '2-digit' });
        }
        setInterval(updateClock, 1000);
        updateClock();

        function focusInput() {
            if (!activeApp && !isAiProcessing) {
                cmdInput.focus();
            }
        }

        window.addEventListener('click', focusInput);

        window.addEventListener('keydown', (e) => {
            if (e.key === 'Escape' && activeApp) {
                closeAppModal();
            }
        });

        function getDirectoryNode(pathArray) {
            let current = fileSystem;
            for (let dir of pathArray) {
                if (current[dir] && current[dir].type === "DIR") {
                    current = current[dir].children;
                } else {
                    return null;
                }
            }
            return current;
        }

        function getPathString() {
            if (currentPath.length === 1) return currentPath[0] + "\\";
            return currentPath.join("\\");
        }

        function updatePrompt() {
            promptText.innerText = getPathString() + ">";
        }

        function printBox(title, content) {
            const box = document.createElement('div');
            box.className = 'dos-box';
            
            const titleEl = document.createElement('div');
            titleEl.className = 'dos-box-title';
            titleEl.textContent = title;
            box.appendChild(titleEl);

            const contentEl = document.createElement('div');
            contentEl.style.whiteSpace = 'pre-wrap';
            contentEl.textContent = content;
            box.appendChild(contentEl);

            terminalOutput.appendChild(box);
            terminalContainer.scrollTop = terminalContainer.scrollHeight;
        }

        function printLine(text = "", className = "") {
            const div = document.createElement('div');
            div.className = 'line ' + className;
            div.textContent = text;
            terminalOutput.appendChild(div);
            terminalContainer.scrollTop = terminalContainer.scrollHeight;
        }

        function saveFS() {
            localStorage.setItem('webdos_fs', JSON.stringify(fileSystem));
        }

        function openModal(title, innerHTML) {
            activeApp = title;
            appTitle.textContent = title;
            appBody.innerHTML = innerHTML;
            appModal.classList.remove('hidden');
        }

        function closeModal() {
            appModal.classList.add('hidden');
            appBody.innerHTML = "";
            activeApp = null;
            focusInput();
        }
        const closeAppModal = closeModal;

        function bootDOS() {
            printBox("SYSTEM BOOT", "Simple WebDOS v3.0 [AI Natural Language & App Suite Loaded]\n\nSystem Ready.\nType HELP for command list or launch apps: NET, OFFICE, PAINT, CALC, SNAKE, MINES.");
            updatePrompt();
        }

        async function parseNaturalLanguageWithAI(userInput) {
            const apiKey = "";
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent?key=${apiKey}`;

            const dirNode = getDirectoryNode(currentPath);
            const availableFiles = dirNode ? Object.keys(dirNode) : [];

            const systemPrompt = `You are an AI command parser inside WebDOS for 1st graders and students.
Map plain English user inputs to executable DOS commands.

Current Folder Files: ${JSON.stringify(availableFiles)}

Supported Commands:
- CREATEFILE [filename] [optional text]
- TYPE [filename]
- EDIT [filename]
- DIR
- CD [folder]
- MD [folder]
- DEL [filename]
- NET (Opens Web Browser)
- NET YT [search query] (Opens YouTube video search)
- OFFICE (Opens Offline LibreOffice Calc/Writer Suite)
- PAINT (Opens Paint/Sketch tool)
- CALC (Opens Calculator)
- SNAKE (Plays Snake game)
- MINES (Plays Minesweeper game)
- HELP, CLS

Rules:
1. Fix common typos (e.g. "opne" -> "TYPE", "crate" -> "CREATEFILE").
2. Output JSON in this exact structure:
{
  "status": "SUCCESS" | "SYNTAX_ERROR",
  "correctedSpelling": "string describing typo fix or empty",
  "dosCommand": "exact single executable DOS command string",
  "friendlyMessage": "short simple 1-sentence note for a child"
}`;

            const payload = {
                contents: [{ parts: [{ text: `User query: "${userInput}"` }] }],
                systemInstruction: { parts: [{ text: systemPrompt }] },
                generationConfig: { responseMimeType: "application/json" }
            };

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                if (!response.ok) throw new Error("AI Request Failed");
                const result = await response.json();
                const jsonText = result.candidates?.[0]?.content?.parts?.[0]?.text;
                if (jsonText) return JSON.parse(jsonText);
            } catch (e) {
                console.error("AI Error:", e);
            }
            return null;
        }

        // Keyboard Command Event Listener
        cmdInput.addEventListener('keydown', async function (e) {
            if (e.key === 'Enter') {
                const commandStr = cmdInput.value.trim();
                if (!commandStr) return;

                printLine(getPathString() + "> " + commandStr);
                commandHistory.push(commandStr);
                historyIndex = commandHistory.length;
                cmdInput.value = "";
                playBeep(1200, 0.02);

                const firstWord = commandStr.trim().split(/\s+/)[0].toUpperCase();
                const nativeCommands = ["HELP", "DIR", "CD", "CHDIR", "CLS", "CLR", "TYPE", "EDIT", "CREATEFILE", "TOUCH", "MKDIR", "MD", "RMDIR", "RD", "DEL", "ERASE", "COPY", "DATE", "TIME", "VER", "TREE", "SNAKE", "MINES", "NET", "BROWSER", "SEARCH", "WWW", "YT", "YOUTUBE", "OFFICE", "CALC", "PAINT", "DRAW", "REBOOT", "RESTART", "RESETFS", "ECHO"];

                if (nativeCommands.includes(firstWord)) {
                    processCommand(commandStr);
                } else {
                    if (!tryRunFile(commandStr)) {
                        isAiProcessing = true;
                        cmdInput.disabled = true;
                        
                        const aiDiv = document.createElement('div');
                        aiDiv.className = 'line ai-indicator';
                        aiDiv.textContent = "[ AI Assistant is thinking... ]";
                        terminalOutput.appendChild(aiDiv);
                        terminalContainer.scrollTop = terminalContainer.scrollHeight;

                        const aiResponse = await parseNaturalLanguageWithAI(commandStr);

                        if (aiDiv.parentNode) aiDiv.parentNode.removeChild(aiDiv);
                        cmdInput.disabled = false;
                        isAiProcessing = false;
                        focusInput();

                        if (aiResponse) {
                            if (aiResponse.status === "SYNTAX_ERROR") {
                                playBeep(200, 0.2, "sawtooth");
                                printBox("SYNTAX ERROR", `Could not understand '${commandStr}'.\nTip: Try typing 'NET', 'OFFICE', 'PAINT', or 'DIR'!`);
                            } else if (aiResponse.dosCommand) {
                                if (aiResponse.correctedSpelling) {
                                    printBox("SPELLING FIXED", aiResponse.correctedSpelling);
                                }
                                if (aiResponse.friendlyMessage) {
                                    printLine("AI: " + aiResponse.friendlyMessage);
                                }
                                processCommand(aiResponse.dosCommand);
                            }
                        } else {
                            printBox("SYNTAX ERROR", `Command '${commandStr}' not recognized.\nType HELP for app list.`);
                        }
                    }
                }
            } else if (e.key === 'ArrowUp') {
                if (historyIndex > 0) {
                    historyIndex--;
                    cmdInput.value = commandHistory[historyIndex];
                }
                e.preventDefault();
            } else if (e.key === 'ArrowDown') {
                if (historyIndex < commandHistory.length - 1) {
                    historyIndex++;
                    cmdInput.value = commandHistory[historyIndex];
                } else {
                    historyIndex = commandHistory.length;
                    cmdInput.value = "";
                }
                e.preventDefault();
            }
        });

        function processCommand(rawCmd) {
            const parts = rawCmd.trim().split(/\s+/);
            const cmd = parts[0].toUpperCase();
            const args = parts.slice(1);

            switch (cmd) {
                case 'HELP':
                    printHelp();
                    break;
                case 'DIR':
                    cmdDir(args);
                    break;
                case 'CD':
                case 'CHDIR':
                    cmdCd(args);
                    break;
                case 'CLS':
                case 'CLR':
                    terminalOutput.innerHTML = "";
                    break;
                case 'TYPE':
                    cmdType(args);
                    break;
                case 'EDIT':
                    cmdEdit(args);
                    break;
                case 'CREATEFILE':
                case 'TOUCH':
                    cmdCreateFile(args);
                    break;
                case 'MKDIR':
                case 'MD':
                    cmdMkdir(args);
                    break;
                case 'RMDIR':
                case 'RD':
                case 'DEL':
                case 'ERASE':
                    cmdDelete(args);
                    break;
                case 'COPY':
                    cmdCopy(args);
                    break;
                case 'DATE':
                    printBox("SYSTEM DATE", "Today is " + new Date().toLocaleDateString('en-US', { weekday: 'short', month: '2-digit', day: '2-digit', year: 'numeric' }));
                    break;
                case 'TIME':
                    printBox("SYSTEM TIME", "Current time is " + new Date().toLocaleTimeString());
                    break;
                case 'VER':
                    printBox("SYSTEM VERSION", "Simple WebDOS Version 3.0\n[Web Browser, LibreOffice Suite & AI Engine Active]");
                    break;
                case 'TREE':
                    cmdTree();
                    break;
                case 'YT':
                case 'YOUTUBE':
                    openWebBrowser("YT " + args.join(" "));
                    break;
                case 'NET':
                case 'BROWSER':
                case 'SEARCH':
                case 'WWW':
                    openWebBrowser(args.join(" "));
                    break;
                case 'OFFICE':
                    openOfficeSuite();
                    break;
                case 'PAINT':
                case 'DRAW':
                    openPaintApp();
                    break;
                case 'CALC':
                    openCalculatorApp();
                    break;
                case 'SNAKE':
                case 'SNAKE.EXE':
                    startSnakeGame();
                    break;
                case 'MINES':
                case 'MINES.EXE':
                    startMinesGame();
                    break;
                case 'REBOOT':
                case 'RESTART':
                    terminalOutput.innerHTML = "";
                    bootDOS();
                    break;
                case 'RESETFS':
                    localStorage.removeItem('webdos_fs');
                    fileSystem = JSON.parse(JSON.stringify(initialFS));
                    printBox("SYSTEM RESET", "File system restored to default state.");
                    break;
                case 'ECHO':
                    printBox("ECHO OUTPUT", args.join(" "));
                    break;
                default:
                    if (!tryRunFile(cmd)) {
                        printBox("SYNTAX ERROR", `Unknown command: '${cmd}'.\nType HELP for assistance or NET to browse web.`);
                        playBeep(200, 0.15, "sawtooth");
                    }
            }
        }

        function tryRunFile(cmdName) {
            const clean = cmdName.toUpperCase().replace(/\.EXE$|\.COM$/, '');
            if (clean === "SNAKE") { startSnakeGame(); return true; }
            if (clean === "MINES") { startMinesGame(); return true; }
            if (clean === "NET") { openWebBrowser(""); return true; }
            if (clean === "OFFICE") { openOfficeSuite(); return true; }
            if (clean === "PAINT") { openPaintApp(); return true; }
            if (clean === "CALC") { openCalculatorApp(); return true; }
            if (clean === "EDIT") { cmdEdit(["UNTITLED.TXT"]); return true; }
            return false;
        }

        function printHelp() {
            const helpText = 
`--- SYSTEM APPLICATIONS ---
NET         - Full Internet Web Browser & YouTube Video Player
OFFICE      - LibreOffice Calc Spreadsheet & Writer Suite
PAINT       - Pixel Drawing & Paint App
CALC        - Arithmetic Calculator
EDIT [file] - Plain Text File Editor
SNAKE       - Snake Arcade Game
MINES       - Minesweeper Game

--- FILE & DIRECTORY COMMANDS ---
DIR         - List all files and folders
CREATEFILE  - Create a new text file directly
TYPE [file] - Display contents of a text file
MD [folder] - Create a folder
DEL [file]  - Delete a file
CD [folder] - Open a folder (CD .. to go back)
CLS         - Clear screen output`;

            printBox("WEBDOS HELP MENU", helpText);
        }

        function cmdCreateFile(args) {
            if (!args[0]) {
                printBox("SYNTAX ERROR", "Specify a file name.\nExample: CREATEFILE F1.TXT");
                return;
            }

            let fileName = args[0].toUpperCase().replace(/["']/g, '');
            if (!fileName.includes(".")) fileName += ".TXT";

            const initialContent = args.slice(1).join(" ") || "This is a new file created in WebDOS v3.0.";
            const currentDir = getDirectoryNode(currentPath);

            currentDir[fileName] = { type: "FILE", content: initialContent };
            saveFS();
            playBeep(900, 0.08);
            printBox("FILE CREATED", `Created '${fileName}' successfully!\n\nContent:\n${initialContent}`);
        }

        function cmdDir(args) {
            const dirNode = getDirectoryNode(currentPath);
            if (!dirNode) {
                printBox("ERROR", "Folder not found.");
                return;
            }

            let output = `.              <FOLDER>\n..             <FOLDER>\n`;
            let fileCount = 0;
            let dirCount = 0;

            for (let name in dirNode) {
                const item = dirNode[name];
                const padName = name.padEnd(14, ' ');
                if (item.type === "DIR") {
                    output += `${padName} <FOLDER>\n`;
                    dirCount++;
                } else {
                    const size = (item.content ? item.content.length : 0);
                    const padSize = size.toString().padStart(6, ' ');
                    output += `${padName} ${padSize} bytes\n`;
                    fileCount++;
                }
            }

            output += `\nTotal: ${fileCount} File(s), ${dirCount + 2} Folder(s)`;
            printBox(`DIRECTORY OF ${getPathString()}`, output);
        }

        function cmdCd(args) {
            if (!args[0] || args[0] === ".") return;
            const target = args[0].toUpperCase();

            if (target === "\\") { currentPath = ["C:"]; updatePrompt(); return; }
            if (target === "..") {
                if (currentPath.length > 1) { currentPath.pop(); updatePrompt(); }
                return;
            }

            const currentDirNode = getDirectoryNode(currentPath);
            if (currentDirNode && currentDirNode[target] && currentDirNode[target].type === "DIR") {
                currentPath.push(target);
                updatePrompt();
            } else {
                printBox("ERROR", "Folder not found: " + target);
            }
        }

        function cmdType(args) {
            if (!args[0]) {
                printBox("SYNTAX ERROR", "Specify a file name.\nExample: TYPE README.TXT");
                return;
            }
            let fileName = args[0].toUpperCase().replace(/["']/g, '');
            const currentDir = getDirectoryNode(currentPath);

            if (currentDir && !currentDir[fileName] && currentDir[fileName + ".TXT"]) {
                fileName = fileName + ".TXT";
            }

            if (currentDir && currentDir[fileName]) {
                if (currentDir[fileName].type === "FILE") {
                    printBox(`FILE: ${fileName}`, currentDir[fileName].content);
                } else {
                    printBox("ERROR", "Target is a directory.");
                }
            } else {
                printBox("ERROR", "File not found: " + fileName);
            }
        }

        function cmdEdit(args) {
            let fileName = (args[0] || "UNTITLED.TXT").toUpperCase().replace(/["']/g, '');
            if (!fileName.includes(".")) fileName += ".TXT";

            const currentDir = getDirectoryNode(currentPath);
            let content = "";

            if (currentDir[fileName] && currentDir[fileName].type === "FILE") {
                content = currentDir[fileName].content || "";
            }

            openModal(`EDIT.COM - ${fileName}`, `
                <div class="h-full flex flex-col bg-black text-white p-2">
                    <div class="mb-2 flex justify-between items-center border-b border-white pb-2">
                        <span>EDITING: ${fileName}</span>
                        <button onclick="saveAndExitEdit('${fileName}')" class="bg-white text-black font-bold px-3 py-1 hover:bg-gray-200">SAVE & EXIT</button>
                    </div>
                    <textarea id="edit-textarea" class="dos-editor">${content}</textarea>
                </div>
            `);
        }

        function saveAndExitEdit(fileName) {
            const textarea = document.getElementById('edit-textarea');
            if (textarea) {
                const currentDir = getDirectoryNode(currentPath);
                currentDir[fileName] = { type: "FILE", content: textarea.value };
                saveFS();
                printBox("EDIT SAVED", `File '${fileName}' saved to system memory.`);
            }
            closeModal();
        }

        function cmdMkdir(args) {
            if (!args[0]) { printBox("SYNTAX ERROR", "Specify folder name."); return; }
            const dirName = args[0].toUpperCase().replace(/["']/g, '');
            const currentDir = getDirectoryNode(currentPath);

            if (currentDir[dirName]) {
                printBox("ERROR", "Directory or file already exists.");
            } else {
                currentDir[dirName] = { type: "DIR", children: {} };
                saveFS();
                printBox("SUCCESS", `Folder '${dirName}' created.`);
            }
        }

        function cmdDelete(args) {
            if (!args[0]) { printBox("SYNTAX ERROR", "Specify file to delete."); return; }
            let targetName = args[0].toUpperCase().replace(/["']/g, '');
            const currentDir = getDirectoryNode(currentPath);

            if (currentDir && !currentDir[targetName] && currentDir[targetName + ".TXT"]) {
                targetName = targetName + ".TXT";
            }

            if (currentDir && currentDir[targetName]) {
                delete currentDir[targetName];
                saveFS();
                printBox("SUCCESS", `Deleted '${targetName}'.`);
            } else {
                printBox("ERROR", "File or folder not found.");
            }
        }

        function cmdCopy(args) {
            if (args.length < 2) { printBox("SYNTAX ERROR", "Usage: COPY [src] [dst]"); return; }
            const src = args[0].toUpperCase().replace(/["']/g, '');
            const dst = args[1].toUpperCase().replace(/["']/g, '');
            const currentDir = getDirectoryNode(currentPath);

            if (currentDir && currentDir[src] && currentDir[src].type === "FILE") {
                currentDir[dst] = { type: "FILE", content: currentDir[src].content };
                saveFS();
                printBox("SUCCESS", "Copied 1 file successfully.");
            } else {
                printBox("ERROR", "Source file not found.");
            }
        }

        function cmdTree() {
            let treeOutput = "C:\\\n";
            function buildTree(node, indent) {
                for (let child in node) {
                    if (node[child].type === "DIR") {
                        treeOutput += indent + "+---" + child + "\n";
                        buildTree(node[child].children, indent + "    ");
                    }
                }
            }
            buildTree(getDirectoryNode(["C:"]), "");
            printBox("DIRECTORY TREE", treeOutput);
        }

        function openWebBrowser(initialQuery = "") {
            const isYt = initialQuery.toUpperCase().startsWith("YT");
            const queryText = isYt ? initialQuery.substring(2).trim() : initialQuery;

            openModal("NET.EXE - Web Browser & YouTube", `
                <div class="h-full flex flex-col bg-black text-white p-2">
                    <div class="border border-white p-2 mb-2 flex justify-between items-center">
                        <div>
                            <div class="font-bold text-lg">DOS WEB BROWSER v3.0</div>
                            <div class="text-xs">Live Gemini Search Grounding & YouTube Player Active</div>
                        </div>
                        <div class="flex gap-2">
                            <button onclick="loadQuickBookmark('space')" class="border border-white px-2 py-0.5 text-xs hover:bg-white hover:text-black">SPACE</button>
                            <button onclick="loadQuickBookmark('dinosaurs')" class="border border-white px-2 py-0.5 text-xs hover:bg-white hover:text-black">DINOSAURS</button>
                            <button onclick="loadQuickBookmark('math')" class="border border-white px-2 py-0.5 text-xs hover:bg-white hover:text-black">MATH</button>
                        </div>
                    </div>

                    <div class="flex gap-2 mb-2">
                        <input id="web-search-input" type="text" value="${queryText}" placeholder="Search web topics or YouTube videos..." class="flex-1 bg-black text-white border border-white px-2 py-1 focus:outline-none font-mono" />
                        <button onclick="performWebSearch()" class="bg-white text-black font-bold px-3 hover:bg-gray-200">WEB SEARCH</button>
                        <button onclick="performYtSearch()" class="bg-red-600 text-white font-bold px-3 hover:bg-red-700">YOUTUBE</button>
                    </div>

                    <div id="web-content-area" class="flex-1 border border-white p-3 overflow-y-auto font-mono text-sm">
                        <div class="text-center mt-6">
                            <p class="font-bold text-lg">WELCOME TO WEBDOS INTERNET</p>
                            <p class="mt-2">Type any question above to search the live web or search YouTube videos.</p>
                        </div>
                    </div>
                </div>
            `);

            if (queryText) {
                if (isYt) performYtSearch(); else performWebSearch();
            }
        }

        function loadQuickBookmark(topic) {
            const input = document.getElementById('web-search-input');
            if (input) {
                input.value = topic;
                performWebSearch();
            }
        }

        async function performWebSearch() {
            const input = document.getElementById('web-search-input');
            const contentArea = document.getElementById('web-content-area');
            if (!input || !contentArea || !input.value.trim()) return;

            const query = input.value.trim();
            contentArea.innerHTML = `<div class="p-4 text-center text-white italic">[ Searching web page for '${query}'... ]</div>`;

            const apiKey = "";
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent?key=${apiKey}`;

            const payload = {
                contents: [{ parts: [{ text: `Provide a simple, clean, 3-paragraph educational web overview suitable for students about: ${query}` }] }],
                tools: [{ "google_search": {} }]
            };

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                const result = await response.json();
                const candidate = result.candidates?.[0];
                const text = candidate?.content?.parts?.[0]?.text || "No response received.";

                let citationsHTML = "";
                const grounding = candidate?.groundingMetadata?.groundingAttributions;
                if (grounding && grounding.length > 0) {
                    citationsHTML = `<div class="mt-4 border-t border-white pt-2 text-xs"><strong>Web Sources:</strong><ul class="list-disc ml-4">`;
                    grounding.forEach(attr => {
                        if (attr.web?.uri && attr.web?.title) {
                            citationsHTML += `<li><a href="${attr.web.uri}" target="_blank" class="underline hover:text-gray-300">${attr.web.title}</a></li>`;
                        }
                    });
                    citationsHTML += `</ul></div>`;
                }

                contentArea.innerHTML = `
                    <div class="border border-white p-3 mb-2 bg-black">
                        <div class="font-bold border-b border-white pb-1 mb-2">RESULTS FOR: ${query.toUpperCase()}</div>
                        <div class="whitespace-pre-wrap leading-relaxed">${text}</div>
                        ${citationsHTML}
                    </div>
                `;
            } catch (err) {
                contentArea.innerHTML = `<div class="p-4 text-red-400">Error connecting to web service. Please check your query.</div>`;
            }
        }

        function performYtSearch() {
            const input = document.getElementById('web-search-input');
            const contentArea = document.getElementById('web-content-area');
            if (!input || !contentArea) return;

            const query = input.value.trim() || "educational videos";
            const embedQuery = encodeURIComponent(query);

            contentArea.innerHTML = `
                <div class="border border-white p-3 bg-black flex flex-col h-full">
                    <div class="font-bold border-b border-white pb-1 mb-2 flex justify-between">
                        <span>YOUTUBE PLAYER - ${query.toUpperCase()}</span>
                        <span class="text-xs">WebDOS Embedded Media</span>
                    </div>
                    <div class="flex-1 w-full bg-black min-h-[250px] mb-2 border border-white">
                        <iframe class="w-full h-full" src="https://www.youtube.com/embed?listType=search&list=${embedQuery}" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
                    </div>
                    <p class="text-xs text-gray-300">Tip: Click video titles inside player to play directly inside WebDOS!</p>
                </div>
            `;
        }

        function openOfficeSuite() {
            openModal("OFFICE.EXE - LibreOffice Calc & Writer Suite", `
                <div class="h-full flex flex-col bg-black text-white p-2">
                    <div class="border border-white p-2 mb-2 flex justify-between items-center">
                        <span class="font-bold">LIBREOFFICE SUITE v1.0 (OFFLINE)</span>
                        <div class="flex gap-2">
                            <button onclick="switchOfficeTab('calc')" id="tab-calc" class="bg-white text-black font-bold px-3 py-1 text-xs">SPREADSHEET (CALC)</button>
                            <button onclick="switchOfficeTab('writer')" id="tab-writer" class="border border-white px-3 py-1 text-xs hover:bg-white hover:text-black">DOCUMENTS (WRITER)</button>
                        </div>
                    </div>

                    <!-- Spreadsheet App -->
                    <div id="office-calc" class="flex-1 overflow-auto border border-white p-2">
                        <div class="mb-2 text-xs flex gap-2">
                            <button onclick="addCalcRow()" class="border border-white px-2 py-0.5 hover:bg-white hover:text-black">+ ADD ROW</button>
                            <button onclick="calculateCalcTotal()" class="border border-white px-2 py-0.5 hover:bg-white hover:text-black">= AUTO SUM</button>
                        </div>
                        <table class="dos-table text-xs" id="calc-table">
                            <thead>
                                <tr>
                                    <th style="width: 40px;">#</th>
                                    <th>ITEM / ITEM NAME</th>
                                    <th>QTY / COUNT</th>
                                    <th>PRICE / VALUE</th>
                                    <th>TOTAL</th>
                                </tr>
                            </thead>
                            <tbody id="calc-body">
                                <tr><td>1</td><td><input type="text" value="Books" /></td><td><input type="number" class="qty" value="2" /></td><td><input type="number" class="price" value="10" /></td><td class="row-total">20</td></tr>
                                <tr><td>2</td><td><input type="text" value="Pencils" /></td><td><input type="number" class="qty" value="5" /></td><td><input type="number" class="price" value="2" /></td><td class="row-total">10</td></tr>
                            </tbody>
                        </table>
                        <div class="mt-4 font-bold text-right" id="calc-sum-display">TOTAL SUM: 30</div>
                    </div>

                    <!-- Writer App -->
                    <div id="office-writer" class="flex-1 border border-white p-2 hidden">
                        <textarea id="writer-textarea" class="dos-editor" placeholder="Write your document or story here..."></textarea>
                    </div>
                </div>
            `);
        }

        function switchOfficeTab(tab) {
            const calc = document.getElementById('office-calc');
            const writer = document.getElementById('office-writer');
            const btnCalc = document.getElementById('tab-calc');
            const btnWriter = document.getElementById('tab-writer');

            if (tab === 'calc') {
                calc.classList.remove('hidden');
                writer.classList.add('hidden');
                btnCalc.className = "bg-white text-black font-bold px-3 py-1 text-xs";
                btnWriter.className = "border border-white px-3 py-1 text-xs hover:bg-white hover:text-black";
            } else {
                writer.classList.remove('hidden');
                calc.classList.add('hidden');
                btnWriter.className = "bg-white text-black font-bold px-3 py-1 text-xs";
                btnCalc.className = "border border-white px-3 py-1 text-xs hover:bg-white hover:text-black";
            }
        }

        function addCalcRow() {
            const tbody = document.getElementById('calc-body');
            if (!tbody) return;
            const rowCount = tbody.children.length + 1;
            const tr = document.createElement('tr');
            tr.innerHTML = `<td>${rowCount}</td><td><input type="text" value="New Item" /></td><td><input type="number" class="qty" value="1" /></td><td><input type="number" class="price" value="0" /></td><td class="row-total">0</td>`;
            tbody.appendChild(tr);
        }

        function calculateCalcTotal() {
            const rows = document.querySelectorAll('#calc-body tr');
            let totalSum = 0;
            rows.forEach(r => {
                const qty = parseFloat(r.querySelector('.qty')?.value || 0);
                const price = parseFloat(r.querySelector('.price')?.value || 0);
                const total = qty * price;
                const totEl = r.querySelector('.row-total');
                if (totEl) totEl.innerText = total.toString();
                totalSum += total;
            });
            const sumDisplay = document.getElementById('calc-sum-display');
            if (sumDisplay) sumDisplay.innerText = "TOTAL SUM: " + totalSum;
        }

        function openPaintApp() {
            openModal("PAINT.EXE - Retro Sketch & Canvas", `
                <div class="h-full flex flex-col bg-black text-white p-2">
                    <div class="mb-2 flex justify-between items-center border-b border-white pb-2">
                        <span class="font-bold">PAINT / CANVAS TOOL</span>
                        <div class="flex gap-2">
                            <button onclick="clearPaintCanvas()" class="border border-white px-2 py-0.5 text-xs hover:bg-white hover:text-black">CLEAR CANVAS</button>
                        </div>
                    </div>
                    <div class="flex-1 flex justify-center items-center border border-white bg-black">
                        <canvas id="paint-canvas" width="600" height="350" class="border border-white cursor-crosshair bg-black"></canvas>
                    </div>
                </div>
            `);

            setTimeout(() => {
                const canvas = document.getElementById('paint-canvas');
                if (!canvas) return;
                const ctx = canvas.getContext('2d');
                ctx.strokeStyle = '#ffffff';
                ctx.lineWidth = 3;

                let drawing = false;
                canvas.onmousedown = () => drawing = true;
                canvas.onmouseup = () => { drawing = false; ctx.beginPath(); };
                canvas.onmousemove = (e) => {
                    if (!drawing) return;
                    const rect = canvas.getBoundingClientRect();
                    ctx.lineTo(e.clientX - rect.left, e.clientY - rect.top);
                    ctx.stroke();
                    ctx.beginPath();
                    ctx.moveTo(e.clientX - rect.left, e.clientY - rect.top);
                };
            }, 100);
        }

        function clearPaintCanvas() {
            const canvas = document.getElementById('paint-canvas');
            if (canvas) {
                const ctx = canvas.getContext('2d');
                ctx.clearRect(0, 0, canvas.width, canvas.height);
            }
        }

        function openCalculatorApp() {
            openModal("CALC.EXE - System Calculator", `
                <div class="h-full flex flex-col items-center justify-center bg-black text-white p-2">
                    <div class="border-2 border-white p-4 w-64 bg-black">
                        <div class="font-bold text-center mb-2 border-b border-white pb-1">CALCULATOR v1.0</div>
                        <input id="calc-display" type="text" readonly value="0" class="w-full bg-black border border-white text-right p-2 mb-3 text-xl font-mono text-white" />
                        <div class="grid grid-cols-4 gap-2">
                            <button onclick="calcPress('7')" class="border border-white p-2 text-center hover:bg-white hover:text-black">7</button>
                            <button onclick="calcPress('8')" class="border border-white p-2 text-center hover:bg-white hover:text-black">8</button>
                            <button onclick="calcPress('9')" class="border border-white p-2 text-center hover:bg-white hover:text-black">9</button>
                            <button onclick="calcPress('/')" class="border border-white p-2 text-center hover:bg-white hover:text-black">/</button>
                            
                            <button onclick="calcPress('4')" class="border border-white p-2 text-center hover:bg-white hover:text-black">4</button>
                            <button onclick="calcPress('5')" class="border border-white p-2 text-center hover:bg-white hover:text-black">5</button>
                            <button onclick="calcPress('6')" class="border border-white p-2 text-center hover:bg-white hover:text-black">6</button>
                            <button onclick="calcPress('*')" class="border border-white p-2 text-center hover:bg-white hover:text-black">*</button>

                            <button onclick="calcPress('1')" class="border border-white p-2 text-center hover:bg-white hover:text-black">1</button>
                            <button onclick="calcPress('2')" class="border border-white p-2 text-center hover:bg-white hover:text-black">2</button>
                            <button onclick="calcPress('3')" class="border border-white p-2 text-center hover:bg-white hover:text-black">3</button>
                            <button onclick="calcPress('-')" class="border border-white p-2 text-center hover:bg-white hover:text-black">-</button>

                            <button onclick="calcPress('0')" class="border border-white p-2 text-center hover:bg-white hover:text-black">0</button>
                            <button onclick="calcClear()" class="border border-white p-2 text-center hover:bg-white hover:text-black">C</button>
                            <button onclick="calcEval()" class="border border-white p-2 text-center bg-white text-black font-bold">=</button>
                            <button onclick="calcPress('+')" class="border border-white p-2 text-center hover:bg-white hover:text-black">+</button>
                        </div>
                    </div>
                </div>
            `);
        }

        let calcVal = "0";
        function calcPress(val) {
            const disp = document.getElementById('calc-display');
            if (!disp) return;
            if (calcVal === "0") calcVal = val; else calcVal += val;
            disp.value = calcVal;
        }
        function calcClear() {
            calcVal = "0";
            const disp = document.getElementById('calc-display');
            if (disp) disp.value = calcVal;
        }
        function calcEval() {
            const disp = document.getElementById('calc-display');
            if (!disp) return;
            try {
                calcVal = eval(calcVal).toString();
                disp.value = calcVal;
            } catch (e) {
                disp.value = "ERROR";
                calcVal = "0";
            }
        }

        function startSnakeGame() {
            openModal("SNAKE.EXE - Snake Game", `
                <div class="h-full flex flex-col items-center justify-center bg-black text-white p-2">
                    <div class="mb-2 font-bold">SNAKE GAME - Use Arrow Keys to Move</div>
                    <canvas id="snake-canvas" width="400" height="300" class="border-2 border-white bg-black"></canvas>
                    <div class="mt-2 text-sm" id="snake-score">SCORE: 0</div>
                </div>
            `);

            setTimeout(() => {
                const canvas = document.getElementById('snake-canvas');
                if (!canvas) return;
                const ctx = canvas.getContext('2d');
                const scoreEl = document.getElementById('snake-score');

                let snake = [{x: 10, y: 10}];
                let food = {x: 15, y: 15};
                let dx = 1, dy = 0;
                let score = 0;
                let gameInterval = null;

                function loop() {
                    const head = {x: snake[0].x + dx, y: snake[0].y + dy};

                    if (head.x < 0 || head.x >= 20 || head.y < 0 || head.y >= 15) {
                        clearInterval(gameInterval);
                        alert("GAME OVER! Score: " + score);
                        closeModal();
                        return;
                    }

                    snake.unshift(head);
                    if (head.x === food.x && head.y === food.y) {
                        score += 10;
                        if (scoreEl) scoreEl.innerText = "SCORE: " + score;
                        food = {x: Math.floor(Math.random()*20), y: Math.floor(Math.random()*15)};
                    } else {
                        snake.pop();
                    }

                    ctx.fillStyle = "#000000";
                    ctx.fillRect(0, 0, 400, 300);

                    ctx.fillStyle = "#ffffff";
                    snake.forEach(s => ctx.fillRect(s.x*20, s.y*20, 18, 18));

                    ctx.fillStyle = "#ffffff";
                    ctx.beginPath();
                    ctx.arc(food.x*20+10, food.y*20+10, 8, 0, Math.PI*2);
                    ctx.fill();
                }

                window.onkeydown = (e) => {
                    if (e.key === "ArrowUp" && dy === 0) { dx = 0; dy = -1; }
                    if (e.key === "ArrowDown" && dy === 0) { dx = 0; dy = 1; }
                    if (e.key === "ArrowLeft" && dx === 0) { dx = -1; dy = 0; }
                    if (e.key === "ArrowRight" && dx === 0) { dx = 1; dy = 0; }
                };

                gameInterval = setInterval(loop, 120);
            }, 100);
        }

        function startMinesGame() {
            openModal("MINES.EXE - Minesweeper", `
                <div class="h-full flex flex-col items-center justify-center bg-black text-white p-2">
                    <div class="mb-2 font-bold">MINESWEEPER</div>
                    <div id="mines-grid" class="grid grid-cols-6 gap-1 border-2 border-white p-2"></div>
                </div>
            `);

            setTimeout(() => {
                const grid = document.getElementById('mines-grid');
                if (!grid) return;
                grid.innerHTML = "";

                for (let i = 0; i < 36; i++) {
                    const btn = document.createElement('button');
                    btn.className = "w-10 h-10 border border-white text-center font-bold hover:bg-white hover:text-black";
                    btn.innerText = "?";
                    btn.onclick = () => {
                        if (Math.random() < 0.2) {
                            btn.innerText = "X";
                            btn.className = "w-10 h-10 border border-white text-center font-bold bg-white text-black";
                            alert("BOOM! Game Over.");
                            closeModal();
                        } else {
                            btn.innerText = "0";
                            btn.disabled = true;
                        }
                    };
                    grid.appendChild(btn);
                }
            }, 100);
        }

        // System Initialization
        window.onload = function() {
            bootDOS();
        };
    </script>
</body>
</html>
