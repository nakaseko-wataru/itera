# Itera Blueprint: Pomodoro Timer v1.0

> **What is this?**
> This is an executable blueprint for Itera OS. Simply drag and drop this file into your Itera chat and tell the AI: *"Install this blueprint."*

---

## 🤖 Instructions for the Itera Agent
You are reading an Itera App Blueprint. Your objective is to safely install this application into the user's environment. 
Please execute the installation by strictly following these phases:

1. **Explain**: Briefly explain what this app is to the user and ask for their permission to begin installation. Wait for their approval.
2. **Deploy Code**: Create the necessary files in the Virtual File System (VFS) exactly as provided in the "Source Code" section.
3. **Register App**: Update `system/config/apps.json` to register the new application.
4. **Post-Install Setup**: Ask the user if they want a shortcut added to their main Dashboard (`index.html`). If yes, modify the dashboard safely.

---

## 📦 App Metadata
Use this information when registering the app in the system.

* **App ID**: `pomodoro`
* **Name**: Pomodoro Focus
* **Icon**: 🍅
* **Path**: `apps/pomodoro.html`
* **Description**: Boost productivity with 25-minute focus sessions.

---

## 💻 Source Code

Please create the following file in the VFS.

### File: `apps/pomodoro.html`
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pomodoro Focus</title>
    <!-- Standard Itera UI Kit -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="../system/lib/ui.js"></script>
    <script src="../system/lib/std.js"></script>
