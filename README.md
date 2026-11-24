# Mayuresh-game-```html
<!DOCTYPE html>
<html lang="mr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>मयुरेश गेम (MAYURESH GAME) - 3D बॅटल रॉयल</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+Devanagari:wght@400;600;700;900&family=Rajdhani:wght@600;700&display=swap" rel="stylesheet">
    <style>
        * {
            box-sizing: border-box;
            user-select: none;
            -webkit-user-select: none;
            font-family: 'Noto Sans Devanagari', 'Rajdhani', sans-serif;
        }
        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: #090d16;
            color: white;
        }
        #gameCanvas {
            display: block;
            width: 100vw;
            height: 100vh;
            position: fixed;
            top: 0;
            left: 0;
            z-index: 1;
        }
        .hud-layer {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            z-index: 10;
            pointer-events: none;
        }
        .interactive {
            pointer-events: auto;
        }
        /* FF Max Style Skewed Badges */
        .ff-skew {
            transform: skewX(-12deg);
        }
        .ff-unskew {
            transform: skewX(12deg);
        }
        /* Crosshair */
        #crosshair {
            position: fixed;
            top: 50%;
            left: 50%;
            width: 28px;
            height: 28px;
            transform: translate(-50%, -50%);
            pointer-events: none;
            z-index: 15;
            transition: transform 0.08s ease;
        }
        .ch-line {
            position: absolute;
            background-color: #ffffff;
            box-shadow: 0 0 3px #000;
            transition: background-color 0.1s;
        }
        .ch-top { width: 2px; height: 8px; top: 0; left: 13px; }
        .ch-bottom { width: 2px; height: 8px; bottom: 0; left: 13px; }
        .ch-left { height: 2px; width: 8px; left: 0; top: 13px; }
        .ch-right { height: 2px; width: 8px; right: 0; top: 13px; }
        .ch-dot { width: 3px; height: 3px; border-radius: 50%; top: 12.5px; left: 12.5px; background: #fbbf24; }
        
        .crosshair-enemy .ch-line {
            background-color: #ef4444 !important;
            box-shadow: 0 0 6px #ef4444;
        }
        .crosshair-head .ch-line {
            background-color: #f59e0b !important;
            box-shadow: 0 0 8px #f59e0b;
        }
        /* Hitmarker X Animation */
        #hitmarker {
            position: fixed;
            top: 50%;
            left: 50%;
            width: 32px;
            height: 32px;
            transform: translate(-50%, -50%) rotate(45deg);
            pointer-events: none;
            z-index: 16;
            opacity: 0;
            transition: opacity 0.1s ease-out;
        }
        #hitmarker::before, #hitmarker::after {
            content: '';
            position: absolute;
            background-color: #ef4444;
            box-shadow: 0 0 6px #000;
        }
        #hitmarker::before { top: 14px; left: 0; width: 32px; height: 4px; }
        #hitmarker::after { left: 14px; top: 0; width: 4px; height: 32px; }
        #hitmarker.head-hit::before, #hitmarker.head-hit::after {
            background-color: #fbbf24;
            box-shadow: 0 0 10px #f59e0b;
        }

        /* Damage Vignette */
        #damageOverlay {
            position: fixed;
            inset: 0;
            background: radial-gradient(circle, transparent 55%, rgba(220, 38, 38, 0.75) 100%);
            opacity: 0;
            pointer-events: none;
            z-index: 8;
            transition: opacity 0.15s ease-out;
        }

        /* Scope Overlay for Sniper */
        #sniperScope {
            position: fixed;
            inset: 0;
            background: radial-gradient(circle, transparent 28%, rgba(0, 0, 0, 0.96) 32%);
            display: none;
            z-index: 9;
            pointer-events: none;
        }
        #sniperScope::before {
            content: '';
            position: absolute;
            top: 50%; left: 0; right: 0; height: 1px;
            background: rgba(239, 68, 68, 0.7);
        }
        #sniperScope::after {
            content: '';
            position: absolute;
            left: 50%; top: 0; bottom: 0; width: 1px;
            background: rgba(239, 68, 68, 0.7);
        }

        /* Headshot Pop Animation */
        @keyframes popScale {
            0% { transform: translate(-50%, -50%) scale(0.5); opacity: 0; }
            30% { transform: translate(-50%, -60%) scale(1.25); opacity: 1; }
            100% { transform: translate(-50%, -95%) scale(1); opacity: 0; }
        }
        .headshot-banner {
            animation: popScale 0.95s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
        }

        /* Mobile / Touch Joystick */
        #joystickZone {
            width: 120px;
            height: 120px;
            border-radius: 50%;
            background: rgba(255, 255, 255, 0.12);
            border: 2px solid rgba(255, 255, 255, 0.3);
            position: relative;
            touch-action: none;
        }
        #joystickKnob {
            width: 46px;
            height: 46px;
            border-radius: 50%;
            background: linear-gradient(135deg, #f59e0b, #ea580c);
            position: absolute;
            top: 35px;
            left: 35px;
            box-shadow: 0 4px 12px rgba(0,0,0,0.5);
        }
    </style>
