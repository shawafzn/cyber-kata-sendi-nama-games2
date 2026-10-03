<!DOCTYPE html>
<html lang="ms" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Cyber Sky — Misi Kata Sendi Nama Tatabahasa</title>

    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    colors: {
                        cyber: {
                            cyan: '#00f0ff',
                            blue: '#0077ff',
                            violet: '#8a2be2',
                            purple: '#b026ff',
                            dark: '#050714',
                            panel: 'rgba(10, 15, 30, 0.85)',
                            amber: '#ffb700',
                            red: '#ff2a6d',
                            green: '#00ff66'
                        }
                    },
                    fontFamily: {
                        orbitron: ['Orbitron', 'sans-serif'],
                        rajdhani: ['Rajdhani', 'sans-serif'],
                        kanit: ['Kanit', 'sans-serif'],
                        prompt: ['Prompt', 'sans-serif'],
                        inter: ['Inter', 'sans-serif']
                    }
                }
            }
        }
    </script>

    <!-- Google Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;800&family=Kanit:wght@400;600;700;900&family=Orbitron:wght@500;700;900&family=Prompt:wght@400;600;700&family=Rajdhani:wght@500;700&display=swap" rel="stylesheet">

    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

    <!-- MediaPipe Libraries for Gesture Tracking -->
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js" crossorigin="anonymous"></script>

    <style>
        * {
            user-select: none;
            -webkit-user-select: none;
            touch-action: manipulation;
        }

        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: #050714;
            font-family: 'Prompt', sans-serif;
            color: #ffffff;
        }

        /* Glassmorphism styling */
        .cyber-glass {
            background: rgba(10, 16, 32, 0.8);
            backdrop-filter: blur(14px);
            -webkit-backdrop-filter: blur(14px);
            border: 1px solid rgba(0, 240, 255, 0.3);
            box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.7),
                        inset 0 0 15px rgba(0, 240, 255, 0.12);
        }

        .cyber-glass-red {
            background: rgba(32, 10, 18, 0.88);
            backdrop-filter: blur(14px);
            border: 1px solid rgba(255, 42, 109, 0.4);
            box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.75), inset 0 0 15px rgba(255, 42, 109, 0.2);
        }

        .cyber-btn {
            background: linear-gradient(135deg, rgba(0, 240, 255, 0.25) 0%, rgba(138, 43, 226, 0.45) 100%);
            border: 1px solid #00f0ff;
            box-shadow: 0 0 15px rgba(0, 240, 255, 0.35);
            transition: all 0.2s ease-in-out;
            text-shadow: 0 0 8px rgba(0, 240, 255, 0.8);
        }

        .cyber-btn:hover {
            background: linear-gradient(135deg, rgba(0, 240, 255, 0.45) 0%, rgba(138, 43, 226, 0.75) 100%);
            box-shadow: 0 0 25px rgba(0, 240, 255, 0.75), inset 0 0 12px rgba(255, 255, 255, 0.5);
            transform: translateY(-2px) scale(1.02);
        }

        .cyber-btn:active {
            transform: translateY(1px) scale(0.98);
        }

        .text-glow-cyan {
            text-shadow: 0 0 10px rgba(0, 240, 255, 0.8), 0 0 20px rgba(0, 240, 255, 0.4);
        }

        .text-glow-amber {
            text-shadow: 0 0 10px rgba(255, 183, 0, 0.8), 0 0 20px rgba(255, 183, 0, 0.4);
        }

        .scanlines {
            background: linear-gradient(
                rgba(18, 16, 16, 0) 50%, 
                rgba(0, 240, 255, 0.04) 50%
            );
            background-size: 100% 4px;
        }

        @keyframes starPop {
            0% { transform: scale(0) rotate(-45deg); opacity: 0; }
            70% { transform: scale(1.3) rotate(10deg); opacity: 1; }
            100% { transform: scale(1) rotate(0deg); opacity: 1; }
        }

        .star-anim {
            animation: starPop 0.6s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
        }

        /* Center feedback toast popup */
        #feedback-toast {
            pointer-events: none;
            transition: all 0.3s ease;
        }

        @media (max-width: 640px) {
            .cyber-glass {
                padding: 0.625rem !important;
            }
            #target-question {
                font-size: 0.95rem !important;
                line-height: 1.3 !important;
            }
        }
    </style>
