
<html lang="pt-BR" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulador de Rastreamento de Alvo por IA & Controle Físico</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    colors: {
                        tactical: {
                            900: '#0a0f1d',
                            800: '#111827',
                            700: '#1f2937',
                            600: '#374151',
                            accent: '#10b981',
                            glow: '#34d399',
                            alert: '#ef4444'
                        }
                    }
                }
            }
        }
    </script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;600;700&family=Inter:wght@400;500;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; }
        .font-mono { font-family: 'JetBrains Mono', monospace; }
        /* Custom scrollbar for terminal */
        ::-webkit-scrollbar { width: 6px; height: 6px; }
        ::-webkit-scrollbar-track { background: #0a0f1d; }
        ::-webkit-scrollbar-thumb { background: #374151; border-radius: 3px; }
        ::-webkit-scrollbar-thumb:hover { background: #10b981; }
    </style>
</head>
<body class="bg-tactical-900 text-gray-100 min-h-screen flex flex-col antialiased selection:bg-tactical-accent selection:text-black">

    <header class="bg-tactical-800 border-b border-gray-800 px-4 py-3 sticky top-0 z-50 shadow-lg">
        <div class="max-w-7xl mx-auto flex flex-col sm:flex-row justify-between items-center gap-4">
            <div class="flex items-center space-x-3">
                <div class="w-3 h-3 rounded-full bg-tactical-accent animate-pulse"></div>
                <div>
                    <h1 class="font-bold text-lg text-white tracking-wider flex items-center gap-2">
                        <span>SYS_TRACKER_V2</span>
                        <span class="text-xs px-2 py-0.5 rounded bg-tactical-accent/20 text-tactical-accent border border-tactical-accent/30 font-mono">OPENCV + ARDUINO</span>
                    </h1>
                    <p class="text-xs text-gray-400">Simulação Unificada de Visão Computacional (CSRT) & Atuação Física via Serial</p>
                </div>
            </div>
            <div class="flex items-center space-x-2 text-xs font-mono">
                <div class="bg-tactical-700 px-3 py-1.5 rounded-lg border border-gray-700 flex items-center gap-2">
                    <span class="text-gray-400">STATUS:</span>
                    <span id="sys-status-badge" class="text-tactical-alert font-bold">DESCONECTADO / LIVRE</span>
                </div>
                <div class="bg-tactical-700 px-3 py-1.5 rounded-lg border border-gray-700 flex items-center gap-2 hidden sm:flex">
                    <span class="text-gray-400">BAUD:</span>
                    <span class="text-tactical-accent">115200</span>
                </div>
            </div>
        </div>
    </header>

    <main class="flex-1 max-w-7xl w-full mx-auto p-4 lg:p-6 grid grid-cols-1 lg:grid-cols-12 gap-6">
        
        <!-- Left Column: Canvas & Target Control (8 cols) -->
        <div class="lg:col-span-8 flex flex-col gap-4">
            <!-- Canvas Wrapper -->
            <div class="bg-tactical-800 border border-gray-700/80 rounded-2xl p-4 shadow-xl flex flex-col relative overflow-hidden">
                <div class="flex justify-between items-center mb-3">
                    <div class="flex items-center gap-2 text-xs font-mono text-gray-300">
                        <svg class="w-4 h-4 text-tactical-accent" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"/><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"/></svg>
                        <span>CÂMERA DE BORDA (OpenCV 2D Feed)</span>
                    </div>
                    <div class="flex items-center gap-2">
                        <span id="fps-counter" class="text-xs font-mono bg-tactical-900 px-2.5 py-1 rounded text-tactical-accent border border-gray-800">FPS: 60</span>
                        <button id="btn-toggle-source" class="text-xs font-mono bg-tactical-700 hover:bg-tactical-600 text-gray-200 px-3 py-1 rounded transition border border-gray-600">Alternar Alvo Móvel</button>
                    </div>
                </div>

                <!-- Canvas Container -->
                <div class="relative w-full aspect-[4/3] bg-black rounded-xl overflow-hidden border border-gray-800 cursor-crosshair shadow-inner">
                    <canvas id="simCanvas" class="w-full h-full block"></canvas>
                    
                    <!-- HUD Overlay Instructions -->
                    <div id="hud-instruction" class="absolute bottom-3 left-3 bg-black/70 backdrop-blur-md px-3 py-1.5 rounded-lg border border-gray-700/60 text-xs text-gray-300 font-mono pointer-events-none flex items-center gap-2">
                        <span class="w-2 h-2 rounded-full bg-tactical-accent animate-ping"></span>
                        <span>Clique e arraste no vídeo para travar o Alvo (CSRT Tracker)</span>
                    </div>
                </div>

                <!-- Quick Action Bar -->
                <div class="flex flex-wrap items-center justify-between gap-3 mt-4 pt-3 border-t border-gray-700/50">
                    <div class="flex items-center gap-2">
                        <button id="btn-reset-lock" class="px-4 py-2 bg-red-500/20 hover:bg-red-500/30 text-red-400 border border-red-500/30 text-xs font-mono rounded-lg transition flex items-center gap-2">
                            <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/></svg>
                            Liberar Alvo
                        </button>
                        <button id="btn-auto-lock" class="px-4 py-2 bg-tactical-accent/20 hover:bg-tactical-accent/30 text-tactical-accent border border-tactical-accent/30 text-xs font-mono rounded-lg transition flex items-center gap-2">
                            <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z"/></svg>
                            Auto-Travar Alvo
                        </button>
                    </div>
                    <div class="text-xs font-mono text-gray-400">
                        Erro Central (X, Y): <span id="error-display" class="text-white font-bold">[0, 0]</span>
                    </div>
                </div>
            </div>

            <!-- Tabs Section: Overview & Source Code -->
            <div class="bg-tactical-800 border border-gray-700/80 rounded-2xl p-4 shadow-xl">
                <div class="flex border-b border-gray-700 gap-4 mb-4 text-xs font-mono">
                    <button class="tab-btn pb-2 border-b-2 border-tactical-accent text-tactical-accent font-semibold transition" data-target="tab-overview">Visão Geral & Arquitetura</button>
                    <button class="tab-btn pb-2 border-b-2 border-transparent text-gray-400 hover:text-gray-200 transition" data-target="tab-python">Python (OpenCV CSRT)</button>
                    <button class="tab-btn pb-2 border-b-2 border-transparent text-gray-400 hover:text-gray-200 transition" data-target="tab-arduino">C++ (Arduino Uno)</button>
                </div>

                <!-- Tab 1: Overview -->
                <div id="tab-overview" class="tab-content space-y-3 text-sm text-gray-300">
                    <p class="leading-relaxed">
                        Este aplicativo unifica o pipeline completo de rastreamento visual de alta precisão com atuação mecânica de 2 eixos (Pan/Tilt). A arquitetura modular divide-se em:
                    </p>
                    <div class="grid grid-cols-1 sm:grid-cols-2 gap-3 pt-2">
                        <div class="bg-tactical-900/60 p-3 rounded-xl border border-gray-800">
                            <h4 class="font-bold text-tactical-accent text-xs font-mono mb-1">1. IA Local (Edge AI / OpenCV)</h4>
                            <p class="text-xs text-gray-400">Simula o algoritmo <code class="text-gray-200">cv2.TrackerCSRT_create()</code>. Inicializa a assinatura óptica e calcula continuamente o vetor de erro espacial centralizado entre o centro do alvo e o centro do frame da câmera.</p>
                        </div>
                        <div class="bg-tactical-900/60 p-3 rounded-xl border border-gray-800">
                            <h4 class="font-bold text-tactical-accent text-xs font-mono mb-1">2. Controle Físico (Arduino Uno)</h4>
                            <p class="text-xs text-gray-400">Simula a recepção via porta serial (115200 baud). Mapeia os erros espaciais de <code class="text-gray-200">[-320, 320]</code> e <code class="text-gray-200">[-240, 240]</code> para ângulos de <code class="text-gray-200">45° a 135°</code> nos Servos X e Y.</p>
                        </div>
                    </div>
                </div>

                <!-- Tab 2: Python Code -->
                <div id="tab-python" class="tab-content hidden">
                    <pre class="bg-tactical-900 p-3 rounded-xl text-xs font-mono text-emerald-400 overflow-x-auto border border-gray-800 leading-relaxed"><code>import cv2
import numpy as np

class LocalTargetTracker:
    def __init__(self):
        self.tracker = cv2.TrackerCSRT_create()
        self.locked = False

    def lock_target(self, frame, bbox):
        # Inicializa assinatura digital tridimensional/óptica do alvo móvel
        self.tracker.init(frame, bbox)
        self.locked = True

    def update_tracking(self, frame):
        if not self.locked: return None
        success, bbox = self.tracker.update(frame)
        if success:
            # Calcula o vetor de erro espacial centralizado local
            x, y, w, h = [int(v) for v in bbox]
            centro_x, centro_y = x + w//2, y + h//2
            return (centro_x, centro_y)
        return None</code></pre>
                </div>

                <!-- Tab 3: Arduino Code -->
                <div id="tab-arduino" class="tab-content hidden">
                    <pre class="bg-tactical-900 p-3 rounded-xl text-xs font-mono text-cyan-400 overflow-x-auto border border-gray-800 leading-relaxed"><code>#include &lt;Servo.h&gt;

Servo servo_eixo_x;
Servo servo_eixo_y;

void setup() {
  Serial.begin(115200); // Barramento blindado local de comunicação
  servo_eixo_x.attach(9);
  servo_eixo_y.attach(10);
}

void loop() {
  if (Serial.available() > 0) {
    // Recebe as correções locais calculadas pela IA de borda
    int erro_x = Serial.parseInt();
    int erro_y = Serial.parseInt();
    int posX = map(erro_x, -320, 320, 45, 135);
    int posY = map(erro_y, -240, 240, 45, 135);
    servo_eixo_x.write(posX);
    servo_eixo_y.write(posY);
  }
}</code></pre>
                </div>
            </div>
        </div>

        <!-- Right Column: Arduino Servos & Serial Terminal (4 cols) -->
        <div class="lg:col-span-4 flex flex-col gap-4">
            
            <!-- Servo Telemetry Card -->
            <div class="bg-tactical-800 border border-gray-700/80 rounded-2xl p-4 shadow-xl flex flex-col gap-4">
                <div class="flex items-center justify-between border-b border-gray-700 pb-3">
                    <div class="flex items-center gap-2 text-xs font-mono text-gray-200 font-semibold">
                        <svg class="w-4 h-4 text-cyan-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"/><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"/></svg>
                        <span>ATUAÇÃO FÍSICA (ARDUINO UNO)</span>
                    </div>
                    <span class="text-[10px] font-mono px-2 py-0.5 rounded bg-cyan-500/20 text-cyan-300 border border-cyan-500/30">PINS 9 & 10</span>
                </div>

                <!-- Servo X Gauge -->
                <div class="bg-tactical-900/80 p-3 rounded-xl border border-gray-800 flex flex-col gap-2">
                    <div class="flex justify-between items-center text-xs font-mono">
                        <span class="text-gray-400">SERVO EIXO X (Pan - Pin 9)</span>
                        <span id="servo-x-val" class="text-tactical-accent font-bold">90°</span>
                    </div>
                    <div class="w-full bg-gray-800 h-2.5 rounded-full overflow-hidden p-0.5 border border-gray-700">
                        <div id="servo-x-bar" class="bg-tactical-accent h-full rounded-full transition-all duration-150" style="width: 50%;"></div>
                    </div>
                    <div class="flex justify-between text-[10px] font-mono text-gray-500">
                        <span>45° (-320px)</span>
                        <span>90° (Centro)</span>
                        <span>135° (+320px)</span>
                    </div>
                </div>

                <!-- Servo Y Gauge -->
                <div class="bg-tactical-900/80 p-3 rounded-xl border border-gray-800 flex flex-col gap-2">
                    <div class="flex justify-between items-center text-xs font-mono">
                        <span class="text-gray-400">SERVO EIXO Y (Tilt - Pin 10)</span>
                        <span id="servo-y-val" class="text-cyan-400 font-bold">90°</span>
                    </div>
                    <div class="w-full bg-gray-800 h-2.5 rounded-full overflow-hidden p-0.5 border border-gray-700">
                        <div id="servo-y-bar" class="bg-cyan-400 h-full rounded-full transition-all duration-150" style="width: 50%;"></div>
                    </div>
                    <div class="flex justify-between text-[10px] font-mono text-gray-500">
                        <span>45° (-240px)</span>
                        <span>90° (Centro)</span>
                        <span>135° (+240px)</span>
                    </div>
                </div>
            </div>

            <!-- Serial Monitor Card -->
            <div class="bg-tactical-800 border border-gray-700/80 rounded-2xl p-4 shadow-xl flex flex-col flex-1 min-h-[280px]">
                <div class="flex items-center justify-between border-b border-gray-700 pb-3 mb-3">
                    <div class="flex items-center gap-2 text-xs font-mono text-gray-200 font-semibold">
                        <svg class="w-4 h-4 text-emerald-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 9l3 3-3 3m5 0h3M5 20h14a2 2 0 002-2V6a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"/></svg>
                        <span>MONITOR SERIAL (115200 BAUD)</span>
                    </div>
                    <button id="btn-clear-serial" class="text-[10px] font-mono text-gray-400 hover:text-white bg-tactical-900 px-2 py-1 rounded border border-gray-700 transition">Limpar</button>
                </div>

                <!-- Terminal Output Box -->
                <div id="serial-terminal" class="bg-tactical-900 rounded-xl p-3 font-mono text-[11px] text-gray-300 flex-1 overflow-y-auto max-h-[220px] border border-gray-800 flex flex-col gap-1 select-text">
                    <div class="text-gray-500">[SYSTEM] Inicializando barramento serial em 115200 bps...</div>
                    <div class="text-gray-500">[ARDUINO] Setup concluído. Servos anexados aos pinos 9 e 10.</div>
                    <div class="text-emerald-400">[IA_EDGE] LocalTargetTracker pronto. Aguardando seleção de alvo...</div>
                </div>
            </div>

        </div>

    </main>

    <footer class="bg-tactical-800 border-t border-gray-800 py-3 text-center text-xs text-gray-500 font-mono">
        Simulador de Visão Computacional & Atuação Mecânica Unificada &copy; 2026
    </footer>

    <script>
        // --- Tab Switching Logic ---
        document.querySelectorAll('.tab-btn').forEach(btn => {
            btn.addEventListener('click', (e) => {
                document.querySelectorAll('.tab-btn').forEach(b => {
                    b.classList.remove('border-tactical-accent', 'text-tactical-accent', 'font-semibold');
                    b.classList.add('border-transparent', 'text-gray-400');
                });
                btn.classList.add('border-tactical-accent', 'text-tactical-accent', 'font-semibold');
                btn.classList.remove('border-transparent', 'text-gray-400');

                document.querySelectorAll('.tab-content').forEach(c => c.classList.add('hidden'));
                document.getElementById(btn.dataset.target).classList.remove('hidden');
            });
        });

        // --- Canvas & Simulation State ---
        const canvas = document.getElementById('simCanvas');
        const ctx = canvas.getContext('2d');
        
        let canvasWidth = 640;
        let canvasHeight = 480;

        function resizeCanvas() {
            const rect = canvas.parentElement.getBoundingClientRect();
            canvas.width = rect.width;
            canvas.height = rect.height;
            canvasWidth = rect.width;
            canvasHeight = rect.height;
        }

        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();

        // Target Object State
        let target = {
            x: canvasWidth / 2 + 100,
            y: canvasHeight / 2 - 50,
            vx: 2.5,
            vy: 1.8,
            w: 70,
            h: 70,
            color: '#ef4444'
        };

        // Tracker State (Simulating OpenCV CSRT)
        let trackerState = {
            locked: false,
            bbox: { x: 0, y: 0, w: 70, h: 70 },
            lastUpdate: 0
        };

        // Mouse Dragging State for manual box selection
        let isDragging = false;
        let dragStart = { x: 0, y: 0 };
        let dragCurrent = { x: 0, y: 0 };

        // Serial Terminal Logger
        function logSerial(msg, type = 'info') {
            const term = document.getElementById('serial-terminal');
            const div = document.createElement('div');
            let colorClass = 'text-gray-300';
            if (type === 'rx') colorClass = 'text-cyan-400';
            if (type === 'tx') colorClass = 'text-emerald-400';
            if (type === 'warn') colorClass = 'text-amber-400';
            
            div.className = colorClass;
            div.textContent = `[${new Date().toLocaleTimeString()}] ${msg}`;
            term.appendChild(div);
            term.scrollTop = term.scrollHeight;

            // Limit lines
            if (term.children.length > 50) {
                term.removeChild(term.firstChild);
            }
        }

        document.getElementById('btn-clear-serial').addEventListener('click', () => {
            document.getElementById('serial-terminal').innerHTML = '';
            logSerial('Buffer serial limpo.', 'info');
        });

        // Event Listeners for Canvas Interaction (Drawing Bounding Box)
        canvas.addEventListener('mousedown', (e) => {
            const rect = canvas.getBoundingClientRect();
            const scaleX = canvas.width / rect.width;
            const scaleY = canvas.height / rect.height;
            dragStart.x = (e.clientX - rect.left) * scaleX;
            dragStart.y = (e.clientY - rect.top) * scaleY;
            isDragging = true;
        });

        canvas.addEventListener('mousemove', (e) => {
            if (!isDragging) return;
            const rect = canvas.getBoundingClientRect();
            const scaleX = canvas.width / rect.width;
            const scaleY = canvas.height / rect.height;
            dragCurrent.x = (e.clientX - rect.left) * scaleX;
            dragCurrent.y = (e.clientY - rect.top) * scaleY;
        });

        canvas.addEventListener('mouseup', (e) => {
            if (!isDragging) return;
            isDragging = false;
            
            const x = Math.min(dragStart.x, dragCurrent.x);
            const y = Math.min(dragStart.y, dragCurrent.y);
            const w = Math.abs(dragCurrent.x - dragStart.x);
            const h = Math.abs(dragCurrent.y - dragStart.y);

            if (w > 15 && h > 15) {
                // Lock target using selected box
                trackerState.bbox = { x, y, w, h };
                trackerState.locked = true;
                document.getElementById('sys-status-badge').textContent = 'ALVO TRAVADO (CSRT)';
                document.getElementById('sys-status-badge').className = 'text-tactical-accent font-bold';
                logSerial(`[OPENCV] cv2.TrackerCSRT_create() inicializado com bbox: [${Math.round(x)}, ${Math.round(y)}, ${Math.round(w)}, ${Math.round(h)}]`, 'tx');
            }
        });

        // Buttons handlers
        document.getElementById('btn-reset-lock').addEventListener('click', () => {
            trackerState.locked = false;
            document.getElementById('sys-status-badge').textContent = 'DESCONECTADO / LIVRE';
            document.getElementById('sys-status-badge').className = 'text-tactical-alert font-bold';
            logSerial('[OPENCV] Alvo destravado pelo usuário.', 'warn');
        });

        document.getElementById('btn-auto-lock').addEventListener('click', () => {
            trackerState.bbox = {
                x: target.x - target.w/2,
                y: target.y - target.h/2,
                w: target.w,
                h: target.h
            };
            trackerState.locked = true;
            document.getElementById('sys-status-badge').textContent = 'ALVO TRAVADO (AUTO)';
            document.getElementById('sys-status-badge').className = 'text-tactical-accent font-bold';
            logSerial('[OPENCV] Auto-traqueamento ativado no alvo móvel principal.', 'tx');
        });

        let syntheticMoving = true;
        document.getElementById('btn-toggle-source').addEventListener('click', () => {
            syntheticMoving = !syntheticMoving;
            logSerial(`[CÂMERA] Modo de alvo móvel alterado: ${syntheticMoving ? 'Dinâmico (Automático)' : 'Estático'}`, 'info');
        });

        // --- Arduino Mapping Helper ---
        function mapValue(val, in_min, in_max, out_min, out_max) {
            return (val - in_min) * (out_max - out_min) / (in_max - in_min) + out_min;
        }

        let lastSerialSend = 0;

        // --- Main Animation & Processing Loop ---
        function animate(timestamp) {
            // Update target position if synthetic motion is enabled
            if (syntheticMoving) {
                target.x += target.vx;
                target.y += target.vy;

                // Bounce off canvas borders
                if (target.x - target.w/2 < 20 || target.x + target.w/2 > canvas.width - 20) target.vx *= -1;
                if (target.y - target.h/2 < 20 || target.y + target.h/2 > canvas.height - 20) target.vy *= -1;
            }

            // Clear canvas
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            // Draw tactical grid background
            ctx.strokeStyle = 'rgba(31, 41, 55, 0.4)';
            ctx.lineWidth = 1;
            const gridSize = 40;
            for (let x = 0; x < canvas.width; x += gridSize) {
                ctx.beginPath();
                ctx.moveTo(x, 0);
                ctx.lineTo(x, canvas.height);
                ctx.stroke();
            }
            for (let y = 0; y < canvas.height; y += gridSize) {
                ctx.beginPath();
                ctx.moveTo(0, y);
                ctx.lineTo(canvas.width, y);
                ctx.stroke();
            }

            // Draw Center Crosshair (Camera Optical Center)
            const cx = canvas.width / 2;
            const cy = canvas.height / 2;
            ctx.strokeStyle = 'rgba(16, 185, 129, 0.4)';
            ctx.lineWidth = 1.5;
            ctx.beginPath();
            ctx.moveTo(cx - 15, cy); ctx.lineTo(cx + 15, cy);
            ctx.moveTo(cx, cy - 15); ctx.lineTo(cx, cy + 15);
            ctx.stroke();

            // Draw Moving Target (Drone / Object)
            ctx.fillStyle = target.color;
            ctx.strokeStyle = '#fff';
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.roundRect(target.x - target.w/2, target.y - target.h/2, target.w, target.h, 10);
            ctx.fill();
            ctx.stroke();

            // Inner target marker
            ctx.fillStyle = '#ffffff';
            ctx.beginPath();
            ctx.arc(target.x, target.y, 4, 0, Math.PI * 2);
            ctx.fill();

            // If tracker is locked, update bbox to follow target with slight inertia (CSRT simulation)
            if (trackerState.locked) {
                // Smooth tracking interpolation
                const targetBboxX = target.x - trackerState.bbox.w / 2;
                const targetBboxY = target.y - trackerState.bbox.h / 2;
                trackerState.bbox.x += (targetBboxX - trackerState.bbox.x) * 0.2;
                trackerState.bbox.y += (targetBboxY - trackerState.bbox.y) * 0.2;

                const centro_x = trackerState.bbox.x + trackerState.bbox.w / 2;
                const centro_y = trackerState.bbox.y + trackerState.bbox.h / 2;

                // Calculate Spatial Error Vector relative to frame center
                const erro_x = Math.round(centro_x - cx);
                const erro_y = Math.round(centro_y - cy);

                document.getElementById('error-display').textContent = `[${erro_x}, ${erro_y}]`;

                // Draw CSRT Tracker Bounding Box & Center Vector Line
                ctx.strokeStyle = '#10b981';
                ctx.lineWidth = 2;
                ctx.strokeRect(trackerState.bbox.x, trackerState.bbox.y, trackerState.bbox.w, trackerState.bbox.h);

                // Error vector line
                ctx.strokeStyle = 'rgba(52, 211, 153, 0.7)';
                ctx.setLineDash([4, 4]);
                ctx.beginPath();
                ctx.moveTo(cx, cy);
                ctx.lineTo(centro_x, centro_y);
                ctx.stroke();
                ctx.setLineDash([]);

                // Target Lock crosshairs on corners
                const bx = trackerState.bbox.x;
                const by = trackerState.bbox.y;
                const bw = trackerState.bbox.w;
                const bh = trackerState.bbox.h;
                
                ctx.fillStyle = '#10b981';
                // Corners
                const cLen = 10;
                ctx.fillRect(bx, by, cLen, 3);
                ctx.fillRect(bx, by, 3, cLen);
                ctx.fillRect(bx + bw - cLen, by, cLen, 3);
                ctx.fillRect(bx + bw - 3, by, 3, cLen);
                ctx.fillRect(bx, by + bh - 3, cLen, 3);
                ctx.fillRect(bx, by + bh - cLen, 3, cLen);
                ctx.fillRect(bx + bw - cLen, by + bh - 3, cLen, 3);
                ctx.fillRect(bx + bw - 3, by + bh - cLen, 3, cLen);

                // Arduino Servo Mapping & Serial Transmission simulation (every 100ms)
                if (timestamp - lastSerialSend > 100) {
                    lastSerialSend = timestamp;
                    
                    // Arduino mapping simulation: map(erro_x, -320, 320, 45, 135)
                    const posX = Math.round(mapValue(Math.max(-320, Math.min(320, erro_x)), -320, 320, 45, 135));
                    const posY = Math.round(mapValue(Math.max(-240, Math.min(240, erro_y)), -240, 240, 45, 135));

                    // Update UI Servos
                    document.getElementById('servo-x-val').textContent = `${posX}°`;
                    document.getElementById('servo-x-bar').style.width = `${((posX - 45) / 90) * 100}%`;

                    document.getElementById('servo-y-val').textContent = `${posY}°`;
                    document.getElementById('servo-y-bar').style.width = `${((posY - 45) / 90) * 100}%`;

                    logSerial(`[SERIAL_TX] erro_x: ${erro_x}, erro_y: ${erro_y} -> posX: ${posX}°, posY: ${posY}°`, 'rx');
                }
            } else {
                document.getElementById('error-display').textContent = '[0, 0]';
            }

            // Draw manual selection box if dragging
            if (isDragging) {
                ctx.strokeStyle = '#3b82f6';
                ctx.lineWidth = 2;
                ctx.setLineDash([2, 2]);
                const rx = Math.min(dragStart.x, dragCurrent.x);
                const ry = Math.min(dragStart.y, dragCurrent.y);
                const rw = Math.abs(dragCurrent.x - dragStart.x);
                const rh = Math.abs(dragCurrent.y - dragStart.y);
                ctx.strokeRect(rx, ry, rw, rh);
                ctx.setLineDash([]);
            }

            requestAnimationFrame(animate);
        }

        requestAnimationFrame(animate);
        logSerial('[SYSTEM] Loop de renderização Canvas 2D iniciado a 60 FPS.', 'info');
    </script>
</body>
</html>