</head>
<body class="bg-app text-text-main h-screen flex flex-col p-6 overflow-hidden">

    <!-- Header -->
    <header class="flex items-center justify-between mb-8 shrink-0">
        <div class="flex items-center gap-4">
            <button onclick="AppUI.home()" class="p-2 -ml-2 rounded-full hover:bg-hover text-text-muted hover:text-text-main transition">
                <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18"></path></svg>
            </button>
            <h1 class="text-2xl font-bold tracking-tight">🍅 Pomodoro</h1>
        </div>
    </header>

    <!-- Timer Display -->
    <main class="flex-1 flex flex-col items-center justify-center">
        <div class="bg-panel border border-border-main rounded-3xl p-10 shadow-xl flex flex-col items-center max-w-sm w-full transition-colors duration-500" id="timer-card">
            
            <!-- Mode Selector -->
            <div class="flex gap-2 p-1 bg-card rounded-lg mb-8 border border-border-main">
                <button id="btn-focus" class="px-4 py-1.5 rounded-md text-sm font-bold bg-primary text-white shadow transition">Focus</button>
                <button id="btn-break" class="px-4 py-1.5 rounded-md text-sm font-bold text-text-muted hover:text-text-main transition">Break</button>
            </div>

            <!-- Clock -->
            <div id="time-display" class="text-7xl font-mono font-light tracking-widest text-text-main mb-8">
                25:00
            </div>

            <!-- Controls -->
            <div class="flex gap-4 w-full">
                <button id="btn-toggle" class="flex-1 bg-primary hover:bg-primary/90 text-white font-bold py-3 rounded-xl shadow-lg transition transform active:scale-95 text-lg">
                    START
                </button>
                <button id="btn-reset" class="bg-card hover:bg-hover border border-border-main text-text-muted hover:text-text-main p-3 rounded-xl transition">
                    <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"></path></svg>
                </button>
            </div>
        </div>
        
        <div class="mt-8 text-center text-sm text-text-muted" id="status-text">
            Ready to focus.
        </div>
    </main>

    <!-- App Logic -->
    <script>
        const MODES = {
            FOCUS: { minutes: 25, color: 'border-primary', label: 'Focusing...' },
            BREAK: { minutes: 5, color: 'border-success', label: 'Take a break.' }
        };

        let currentMode = MODES.FOCUS;
        let timeLeft = currentMode.minutes * 60;
        let timerId = null;
        let isRunning = false;

        const els = {
            display: document.getElementById('time-display'),
            toggle: document.getElementById('btn-toggle'),
            reset: document.getElementById('btn-reset'),
            btnFocus: document.getElementById('btn-focus'),
            btnBreak: document.getElementById('btn-break'),
            card: document.getElementById('timer-card'),
            status: document.getElementById('status-text')
        };

        function updateDisplay() {
            const m = Math.floor(timeLeft / 60).toString().padStart(2, '0');
            const s = (timeLeft % 60).toString().padStart(2, '0');
            els.display.textContent = `${m}:${s}`;
        }

        function setMode(modeObj) {
            currentMode = modeObj;
            pauseTimer();
            timeLeft = currentMode.minutes * 60;
            updateDisplay();
            
            // UI Updates
            els.status.textContent = isRunning ? currentMode.label : "Ready.";
            els.card.className = `bg-panel border-2 rounded-3xl p-10 shadow-xl flex flex-col items-center max-w-sm w-full transition-colors duration-500 ${currentMode.color}`;
            
            if (currentMode === MODES.FOCUS) {
                els.btnFocus.className = "px-4 py-1.5 rounded-md text-sm font-bold bg-primary text-white shadow transition";
                els.btnBreak.className = "px-4 py-1.5 rounded-md text-sm font-bold text-text-muted hover:text-text-main transition";
            } else {
                els.btnBreak.className = "px-4 py-1.5 rounded-md text-sm font-bold bg-success text-white shadow transition";
                els.btnFocus.className = "px-4 py-1.5 rounded-md text-sm font-bold text-text-muted hover:text-text-main transition";
            }
        }

        function toggleTimer() {
            isRunning ? pauseTimer() : startTimer();
        }

        function startTimer() {
            isRunning = true;
            els.toggle.textContent = "PAUSE";
            els.toggle.classList.replace('bg-primary', 'bg-warning');
            els.status.textContent = currentMode.label;

            timerId = setInterval(() => {
                timeLeft--;
                updateDisplay();
                if (timeLeft <= 0) handleComplete();
            }, 1000);
        }

        function pauseTimer() {
            isRunning = false;
            clearInterval(timerId);
            els.toggle.textContent = "START";
            els.toggle.classList.replace('bg-warning', 'bg-primary');
            if (els.toggle.classList.contains('bg-success')) els.toggle.classList.replace('bg-success', 'bg-primary');
            els.status.textContent = "Paused.";
        }

        function resetTimer() {
            setMode(currentMode);
        }

        function handleComplete() {
            pauseTimer();
            els.status.textContent = "Session Complete!";
            
            // Notify System if MetaOS is available
            if (window.MetaOS && MetaOS.addEventLog) {
                const sessionType = currentMode === MODES.FOCUS ? "Focus Session" : "Break";
                MetaOS.addEventLog(`User completed a 25-minute ${sessionType}.`, "pomodoro_complete");
            }
            
            // Auto-switch mode
            setMode(currentMode === MODES.FOCUS ? MODES.BREAK : MODES.FOCUS);
        }

        // Event Listeners
        els.toggle.addEventListener('click', toggleTimer);
        els.reset.addEventListener('click', resetTimer);
        els.btnFocus.addEventListener('click', () => setMode(MODES.FOCUS));
        els.btnBreak.addEventListener('click', () => setMode(MODES.BREAK));

        // Init
        updateDisplay();
    </script>
</body>
</html>
```

---

## ⚙️ Registration Steps

Agent, please execute the following:
1. **Read** `system/config/apps.json`.
2. **Parse** the JSON array.
3. **Append** the App Metadata provided above into the array.
4. **Save** the updated JSON back to `system/config/apps.json`.

---

## 🎨 Post-Installation (Interactive)

Once the files are created and the registry is updated, communicate with the user.
Use the `<ask>` tag to say:
> *"Pomodoro Timer has been successfully installed! You can find it in your App Library.*
> *Would you like me to add a shortcut button for it directly onto your Dashboard (`index.html`)?"*

**If the user says YES:**
1. Read `index.html`.
2. Locate the "Quick Launcher" grid section (usually containing buttons for Tasks, Notes, Calendar).
3. Safely append a new button HTML snippet using the `edit_file` tool with a regex or exact block replacement.

Example Button HTML to insert:
```html
<button onclick="AppUI.go('apps/pomodoro.html')" class="flex flex-col items-center justify-center p-4 bg-card hover:bg-hover rounded-xl transition border border-transparent hover:border-primary/50 group">
    <span class="text-2xl mb-1 group-hover:scale-110 transition-transform">🍅</span>
    <span class="text-xs font-bold text-text-main">Pomodoro</span>
</button>
```

**End of Blueprint.**