</head>
<body class="relative bg-cyber-dark text-white select-none overflow-hidden h-screen w-screen">

    <!-- Hidden webcam feed element for MediaPipe -->
    <video id="webcam-feed" class="hidden" playsinline muted></video>

    <!-- Main Game Canvas layer -->
    <canvas id="game-canvas" class="absolute inset-0 w-full h-full z-0 block cursor-crosshair"></canvas>

    <!-- Holographic Overlay FX -->
    <div class="pointer-events-none absolute inset-0 z-10 scanlines opacity-40"></div>

    <!-- MAIN HUD OVERLAY -->
    <div id="hud-layer" class="relative z-20 inset-0 w-full h-full flex flex-col justify-between p-2 md:p-5 pointer-events-none">
        
        <!-- TOP HUD: PROGRESS & QUESTION BANNER & SCORE -->
        <div class="flex flex-col sm:flex-row items-center justify-between gap-2 w-full max-w-7xl mx-auto">
            
            <!-- LEFT: Mission Progress -->
            <div class="cyber-glass px-3 py-1.5 md:px-4 md:py-2.5 rounded-xl md:rounded-2xl flex items-center gap-2.5 w-full sm:w-auto justify-between sm:justify-start">
                <div class="flex items-center gap-2">
                    <div class="w-3 h-3 rounded-full bg-cyber-cyan animate-pulse shadow-[0_0_10px_#00f0ff]"></div>
                    <div>
                        <div class="text-[9px] md:text-[10px] tracking-widest text-cyber-cyan font-orbitron uppercase">TATABAHASA</div>
                        <div class="text-xs md:text-sm font-bold font-orbitron text-white">SENDI NAMA</div>
                    </div>
                </div>
                <div class="flex flex-col items-end sm:items-start ml-2">
                    <div class="text-[9px] md:text-[10px] font-orbitron text-gray-300">SOALAN <span id="progress-text" class="text-cyber-cyan font-bold">01 / 06</span></div>
                    <div id="progress-nodes" class="flex gap-1 mt-0.5">
                        <!-- Progress nodes injected via JS -->
                    </div>
                </div>
            </div>

            <!-- CENTER: BANNER SOALAN TATABAHASA -->
            <div id="target-banner-container" class="cyber-glass px-3 py-2 md:px-6 md:py-3 rounded-xl md:rounded-2xl border border-cyber-cyan/50 text-center shadow-[0_0_20px_rgba(0,240,255,0.25)] w-full sm:w-2/3 max-w-2xl">
                <div class="text-[9px] md:text-[10px] font-orbitron tracking-widest text-cyber-cyan uppercase mb-0.5">
                    <i class="fa-solid fa-bullseye mr-1"></i> ISI TEMPAT KOSONG
                </div>
                <div id="target-question" class="text-xs sm:text-base md:text-xl font-bold font-kanit text-yellow-300 tracking-wide leading-tight">
                    Memuatkan soalan...
                </div>
            </div>

            <!-- RIGHT: SCORE & COMBO -->
            <div class="cyber-glass px-3 py-1.5 md:px-5 md:py-2.5 rounded-xl md:rounded-2xl flex items-center justify-between sm:justify-end gap-3 md:gap-5 w-full sm:w-auto">
                <div>
                    <div class="text-[9px] md:text-[10px] tracking-widest text-gray-400 font-orbitron">MARKAH</div>
                    <div id="hud-score" class="text-lg md:text-2xl font-black font-orbitron text-cyber-cyan text-glow-cyan">0</div>
                </div>
                <div class="border-l border-cyan-500/30 pl-3 md:pl-4">
                    <div class="text-[9px] md:text-[10px] tracking-widest text-gray-400 font-orbitron">COMBO</div>
                    <div id="hud-combo" class="text-lg md:text-2xl font-black font-orbitron text-cyber-amber text-glow-amber">×1</div>
                </div>
            </div>
        </div>

        <!-- CENTER FEEDBACK TOAST / POPUP -->
        <div id="feedback-toast" class="self-center my-auto pointer-events-none opacity-0 transform scale-75 transition-all duration-300 flex flex-col items-center justify-center text-center px-2">
            <div id="toast-box" class="cyber-glass px-5 py-3 md:px-8 md:py-5 rounded-2xl md:rounded-3xl border-2 text-center max-w-md shadow-2xl">
                <div id="toast-icon" class="text-3xl md:text-5xl mb-1">✅</div>
                <div id="toast-title" class="text-base md:text-2xl font-black font-kanit uppercase tracking-wider text-cyber-cyan">TAHNIAH! JAWAPAN BETUL</div>
                <div id="toast-answer" class="text-xl md:text-3xl font-black text-yellow-300 font-orbitron my-1">OLEH</div>
                <div id="toast-explanation" class="text-xs md:text-sm font-medium text-gray-200 leading-snug font-prompt bg-slate-900/80 p-2.5 rounded-xl border border-cyan-500/30">
                    Kata sendi nama 'oleh' digunakan selepas kata kerja pasif.
                </div>
            </div>
        </div>

        <!-- BOTTOM HUD: AI NOVA ASSISTANT & CONTROLS -->
        <div class="flex flex-col sm:flex-row items-end justify-between gap-2 w-full max-w-7xl mx-auto">
            
            <!-- AI NOVA ASSISTANT BOX -->
            <div class="cyber-glass p-2.5 md:p-4 rounded-xl md:rounded-2xl flex items-center gap-2.5 w-full sm:w-3/4 border-l-4 border-l-cyber-cyan">
                <div class="relative flex-shrink-0">
                    <div class="w-8 h-8 md:w-11 md:h-11 rounded-xl bg-cyber-cyan/20 border border-cyber-cyan flex items-center justify-center text-cyber-cyan text-sm md:text-lg shadow-[0_0_12px_rgba(0,240,255,0.4)]">
                        <i class="fa-solid fa-robot animate-pulse"></i>
                    </div>
                </div>
                <div class="overflow-hidden">
                    <div class="flex items-center gap-1.5 mb-0.5">
                        <span class="text-[10px] md:text-xs font-black font-orbitron text-cyber-cyan tracking-wider">A.I. NOVA</span>
                    </div>
                    <p id="nova-dialog" class="text-[11px] md:text-xs font-prompt text-gray-200 truncate font-medium">
                        "Tekan / halakan ke drone Kata Sendi Nama yang tepat!"
                    </p>
                </div>
            </div>

            <!-- CONTROLS TOGGLE & FORMULA QUICK LINK -->
            <div class="pointer-events-auto flex items-center gap-2 w-full sm:w-auto justify-end">
                <button id="btn-quick-formula" class="cyber-glass px-2.5 py-1.5 md:px-3.5 md:py-2.5 rounded-lg md:rounded-xl text-[10px] md:text-xs font-bold font-orbitron text-yellow-300 flex items-center gap-1.5 hover:bg-yellow-400/20 transition border border-yellow-500/40">
                    <i class="fa-solid fa-lightbulb"></i>
                    <span>FORMULA</span>
                </button>
                <button id="toggle-control-btn" class="cyber-glass px-2.5 py-1.5 md:px-3.5 md:py-2.5 rounded-lg md:rounded-xl text-[10px] md:text-xs font-bold font-orbitron text-cyber-cyan flex items-center gap-1.5 hover:bg-cyber-cyan/20 transition">
                    <i class="fa-solid fa-hand"></i>
                    <span id="control-mode-text">MOD: AR KAMERA</span>
                </button>
            </div>
        </div>

    </div>

    <!-- 1. START MODAL -->
    <div id="modal-start" class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/85 backdrop-blur-md">
        <div class="cyber-glass max-w-2xl w-full p-6 md:p-8 rounded-3xl border-2 border-cyber-cyan/50 text-center relative overflow-hidden shadow-[0_0_60px_rgba(0,240,255,0.35)] my-auto">
            
            <div class="text-xs font-orbitron tracking-widest text-cyber-cyan mb-1">APLIKASI GAMIFIKASI TATABAHASA</div>
            <h1 class="text-2xl md:text-4xl font-black font-orbitron text-white tracking-wide mb-2 text-glow-cyan">
                CYBER SKY: KATA SENDI NAMA
            </h1>
            <p class="text-gray-300 text-xs md:text-sm font-prompt mb-6 leading-relaxed">
                Uji dan kukuhkan kefahaman penggunaan Kata Sendi Nama Bahasa Melayu (oleh, dari, daripada, ke, di dalam, pada, dll.) dengan menembak Cyber Drones jawapan betul!
            </p>

            <!-- FORMULA QUICK PREVIEW CARDS -->
            <div class="grid grid-cols-1 md:grid-cols-2 gap-3 mb-6 text-left">
                <div class="bg-cyan-950/50 p-3.5 rounded-2xl border border-cyber-cyan/30">
                    <div class="text-xs font-bold text-cyber-cyan font-orbitron mb-1"><i class="fa-solid fa-compass mr-1"></i> FORMULA DARI (A.T.M.)</div>
                    <p class="text-xs text-gray-200 font-prompt"><strong>A</strong>rah, <strong>T</strong>empat, <strong>M</strong>asa/Waktu (contoh: <em>dari selatan, dari sekolah, dari pukul 8 pagi</em>)</p>
                </div>
                <div class="bg-purple-950/50 p-3.5 rounded-2xl border border-purple-500/30">
                    <div class="text-xs font-bold text-purple-300 font-orbitron mb-1"><i class="fa-solid fa-cubes mr-1"></i> FORMULA DARIPADA (K.O.S.A.)</div>
                    <p class="text-xs text-gray-200 font-prompt"><strong>K</strong>eturunan, <strong>O</strong>rang/Sumber, <strong>S</strong>ebab/Perbandingan, <strong>A</strong>sal kejadian</p>
                </div>
            </div>

            <!-- CONTROLS GUIDE -->
            <div class="grid grid-cols-2 md:grid-cols-4 gap-2.5 mb-6 text-center">
                <div class="bg-slate-900/80 p-3 rounded-xl border border-cyan-500/20">
                    <div class="text-xl mb-0.5">✋</div>
                    <div class="text-[11px] font-bold text-cyber-cyan font-orbitron">1. BUKA TAPAK</div>
                    <div class="text-[10px] text-gray-300 font-prompt">Angkat tapak tangan</div>
                </div>
                <div class="bg-slate-900/80 p-3 rounded-xl border border-cyan-500/20">
                    <div class="text-xl mb-0.5">🎯</div>
                    <div class="text-[11px] font-bold text-cyber-cyan font-orbitron">2. HALA DRONE</div>
                    <div class="text-[10px] text-gray-300 font-prompt">Tujukan pada jawapan</div>
                </div>
                <div class="bg-slate-900/80 p-3 rounded-xl border border-cyan-500/20">
                    <div class="text-xl mb-0.5">⚡</div>
                    <div class="text-[11px] font-bold text-cyber-cyan font-orbitron">3. CAS LASER</div>
                    <div class="text-[10px] text-gray-300 font-prompt">Tunggu sasaran dikunci</div>
                </div>
                <div class="bg-slate-900/80 p-3 rounded-xl border border-cyan-500/20">
                    <div class="text-xl mb-0.5">💥</div>
                    <div class="text-[11px] font-bold text-cyber-cyan font-orbitron">4. TEMBAK</div>
                    <div class="text-[10px] text-gray-300 font-prompt">Musnahkan drone betul</div>
                </div>
            </div>

            <button id="btn-start-game" class="cyber-btn w-full py-4 rounded-2xl font-black font-orbitron text-lg md:text-xl text-white tracking-widest flex items-center justify-center gap-3">
                <i class="fa-solid fa-play text-cyber-cyan"></i>
                <span>MULA MISI TATABAHASA</span>
            </button>
        </div>
    </div>

    <!-- 2. LOADING MODAL -->
    <div id="modal-loading" class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/90 backdrop-blur-md hidden">
        <div class="cyber-glass max-w-md w-full p-8 rounded-3xl border border-cyber-cyan/40 text-center relative">
            <div class="relative w-24 h-24 mx-auto mb-6 flex items-center justify-center">
                <div class="absolute inset-0 rounded-full border-2 border-cyber-cyan/20 animate-ping"></div>
                <div class="absolute inset-0 rounded-full border-2 border-t-cyber-cyan border-r-transparent border-b-cyber-violet border-l-transparent animate-spin"></div>
                <i class="fa-solid fa-microchip text-4xl text-cyber-cyan"></i>
            </div>
            
            <div class="text-xs font-orbitron tracking-widest text-cyber-cyan mb-1">MENGHUBUNGKAN A.I. SENSOR</div>
            <h2 id="loading-status" class="text-base font-bold font-prompt text-white mb-4">Mengaktifkan sistem pengesanan tangan...</h2>
            
            <div class="w-full bg-slate-800 rounded-full h-2 mb-2 overflow-hidden border border-cyan-500/30">
                <div id="loading-bar" class="bg-gradient-to-r from-cyber-cyan to-cyber-violet h-full w-0 transition-all duration-300"></div>
            </div>
            <div id="loading-percent" class="text-xs font-mono text-gray-400">0%</div>
        </div>
    </div>

    <!-- 3. CAMERA ERROR / FALLBACK MODAL -->
    <div id="modal-error" class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/90 backdrop-blur-md hidden">
        <div class="cyber-glass-red max-w-md w-full p-8 rounded-3xl text-center relative">
            <div class="w-16 h-16 mx-auto mb-4 rounded-full bg-red-500/20 border border-red-500 flex items-center justify-center text-red-500 text-3xl">
                <i class="fa-solid fa-triangle-exclamation"></i>
            </div>
            <h2 class="text-xl font-bold font-orbitron text-red-400 mb-2">KAMERA TIDAK TERSEDIA</h2>
            <p class="text-sm font-prompt text-gray-300 mb-6">
                Permainan akan bertukar secara automatik ke mod Skrin Sentuh / Tetikus. Anda masih boleh bermain dengan menekan skrin!
            </p>
            <div class="flex flex-col gap-3">
                <button id="btn-retry-camera" class="cyber-btn py-3 rounded-xl font-bold font-orbitron text-white text-sm">
                    CUBA KAMERA SEMULA
                </button>
                <button id="btn-use-mouse" class="bg-slate-800 hover:bg-slate-700 text-cyan-400 py-3 rounded-xl font-bold font-orbitron text-sm border border-cyan-500/30 transition">
                    GUNAKAN TETIKUS / SKRIN SENTUH
                </button>
            </div>
        </div>
    </div>

    <!-- 4. VICTORY MODAL (MISSION COMPLETE) -->
    <div id="modal-victory" class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/90 backdrop-blur-lg hidden overflow-y-auto">
        <div class="cyber-glass max-w-2xl w-full p-6 md:p-8 rounded-3xl border-2 border-cyber-cyan text-center relative my-auto shadow-[0_0_60px_rgba(0,240,255,0.4)]">
            
            <div class="text-xs font-orbitron tracking-widest text-cyber-cyan uppercase mb-1">MISI KATA SENDI NAMA SELESAI</div>
            <h1 class="text-3xl md:text-4xl font-black font-orbitron text-white tracking-wider mb-2 text-glow-cyan">
                TAHNIAH! MISI BERJAYA
            </h1>

            <!-- Holographic Stars Container -->
            <div id="stars-container" class="flex justify-center items-center gap-4 my-4 h-14">
                <!-- Stars rendered via JS -->
            </div>

            <!-- STATS GRID -->
            <div class="grid grid-cols-2 md:grid-cols-4 gap-3 my-6">
                <div class="bg-slate-900/80 p-3 rounded-2xl border border-cyan-500/30">
                    <div class="text-[10px] font-orbitron text-gray-400">JUMLAH MARKAH</div>
                    <div id="final-score" class="text-2xl font-black font-orbitron text-cyber-cyan">600</div>
                </div>
                <div class="bg-slate-900/80 p-3 rounded-2xl border border-cyan-500/30">
                    <div class="text-[10px] font-orbitron text-gray-400">COMBO MAKSIMUM</div>
                    <div id="final-combo" class="text-2xl font-black font-orbitron text-cyber-amber">×5</div>
                </div>
                <div class="bg-slate-900/80 p-3 rounded-2xl border border-cyan-500/30">
                    <div class="text-[10px] font-orbitron text-gray-400">TEMBAKAN TERSASAR</div>
                    <div id="final-misses" class="text-2xl font-black font-orbitron text-red-400">0</div>
                </div>
                <div class="bg-slate-900/80 p-3 rounded-2xl border border-cyan-500/30">
                    <div class="text-[10px] font-orbitron text-gray-400">SOALAN BETUL</div>
                    <div id="final-mastered" class="text-2xl font-black font-orbitron text-green-400">6 / 6</div>
                </div>
            </div>

            <!-- BUTTON ACTIONS -->
            <div class="grid grid-cols-1 md:grid-cols-3 gap-3">
                <button id="btn-replay" class="cyber-btn py-3.5 rounded-2xl font-bold font-orbitron text-white text-sm flex items-center justify-center gap-2">
                    <i class="fa-solid fa-rotate-right"></i> MAIN SEMULA
                </button>
                <button id="btn-review" class="bg-slate-800 hover:bg-slate-700 text-yellow-300 border border-yellow-500/40 py-3.5 rounded-2xl font-bold font-orbitron text-sm flex items-center justify-center gap-2 transition">
                    <i class="fa-solid fa-book-open"></i> ULANG KAJI & NOTA
                </button>
                <button id="btn-home" class="bg-slate-900 hover:bg-slate-800 text-gray-300 border border-slate-700 py-3.5 rounded-2xl font-bold font-orbitron text-sm flex items-center justify-center gap-2 transition">
                    <i class="fa-solid fa-house"></i> MENU UTAMA
                </button>
            </div>
        </div>
    </div>

    <!-- 5. REVISION & DETAILED GRAMMAR EXPLANATIONS MODAL -->
    <div id="modal-revision" class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black/90 backdrop-blur-md hidden">
        <div class="cyber-glass max-w-4xl w-full max-h-[90vh] p-6 rounded-3xl border border-cyber-cyan flex flex-col relative shadow-2xl">
            
            <div class="flex items-center justify-between pb-4 border-b border-cyan-500/30">
                <div>
                    <h2 class="text-xl md:text-2xl font-black font-orbitron text-cyber-cyan">ANALISIS & PENERANGAN TATABAHASA</h2>
                    <p class="text-xs font-prompt text-gray-300">Ringkasan jawapan beserta formula penjelasan bagi soalan Kata Sendi Nama.</p>
                </div>
                <button id="btn-close-revision" class="w-10 h-10 rounded-full bg-slate-800 hover:bg-slate-700 text-gray-300 flex items-center justify-center text-lg">
                    <i class="fa-solid fa-xmark"></i>
                </button>
            </div>

            <!-- CONTENT SCROLL AREA -->
            <div class="overflow-y-auto my-4 pr-2 flex-grow space-y-4">
                
                <!-- FORMULA QUICK SUMMARY BOX -->
                <div class="grid grid-cols-1 md:grid-cols-2 gap-3">
                    <div class="bg-slate-900/90 p-4 rounded-2xl border border-cyan-500/40">
                        <div class="text-sm font-bold font-orbitron text-cyber-cyan mb-1">📌 FORMULA DARI (A.T.M.)</div>
                        <ul class="text-xs space-y-1 text-gray-200 font-prompt">
                            <li>• <strong>Arah:</strong> Contoh: <em>dari selatan, dari arah timur</em></li>
                            <li>• <strong>Tempat:</strong> Contoh: <em>dari Kuala Lumpur, dari bilik</em></li>
                            <li>• <strong>Masa:</strong> Contoh: <em>dari pukul 8.00 pagi, dari dulu</em></li>
                        </ul>
                    </div>
                    <div class="bg-slate-900/90 p-4 rounded-2xl border border-purple-500/40">
                        <div class="text-sm font-bold font-orbitron text-purple-300 mb-1">📌 FORMULA DARIPADA (K.O.S.A.)</div>
                        <ul class="text-xs space-y-1 text-gray-200 font-prompt">
                            <li>• <strong>Keturunan / Sumber:</strong> Contoh: <em>daripada keturunan raja</em></li>
                            <li>• <strong>Orang / Ahli:</strong> Contoh: <em>daripada cikgu, daripada peserta</em></li>
                            <li>• <strong>Sebab / Perbandingan:</strong> Contoh: <em>lebih tinggi daripada</em></li>
                            <li>• <strong>Asal kejadian:</strong> Contoh: <em>dibuat daripada kayu</em></li>
                        </ul>
                    </div>
                </div>

                <!-- DETAILED QUESTIONS LIST -->
                <div class="text-sm font-bold font-orbitron text-yellow-300 pt-2 border-t border-slate-700">
                    <i class="fa-solid fa-list-check mr-1"></i> ANALISIS SETIAP SOALAN:
                </div>

                <div id="revision-questions-container" class="space-y-3">
                    <!-- Injected dynamically via JS -->
                </div>

            </div>

            <div class="pt-3 border-t border-cyan-500/30 flex justify-end">
                <button id="btn-back-from-revision" class="cyber-btn px-6 py-2.5 rounded-xl font-bold font-orbitron text-sm text-white">
                    KEMBALI
                </button>
            </div>
        </div>
    </div>

    <script>
        /* =========================================================================
         * 1. KATA SENDI NAMA QUESTION DATASET & GRAMMAR EXPLANATIONS
         * ========================================================================= */
        const QUESTION_BANK = [
            {
                id: 1,
                question: "Murid-murid yang ponteng kelas ditangkap ______ guru disiplin.",
                correct: "oleh",
                options: ["oleh", "dari", "daripada", "pada"],
                explanation: "Kata sendi nama 'oleh' digunakan selepas kata kerja pasif ('ditangkap') untuk menunjukkan pelaku atau pembuat tindakan tersebut.",
                tag: "Pelaku / Pembuat"
            },
            {
                id: 2,
                question: "Pameran Seni ini dibuka ______ pukul 10.00 pagi hingga pukul 1.00 tengah hari.",
                correct: "dari",
                options: ["dari", "daripada", "pada", "ke"],
                explanation: "Kata sendi nama 'dari' digunakan untuk menyatakan Arah, Tempat, dan Masa/Waktu (Formula A.T.M.). 'Pukul 10.00 pagi' merujuk kepada Masa/Waktu.",
                tag: "Masa / Waktu (Formula A.T.M.)"
            },
            {
                id: 3,
                question: "Amir hendak pergi ______ Perak pada minggu hadapan.",
                correct: "ke",
                options: ["ke", "di", "dari", "kepada"],
                explanation: "Kata sendi nama 'ke' digunakan untuk menunjukkan tempat atau arah yang dituju, serta masa. 'Perak' merujuk tempat yang dituju.",
                tag: "Tempat / Arah Dituju"
            },
            {
                id: 4,
                question: "Kahar merupakan ketua pengawas di SK Batu Yon. Dia berasal ______ Kuala Nerus, Terengganu.",
                correct: "dari",
                options: ["dari", "daripada", "di", "pada"],
                explanation: "Kata sendi nama 'dari' digunakan untuk menyatakan tempat permulaan atau tempat asal seseorang/sesuatu (Formula A.T.M. — Tempat).",
                tag: "Tempat Asal (Formula A.T.M.)"
            },
            {
                id: 5,
                question: "Mia sedang berehat ______ biliknya setelah mengulang kaji pelajaran.",
                correct: "di dalam",
                options: ["di dalam", "di", "pada", "dalam"],
                explanation: "Kata sendi nama 'di' menunjukkan tempat, manakala kata arah 'dalam' menunjukkan lingkungan atau ruang tertutup seperti bilik.",
                tag: "Tempat / Ruang Tertutup"
            },
            {
                id: 6,
                question: "Di kampung Adham, hujan turun dengan lebat ______ waktu pagi.",
                correct: "pada",
                options: ["pada", "di", "dari", "ke"],
                explanation: "Kata sendi nama 'pada' digunakan untuk menyatakan masa, waktu, tempat bagi perbuatan, atau orang/benda. 'Waktu pagi' merujuk keterangan masa.",
                tag: "Masa / Waktu"
            }
        ];

        /* =========================================================================
         * 2. PROCEDURAL SOUND SYNTHESIZER (WEB AUDIO API)
         * ========================================================================= */
        class AudioSynthesizer {
            constructor() {
                this.ctx = null;
            }

            init() {
                if (!this.ctx) {
                    const AudioContext = window.AudioContext || window.webkitAudioContext;
                    this.ctx = new AudioContext();
                }
                if (this.ctx.state === 'suspended') {
                    this.ctx.resume();
                }
            }

            playChargeRamp(duration = 0.75) {
                if (!this.ctx) return;
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();

                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(160, now);
                osc.frequency.exponentialRampToValueAtTime(850, now + duration);

                gain.gain.setValueAtTime(0.01, now);
                gain.gain.linearRampToValueAtTime(0.15, now + duration);
                gain.gain.linearRampToValueAtTime(0.01, now + duration + 0.05);

                osc.connect(gain);
                gain.connect(this.ctx.destination);

                osc.start(now);
                osc.stop(now + duration + 0.05);
            }

            playAeroPulseBlast() {
                if (!this.ctx) return;
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();

                osc.type = 'sine';
                osc.frequency.setValueAtTime(950, now);
                osc.frequency.exponentialRampToValueAtTime(90, now + 0.3);

                gain.gain.setValueAtTime(0.3, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.3);

                osc.connect(gain);
                gain.connect(this.ctx.destination);

                osc.start(now);
                osc.stop(now + 0.3);
            }

            playDroneExplosion() {
                if (!this.ctx) return;
                const now = this.ctx.currentTime;
                const bufferSize = this.ctx.sampleRate * 0.4;
                const buffer = this.ctx.createBuffer(1, bufferSize, this.ctx.sampleRate);
                const data = buffer.getChannelData(0);
                for (let i = 0; i < bufferSize; i++) {
                    data[i] = Math.random() * 2 - 1;
                }

                const noise = this.ctx.createBufferSource();
                noise.buffer = buffer;

                const filter = this.ctx.createBiquadFilter();
                filter.type = 'lowpass';
                filter.frequency.setValueAtTime(800, now);
                filter.frequency.linearRampToValueAtTime(100, now + 0.4);

                const gain = this.ctx.createGain();
                gain.gain.setValueAtTime(0.4, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.4);

                noise.connect(filter);
                filter.connect(gain);
                gain.connect(this.ctx.destination);

                noise.start(now);
                noise.stop(now + 0.4);
            }

            playDeflectBuzz() {
                if (!this.ctx) return;
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();

                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(120, now);
                osc.frequency.setValueAtTime(70, now + 0.1);

                gain.gain.setValueAtTime(0.25, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.25);

                osc.connect(gain);
                gain.connect(this.ctx.destination);

                osc.start(now);
                osc.stop(now + 0.25);
            }

            playVictoryArpeggio() {
                if (!this.ctx) return;
                const now = this.ctx.currentTime;
                const notes = [523.25, 659.25, 783.99, 1046.50];
                notes.forEach((freq, i) => {
                    const osc = this.ctx.createOscillator();
                    const gain = this.ctx.createGain();
                    osc.type = 'triangle';
                    osc.frequency.setValueAtTime(freq, now + i * 0.08);

                    gain.gain.setValueAtTime(0.2, now + i * 0.08);
                    gain.gain.exponentialRampToValueAtTime(0.001, now + i * 0.08 + 0.25);

                    osc.connect(gain);
                    gain.connect(this.ctx.destination);

                    osc.start(now + i * 0.08);
                    osc.stop(now + i * 0.08 + 0.25);
                });
            }

            playTargetLock() {
                if (!this.ctx) return;
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();

                osc.type = 'sine';
                osc.frequency.setValueAtTime(1200, now);

                gain.gain.setValueAtTime(0.15, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.08);

                osc.connect(gain);
                gain.connect(this.ctx.destination);

                osc.start(now);
                osc.stop(now + 0.08);
            }

            playMissionComplete() {
                if (!this.ctx) return;
                const now = this.ctx.currentTime;
                const chords = [
                    { time: 0.0, freq: 523.25 },
                    { time: 0.1, freq: 659.25 },
                    { time: 0.2, freq: 783.99 },
                    { time: 0.3, freq: 1046.50 },
                    { time: 0.5, freq: 880.00 },
                    { time: 0.6, freq: 1046.50 },
                    { time: 0.8, freq: 1174.66 }
                ];

                chords.forEach(note => {
                    const osc = this.ctx.createOscillator();
                    const gain = this.ctx.createGain();
                    osc.type = 'sawtooth';
                    osc.frequency.setValueAtTime(note.freq, now + note.time);

                    gain.gain.setValueAtTime(0.2, now + note.time);
                    gain.gain.exponentialRampToValueAtTime(0.001, now + note.time + 0.4);

                    osc.connect(gain);
                    gain.connect(this.ctx.destination);

                    osc.start(now + note.time);
                    osc.stop(now + note.time + 0.4);
                });
            }
        }

        const soundSynth = new AudioSynthesizer();

        /* =========================================================================
         * 3. GLOBAL GAME STATE
         * ========================================================================= */
        const gameState = {
            active: false,
            score: 0,
            combo: 1,
            maxCombo: 1,
            misses: 0,
            currentQuestionIndex: 0,
            currentQuestion: null,
            drones: [],
            particles: [],
            beams: [],
            stars: [],
            
            // Aiming & Lock State
            pointer: { x: 0.5, y: 0.5, active: false, isOpenPalm: false, lastSeen: 0 },
            targetedDrone: null,
            chargeTimer: 0,
            chargeDuration: 0.7, // 0.7 seconds lock-on time
            isCharging: false,
            
            // Camera & MediaPipe
            useWebcam: true,
            cameraInstance: null,
            handsInstance: null,
            handLandmarks: null,
            
            // Visual FX
            shakeIntensity: 0,
            flashOpacity: 0
        };

        let canvas, ctx;
        let animationFrameId;

        // Initialize DOM & listeners
        window.addEventListener('DOMContentLoaded', () => {
            initDOMReferences();
            initProgressNodes();
            initBackgroundStars();
            setupCanvas();
            setupEventListeners();
        });

        function initDOMReferences() {
            canvas = document.getElementById('game-canvas');
            ctx = canvas.getContext('2d');
        }

        function initProgressNodes() {
            const container = document.getElementById('progress-nodes');
            container.innerHTML = '';
            for (let i = 0; i < QUESTION_BANK.length; i++) {
                const node = document.createElement('div');
                node.id = `node-${i}`;
                node.className = 'w-2.5 h-2.5 rounded-full bg-slate-700 transition-all duration-300 border border-slate-600';
                container.appendChild(node);
            }
        }

        function updateProgressNodes() {
            document.getElementById('progress-text').innerText = `${String(gameState.currentQuestionIndex + 1).padStart(2, '0')} / ${String(QUESTION_BANK.length).padStart(2, '0')}`;
            for (let i = 0; i < QUESTION_BANK.length; i++) {
                const node = document.getElementById(`node-${i}`);
                if (i < gameState.currentQuestionIndex) {
                    node.className = 'w-3 h-3 rounded-full bg-cyber-cyan shadow-[0_0_8px_#00f0ff] border border-white';
                } else if (i === gameState.currentQuestionIndex) {
                    node.className = 'w-3 h-3 rounded-full bg-yellow-400 animate-pulse border border-yellow-200';
                } else {
                    node.className = 'w-2.5 h-2.5 rounded-full bg-slate-800 border border-slate-700';
                }
            }
        }

        function initBackgroundStars() {
            gameState.stars = [];
            for (let i = 0; i < 120; i++) {
                gameState.stars.push({
                    x: Math.random(),
                    y: Math.random(),
                    size: Math.random() * 2 + 0.5,
                    speed: Math.random() * 0.0008 + 0.0002,
                    alpha: Math.random() * 0.7 + 0.3
                });
            }
        }

        function setupCanvas() {
            const dpr = window.devicePixelRatio || 1;
            canvas.width = window.innerWidth * dpr;
            canvas.height = window.innerHeight * dpr;
            ctx.scale(dpr, dpr);
        }

        window.addEventListener('resize', () => {
            setupCanvas();
        });

        function setupEventListeners() {
            // Start Game
            document.getElementById('btn-start-game').addEventListener('click', () => {
                soundSynth.init();
                document.getElementById('modal-start').classList.add('hidden');
                startCameraAndMediaPipe();
            });

            // Camera Retry / Mouse Fallback
            document.getElementById('btn-retry-camera').addEventListener('click', () => {
                document.getElementById('modal-error').classList.add('hidden');
                startCameraAndMediaPipe();
            });

            document.getElementById('btn-use-mouse').addEventListener('click', () => {
                document.getElementById('modal-error').classList.add('hidden');
                gameState.useWebcam = false;
                document.getElementById('control-mode-text').innerText = 'MOD: TETIKUS / SKRIN';
                startGameplay();
            });

            document.getElementById('toggle-control-btn').addEventListener('click', () => {
                gameState.useWebcam = !gameState.useWebcam;
                document.getElementById('control-mode-text').innerText = gameState.useWebcam ? 'MOD: KAMERA AR' : 'MOD: TETIKUS / SKRIN';
                if (gameState.useWebcam && !gameState.cameraInstance) {
                    startCameraAndMediaPipe();
                }
            });

            document.getElementById('btn-quick-formula').addEventListener('click', () => {
                openRevisionModal();
            });

            // Victory Actions
            document.getElementById('btn-replay').addEventListener('click', () => {
                document.getElementById('modal-victory').classList.add('hidden');
                resetGame();
                startGameplay();
            });

            document.getElementById('btn-review').addEventListener('click', () => {
                openRevisionModal();
            });

            document.getElementById('btn-home').addEventListener('click', () => {
                document.getElementById('modal-victory').classList.add('hidden');
                document.getElementById('modal-start').classList.remove('hidden');
            });

            document.getElementById('btn-close-revision').addEventListener('click', () => {
                document.getElementById('modal-revision').classList.add('hidden');
            });

            document.getElementById('btn-back-from-revision').addEventListener('click', () => {
                document.getElementById('modal-revision').classList.add('hidden');
                if (!gameState.active) {
                    document.getElementById('modal-victory').classList.remove('hidden');
                }
            });

            // Touch / Mouse Pointer Aiming Fallback
            const handlePointerMove = (e) => {
                if (!gameState.useWebcam) {
                    const rect = canvas.getBoundingClientRect();
                    const clientX = e.touches ? e.touches[0].clientX : e.clientX;
                    const clientY = e.touches ? e.touches[0].clientY : e.clientY;
                    gameState.pointer.x = (clientX - rect.left) / rect.width;
                    gameState.pointer.y = (clientY - rect.top) / rect.height;
                    gameState.pointer.active = true;
                    gameState.pointer.isOpenPalm = true;
                    gameState.pointer.lastSeen = Date.now();
                }
            };

            window.addEventListener('mousemove', handlePointerMove);
            window.addEventListener('touchstart', handlePointerMove);
            window.addEventListener('touchmove', handlePointerMove);
        }

        /* Start MediaPipe Hands Detection */
        function startCameraAndMediaPipe() {
            const loadingModal = document.getElementById('modal-loading');
            const loadingStatus = document.getElementById('loading-status');
            const loadingBar = document.getElementById('loading-bar');
            const loadingPercent = document.getElementById('loading-percent');

            loadingModal.classList.remove('hidden');
            loadingStatus.innerText = 'Mengaktifkan pengesanan MediaPipe Hands...';
            loadingBar.style.width = '35%';
            loadingPercent.innerText = '35%';

            try {
                const hands = new Hands({
                    locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`
                });

                hands.setOptions({
                    maxNumHands: 1,
                    modelComplexity: 1,
                    minDetectionConfidence: 0.6,
                    minTrackingConfidence: 0.6
                });

                hands.onResults(onHandResults);
                gameState.handsInstance = hands;

                const videoElement = document.getElementById('webcam-feed');
                
                loadingBar.style.width = '70%';
                loadingPercent.innerText = '70%';
                loadingStatus.innerText = 'Mengakses kamera peranti...';

                const camera = new Camera(videoElement, {
                    onFrame: async () => {
                        if (gameState.useWebcam && gameState.handsInstance) {
                            await gameState.handsInstance.send({ image: videoElement });
                        }
                    },
                    width: 1280,
                    height: 720
                });

                camera.start().then(() => {
                    loadingBar.style.width = '100%';
                    loadingPercent.innerText = '100%';
                    setTimeout(() => {
                        loadingModal.classList.add('hidden');
                        gameState.cameraInstance = camera;
                        startGameplay();
                    }, 400);
                }).catch(err => {
                    console.warn("Camera fallback to touch/mouse:", err);
                    loadingModal.classList.add('hidden');
                    document.getElementById('modal-error').classList.remove('hidden');
                });

            } catch (err) {
                console.error("MediaPipe initialization error:", err);
                loadingModal.classList.add('hidden');
                document.getElementById('modal-error').classList.remove('hidden');
            }
        }

        /* Process Hand Landmarks */
        function onHandResults(results) {
            if (results.multiHandLandmarks && results.multiHandLandmarks.length > 0) {
                const landmarks = results.multiHandLandmarks[0];
                gameState.handLandmarks = landmarks;

                const palmCenterX = (landmarks[0].x + landmarks[5].x + landmarks[17].x) / 3;
                const palmCenterY = (landmarks[0].y + landmarks[5].y + landmarks[17].y) / 3;

                // Mirrored horizontal orientation
                gameState.pointer.x = 1 - palmCenterX;
                gameState.pointer.y = palmCenterY;
                gameState.pointer.active = true;
                gameState.pointer.lastSeen = Date.now();

                const isExtended = (tipIdx, pipIdx) => {
                    const dTip = Math.hypot(landmarks[tipIdx].x - landmarks[0].x, landmarks[tipIdx].y - landmarks[0].y);
                    const dPip = Math.hypot(landmarks[pipIdx].x - landmarks[0].x, landmarks[pipIdx].y - landmarks[0].y);
                    return dTip > dPip * 1.12;
                };

                const fingersOpen = [
                    isExtended(8, 6),   // Index
                    isExtended(12, 10), // Middle
                    isExtended(16, 14), // Ring
                    isExtended(20, 18)  // Pinky
                ];

                gameState.pointer.isOpenPalm = fingersOpen.filter(Boolean).length >= 3;

            } else {
                gameState.handLandmarks = null;
                if (Date.now() - gameState.pointer.lastSeen > 800) {
                    gameState.pointer.active = false;
                }
            }
        }

        function resetGame() {
            gameState.score = 0;
            gameState.combo = 1;
            gameState.maxCombo = 1;
            gameState.misses = 0;
            gameState.currentQuestionIndex = 0;
            gameState.drones = [];
            gameState.particles = [];
            gameState.beams = [];
            gameState.targetedDrone = null;
            gameState.chargeTimer = 0;
            gameState.isCharging = false;
            
            updateHUDScore();
            updateProgressNodes();
        }

        function startGameplay() {
            resetGame();
            gameState.active = true;
            loadQuestionRound(gameState.currentQuestionIndex);

            if (!animationFrameId) {
                gameLoop();
            }
        }

        function loadQuestionRound(index) {
            if (index >= QUESTION_BANK.length) {
                onMissionComplete();
                return;
            }

            gameState.currentQuestion = QUESTION_BANK[index];
            updateProgressNodes();

            // Update UI Banner
            document.getElementById('target-question').innerText = `"${gameState.currentQuestion.question}"`;
            document.getElementById('nova-dialog').innerText = `"Petua A.I. NOVA: Tembak drone dengan Kata Sendi Nama yang tepat untuk melengkapkan ayat ini."`;

            // Spawn Drones carrying options
            spawnDronesForQuestion(gameState.currentQuestion);
        }

        function spawnDronesForQuestion(qObj) {
            gameState.drones = [];
            const options = [...qObj.options];
            shuffleArray(options);

            const count = options.length;
            const screenW = window.innerWidth;
            const screenH = window.innerHeight;
            const isMobile = screenW < 640;

            // Dynamically scale drone size for small mobile screens
            const droneRadius = isMobile ? Math.min(screenW * 0.1, 46) : 65;

            options.forEach((optText, i) => {
                let targetX, targetY;

                if (isMobile) {
                    // Responsive 2x2 Grid Layout on Mobile Phones
                    const col = i % 2;
                    const row = Math.floor(i / 2);

                    const startX = screenW * 0.28;
                    const gapX = screenW * 0.44;
                    targetX = startX + col * gapX;

                    const startY = screenH * 0.38;
                    const gapY = screenH * 0.15;
                    targetY = startY + row * gapY;
                } else {
                    // Horizontal wave row layout on Laptops and Tablets
                    const marginX = screenW * 0.15;
                    const availableW = screenW - marginX * 2;
                    const spacingX = availableW / (count - 1 || 1);

                    targetX = marginX + spacingX * i;
                    targetY = screenH * 0.38 + (Math.sin(i * 1.8) * 32);
                }

                gameState.drones.push({
                    id: i,
                    text: optText,
                    isCorrect: optText === qObj.correct,
                    x: targetX,
                    y: targetY,
                    baseX: targetX,
                    baseY: targetY,
                    radius: droneRadius,
                    glowAlpha: 0,
                    shieldHitAlpha: 0,
                    destroyed: false
                });
            });
        }

        function updateAndDrawDrones(w, h) {
            const time = Date.now() * 0.002;
            const isMobile = w < 640;

            gameState.drones.forEach(drone => {
                if (drone.destroyed) return;

                drone.x = drone.baseX + Math.sin(time + drone.id) * (isMobile ? 8 : 16);
                drone.y = drone.baseY + Math.cos(time * 1.2 + drone.id) * (isMobile ? 8 : 14);

                ctx.save();
                ctx.translate(drone.x, drone.y);

                // Propellers
                ctx.strokeStyle = '#475569';
                ctx.lineWidth = isMobile ? 2.5 : 4;
                const propOffset = drone.radius * 0.85;

                for (let i = 0; i < 4; i++) {
                    const armAngle = (Math.PI / 2) * i + Math.PI / 4;
                    const armX = Math.cos(armAngle) * propOffset;
                    const armY = Math.sin(armAngle) * propOffset;

                    ctx.beginPath();
                    ctx.moveTo(0, 0);
                    ctx.lineTo(armX, armY);
                    ctx.stroke();

                    ctx.save();
                    ctx.translate(armX, armY);
                    ctx.rotate(time * 12 + i);
                    ctx.fillStyle = 'rgba(0, 240, 255, 0.4)';
                    ctx.fillRect(-drone.radius * 0.35, -2, drone.radius * 0.7, 4);
                    ctx.restore();
                }

                // Core Sphere
                ctx.beginPath();
                ctx.arc(0, 0, drone.radius, 0, Math.PI * 2);
                ctx.fillStyle = 'rgba(15, 23, 42, 0.92)';
                ctx.fill();
                ctx.lineWidth = 2.5;
                ctx.strokeStyle = drone === gameState.targetedDrone ? '#00f0ff' : 'rgba(0, 240, 255, 0.45)';
                ctx.stroke();

                // Target Lock Glow
                if (drone.glowAlpha > 0) {
                    ctx.beginPath();
                    ctx.arc(0, 0, drone.radius + 8, 0, Math.PI * 2);
                    ctx.strokeStyle = `rgba(0, 240, 255, ${drone.glowAlpha})`;
                    ctx.lineWidth = 4;
                    ctx.stroke();
                    drone.glowAlpha *= 0.92;
                }

                // Deflect Shield Hit
                if (drone.shieldHitAlpha > 0) {
                    ctx.beginPath();
                    ctx.arc(0, 0, drone.radius + 12, 0, Math.PI * 2);
                    ctx.fillStyle = `rgba(255, 42, 109, ${drone.shieldHitAlpha * 0.4})`;
                    ctx.fill();
                    ctx.strokeStyle = `rgba(255, 42, 109, ${drone.shieldHitAlpha})`;
                    ctx.lineWidth = 3;
                    ctx.stroke();
                    drone.shieldHitAlpha *= 0.9;
                }

                // Drone Word Text
                const fontSize = Math.max(13, Math.floor(drone.radius * 0.38));
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.font = `bold ${fontSize}px "Kanit", "Orbitron", sans-serif`;
                ctx.fillStyle = '#ffffff';
                ctx.shadowColor = '#00f0ff';
                ctx.shadowBlur = 8;
                ctx.fillText(drone.text, 0, 0);

                ctx.restore();
            });
        }

        function setupEventListeners() {

            // Direct Click / Tap on canvas for instant response on mobile devices
            canvas.addEventListener('click', (e) => {
                if (!gameState.active) return;
                const rect = canvas.getBoundingClientRect();
                const clickX = e.clientX - rect.left;
                const clickY = e.clientY - rect.top;

                gameState.drones.forEach(drone => {
                    if (drone.destroyed) return;
                    const dist = Math.hypot(drone.x - clickX, drone.y - clickY);
                    if (dist <= drone.radius + 18) {
                        fireBeamAtDrone(drone, clickX, clickY);
                    }
                });
            });
        }

        function shuffleArray(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
        }

        /* Main Game Animation Loop */
        function gameLoop() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);

            const w = window.innerWidth;
            const h = window.innerHeight;

            // 1. Draw Background
            drawCyberBackground(w, h);

            if (gameState.active) {
                // 2. Update & Draw Drones
                updateAndDrawDrones(w, h);

                // 3. Process Lock-on Targeting
                processTargetLocking(w, h);

                // 4. Beams & Particles
                updateAndDrawBeamsAndParticles();

                // 5. Draw Cyber Gauntlet Overlay
                drawCyberGauntlet(w, h);
            }

            // 6. Draw Screen FX
            drawScreenEffects(w, h);

            animationFrameId = requestAnimationFrame(gameLoop);
        }

        function drawCyberBackground(w, h) {
            const skyGrad = ctx.createLinearGradient(0, 0, 0, h);
            skyGrad.addColorStop(0, '#050714');
            skyGrad.addColorStop(0.5, '#0a1128');
            skyGrad.addColorStop(1, '#1c2541');
            ctx.fillStyle = skyGrad;
            ctx.fillRect(0, 0, w, h);

            // Stars
            ctx.fillStyle = '#00f0ff';
            gameState.stars.forEach(star => {
                star.y += star.speed;
                if (star.y > 1) star.y = 0;
                ctx.globalAlpha = star.alpha * (0.5 + Math.sin(Date.now() * 0.003 + star.x * 10) * 0.5);
                ctx.beginPath();
                ctx.arc(star.x * w, star.y * h, star.size, 0, Math.PI * 2);
                ctx.fill();
            });
            ctx.globalAlpha = 1.0;

            // Perspective Grid
            ctx.strokeStyle = 'rgba(0, 240, 255, 0.1)';
            ctx.lineWidth = 1;
            const horizonY = h * 0.72;
            const centerX = w / 2;

            for (let x = -w; x < w * 2; x += 85) {
                ctx.beginPath();
                ctx.moveTo(centerX, horizonY);
                ctx.lineTo(x, h);
                ctx.stroke();
            }

            for (let y = horizonY; y < h; y += 28) {
                ctx.beginPath();
                ctx.moveTo(0, y);
                ctx.lineTo(w, y);
                ctx.stroke();
            }
        }

        function updateAndDrawDrones(w, h) {
            const time = Date.now() * 0.002;

            gameState.drones.forEach(drone => {
                if (drone.destroyed) return;

                drone.x = drone.baseX + Math.sin(time + drone.id) * 18;
                drone.y = drone.baseY + Math.cos(time * 1.2 + drone.id) * 15;

                ctx.save();
                ctx.translate(drone.x, drone.y);

                // Propellers
                ctx.strokeStyle = '#475569';
                ctx.lineWidth = 4;
                for (let i = 0; i < 4; i++) {
                    const armAngle = (Math.PI / 2) * i + Math.PI / 4;
                    const armX = Math.cos(armAngle) * 58;
                    const armY = Math.sin(armAngle) * 58;

                    ctx.beginPath();
                    ctx.moveTo(0, 0);
                    ctx.lineTo(armX, armY);
                    ctx.stroke();

                    ctx.save();
                    ctx.translate(armX, armY);
                    ctx.rotate(time * 12 + i);
                    ctx.fillStyle = 'rgba(0, 240, 255, 0.4)';
                    ctx.fillRect(-22, -2, 44, 4);
                    ctx.restore();
                }

                // Core Sphere
                ctx.beginPath();
                ctx.arc(0, 0, drone.radius, 0, Math.PI * 2);
                ctx.fillStyle = 'rgba(15, 23, 42, 0.9)';
                ctx.fill();
                ctx.lineWidth = 3;
                ctx.strokeStyle = drone === gameState.targetedDrone ? '#00f0ff' : 'rgba(0, 240, 255, 0.4)';
                ctx.stroke();

                // Target Lock Glow
                if (drone.glowAlpha > 0) {
                    ctx.beginPath();
                    ctx.arc(0, 0, drone.radius + 12, 0, Math.PI * 2);
                    ctx.strokeStyle = `rgba(0, 240, 255, ${drone.glowAlpha})`;
                    ctx.lineWidth = 6;
                    ctx.stroke();
                    drone.glowAlpha *= 0.92;
                }

                // Deflect Shield Hit
                if (drone.shieldHitAlpha > 0) {
                    ctx.beginPath();
                    ctx.arc(0, 0, drone.radius + 16, 0, Math.PI * 2);
                    ctx.fillStyle = `rgba(255, 42, 109, ${drone.shieldHitAlpha * 0.4})`;
                    ctx.fill();
                    ctx.strokeStyle = `rgba(255, 42, 109, ${drone.shieldHitAlpha})`;
                    ctx.lineWidth = 4;
                    ctx.stroke();
                    drone.shieldHitAlpha *= 0.9;
                }

                // Drone Word Text
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.font = 'bold 22px "Kanit", "Orbitron", sans-serif';
                ctx.fillStyle = '#ffffff';
                ctx.shadowColor = '#00f0ff';
                ctx.shadowBlur = 10;
                ctx.fillText(drone.text, 0, 0);

                ctx.restore();
            });
        }

        function processTargetLocking(w, h) {
            if (!gameState.pointer.active || !gameState.pointer.isOpenPalm) {
                gameState.targetedDrone = null;
                gameState.chargeTimer = 0;
                gameState.isCharging = false;
                return;
            }

            const pointerX = gameState.pointer.x * w;
            const pointerY = gameState.pointer.y * h;

            let closestDrone = null;
            let minDist = 115;

            gameState.drones.forEach(drone => {
                if (drone.destroyed) return;
                const dist = Math.hypot(drone.x - pointerX, drone.y - pointerY);
                if (dist < minDist) {
                    minDist = dist;
                    closestDrone = drone;
                }
            });

            if (closestDrone) {
                if (gameState.targetedDrone !== closestDrone) {
                    gameState.targetedDrone = closestDrone;
                    gameState.chargeTimer = 0;
                    gameState.isCharging = true;
                    soundSynth.playTargetLock();
                    document.getElementById('nova-dialog').innerText = `"KUNCI SASARAN! Mengecas tenaga tembakan ke drone '${closestDrone.text}'..."`;
                } else {
                    gameState.chargeTimer += 1 / 60;
                    closestDrone.glowAlpha = 0.85;

                    if (Math.random() < 0.15) {
                        soundSynth.playChargeRamp(0.2);
                    }

                    drawTargetReticle(closestDrone, gameState.chargeTimer / gameState.chargeDuration);

                    if (gameState.chargeTimer >= gameState.chargeDuration) {
                        fireBeamAtDrone(closestDrone, pointerX, pointerY);
                        gameState.targetedDrone = null;
                        gameState.chargeTimer = 0;
                        gameState.isCharging = false;
                    }
                }
            } else {
                gameState.targetedDrone = null;
                gameState.chargeTimer = 0;
                gameState.isCharging = false;
            }
        }

        function drawTargetReticle(drone, progress) {
            const time = Date.now() * 0.005;
            ctx.save();
            ctx.translate(drone.x, drone.y);

            ctx.save();
            ctx.rotate(time);
            ctx.strokeStyle = '#00f0ff';
            ctx.lineWidth = 2.5;
            ctx.setLineDash([12, 8]);
            ctx.beginPath();
            ctx.arc(0, 0, drone.radius + 20, 0, Math.PI * 2);
            ctx.stroke();
            ctx.restore();

            ctx.beginPath();
            ctx.arc(0, 0, drone.radius + 26, -Math.PI / 2, -Math.PI / 2 + (Math.PI * 2 * Math.min(progress, 1)));
            ctx.strokeStyle = '#00f0ff';
            ctx.lineWidth = 6;
            ctx.lineCap = 'round';
            ctx.shadowColor = '#00f0ff';
            ctx.shadowBlur = 15;
            ctx.stroke();

            ctx.fillStyle = '#00f0ff';
            ctx.font = 'bold 12px "Orbitron", sans-serif';
            ctx.textAlign = 'center';
            ctx.fillText(`LOCK ${Math.floor(Math.min(progress, 1) * 100)}%`, 0, drone.radius + 45);

            ctx.restore();
        }

        function fireBeamAtDrone(drone, startX, startY) {
            soundSynth.playAeroPulseBlast();

            gameState.beams.push({
                x1: startX,
                y1: startY,
                x2: drone.x,
                y2: drone.y,
                alpha: 1.0,
                width: 18
            });

            if (drone.isCorrect) {
                // CORRECT JAWAPAN!
                soundSynth.playDroneExplosion();
                soundSynth.playVictoryArpeggio();

                drone.destroyed = true;
                spawnExplosionDebris(drone.x, drone.y);
                triggerScreenShake(12);
                triggerScreenFlash('#00f0ff');

                const earnedScore = 100 * gameState.combo;
                gameState.score += earnedScore;
                gameState.combo = Math.min(gameState.combo + 1, 5);
                if (gameState.combo > gameState.maxCombo) gameState.maxCombo = gameState.combo;

                updateHUDScore();

                showFeedbackToast(true, gameState.currentQuestion, earnedScore);

                document.getElementById('nova-dialog').innerText = `"SYABAS! '${drone.text}' ialah Kata Sendi Nama yang tepat!"`;

                setTimeout(() => {
                    gameState.currentQuestionIndex++;
                    loadQuestionRound(gameState.currentQuestionIndex);
                }, 2200);

            } else {
                // WRONG JAWAPAN!
                soundSynth.playDeflectBuzz();

                drone.shieldHitAlpha = 1.0;
                triggerScreenShake(8);
                triggerScreenFlash('#ff2a6d');

                gameState.misses++;
                gameState.combo = 1;

                updateHUDScore();

                showFeedbackToast(false, gameState.currentQuestion, 0);

                document.getElementById('nova-dialog').innerText = `"SALAH! '${drone.text}' bukan jawapan yang betul. Sila perhati semula ayat di atas."`;
            }
        }

        function spawnExplosionDebris(x, y) {
            for (let i = 0; i < 35; i++) {
                const angle = Math.random() * Math.PI * 2;
                const speed = Math.random() * 8 + 2;
                gameState.particles.push({
                    x: x,
                    y: y,
                    vx: Math.cos(angle) * speed,
                    vy: Math.sin(angle) * speed,
                    size: Math.random() * 6 + 2,
                    color: Math.random() > 0.5 ? '#00f0ff' : '#ffb700',
                    alpha: 1.0,
                    life: 1.0,
                    decay: Math.random() * 0.03 + 0.02
                });
            }
        }

        function updateAndDrawBeamsAndParticles() {
            for (let i = gameState.beams.length - 1; i >= 0; i--) {
                const b = gameState.beams[i];
                ctx.save();
                ctx.strokeStyle = `rgba(0, 240, 255, ${b.alpha})`;
                ctx.lineWidth = b.width;
                ctx.shadowColor = '#00f0ff';
                ctx.shadowBlur = 20;
                ctx.beginPath();
                ctx.moveTo(b.x1, b.y1);
                ctx.lineTo(b.x2, b.y2);
                ctx.stroke();
                ctx.restore();

                b.alpha -= 0.08;
                b.width *= 0.85;
                if (b.alpha <= 0) {
                    gameState.beams.splice(i, 1);
                }
            }

            for (let i = gameState.particles.length - 1; i >= 0; i--) {
                const p = gameState.particles[i];
                p.x += p.vx;
                p.y += p.vy;
                p.vy += 0.1;
                p.alpha -= p.decay;

                ctx.fillStyle = p.color;
                ctx.globalAlpha = Math.max(0, p.alpha);
                ctx.beginPath();
                ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
                ctx.fill();

                if (p.alpha <= 0) {
                    gameState.particles.splice(i, 1);
                }
            }
            ctx.globalAlpha = 1.0;
        }

        function drawCyberGauntlet(w, h) {
            if (!gameState.pointer.active) return;

            const px = gameState.pointer.x * w;
            const py = gameState.pointer.y * h;

            ctx.save();

            if (gameState.handLandmarks) {
                const lm = gameState.handLandmarks;
                const connections = [
                    [0,1],[1,2],[2,3],[3,4],
                    [0,5],[5,6],[6,7],[7,8],
                    [9,10],[10,11],[11,12],
                    [13,14],[14,15],[15,16],
                    [0,17],[17,18],[18,19],[19,20],
                    [5,9],[9,13],[13,17]
                ];

                ctx.strokeStyle = '#8a2be2';
                ctx.lineWidth = 3.5;
                ctx.shadowColor = '#00f0ff';
                ctx.shadowBlur = 8;

                connections.forEach(([i, j]) => {
                    const x1 = (1 - lm[i].x) * w;
                    const y1 = lm[i].y * h;
                    const x2 = (1 - lm[j].x) * w;
                    const y2 = lm[j].y * h;

                    ctx.beginPath();
                    ctx.moveTo(x1, y1);
                    ctx.lineTo(x2, y2);
                    ctx.stroke();
                });

                lm.forEach(pt => {
                    const jx = (1 - pt.x) * w;
                    const jy = pt.y * h;
                    ctx.fillStyle = '#00f0ff';
                    ctx.beginPath();
                    ctx.arc(jx, jy, 3.5, 0, Math.PI * 2);
                    ctx.fill();
                });
            }

            const time = Date.now() * 0.008;

            ctx.save();
            ctx.translate(px, py);
            ctx.rotate(time);

            ctx.strokeStyle = '#00f0ff';
            ctx.lineWidth = 3;
            ctx.shadowColor = '#00f0ff';
            ctx.shadowBlur = 15;
            ctx.beginPath();
            ctx.arc(0, 0, 22, 0, Math.PI * 2);
            ctx.stroke();

            for (let i = 0; i < 4; i++) {
                ctx.rotate(Math.PI / 2);
                ctx.fillStyle = '#ffffff';
                ctx.fillRect(10, -2, 8, 4);
            }
            ctx.restore();

            ctx.fillStyle = gameState.pointer.isOpenPalm ? '#00f0ff' : '#ff2a6d';
            ctx.shadowColor = gameState.pointer.isOpenPalm ? '#00f0ff' : '#ff2a6d';
            ctx.shadowBlur = 18;
            ctx.beginPath();
            ctx.arc(px, py, 10, 0, Math.PI * 2);
            ctx.fill();

            ctx.restore();
        }

        function triggerScreenShake(intensity) {
            gameState.shakeIntensity = intensity;
        }

        function triggerScreenFlash(color) {
            gameState.flashOpacity = 0.35;
        }

        function drawScreenEffects(w, h) {
            if (gameState.shakeIntensity > 0) {
                const dx = (Math.random() - 0.5) * gameState.shakeIntensity;
                const dy = (Math.random() - 0.5) * gameState.shakeIntensity;
                canvas.style.transform = `translate(${dx}px, ${dy}px)`;
                gameState.shakeIntensity *= 0.85;
                if (gameState.shakeIntensity < 0.5) {
                    gameState.shakeIntensity = 0;
                    canvas.style.transform = 'none';
                }
            }

            if (gameState.flashOpacity > 0) {
                ctx.fillStyle = `rgba(0, 240, 255, ${gameState.flashOpacity})`;
                ctx.fillRect(0, 0, w, h);
                gameState.flashOpacity -= 0.03;
            }
        }

        function updateHUDScore() {
            document.getElementById('hud-score').innerText = gameState.score;
            document.getElementById('hud-combo').innerText = `×${gameState.combo}`;
        }

        function showFeedbackToast(isCorrect, qObj, scoreGain) {
            const toast = document.getElementById('feedback-toast');
            const box = document.getElementById('toast-box');
            const icon = document.getElementById('toast-icon');
            const title = document.getElementById('toast-title');
            const answer = document.getElementById('toast-answer');
            const exp = document.getElementById('toast-explanation');

            if (isCorrect) {
                box.className = 'cyber-glass px-8 py-5 rounded-3xl border-2 border-cyber-cyan text-center max-w-lg shadow-[0_0_30px_rgba(0,240,255,0.4)]';
                icon.innerText = '✅';
                title.className = 'text-xl font-black font-kanit uppercase tracking-wider text-cyber-cyan';
                title.innerText = `JAWAPAN BETUL! +${scoreGain}`;
                answer.innerText = qObj.correct.toUpperCase();
            } else {
                box.className = 'cyber-glass-red px-8 py-5 rounded-3xl border-2 border-red-500 text-center max-w-lg shadow-[0_0_30px_rgba(255,42,109,0.4)]';
                icon.innerText = '❌';
                title.className = 'text-xl font-black font-kanit uppercase tracking-wider text-red-400';
                title.innerText = 'JAWAPAN TERSASAR!';
                answer.innerText = `JAWAPAN SEBENAR: ${qObj.correct.toUpperCase()}`;
            }

            exp.innerText = qObj.explanation;

            toast.classList.remove('opacity-0', 'scale-75');
            toast.classList.add('opacity-100', 'scale-100');

            setTimeout(() => {
                toast.classList.remove('opacity-100', 'scale-100');
                toast.classList.add('opacity-0', 'scale-75');
            }, 2000);
        }

        function onMissionComplete() {
            gameState.active = false;
            soundSynth.playMissionComplete();

            const victoryModal = document.getElementById('modal-victory');
            document.getElementById('final-score').innerText = gameState.score;
            document.getElementById('final-combo').innerText = `×${gameState.maxCombo}`;
            document.getElementById('final-misses').innerText = gameState.misses;
            document.getElementById('final-mastered').innerText = `${QUESTION_BANK.length} / ${QUESTION_BANK.length}`;

            let starCount = 3;
            if (gameState.misses >= 4) starCount = 1;
            else if (gameState.misses >= 2) starCount = 2;

            renderHolographicStars(starCount);
            victoryModal.classList.remove('hidden');
        }

        function renderHolographicStars(count) {
            const container = document.getElementById('stars-container');
            container.innerHTML = '';

            for (let i = 1; i <= 3; i++) {
                const star = document.createElement('div');
                star.className = `text-5xl transition-all ${i <= count ? 'text-yellow-400 text-glow-amber star-anim' : 'text-slate-700 opacity-40'}`;
                star.style.animationDelay = `${i * 0.25}s`;
                star.innerHTML = '<i class="fa-solid fa-star"></i>';
                container.appendChild(star);
            }
        }

        function openRevisionModal() {
            const container = document.getElementById('revision-questions-container');
            container.innerHTML = '';

            QUESTION_BANK.forEach(item => {
                const qCard = document.createElement('div');
                qCard.className = 'cyber-glass p-4 rounded-2xl border border-cyan-500/30 text-left';
                
                // Highlight correct answer in question string
                const highlightedQ = item.question.replace('______', `<span class="text-yellow-300 font-bold underline px-1">${item.correct}</span>`);

                qCard.innerHTML = `
                    <div class="flex items-center justify-between gap-2 mb-2">
                        <span class="text-xs font-bold font-orbitron text-cyber-cyan">SOALAN ${item.id}</span>
                        <span class="text-[10px] px-2 py-0.5 rounded-full bg-cyan-950 text-cyan-300 border border-cyan-500/40 font-mono">${item.tag}</span>
                    </div>
                    <div class="text-sm font-medium font-kanit text-white mb-2 leading-relaxed">${highlightedQ}</div>
                    <div class="bg-slate-900/80 p-3 rounded-xl text-xs text-gray-200 font-prompt border-l-2 border-l-yellow-400">
                        <strong>Penerangan Tatabahasa:</strong> ${item.explanation}
                    </div>
                `;
                container.appendChild(qCard);
            });

            document.getElementById('modal-revision').classList.remove('hidden');
        }
    </script>
</body>
</html>