</head>
<body>

    <!-- 3D WebGL Canvas -->
    <canvas id="gameCanvas"></canvas>

    <!-- Damage Red Flash & Sniper Scope -->
    <div id="damageOverlay"></div>
    <div id="sniperScope"></div>
    <!-- Center Crosshair & Hitmarker -->
    <div id="crosshair">
        <div class="ch-line ch-top"></div>
        <div class="ch-line ch-bottom"></div>
        <div class="ch-line ch-left"></div>
        <div class="ch-line ch-right"></div>
        <div class="ch-line ch-dot"></div>
    </div>
    <div id="hitmarker"></div>

    <!-- Floating Headshot / Kill Notification Container -->
    <div id="killFeedContainer" class="fixed top-1/3 left-1/2 pointer-events-none z-20"></div>

    <!-- HUD Layer -->
    <div id="hud" class="hud-layer hidden flex flex-col justify-between p-3 sm:p-5">
        
        <!-- TOP BAR: Rank, Level, Zone Info, Alive Counter & Mini-Radar -->
        <div class="flex justify-between items-start w-full">
            
            <!-- Left: Mini Radar + Zone Indicator -->
            <div class="flex items-start gap-3">
                <div class="relative bg-slate-900/85 border-2 border-amber-500/70 rounded-xl p-1 shadow-lg">
                    <canvas id="radarCanvas" width="110" height="110" class="rounded-lg block"></canvas>
                    <div class="text-[10px] text-center font-bold text-amber-400 tracking-wider mt-0.5">रडार (RADAR)</div>
                </div>

                <div class="flex flex-col gap-1.5">
                    <!-- Official Game Title Header Badge -->
                    <div class="bg-gradient-to-r from-amber-500 via-orange-600 to-red-600 px-3 py-1 rounded-r-lg shadow-lg border-l-4 border-white flex items-center gap-2">
                        <span class="text-xs sm:text-sm font-black text-white tracking-wide drop-shadow">🔥 मयुरेश गेम</span>
                        <span class="text-[9px] font-bold bg-black/40 text-amber-200 px-1.5 py-0.5 rounded">MAYURESH GAME</span>
                    </div>

                    <!-- Current Area Banner -->
                    <div class="bg-gradient-to-r from-slate-900/90 to-slate-800/70 border-l-4 border-amber-500 px-3 py-1 rounded-r-lg shadow">
                        <div class="text-[10px] text-gray-300 uppercase">सध्याचा एरिया (Zone)</div>
                        <div id="currentZoneText" class="text-xs sm:text-sm font-extrabold text-amber-400">ओपन ग्राउंड (Open Ground)</div>
                    </div>

                    <!-- Rank & Level Pill -->
                    <div class="flex items-center gap-2 bg-slate-900/85 border border-slate-700 px-3 py-1 rounded-lg">
                        <span id="hudRankBadge" class="text-xs font-black px-2 py-0.5 rounded bg-amber-500 text-black">BRONZE I</span>
                        <span class="text-xs font-bold text-cyan-400">लेव्हल <span id="hudLevel">1</span></span>
                    </div>
                </div>
            </div>

            <!-- Right: Alive & Kills Counter (FF Max Style) -->
            <div class="flex flex-col items-end gap-2">
                <div class="flex gap-2">
                    <!-- Alive Box -->
                    <div class="ff-skew bg-slate-900/90 border border-slate-600 px-3 sm:px-4 py-1.5 rounded shadow-lg">
                        <div class="ff-unskew flex items-center gap-2">
                            <span class="text-xs text-gray-300 font-bold">जिवंत (ALIVE)</span>
                            <span id="aliveCount" class="text-lg sm:text-xl font-black text-amber-400">6</span>
                        </div>
                    </div>
                    <!-- Kills Box -->
                    <div class="ff-skew bg-gradient-to-r from-red-700/90 to-orange-600/90 border border-amber-400/50 px-3 sm:px-4 py-1.5 rounded shadow-lg">
                        <div class="ff-unskew flex items-center gap-2">
                            <span class="text-xs text-white font-bold">किल्स (KILLS)</span>
                            <span id="killCount" class="text-lg sm:text-xl font-black text-white">0</span>
                        </div>
                    </div>
                </div>

                <!-- Headshot Counter + Pause/Lock Help -->
                <div class="flex items-center gap-2">
                    <div class="bg-slate-900/80 border border-red-500/40 px-2.5 py-1 rounded text-xs font-bold text-red-400">
                        🎯 हेडशॉट्स: <span id="headshotCount" class="text-white">0</span>
                    </div>
                    <button id="lockMouseBtn" class="interactive bg-amber-500 hover:bg-amber-400 text-black font-bold text-xs px-3 py-1 rounded shadow transition">
                        🖱️ माऊस लॉक (Aim)
                    </button>
                </div>
            </div>
        </div>

        <!-- CENTER-BOTTOM: Action Prompts (Loot / Enter House) -->
        <div id="promptMessage" class="self-center bg-black/75 border border-amber-400/60 text-amber-300 px-4 py-1.5 rounded-full text-xs sm:text-sm font-bold hidden">
            मेडकिट आणि अ‍ॅमो गोळा केले! (+HP / +Ammo)
        </div>

        <!-- BOTTOM BAR: Mobile Touch Controls + Health Bar + Weapons -->
        <div class="flex flex-col gap-2 w-full">
            
            <!-- Mobile / Touch On-screen Controls Row -->
            <div class="flex justify-between items-end w-full px-1 sm:px-4 mb-1">
                <!-- Left Touch Joystick -->
                <div id="joystickZone" class="interactive sm:opacity-75 hover:opacity-100">
                    <div id="joystickKnob"></div>
                </div>

                <!-- Right Touch Action Buttons (Fire, Scope, Jump, Reload, Gloo Wall) -->
                <div class="interactive flex flex-col items-end gap-2">
                    <div class="flex gap-2">
                        <button id="btnGloo" class="bg-cyan-600/90 active:scale-95 border-2 border-cyan-300 text-white font-bold px-3 py-2 rounded-xl shadow-lg text-xs">
                            🛡️ ग्लू वॉल (G)
                        </button>
                        <button id="btnScope" class="bg-slate-800/90 active:scale-95 border-2 border-amber-400 text-amber-300 font-bold px-3 py-2 rounded-xl shadow-lg text-xs">
                            🔭 स्कोप (Right Click)
                        </button>
                    </div>
                    <div class="flex items-center gap-3">
                        <button id="btnCrouch" class="bg-slate-800/90 active:scale-95 border border-gray-400 text-white font-bold w-11 h-11 rounded-full shadow-lg text-[11px]">
                            🧎 C
                        </button>
                        <button id="btnReload" class="bg-slate-800/90 active:scale-95 border border-gray-400 text-white font-bold w-11 h-11 rounded-full shadow-lg text-xs">
                            🔄 R
                        </button>
                        <button id="btnJump" class="bg-slate-800/90 active:scale-95 border border-gray-400 text-white font-bold w-11 h-11 rounded-full shadow-lg text-xs">
                            ⬆️ Jump
                        </button>
                        <button id="btnFire" class="bg-gradient-to-br from-red-500 to-orange-600 active:scale-90 border-2 border-amber-300 text-white font-black w-16 h-16 rounded-full shadow-xl text-sm flex items-center justify-center">
                            🔥 FIRE
                        </button>
                    </div>
                </div>
            </div>

            <!-- Player Status: HP Bar & Weapon Switcher -->
            <div class="flex flex-col sm:flex-row justify-between items-center gap-3 bg-slate-950/85 border-t border-slate-700/80 px-4 py-2.5 rounded-2xl backdrop-blur-md">
                
                <!-- Health & Shield -->
                <div class="w-full sm:w-1/3 flex flex-col gap-1">
                    <div class="flex justify-between text-xs font-bold">
                        <span class="text-emerald-400">❤️ HP (आरोग्य): <span id="hpText">200</span>/200</span>
                        <span class="text-cyan-400">🛡️ कवच: <span id="shieldText">100</span></span>
                    </div>
                    <div class="w-full h-3.5 bg-slate-800 rounded-full overflow-hidden border border-slate-600 p-0.5">
                        <div id="hpBar" class="h-full bg-gradient-to-r from-emerald-500 to-lime-400 rounded-full transition-all duration-150" style="width: 100%;"></div>
                    </div>
                </div>

                <!-- Weapon Slots (1: AK47 AR, 2: AWM Sniper, 3: M1887 Shotgun) -->
                <div class="interactive flex items-center gap-2 w-full sm:w-auto justify-center">
                    <button onclick="selectWeapon(0)" id="wepSlot0" class="flex-1 sm:flex-initial bg-amber-500/20 border-2 border-amber-400 px-3 py-1.5 rounded-xl text-left transition">
                        <div class="text-[10px] text-amber-300 font-bold">[1] AR रायफल</div>
                        <div class="text-xs sm:text-sm font-black text-white">AK-47 मॅक्स</div>
                    </button>
                    <button onclick="selectWeapon(1)" id="wepSlot1" class="flex-1 sm:flex-initial bg-slate-800/80 border border-slate-600 px-3 py-1.5 rounded-xl text-left transition">
                        <div class="text-[10px] text-gray-400 font-bold">[2] स्निपर</div>
                        <div class="text-xs sm:text-sm font-black text-white">AWM स्निपर</div>
                    </button>
                    <button onclick="selectWeapon(2)" id="wepSlot2" class="flex-1 sm:flex-initial bg-slate-800/80 border border-slate-600 px-3 py-1.5 rounded-xl text-left transition">
                        <div class="text-[10px] text-gray-400 font-bold">[3] शॉटगन</div>
                        <div class="text-xs sm:text-sm font-black text-white">M1887 डबल</div>
                    </button>
                </div>

                <!-- Current Ammo Counter -->
                <div class="flex items-center gap-3">
                    <div class="text-right">
                        <div id="weaponNameDisplay" class="text-xs font-bold text-amber-400">AK-47 मॅक्स</div>
                        <div class="text-xl sm:text-2xl font-black tracking-wider">
                            <span id="magAmmo" class="text-white">30</span>
                            <span class="text-sm text-gray-400">/ <span id="reserveAmmo">150</span></span>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- START / LOBBY MODAL (FF MAX Style Lobby) -->
    <div id="lobbyModal" class="fixed inset-0 z-30 bg-gradient-to-br from-slate-950 via-slate-900 to-indigo-950 flex items-center justify-center p-4">
        <div class="max-w-3xl w-full bg-slate-900/90 border-2 border-amber-500/60 rounded-3xl p-6 sm:p-8 shadow-2xl relative overflow-hidden">
            
            <!-- Decorative Glow -->
            <div class="absolute -top-24 -right-24 w-64 h-64 bg-amber-500/20 rounded-full blur-3xl pointer-events-none"></div>
            <div class="absolute -bottom-24 -left-24 w-64 h-64 bg-orange-600/20 rounded-full blur-3xl pointer-events-none"></div>

            <!-- Header -->
            <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4 border-b border-slate-700 pb-5">
                <div class="flex items-center gap-4">
                    <!-- Prominent Red Glowing 'M G' Emblem on Start Screen -->
                    <div class="relative w-20 h-20 sm:w-24 sm:h-24 rounded-2xl bg-gradient-to-br from-red-600 via-red-700 to-red-950 border-2 border-red-400 shadow-[0_0_28px_rgba(239,68,68,0.85)] flex flex-col items-center justify-center shrink-0 transform -skew-x-6">
                        <div class="text-3xl sm:text-4xl font-black tracking-tighter text-white drop-shadow-[0_2px_10px_rgba(255,255,255,0.6)]">
                            M<span class="text-red-200">G</span>
                        </div>
                        <div class="text-[9px] font-extrabold tracking-widest text-red-200 uppercase mt-0.5 bg-black/40 px-1.5 rounded">
                            START
                        </div>
                    </div>

                    <div>
                        <div class="inline-block bg-gradient-to-r from-red-600 to-amber-500 text-white font-black text-xs px-3 py-1 rounded-full mb-1.5 tracking-wider shadow">
                            🔥 MAYURESH GAME • FF MAX 3D EDITION
                        </div>
                        <h1 class="text-3xl sm:text-5xl font-black text-white tracking-tight drop-shadow-md">
                            मयुरेश <span class="text-red-500 drop-shadow-[0_0_12px_rgba(239,68,68,0.7)]">गेम</span>
                        </h1>
                        <p class="text-gray-300 text-xs sm:text-sm mt-1">
                            हाऊस सखी, कॉलनी आणि ओपन ग्राउंडमध्ये शत्रूंना हेडशॉट मारून रँक वाढवा!
                        </p>
                    </div>
                </div>

                <!-- Player Rank Card -->
                <div class="bg-slate-800/90 border border-amber-400/50 rounded-2xl p-3.5 text-center min-w-[160px]">
                    <div class="text-[11px] text-gray-400 font-bold">तुमची सध्याची रँक (RANK)</div>
                    <div id="lobbyRankName" class="text-xl font-black text-amber-400 mt-0.5">BRONZE I</div>
                    <div class="text-xs text-cyan-300 font-bold mt-1">
                        ⭐ रँक पॉइंट्स: <span id="lobbyRankPoints">0</span> | लेव्हल: <span id="lobbyLevel">1</span>
                    </div>
                </div>
            </div>

            <!-- Map Areas & Rules Grid -->
            <div class="grid grid-cols-1 sm:grid-cols-3 gap-3 my-5">
                <div class="bg-slate-800/60 border border-slate-700 p-3.5 rounded-2xl">
                    <div class="text-amber-400 font-black text-sm mb-1">🏠 १. हाऊस सखी (House Sakhi)</div>
                    <p class="text-xs text-gray-300 leading-relaxed">
                        मुख्य मोठे घर ज्यात अनेक खोल्या (Rooms), दरवाजे आणि छत आहे. आत लपलेल्या शत्रूंना शोधा आणि खात्मा करा.
                    </p>
                </div>
                <div class="bg-slate-800/60 border border-slate-700 p-3.5 rounded-2xl">
                    <div class="text-cyan-400 font-black text-sm mb-1">🏘️ २. साईड कॉलनी (Colony)</div>
                    <p class="text-xs text-gray-300 leading-relaxed">
                        हाऊस सखीच्या बाजूला असलेली घरे आणि गल्ल्या. कव्हर घेऊन शत्रूंवर अचूक नेम धरा.
                    </p>
                </div>
                <div class="bg-slate-800/60 border border-slate-700 p-3.5 rounded-2xl">
                    <div class="text-emerald-400 font-black text-sm mb-1">🏟️ ३. ओपन ग्राउंड (Open Ground)</div>
                    <p class="text-xs text-gray-300 leading-relaxed">
                        विस्तीर्ण मैदान, कंटेनर्स, झाडे आणि लूट बॉक्स. लांबून स्निपरने वन-शॉट हेडशॉट मारा!
                    </p>
                </div>
            </div>

            <!-- Controls Guide -->
            <div class="bg-slate-950/70 border border-slate-800 rounded-2xl p-4 mb-6">
                <div class="text-xs font-bold text-amber-400 mb-2">🎮 मयुरेश गेम कंट्रोल्स (Controls):</div>
                <div class="grid grid-cols-2 sm:grid-cols-4 gap-2 text-xs text-gray-300">
                    <div><span class="text-white font-bold">W A S D / Joystick:</span> चालणे</div>
                    <div><span class="text-white font-bold">Shift / Space / C:</span> धावणे, उडी, खाली वाकणे</div>
                    <div><span class="text-white font-bold">Click / Right-Click:</span> गोळीबार व स्कोप</div>
                    <div><span class="text-white font-bold">1, 2, 3 / R / G:</span> गन, रीलोड, ग्लू वॉल</div>
                </div>
                <div class="mt-2 text-xs text-red-400 font-bold">
                    💡 टीप: शत्रूच्या डोक्यावर (Head) गोळी मारल्यास थेट <span class="text-amber-300 underline">HEADSHOT!</span> होऊन शत्रू एका फटक्यात आऊट होईल!
                </div>
            </div>

            <!-- Start Button -->
            <div class="flex flex-col sm:flex-row gap-3 justify-end items-center">
                <button id="resetProgressBtn" class="text-xs text-gray-400 hover:text-red-400 underline px-3 py-2">
                    रँक आणि लेव्हल रीसेट करा
                </button>
                <button id="startGameBtn" class="w-full sm:w-auto bg-gradient-to-r from-amber-500 via-orange-500 to-red-600 hover:from-amber-400 hover:to-red-500 text-white font-black text-lg px-10 py-3.5 rounded-2xl shadow-lg shadow-orange-600/30 transform hover:scale-105 transition cursor-pointer">
                    ⚔️ मयुरेश गेम सुरू करा (START)
                </button>
            </div>
        </div>
    </div>

    <!-- VICTORY / DEFEAT END MODAL -->
    <div id="endModal" class="fixed inset-0 z-40 bg-black/85 backdrop-blur-sm hidden flex items-center justify-center p-4">
        <div class="max-w-md w-full bg-slate-900 border-2 border-amber-500 rounded-3xl p-6 text-center shadow-2xl">
            <div class="text-xs font-black text-amber-400 tracking-widest uppercase mb-1">🔥 मयुरेश गेम (MAYURESH GAME)</div>
            <div id="endBadge" class="inline-block px-4 py-1 rounded-full text-xs font-black uppercase mb-2 bg-amber-500 text-black">
                BOOYAH! बुयाह!
            </div>
            <h2 id="endTitle" class="text-3xl font-black text-white mb-2">तुम्ही जिंकलात!</h2>
            <p id="endSubtitle" class="text-sm text-gray-300 mb-5">हाऊस सखी आणि संपूर्ण एरिया सुरक्षित झाला आहे.</p>

            <!-- Match Stats -->
            <div class="bg-slate-800/80 rounded-2xl p-4 mb-5 space-y-2 text-sm">
                <div class="flex justify-between">
                    <span class="text-gray-400">एकूण किल्स (Kills):</span>
                    <span id="endKills" class="font-black text-white">0</span>
                </div>
                <div class="flex justify-between">
                    <span class="text-gray-400">हेडशॉट्स (Headshots):</span>
                    <span id="endHeadshots" class="font-black text-red-400">0</span>
                </div>
                <div class="flex justify-between">
                    <span class="text-gray-400">मिळालेले रँक पॉइंट्स:</span>
                    <span id="endPoints" class="font-black text-emerald-400">+50 RP</span>
                </div>
                <div class="border-t border-slate-700 pt-2 flex justify-between items-center">
                    <span class="text-gray-300 font-bold">नवीन रँक (Current Rank):</span>
                    <span id="endNewRank" class="font-black text-amber-400 text-base">SILVER I</span>
                </div>
            </div>

            <button id="nextLevelBtn" class="w-full bg-gradient-to-r from-amber-500 to-orange-600 hover:from-amber-400 hover:to-orange-500 text-black font-black text-base py-3.5 rounded-2xl shadow-lg transition cursor-pointer">
                पुढील लेव्हल खेळा (NEXT LEVEL) ➡️
            </button>
        </div>
    </div>

    <script>
        // ==========================================
        // 1. AUDIO SYNTHESIZER (WEB AUDIO API)
        // ==========================================
        const AudioSys = {
            ctx: null,
            init() {
                if (!this.ctx) {
                    this.ctx = new (window.AudioContext || window.webkitAudioContext)();
                }
                if (this.ctx.state === 'suspended') {
                    this.ctx.resume();
                }
            },
            playGunshot(type = 'ar') {
                if (!this.ctx) return;
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                
                // Noise burst for realistic punch
                const bufferSize = this.ctx.sampleRate * (type === 'sniper' ? 0.35 : 0.15);
                const buffer = this.ctx.createBuffer(1, bufferSize, this.ctx.sampleRate);
                const data = buffer.getChannelData(0);
                for (let i = 0; i < bufferSize; i++) {
                    data[i] = (Math.random() * 2 - 1) * Math.exp(-i / (bufferSize * 0.25));
                }
                const noise = this.ctx.createBufferSource();
                noise.buffer = buffer;

                const filter = this.ctx.createBiquadFilter();
                filter.type = 'lowpass';
                filter.frequency.setValueAtTime(type === 'sniper' ? 1200 : (type === 'shotgun' ? 900 : 1800), now);
                filter.frequency.exponentialRampToValueAtTime(120, now + 0.15);

                gain.gain.setValueAtTime(type === 'sniper' ? 0.55 : 0.35, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + (type === 'sniper' ? 0.35 : 0.15));

                noise.connect(filter);
                filter.connect(gain);
                gain.connect(this.ctx.destination);
                noise.start(now);
            },
            playHeadshot() {
                if (!this.ctx) return;
                const now = this.ctx.currentTime;
                // Crisp metallic helmet/headshot ding
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'triangle';
                osc.frequency.setValueAtTime(1250, now);
                osc.frequency.exponentialRampToValueAtTime(2400, now + 0.08);
                gain.gain.setValueAtTime(0.5, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.28);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.29);
            },
            playHit() {
                if (!this.ctx) return;
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(420, now);
                osc.frequency.exponentialRampToValueAtTime(150, now + 0.07);
                gain.gain.setValueAtTime(0.25, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.08);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.09);
            },
            playReload() {
                if (!this.ctx) return;
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'square';
                osc.frequency.setValueAtTime(300, now);
                osc.frequency.setValueAtTime(520, now + 0.12);
                gain.gain.setValueAtTime(0.15, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.22);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.23);
            },
            playPickup() {
                if (!this.ctx) return;
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(600, now);
                osc.frequency.exponentialRampToValueAtTime(1100, now + 0.15);
                gain.gain.setValueAtTime(0.25, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.16);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.17);
            }
        };

        // ==========================================
        // 2. PROGRESSION, RANKS & SAVE SYSTEM
        // ==========================================
        const RANKS = [
            { name: 'BRONZE I', minRP: 0, color: '#cd7f32' },
            { name: 'SILVER II', minRP: 100, color: '#c0c0c0' },
            { name: 'GOLD III', minRP: 250, color: '#ffd700' },
            { name: 'PLATINUM IV', minRP: 450, color: '#00e5ff' },
            { name: 'DIAMOND', minRP: 700, color: '#b9f2ff' },
            { name: 'HEROIC (हिरोइक)', minRP: 1000, color: '#ff3d00' },
            { name: 'GRANDMASTER (ग्रँडमास्टर)', minRP: 1500, color: '#ffab00' }
        ];

        let playerData = {
            level: 1,
            rankPoints: 0,
            totalKills: 0,
            totalHeadshots: 0
        };

        function loadPlayerData() {
            try {
                const saved = localStorage.getItem('mayureshGameSave') || localStorage.getItem('ffMaxHouseSakhiSave');
                if (saved) {
                    playerData = { ...playerData, ...JSON.parse(saved) };
                }
            } catch (e) {}
            updateLobbyUI();
        }

        function savePlayerData() {
            try {
                localStorage.setItem('mayureshGameSave', JSON.stringify(playerData));
            } catch (e) {}
        }

        function getCurrentRank(rp) {
            let current = RANKS[0];
            for (const r of RANKS) {
                if (rp >= r.minRP) current = r;
            }
            return current;
        }

        function updateLobbyUI() {
            const rank = getCurrentRank(playerData.rankPoints);
            document.getElementById('lobbyRankName').textContent = rank.name;
            document.getElementById('lobbyRankName').style.color = rank.color;
            document.getElementById('lobbyRankPoints').textContent = playerData.rankPoints;
            document.getElementById('lobbyLevel').textContent = playerData.level;
        }

        // ==========================================
        // 3. WEAPONS CONFIGURATION
        // ==========================================
        const WEAPONS = [
            {
                id: 'ar',
                name: 'AK-47 मॅक्स',
                damage: 34,
                headshotMultiplier: 4.0, // Instant Kill (136 > 100 HP)
                fireRate: 140, // ms
                magSize: 30,
                mag: 30,
                reserve: 150,
                reloadTime: 1300,
                spread: 0.012,
                zoomFov: 60,
                color: 0xf59e0b
            },
            {
                id: 'sniper',
                name: 'AWM स्निपर',
                damage: 95,
                headshotMultiplier: 3.0, // Instant Kill
                fireRate: 950,
                magSize: 5,
                mag: 5,
                reserve: 25,
                reloadTime: 1800,
                spread: 0.001,
                zoomFov: 24,
                color: 0x38bdf8
            },
            {
                id: 'shotgun',
                name: 'M1887 डबल',
                damage: 85,
                headshotMultiplier: 2.5, // Instant Kill
                fireRate: 550,
                magSize: 2,
                mag: 2,
                reserve: 30,
                reloadTime: 1200,
                spread: 0.045,
                zoomFov: 65,
                color: 0xef4444
            }
        ];

        // ==========================================
        // 4. THREE.JS 3D ENGINE & MAP ARCHITECTURE
        // ==========================================
        let scene, camera, renderer;
        let colliders = []; // AABB bounding boxes for walls/obstacles
        let enemies = [];
        let bullets = [];
        let lootItems = [];
        let glooWalls = [];
        let particles = [];

        // Player State
        const player = {
            pos: new THREE.Vector3(0, 1.75, 42), // Start in Open Ground facing House Sakhi
            vel: new THREE.Vector3(0, 0, 0),
            yaw: 0,
            pitch: 0,
            hp: 200,
            maxHp: 200,
            shield: 100,
            isGrounded: true,
            isCrouching: false,
            currentWeapon: 0,
            isReloading: false,
            isScoped: false,
            lastShotTime: 0,
            glooCount: 3,
            kills: 0,
            headshots: 0
        };

        let isGameRunning = false;
        let isFiring = false;
        const keys = {};
        const joyInput = { x: 0, y: 0 };

        // Gun 3D Model attached to Camera
        let gunGroup, muzzleFlashMesh;

        function initThree() {
            const canvas = document.getElementById('gameCanvas');
            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x87ceeb);
            scene.fog = new THREE.FogExp2(0x93c5fd, 0.0065);

            camera = new THREE.PerspectiveCamera(72, window.innerWidth / window.innerHeight, 0.1, 300);
            camera.rotation.order = 'YXZ';

            renderer = new THREE.WebGLRenderer({ canvas, antialias: true });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.shadowMap.enabled = true;
            renderer.shadowMap.type = THREE.PCFSoftShadowMap;

            // Lighting
            const hemiLight = new THREE.HemisphereLight(0xffffff, 0x3f6212, 0.75);
            hemiLight.position.set(0, 50, 0);
            scene.add(hemiLight);

            const dirLight = new THREE.DirectionalLight(0xfffbeb, 0.95);
            dirLight.position.set(45, 70, 35);
            dirLight.castShadow = true;
            dirLight.shadow.mapSize.width = 2048;
            dirLight.shadow.mapSize.height = 2048;
            dirLight.shadow.camera.near = 10;
            dirLight.shadow.camera.far = 180;
            const d = 75;
            dirLight.shadow.camera.left = -d;
            dirLight.shadow.camera.right = d;
            dirLight.shadow.camera.top = d;
            dirLight.shadow.camera.bottom = -d;
            scene.add(dirLight);

            buildFirstPersonWeapon();
            scene.add(camera);

            window.addEventListener('resize', onWindowResize);
        }

        function onWindowResize() {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        }

        // Helper to create solid wall/box with collision
        function createBox(x, y, z, w, h, d, color, addCollider = true, castShadow = true) {
            const geo = new THREE.BoxGeometry(w, h, d);
            const mat = new THREE.MeshStandardMaterial({ color, roughness: 0.75, metalness: 0.1 });
            const mesh = new THREE.Mesh(geo, mat);
            mesh.position.set(x, y, z);
            mesh.castShadow = castShadow;
            mesh.receiveShadow = true;
            scene.add(mesh);

            if (addCollider) {
                colliders.push({
                    minX: x - w / 2,
                    maxX: x + w / 2,
                    minY: y - h / 2,
                    maxY: y + h / 2,
                    minZ: z - d / 2,
                    maxZ: z + d / 2
                });
            }
            return mesh;
        }

        // Create 3D Text Signboard using Canvas Texture
        function createSignboard(text, subtext, x, y, z, rotY = 0, bgColor = '#1e293b', textColor = '#fbbf24') {
            const c = document.createElement('canvas');
            c.width = 512;
            c.height = 160;
            const ctx = c.getContext('2d');
            ctx.fillStyle = bgColor;
            ctx.fillRect(0, 0, 512, 160);
            ctx.strokeStyle = textColor;
            ctx.lineWidth = 10;
            ctx.strokeRect(8, 8, 496, 144);

            ctx.fillStyle = textColor;
            ctx.font = 'bold 48px sans-serif';
            ctx.textAlign = 'center';
            ctx.fillText(text, 256, 75);

            ctx.fillStyle = '#ffffff';
            ctx.font = 'bold 26px sans-serif';
            ctx.fillText(subtext, 256, 125);

            const tex = new THREE.CanvasTexture(c);
            const mat = new THREE.MeshBasicMaterial({ map: tex });
            const mesh = new THREE.Mesh(new THREE.BoxGeometry(6.5, 2.0, 0.3), mat);
            mesh.position.set(x, y, z);
            mesh.rotation.y = rotY;
            scene.add(mesh);
        }

        // ==========================================
        // 5. BUILD MAP: HOUSE SAKHI, COLONY & OPEN GROUND
        // ==========================================
        function buildMap() {
            // Clear old objects
            while (scene.children.length > 0) {
                scene.remove(scene.children[0]);
            }
            colliders = [];
            enemies = [];
            bullets = [];
            lootItems = [];
            glooWalls = [];
            particles = [];

            // Re-add Lights & Camera
            const hemiLight = new THREE.HemisphereLight(0xffffff, 0x3f6212, 0.78);
            scene.add(hemiLight);

            const dirLight = new THREE.DirectionalLight(0xfffbeb, 0.95);
            dirLight.position.set(45, 75, 35);
            dirLight.castShadow = true;
            dirLight.shadow.mapSize.width = 2048;
            dirLight.shadow.mapSize.height = 2048;
            const d = 80;
            dirLight.shadow.camera.left = -d;
            dirLight.shadow.camera.right = d;
            dirLight.shadow.camera.top = d;
            dirLight.shadow.camera.bottom = -d;
            scene.add(dirLight);
            scene.add(camera);

            // 1. GROUND TERRAIN (160 x 160)
            const groundGeo = new THREE.PlaneGeometry(160, 160);
            const groundMat = new THREE.MeshStandardMaterial({ color: 0x4d7c0f, roughness: 0.9 });
            const ground = new THREE.Mesh(groundGeo, groundMat);
            ground.rotation.x = -Math.PI / 2;
            ground.receiveShadow = true;
            scene.add(ground);

            // Concrete Road & Courtyard around House Sakhi and Colony
            createBox(-15, 0.02, -5, 75, 0.05, 55, 0x64748b, false, false);
            // Open Ground Sandy/Battlefield Arena Patch
            createBox(28, 0.02, 22, 65, 0.05, 65, 0xa16207, false, false);

            // ====================================================
            // RED 'M G' STARTING POINT LANDMARK / ARCHWAY (Spawn at 0, 0, 42)
            // ====================================================
            buildMGStartingPoint(0, 42);

            // Outer Map Boundary Walls
            createBox(0, 4, -80, 160, 8, 2, 0x334155);
            createBox(0, 4, 80, 160, 8, 2, 0x334155);
            createBox(-80, 4, 0, 2, 8, 160, 0x334155);
            createBox(80, 4, 0, 2, 8, 160, 0x334155);

            // ====================================================
            // AREA 1: 'HOUSE SAKHI' (हाऊस सखी) - Center-Left (-22, 0, -15)
            // Multi-room enterable house with doorways, windows & rooms
            // ====================================================
            const hx = -22, hz = -15;
            const wallH = 4.6;
            const wallColor = 0xfef3c7; // Warm cream/yellow house walls
            const trimColor = 0xd97706; // Amber/Orange border

            // House Floor
            createBox(hx, 0.08, hz, 24, 0.15, 20, 0xe2e8f0, false);

            // Front Wall (z = hz + 10 = -5) with wide Main Entrance Doorway (x from -24 to -20)
            createBox(hx - 7.5, wallH / 2, hz + 10, 9, wallH, 0.6, wallColor);
            createBox(hx + 7.5, wallH / 2, hz + 10, 9, wallH, 0.6, wallColor);
            createBox(hx, wallH - 0.6, hz + 10, 6, 1.2, 0.6, trimColor); // Door header

            // Back Wall (z = hz - 10 = -25) with Back Exit Doorway
            createBox(hx - 7.5, wallH / 2, hz - 10, 9, wallH, 0.6, wallColor);
            createBox(hx + 7.5, wallH / 2, hz - 10, 9, wallH, 0.6, wallColor);
            createBox(hx, wallH - 0.6, hz - 10, 6, 1.2, 0.6, trimColor);

            // Left Wall (x = hx - 12 = -34) with Window gap
            createBox(hx - 12, wallH / 2, hz - 5.5, 0.6, wallH, 9, wallColor);
            createBox(hx - 12, wallH / 2, hz + 5.5, 0.6, wallH, 9, wallColor);
            // Spawn Loot Pickups (Medkits & Ammo Boxes)
            // ====================================================
            const lootCoords = [
                { x: hx - 8, z: hz + 4, type: 'medkit' },   // Inside House Sakhi Room 1
                { x: hx + 8, z: hz - 5, type: 'ammo' },     // Inside House Sakhi Room 2
                { x: 10, z: -22, type: 'medkit' },          // Inside Colony A
                { x: 26, z: -20, type: 'ammo' },            // Inside Colony B
                { x: 16, z: 22, type: 'medkit' },           // Open Ground
                { x: -12, z: 24, type: 'ammo' }             // Open Ground near House
            ];
            lootCoords.forEach(l => spawnLoot(l.x, l.z, l.type));
        }

        // Build Glowing Red 'M G' (MAYURESH GAME) 3D Starting Point Archway & Spawn Pad
        function buildMGStartingPoint(sx, sz) {
            const redNeonMat = new THREE.MeshStandardMaterial({
                color: 0xef4444,
                emissive: 0xdc2626,
                emissiveIntensity: 0.75,
                roughness: 0.25,
                metalness: 0.5
            });
            const darkMetalMat = new THREE.MeshStandardMaterial({
                color: 0x111827,
                roughness: 0.4,
                metalness: 0.8
            });

            // Glowing Red Spawn Pad on Ground
            const padGeo = new THREE.CylinderGeometry(4.2, 4.5, 0.14, 32);
            const padMesh = new THREE.Mesh(padGeo, redNeonMat);
            padMesh.position.set(sx, 0.07, sz);
            padMesh.receiveShadow = true;
            scene.add(padMesh);

            const innerPad = new THREE.Mesh(
                new THREE.CylinderGeometry(3.4, 3.4, 0.16, 32),
                darkMetalMat
            );
            innerPad.position.set(sx, 0.08, sz);
            scene.add(innerPad);

            // Archway Pillars (Positioned slightly ahead of player spawn so it's immediately visible!)
            const archZ = sz - 5.5;
            const leftPillar = new THREE.Mesh(new THREE.BoxGeometry(0.9, 6.2, 0.9), darkMetalMat);
            leftPillar.position.set(sx - 4.4, 3.1, archZ);
            leftPillar.castShadow = true;
            scene.add(leftPillar);

            const rightPillar = new THREE.Mesh(new THREE.BoxGeometry(0.9, 6.2, 0.9), darkMetalMat);
            rightPillar.position.set(sx + 4.4, 3.1, archZ);
            rightPillar.castShadow = true;
            scene.add(rightPillar);

            // Red Glowing Top Crossbeam
            const topBeam = new THREE.Mesh(new THREE.BoxGeometry(10.2, 0.7, 1.1), redNeonMat);
            topBeam.position.set(sx, 6.2, archZ);
            scene.add(topBeam);

            // Giant Glowing Red 'M G' + 'मयुरेश गेम' Signboard on Top of the Starting Arch
            const c = document.createElement('canvas');
            c.width = 640;
            c.height = 240;
            const ctx = c.getContext('2d');

            // Dark metallic background with bright red neon border
            ctx.fillStyle = '#090d16';
            ctx.fillRect(0, 0, 640, 240);
            ctx.strokeStyle = '#ef4444';
            ctx.lineWidth = 14;
            ctx.strokeRect(10, 10, 620, 220);

            // Bold Red 'M G' Emblem
            ctx.shadowColor = '#ef4444';
            ctx.shadowBlur = 24;
            ctx.fillStyle = '#ef4444';
            ctx.font = '900 115px sans-serif';
            ctx.textAlign = 'center';
            ctx.fillText('M  G', 320, 128);

            // Subtitle 'मयुरेश गेम - STARTING POINT'
            ctx.shadowBlur = 0;
            ctx.fillStyle = '#ffffff';
            ctx.font = 'bold 30px sans-serif';
            ctx.fillText('मयुरेश गेम • STARTING POINT', 320, 195);

            const mgTex = new THREE.CanvasTexture(c);
            const mgSignMesh = new THREE.Mesh(
                new THREE.BoxGeometry(7.6, 2.8, 0.45),
                new THREE.MeshBasicMaterial({ map: mgTex })
            );
            mgSignMesh.position.set(sx, 7.9, archZ);
            scene.add(mgSignMesh);
        }

        // ==========================================
        // 6. ENEMY AI & HITBOXES (HEADSHOT vs BODY)
        // ==========================================
        function spawnEnemiesForLevel(level) {
            // Base 6 enemies + 2 per level (capped at 14)
            const totalEnemies = Math.min(4 + level * 2, 14);
            
            // Strategic spawn spots: Inside House Sakhi rooms, Colony houses, and Open Ground
            const spawnPoints = [
                // Inside House Sakhi! (User specifically asked for enemies inside the house shooting at player)
                { x: -28, z: -20, zone: 'हाऊस सखी (आत)' },
                { x: -15, z: -18, zone: 'हाऊस सखी (आत)' },
                { x: -22, z: -10, zone: 'हाऊस सखी (हॉल)' },
                // Inside Colony
                { x: 10, z: -21, zone: 'कॉलनी घर A' },
                { x: 26, z: -19, zone: 'कॉलनी घर B' },
                { x: 18, z: -3,  zone: 'कॉलनी घर C' },
                // Open Ground
                { x: 16, z: 22,  zone: 'ओपन ग्राउंड' },
                { x: 32, z: 28,  zone: 'ओपन ग्राउंड' },
                { x: -10, z: 18, zone: 'ओपन ग्राउंड' },
                { x: 4, z: 8,    zone: 'कॉलनी रोड' },
                { x: -30, z: 8,  zone: 'हाऊस सखी बाहेर' },
                { x: 38, z: -8,  zone: 'कॉलनी पूर्व' }
            ];

            for (let i = 0; i < totalEnemies; i++) {
                const sp = spawnPoints[i % spawnPoints.length];
                const jitterX = (Math.random() - 0.5) * 2.5;
                const jitterZ = (Math.random() - 0.5) * 2.5;
                createEnemy(sp.x + jitterX, sp.z + jitterZ, sp.zone, level);
            }
        }

        function createEnemy(x, z, zoneName, level) {
            const group = new THREE.Group();
            group.position.set(x, 0, z);

            // Body Mesh (Torso + Legs)
            const bodyGeo = new THREE.BoxGeometry(0.85, 1.35, 0.55);
            const bodyMat = new THREE.MeshStandardMaterial({ color: 0xdc2626, roughness: 0.5 });
            const bodyMesh = new THREE.Mesh(bodyGeo, bodyMat);
            bodyMesh.position.y = 0.85;
            bodyMesh.castShadow = true;
            group.add(bodyMesh);

            // HEAD MESH (Distinct Hitbox for Instant Headshot!)
            const headGeo = new THREE.BoxGeometry(0.52, 0.52, 0.52);
            const headMat = new THREE.MeshStandardMaterial({ color: 0xfbbf24, roughness: 0.3 }); // Golden/Helmet head for clear visibility
            const headMesh = new THREE.Mesh(headGeo, headMat);
            headMesh.position.y = 1.82;
            headMesh.castShadow = true;
            group.add(headMesh);

            // Enemy Gun Mesh
            const gunMesh = new THREE.Mesh(
                new THREE.BoxGeometry(0.14, 0.16, 0.85),
                new THREE.MeshStandardMaterial({ color: 0x111827 })
            );
            gunMesh.position.set(0.45, 1.25, 0.35);
            group.add(gunMesh);

            // Floating HP Bar above Enemy
            const hpBg = new THREE.Mesh(
                new THREE.PlaneGeometry(1.0, 0.14),
                new THREE.MeshBasicMaterial({ color: 0x000000, side: THREE.DoubleSide })
            );
            hpBg.position.y = 2.35;
            group.add(hpBg);

            const hpFill = new THREE.Mesh(
                new THREE.PlaneGeometry(0.94, 0.1),
                new THREE.MeshBasicMaterial({ color: 0xef4444, side: THREE.DoubleSide })
            );
            hpFill.position.set(0, 2.35, 0.01);
            group.add(hpFill);

            scene.add(group);

            const enemyObj = {
                group,
                bodyMesh,
                headMesh,
                hpFill,
                hpBg,
                hp: 100 + (level - 1) * 15,
                maxHp: 100 + (level - 1) * 15,
                speed: 2.2 + Math.min(level * 0.25, 2.0),
                shootCooldown: 1.4 - Math.min(level * 0.08, 0.6),
                shootTimer: Math.random() * 1.5,
                patrolOrigin: new THREE.Vector3(x, 0, z),
                patrolAngle: Math.random() * Math.PI * 2,
                zoneName,
                alive: true
            };

            // Tag meshes so Raycaster immediately knows whether Head or Body was hit
            headMesh.userData = { enemy: enemyObj, isHead: true };
            bodyMesh.userData = { enemy: enemyObj, isHead: false };

            enemies.push(enemyObj);
        }

        function spawnLoot(x, z, type) {
            const geo = new THREE.BoxGeometry(0.7, 0.5, 0.7);
            const mat = new THREE.MeshStandardMaterial({
                color: type === 'medkit' ? 0x10b981 : 0xf59e0b,
                emissive: type === 'medkit' ? 0x065f46 : 0x78350f,
                roughness: 0.3
            });
            const mesh = new THREE.Mesh(geo, mat);
            mesh.position.set(x, 0.45, z);
            scene.add(mesh);
            lootItems.push({ mesh, type, active: true });
        }

        // ==========================================
        // 7. FIRST-PERSON WEAPON MODEL & SHOOTING
        // ==========================================
        function buildFirstPersonWeapon() {
            gunGroup = new THREE.Group();

            // Receiver
            const body = new THREE.Mesh(
                new THREE.BoxGeometry(0.11, 0.14, 0.62),
                new THREE.MeshStandardMaterial({ color: 0x1f2937, metalness: 0.7, roughness: 0.3 })
            );
            gunGroup.add(body);

            // Gold/Amber Accent Strip (FF Max Skin style)
            const skin = new THREE.Mesh(
                new THREE.BoxGeometry(0.116, 0.04, 0.45),
                new THREE.MeshStandardMaterial({ color: 0xf59e0b, metalness: 0.8, roughness: 0.2 })
            );
            skin.position.y = 0.03;
            gunGroup.add(skin);

            // Barrel
            const barrel = new THREE.Mesh(
                new THREE.CylinderGeometry(0.02, 0.02, 0.35, 8),
                new THREE.MeshStandardMaterial({ color: 0x0f172a })
            );
            barrel.rotation.x = Math.PI / 2;
            barrel.position.set(0, 0.03, -0.45);
            gunGroup.add(barrel);

            // Muzzle Flash
            muzzleFlashMesh = new THREE.Mesh(
                new THREE.SphereGeometry(0.09, 6, 6),
                new THREE.MeshBasicMaterial({ color: 0xfef08a })
            );
            muzzleFlashMesh.position.set(0, 0.03, -0.66);
            muzzleFlashMesh.visible = false;
            gunGroup.add(muzzleFlashMesh);

            gunGroup.position.set(0.28, -0.24, -0.45);
            camera.add(gunGroup);
        }

        function selectWeapon(idx) {
            if (player.isReloading) return;
            player.currentWeapon = idx;
            const wep = WEAPONS[idx];

            // Update UI slots
            for (let i = 0; i < 3; i++) {
                const el = document.getElementById(`wepSlot${i}`);
                if (i === idx) {
                    el.className = 'flex-1 sm:flex-initial bg-amber-500/25 border-2 border-amber-400 px-3 py-1.5 rounded-xl text-left transition';
                } else {
                    el.className = 'flex-1 sm:flex-initial bg-slate-800/80 border border-slate-600 px-3 py-1.5 rounded-xl text-left transition';
                }
            }

            // Reset scope if switching away from sniper
            if (wep.id !== 'sniper' && player.isScoped) {
                toggleScope(false);
            }
            updateAmmoHUD();
        }

        function toggleScope(forceState) {
            const wep = WEAPONS[player.currentWeapon];
            player.isScoped = forceState !== undefined ? forceState : !player.isScoped;

            const scopeOverlay = document.getElementById('sniperScope');
            if (player.isScoped) {
                camera.fov = wep.zoomFov;
                if (wep.id === 'sniper') {
                    scopeOverlay.style.display = 'block';
                    gunGroup.visible = false;
                } else {
                    gunGroup.position.set(0, -0.18, -0.38);
                }
            } else {
                camera.fov = 72;
                scopeOverlay.style.display = 'none';
                gunGroup.visible = true;
                gunGroup.position.set(0.28, -0.24, -0.45);
            }
            camera.updateProjectionMatrix();
        }

        function reloadWeapon() {
            const wep = WEAPONS[player.currentWeapon];
            if (player.isReloading || wep.mag === wep.magSize || wep.reserve <= 0) return;

            player.isReloading = true;
            AudioSys.playReload();
            document.getElementById('weaponNameDisplay').textContent = 'रीलोड होत आहे...';

            // Tilt gun down during reload
            gunGroup.rotation.x = -0.6;

            setTimeout(() => {
                const needed = wep.magSize - wep.mag;
                const toLoad = Math.min(needed, wep.reserve);
                wep.mag += toLoad;
                wep.reserve -= toLoad;
                player.isReloading = false;
                gunGroup.rotation.x = 0;
                updateAmmoHUD();
            }, wep.reloadTime);
        }

        function deployGlooWall() {
            if (player.glooCount <= 0) {
                showPrompt('ग्लू वॉल संपले आहेत! (0 Gloo Walls)');
                return;
            }
            player.glooCount--;
            const forward = new THREE.Vector3(0, 0, -1).applyAxisAngle(new THREE.Vector3(0, 1, 0), player.yaw);
            const gx = player.pos.x + forward.x * 3.2;
            const gz = player.pos.z + forward.z * 3.2;

            const mesh = createBox(gx, 1.3, gz, 3.6, 2.6, 0.5, 0x06b6d4, true);
            mesh.rotation.y = player.yaw;
            glooWalls.push(mesh);
            showPrompt(`🛡️ ग्लू वॉल लावली! (उरलेले: ${player.glooCount})`);
        }

        function shootWeapon() {
            if (!isGameRunning || player.isReloading) return;
            const wep = WEAPONS[player.currentWeapon];
            const now = performance.now();
            if (now - player.lastShotTime < wep.fireRate) return;

            if (wep.mag <= 0) {
                reloadWeapon();
                return;
            }

            player.lastShotTime = now;
            wep.mag--;
            updateAmmoHUD();
            AudioSys.playGunshot(wep.id);

            // Recoil & Muzzle Flash
            gunGroup.position.z += 0.08;
            player.pitch += (wep.id === 'sniper' ? 0.035 : 0.012);
            muzzleFlashMesh.visible = true;
            setTimeout(() => {
                muzzleFlashMesh.visible = false;
                if (!player.isScoped) gunGroup.position.z = -0.45;
            }, 55);

            // Raycast from center of screen
            const raycaster = new THREE.Raycaster();
            const spreadX = (Math.random() - 0.5) * (player.isScoped ? wep.spread * 0.2 : wep.spread);
            const spreadY = (Math.random() - 0.5) * (player.isScoped ? wep.spread * 0.2 : wep.spread);
            raycaster.setFromCamera(new THREE.Vector2(spreadX, spreadY), camera);

            // Collect all enemy hitboxes
            const targetMeshes = [];
            enemies.forEach(en => {
                if (en.alive) {
                    targetMeshes.push(en.headMesh, en.bodyMesh);
                }
            });

            const intersects = raycaster.intersectObjects(targetMeshes, false);

            // Bullet Tracer Visual
            const startPoint = new THREE.Vector3();
            muzzleFlashMesh.getWorldPosition(startPoint);
            let endPoint = raycaster.ray.origin.clone().add(raycaster.ray.direction.clone().multiplyScalar(85));

            if (intersects.length > 0) {
                const hit = intersects[0];
                // Check if a wall blocks the shot before reaching enemy
                const wallDist = getWallHitDistance(raycaster.ray.origin, raycaster.ray.direction, hit.distance);
                if (wallDist === null) {
                    endPoint = hit.point;
                    const { enemy, isHead } = hit.object.userData;
                    applyDamageToEnemy(enemy, wep, isHead, hit.point);
                } else {
                    endPoint = raycaster.ray.origin.clone().add(raycaster.ray.direction.clone().multiplyScalar(wallDist));
                }
            }

            createBulletTracer(startPoint, endPoint, wep.color);
        }

        function getWallHitDistance(origin, dir, maxDist) {
            // Simple ray-AABB check against colliders
            for (const c of colliders) {
                let tmin = (c.minX - origin.x) / dir.x;
                let tmax = (c.maxX - origin.x) / dir.x;
                if (tmin > tmax) [tmin, tmax] = [tmax, tmin];

                let tymin = (c.minY - origin.y) / dir.y;
                let tymax = (c.maxY - origin.y) / dir.y;
                if (tymin > tymax) [tymin, tymax] = [tymax, tymin];

                if ((tmin > tymax) || (tymin > tmax)) continue;
                if (tymin > tmin) tmin = tymin;
                if (tymax < tmax) tmax = tymax;

                let tzmin = (c.minZ - origin.z) / dir.z;
                let tzmax = (c.maxZ - origin.z) / dir.z;
                if (tzmin > tzmax) [tzmin, tzmax] = [tzmax, tzmin];

                if ((tmin > tzmax) || (tzmin > tmax)) continue;
                if (tzmin > tmin) tmin = tzmin;

                if (tmin > 0.2 && tmin < maxDist) {
                    return tmin;
                }
            }
            return null;
        }

        function createBulletTracer(start, end, colorHex) {
            const points = [start, end];
            const geo = new THREE.BufferGeometry().setFromPoints(points);
            const mat = new THREE.LineBasicMaterial({ color: colorHex, transparent: true, opacity: 0.9 });
            const line = new THREE.Line(geo, mat);
            scene.add(line);
            setTimeout(() => scene.remove(line), 65);
        }

        function applyDamageToEnemy(enemy, wep, isHead, hitPoint) {
            if (!enemy.alive) return;

            // Trigger Crosshair Hitmarker X
            const hm = document.getElementById('hitmarker');
            hm.className = isHead ? 'head-hit' : '';
            hm.style.opacity = '1';
            setTimeout(() => { hm.style.opacity = '0'; }, 140);

            // Headshot deals instant elimination damage!
            const dmg = isHead ? Math.round(wep.damage * wep.headshotMultiplier) : wep.damage;
            enemy.hp -= dmg;

            // Spawn hit particles
            for (let i = 0; i < (isHead ? 10 : 4); i++) {
                const p = new THREE.Mesh(
                    new THREE.BoxGeometry(0.12, 0.12, 0.12),
                    new THREE.MeshBasicMaterial({ color: isHead ? 0xf59e0b : 0xef4444 })
                );
                p.position.copy(hitPoint);
                scene.add(p);
                particles.push({
                    mesh: p,
                    vel: new THREE.Vector3((Math.random() - 0.5) * 5, Math.random() * 4, (Math.random() - 0.5) * 5),
                    life: 0.35
                });
            }

            if (enemy.hp <= 0) {
                enemy.alive = false;
                scene.remove(enemy.group);
                player.kills++;
                if (isHead) {
                    player.headshots++;
                    AudioSys.playHeadshot();
                    showHeadshotBanner('🎯 HEADSHOT! (+100)', true);
                } else {
                    AudioSys.playHit();
                    showHeadshotBanner('💀 शत्रू आऊट! (KILL)', false);
                }
                updateMatchHUD();
                checkMatchEnd();
            } else {
                AudioSys.playHit();
                const ratio = Math.max(0, enemy.hp / enemy.maxHp);
                enemy.hpFill.scale.x = ratio;
                showHeadshotBanner(`-${dmg}`, false);
            }
        }

        function showHeadshotBanner(text, isHeadshot) {
            const container = document.getElementById('killFeedContainer');
            const el = document.createElement('div');
            el.className = `headshot-banner font-black tracking-wide px-4 py-1 rounded-full shadow-lg whitespace-nowrap ${
                isHeadshot
                    ? 'bg-gradient-to-r from-red-600 to-amber-500 text-white text-xl sm:text-2xl border-2 border-yellow-200'
                    : 'bg-black/70 text-amber-300 text-sm'
            }`;
            el.textContent = text;
            container.appendChild(el);
            setTimeout(() => el.remove(), 950);
        }

        // ==========================================
        // 8. ENEMY AI & PLAYER COLLISION LOOP
        // ==========================================
        function updateEnemies(dt) {
            enemies.forEach(en => {
                if (!en.alive) return;

                // Make enemy HP bar always face player
                en.hpBg.lookAt(camera.position);
                en.hpFill.lookAt(camera.position);

                const toPlayer = new THREE.Vector3().subVectors(player.pos, en.group.position);
                toPlayer.y = 0;
                const dist = toPlayer.length();

                // Detection & Line of Sight check
                if (dist < 48) {
                    en.group.lookAt(player.pos.x, 0, player.pos.z);

                    const eyePos = en.group.position.clone().add(new THREE.Vector3(0, 1.5, 0));
                    const dirToPlayer = new THREE.Vector3().subVectors(player.pos, eyePos).normalize();
                    const blocked = getWallHitDistance(eyePos, dirToPlayer, dist);

                    if (blocked === null) {
                        // Player is visible! Move slightly and shoot
                        if (dist > 8) {
                            const moveStep = dirToPlayer.clone().multiplyScalar(en.speed * dt);
                            const nextX = en.group.position.x + moveStep.x;
                            const nextZ = en.group.position.z + moveStep.z;
                            if (!checkPointCollision(nextX, nextZ, 0.6)) {
                                en.group.position.x = nextX;
                                en.group.position.z = nextZ;
                            }
                        }

                        en.shootTimer -= dt;
                        if (en.shootTimer <= 0) {
                            en.shootTimer = en.shootCooldown + Math.random() * 0.6;
                            enemyShootAtPlayer(eyePos);
                        }
                    } else {
                        // Patrol inside room or zone
                        patrolEnemy(en, dt);
                    }
                } else {
                    patrolEnemy(en, dt);
                }
            });
        }

        function patrolEnemy(en, dt) {
            en.patrolAngle += dt * 0.8;
            const targetX = en.patrolOrigin.x + Math.cos(en.patrolAngle) * 2.2;
            const targetZ = en.patrolOrigin.z + Math.sin(en.patrolAngle) * 2.2;
            if (!checkPointCollision(targetX, targetZ, 0.6)) {
                en.group.position.x = targetX;
                en.group.position.z = targetZ;
                en.group.lookAt(targetX, 0, targetZ);
            }
        }

        function enemyShootAtPlayer(origin) {
            const target = player.pos.clone().add(new THREE.Vector3((Math.random() - 0.5) * 0.6, -0.3, (Math.random() - 0.5) * 0.6));
            createBulletTracer(origin, target, 0xff2222);

            // 65% hit accuracy
            if (Math.random() < 0.65) {
                const dmg = 10 + playerData.level * 2;
                damagePlayer(dmg);
            }
        }

        function damagePlayer(amount) {
            if (!isGameRunning) return;
            if (player.shield > 0) {
                const absorbed = Math.min(player.shield, amount * 0.6);
                player.shield -= Math.round(absorbed);
                player.hp -= Math.round(amount - absorbed);
            } else {
                player.hp -= amount;
            }

            player.hp = Math.max(0, player.hp);
            updateHealthHUD();

            // Red Damage Vignette Flash
            const overlay = document.getElementById('damageOverlay');
            overlay.style.opacity = '1';
            setTimeout(() => { overlay.style.opacity = '0'; }, 180);

            if (player.hp <= 0) {
                endMatch(false);
            }
        }

        function checkPointCollision(x, z, radius = 0.55) {
            for (const c of colliders) {
                if (
                    x + radius > c.minX &&
                    x - radius < c.maxX &&
                    z + radius > c.minZ &&
                    z - radius < c.maxZ &&
                    c.maxY > 0.4
                ) {
                    return true;
                }
            }
            return false;
        }

        function updatePlayerMovement(dt) {
            // Keyboard + Touch Joystick combined input
            let forward = 0;
            let strafe = 0;
            if (keys['KeyW'] || keys['ArrowUp']) forward += 1;
            if (keys['KeyS'] || keys['ArrowDown']) forward -= 1;
            if (keys['KeyA'] || keys['ArrowLeft']) strafe -= 1;
            if (keys['KeyD'] || keys['ArrowRight']) strafe += 1;

            forward -= joyInput.y;
            strafe += joyInput.x;

            const baseSpeed = player.isCrouching ? 4.2 : (keys['ShiftLeft'] ? 10.5 : 7.5);
            const sin = Math.sin(player.yaw);
            const cos = Math.cos(player.yaw);

            const moveX = (-sin * forward + cos * strafe) * baseSpeed * dt;
            const moveZ = (-cos * forward - sin * strafe) * baseSpeed * dt;

            // Slide along walls independently on X and Z
            if (!checkPointCollision(player.pos.x + moveX, player.pos.z, 0.55)) {
                player.pos.x += moveX;
            }
            if (!checkPointCollision(player.pos.x, player.pos.z + moveZ, 0.55)) {
                player.pos.z += moveZ;
            }

            // Jump, Crouch & Gravity
            const targetEyeHeight = player.isCrouching ? 1.05 : 1.75;
            if (keys['Space'] && player.isGrounded) {
                player.isCrouching = false;
                player.vel.y = 7.2;
                player.isGrounded = false;
            }
            player.vel.y -= 18.0 * dt;
            player.pos.y += player.vel.y * dt;

            if (player.pos.y <= targetEyeHeight) {
                player.pos.y = targetEyeHeight;
                player.vel.y = 0;
                player.isGrounded = true;
            }

            camera.position.copy(player.pos);
            camera.rotation.y = player.yaw;
            camera.rotation.x = player.pitch;

            // Update Current Zone Indicator (House Sakhi vs Colony vs Open Ground)
            updateCurrentZoneUI();

            // Check Loot Pickups
            lootItems.forEach(item => {
                if (!item.active) return;
                item.mesh.rotation.y += dt * 2.5;
                if (player.pos.distanceTo(item.mesh.position) < 2.2) {
                    item.active = false;
                    scene.remove(item.mesh);
                    AudioSys.playPickup();
                    if (item.type === 'medkit') {
                        player.hp = Math.min(player.maxHp, player.hp + 75);
                        player.shield = Math.min(100, player.shield + 50);
                        updateHealthHUD();
                        showPrompt('💊 मेडकिट मिळाली! (+75 HP & +50 Shield)');
                    } else {
                        WEAPONS.forEach(w => { w.reserve += w.magSize * 2; });
                        player.glooCount++;
                        updateAmmoHUD();
                        showPrompt('📦 अ‍ॅमो आणि ग्लू वॉल मिळाली! (+Ammo / +1 Gloo)');
                    }
                }
            });
        }

        function updateCurrentZoneUI() {
            const x = player.pos.x;
            const z = player.pos.z;
            const zoneEl = document.getElementById('currentZoneText');

            if (x >= -34 && x <= -10 && z >= -25 && z <= -5) {
                zoneEl.textContent = '🏠 हाऊस सखी (House Sakhi - आत)';
                zoneEl.className = 'text-sm sm:text-base font-extrabold text-amber-400';
            } else if (x >= 2 && x <= 35 && z >= -30 && z <= 3) {
                zoneEl.textContent = '🏘️ साईड कॉलनी (Side Colony)';
                zoneEl.className = 'text-sm sm:text-base font-extrabold text-cyan-400';
            } else {
                zoneEl.textContent = '🏟️ ओपन ग्राउंड (Open Ground)';
                zoneEl.className = 'text-sm sm:text-base font-extrabold text-emerald-400';
            }
        }

        // Crosshair Hover Highlight (Red on Enemy, Gold on Head!)
        function updateCrosshairHover() {
            const raycaster = new THREE.Raycaster();
            raycaster.setFromCamera(new THREE.Vector2(0, 0), camera);
            const targetMeshes = [];
            enemies.forEach(en => {
                if (en.alive) targetMeshes.push(en.headMesh, en.bodyMesh);
            });

            const ch = document.getElementById('crosshair');
            const hits = raycaster.intersectObjects(targetMeshes, false);
            if (hits.length > 0) {
                ch.className = hits[0].object.userData.isHead ? 'crosshair-head' : 'crosshair-enemy';
            } else {
                ch.className = '';
            }
        }

        // ==========================================
        // 9. MINI RADAR & HUD UPDATES
        // ==========================================
        function drawMiniRadar() {
            const canvas = document.getElementById('radarCanvas');
            const ctx = canvas.getContext('2d');
            const w = canvas.width, h = canvas.height;
            const cx = w / 2, cy = h / 2;

            ctx.clearRect(0, 0, w, h);
            ctx.fillStyle = '#0f172a';
            ctx.fillRect(0, 0, w, h);

            ctx.save();
            ctx.translate(cx, cy);
            ctx.rotate(player.yaw);

            const scale = 0.85;
            // Draw House Sakhi on Radar
            ctx.fillStyle = 'rgba(245, 158, 11, 0.35)';
            ctx.fillRect((-22 - player.pos.x - 12) * scale, (-15 - player.pos.z - 10) * scale, 24 * scale, 20 * scale);

            // Draw Colony on Radar
            ctx.fillStyle = 'rgba(56, 189, 248, 0.3)';
            ctx.fillRect((18 - player.pos.x - 14) * scale, (-14 - player.pos.z - 12) * scale, 28 * scale, 24 * scale);

            // Draw Enemies as Red Dots
            enemies.forEach(en => {
                if (!en.alive) return;
                const dx = (en.group.position.x - player.pos.x) * scale;
                const dz = (en.group.position.z - player.pos.z) * scale;
                ctx.fillStyle = '#ef4444';
                ctx.beginPath();
                ctx.arc(dx, dz, 3.5, 0, Math.PI * 2);
                ctx.fill();
            });

            ctx.restore();

            // Player Arrow at Center
            ctx.fillStyle = '#10b981';
            ctx.beginPath();
            ctx.moveTo(cx, cy - 6);
            ctx.lineTo(cx - 4.5, cy + 5);
            ctx.lineTo(cx + 4.5, cy + 5);
            ctx.closePath();
            ctx.fill();
        }

        function updateHealthHUD() {
            document.getElementById('hpText').textContent = player.hp;
            document.getElementById('shieldText').textContent = player.shield;
            const pct = (player.hp / player.maxHp) * 100;
            document.getElementById('hpBar').style.width = `${pct}%`;
        }

        function updateAmmoHUD() {
            const wep = WEAPONS[player.currentWeapon];
            document.getElementById('weaponNameDisplay').textContent = wep.name;
            document.getElementById('magAmmo').textContent = wep.mag;
            document.getElementById('reserveAmmo').textContent = wep.reserve;
        }

        function updateMatchHUD() {
            const alive = enemies.filter(e => e.alive).length;
            document.getElementById('aliveCount').textContent = alive;
            document.getElementById('killCount').textContent = player.kills;
            document.getElementById('headshotCount').textContent = player.headshots;

            const rank = getCurrentRank(playerData.rankPoints);
            document.getElementById('hudRankBadge').textContent = rank.name;
            document.getElementById('hudLevel').textContent = playerData.level;
        }

        let promptTimeout;
        function showPrompt(msg) {
            const el = document.getElementById('promptMessage');
            el.textContent = msg;
            el.classList.remove('hidden');
            clearTimeout(promptTimeout);
            promptTimeout = setTimeout(() => el.classList.add('hidden'), 2400);
        }

        // ==========================================
        // 10. MATCH FLOW (START, WIN, LOSE)
        // ==========================================
        function startMatch() {
            AudioSys.init();
            document.getElementById('lobbyModal').classList.add('hidden');
            document.getElementById('endModal').classList.add('hidden');
            document.getElementById('hud').classList.remove('hidden');

            // Reset Player
            player.pos.set(0, 1.75, 42);
            player.yaw = 0.25; // Facing towards House Sakhi & Open Ground
            player.pitch = 0;
            player.hp = 200;
            player.shield = 100;
            player.kills = 0;
            player.headshots = 0;
            player.glooCount = 3;
            player.isReloading = false;
            toggleScope(false);

            // Reset Weapons Ammo
            WEAPONS[0].mag = 30; WEAPONS[0].reserve = 150;
            WEAPONS[1].mag = 5;  WEAPONS[1].reserve = 25;
            WEAPONS[2].mag = 2;  WEAPONS[2].reserve = 30;
            selectWeapon(0);

            buildMap();
            updateHealthHUD();
            updateAmmoHUD();
            updateMatchHUD();

            isGameRunning = true;
            requestPointerLockSafe();
        }

        function checkMatchEnd() {
            const alive = enemies.filter(e => e.alive).length;
            if (alive === 0) {
                setTimeout(() => endMatch(true), 600);
            }
        }

        function endMatch(victory) {
            isGameRunning = false;
            isFiring = false;
            toggleScope(false);
            if (document.pointerLockElement) {
                document.exitPointerLock();
            }

            const earnedRP = victory
                ? (50 + player.kills * 10 + player.headshots * 15)
                : (player.kills * 5 + player.headshots * 10);

            playerData.rankPoints += earnedRP;
            playerData.totalKills += player.kills;
            playerData.totalHeadshots += player.headshots;
            if (victory) {
                playerData.level++;
            }
            savePlayerData();

            const newRank = getCurrentRank(playerData.rankPoints);

            document.getElementById('endBadge').textContent = victory ? '🏆 BOOYAH! बुयाह!' : '💀 ELIMINATED (पराभव)';
            document.getElementById('endBadge').className = victory
                ? 'inline-block px-4 py-1 rounded-full text-xs font-black uppercase mb-2 bg-amber-500 text-black'
                : 'inline-block px-4 py-1 rounded-full text-xs font-black uppercase mb-2 bg-red-600 text-white';

            document.getElementById('endTitle').textContent = victory ? 'मयुरेश गेममध्ये तुम्ही जिंकलात!' : 'तुम्ही आऊट झालात!';
            document.getElementById('endSubtitle').textContent = victory
                ? `अभिनंदन! तुम्ही मयुरेश गेमच्या लेव्हल ${playerData.level} वर पोहोचला आहात.`
                : 'पुन्हा प्रयत्न करा आणि शत्रूंना हेडशॉट मारा!';

            document.getElementById('endKills').textContent = player.kills;
            document.getElementById('endHeadshots').textContent = player.headshots;
            document.getElementById('endPoints').textContent = `+${earnedRP} RP`;
            document.getElementById('endNewRank').textContent = newRank.name;
            document.getElementById('nextLevelBtn').textContent = victory
                ? `पुढील लेव्हल (LEVEL ${playerData.level}) खेळा ➡️`
                : 'पुन्हा खेळा (RETRY) 🔄';

            document.getElementById('hud').classList.add('hidden');
            document.getElementById('endModal').classList.remove('hidden');
        }

        // ==========================================
        // 11. INPUT HANDLERS (MOUSE, KEYBOARD & TOUCH)
        // ==========================================
        function requestPointerLockSafe() {
            const canvas = document.getElementById('gameCanvas');
            if (canvas.requestPointerLock) {
                try {
                    canvas.requestPointerLock();
                } catch (e) {}
            }
        }

        function setupControls() {
            window.addEventListener('keydown', e => {
                keys[e.code] = true;
                if (!isGameRunning) return;
                if (e.code === 'Digit1') selectWeapon(0);
                if (e.code === 'Digit2') selectWeapon(1);
                if (e.code === 'Digit3') selectWeapon(2);
                if (e.code === 'KeyR') reloadWeapon();
                if (e.code === 'KeyG') deployGlooWall();
                if (e.code === 'KeyC') {
                    player.isCrouching = !player.isCrouching;
                }
            });

            window.addEventListener('keyup', e => {
                keys[e.code] = false;
            });

            // Mouse Aiming (Works both with PointerLock AND click-drag fallback!)
            let isDraggingLook = false;
            let lastMouseX = 0, lastMouseY = 0;

            document.addEventListener('mousemove', e => {
                if (!isGameRunning) return;
                const sensitivity = player.isScoped ? 0.0011 : 0.0024;
                if (document.pointerLockElement === document.getElementById('gameCanvas')) {
                    player.yaw -= e.movementX * sensitivity;
                    player.pitch -= e.movementY * sensitivity;
                    player.pitch = Math.max(-Math.PI / 2.2, Math.min(Math.PI / 2.2, player.pitch));
                } else if (isDraggingLook) {
                    const dx = e.clientX - lastMouseX;
                    const dy = e.clientY - lastMouseY;
                    lastMouseX = e.clientX;
                    lastMouseY = e.clientY;
                    player.yaw -= dx * sensitivity * 1.4;
                    player.pitch -= dy * sensitivity * 1.4;
                    player.pitch = Math.max(-Math.PI / 2.2, Math.min(Math.PI / 2.2, player.pitch));
                }
            });

            const canvas = document.getElementById('gameCanvas');
            canvas.addEventListener('mousedown', e => {
                if (!isGameRunning) return;
                if (e.button === 0) {
                    isFiring = true;
                    isDraggingLook = true;
                    lastMouseX = e.clientX;
                    lastMouseY = e.clientY;
                    shootWeapon();
                } else if (e.button === 2) {
                    toggleScope();
                }
            });

            window.addEventListener('mouseup', () => {
                isFiring = false;
                isDraggingLook = false;
            });

            canvas.addEventListener('contextmenu', e => e.preventDefault());

            // Touch Look on Canvas (Swipe anywhere on screen to aim)
            let touchLookId = null;
            let lastTouchX = 0, lastTouchY = 0;
            canvas.addEventListener('touchstart', e => {
                if (!isGameRunning) return;
                for (const t of e.changedTouches) {
                    if (touchLookId === null) {
                        touchLookId = t.identifier;
                        lastTouchX = t.clientX;
                        lastTouchY = t.clientY;
                    }
                }
            }, { passive: true });

            canvas.addEventListener('touchmove', e => {
                if (!isGameRunning) return;
                for (const t of e.changedTouches) {
                    if (t.identifier === touchLookId) {
                        const dx = t.clientX - lastTouchX;
                        const dy = t.clientY - lastTouchY;
                        lastTouchX = t.clientX;
                        lastTouchY = t.clientY;
                        const sens = player.isScoped ? 0.002 : 0.0045;
                        player.yaw -= dx * sens;
                        player.pitch -= dy * sens;
                        player.pitch = Math.max(-Math.PI / 2.2, Math.min(Math.PI / 2.2, player.pitch));
                    }
                }
            }, { passive: true });

            canvas.addEventListener('touchend', e => {
                for (const t of e.changedTouches) {
                    if (t.identifier === touchLookId) touchLookId = null;
                }
            });

            // Virtual Touch Joystick
            const joyZone = document.getElementById('joystickZone');
            const joyKnob = document.getElementById('joystickKnob');
            let joyTouchId = null;

            function handleJoyMove(clientX, clientY) {
                const rect = joyZone.getBoundingClientRect();
                const centerX = rect.left + rect.width / 2;
                const centerY = rect.top + rect.height / 2;
                let dx = clientX - centerX;
                let dy = clientY - centerY;
                const maxDist = 38;
                const dist = Math.hypot(dx, dy);
                if (dist > maxDist) {
                    dx = (dx / dist) * maxDist;
                    dy = (dy / dist) * maxDist;
                }
                joyKnob.style.transform = `translate(${dx}px, ${dy}px)`;
                joyInput.x = dx / maxDist;
                joyInput.y = dy / maxDist;
            }

            joyZone.addEventListener('touchstart', e => {
                e.preventDefault();
                const t = e.changedTouches[0];
                joyTouchId = t.identifier;
                handleJoyMove(t.clientX, t.clientY);
            }, { passive: false });

            joyZone.addEventListener('touchmove', e => {
                e.preventDefault();
                for (const t of e.changedTouches) {
                    if (t.identifier === joyTouchId) handleJoyMove(t.clientX, t.clientY);
                }
            }, { passive: false });

            joyZone.addEventListener('touchend', e => {
                for (const t of e.changedTouches) {
                    if (t.identifier === joyTouchId) {
                        joyTouchId = null;
                        joyKnob.style.transform = 'translate(0px, 0px)';
                        joyInput.x = 0;
                        joyInput.y = 0;
                    }
                }
            });

            // On-Screen Buttons
            const btnFire = document.getElementById('btnFire');
            btnFire.addEventListener('mousedown', () => { isFiring = true; shootWeapon(); });
            btnFire.addEventListener('touchstart', e => { e.preventDefault(); isFiring = true; shootWeapon(); });
            btnFire.addEventListener('touchend', () => { isFiring = false; });

            document.getElementById('btnScope').addEventListener('click', () => toggleScope());
            document.getElementById('btnReload').addEventListener('click', () => reloadWeapon());
            document.getElementById('btnGloo').addEventListener('click', () => deployGlooWall());
            document.getElementById('btnCrouch').addEventListener('click', () => {
                player.isCrouching = !player.isCrouching;
            });
            document.getElementById('btnJump').addEventListener('click', () => {
                if (player.isGrounded) {
                    player.isCrouching = false;
                    player.vel.y = 7.2;
                    player.isGrounded = false;
                }
            });

            document.getElementById('lockMouseBtn').addEventListener('click', requestPointerLockSafe);
            document.getElementById('startGameBtn').addEventListener('click', startMatch);
            document.getElementById('nextLevelBtn').addEventListener('click', startMatch);
            document.getElementById('resetProgressBtn').addEventListener('click', () => {
                playerData = { level: 1, rankPoints: 0, totalKills: 0, totalHeadshots: 0 };
                savePlayerData();
                updateLobbyUI();
            });
        }

        // ==========================================
        // 12. MAIN ANIMATION LOOP
        // ==========================================
        let prevTime = performance.now();

        function animate() {
            requestAnimationFrame(animate);

            const now = performance.now();
            const dt = Math.min((now - prevTime) / 1000, 0.08);
            prevTime = now;

            if (isGameRunning) {
                updatePlayerMovement(dt);
                updateEnemies(dt);
                updateCrosshairHover();
                drawMiniRadar();

                if (isFiring && WEAPONS[player.currentWeapon].id === 'ar') {
                    shootWeapon();
                }

                // Update Hit Particles
                for (let i = particles.length - 1; i >= 0; i--) {
                    const p = particles[i];
                    p.life -= dt;
                    p.mesh.position.addScaledVector(p.vel, dt);
                    if (p.life <= 0) {
                        scene.remove(p.mesh);
                        particles.splice(i, 1);
                    }
                }
            }

            renderer.render(scene, camera);
        }

        window.onload = function () {
            loadPlayerData();
            initThree();
            setupControls();
            animate();
        };
    </script>
</body>
</html>
```
