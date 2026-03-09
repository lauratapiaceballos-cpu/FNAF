<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Five Nights at Freddy's - Supervivencia Web</title>
    <style>
        /* ESTILOS GLOBALES Y ATMÓSFERA */
        :root {
            --office-bg: #1a1a1a;
            --door-bg: #333;
            --panel-bg: #444;
            --power-color: #00ff00;
            --red-btn: #cc0000;
            --white-btn: #dddddd;
            --crt-scanline: rgba(0, 0, 0, 0.3);
        }

        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            background-color: #000;
            color: #fff;
            font-family: 'Courier New', Courier, monospace;
            overflow: hidden;
            user-select: none;
        }

        /* EFECTO VHS / CRT */
        .crt-overlay {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: linear-gradient(rgba(18, 16, 16, 0) 50%, rgba(0, 0, 0, 0.25) 50%), linear-gradient(90deg, rgba(255, 0, 0, 0.06), rgba(0, 255, 0, 0.02), rgba(0, 0, 255, 0.06));
            background-size: 100% 4px, 6px 100%;
            z-index: 100;
            pointer-events: none;
            opacity: 0.8;
        }
        
        /* PANTALLA DE INICIO */
        #main-menu {
            position: absolute;
            width: 100%; height: 100%;
            background: radial-gradient(circle at center, #222 0%, #000 70%);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 200;
        }

        #main-menu h1 {
            font-size: 4rem;
            text-align: center;
            text-shadow: 0 0 10px #fff;
            margin-bottom: 50px;
        }

        .btn-start {
            background: transparent;
            color: #fff;
            border: 2px solid #fff;
            padding: 15px 40px;
            font-size: 1.5rem;
            font-family: 'Courier New', Courier, monospace;
            cursor: pointer;
            transition: all 0.3s;
        }

        .btn-start:hover {
            background: #fff;
            color: #000;
        }

        /* JUEGO PRINCIPAL */
        #game-container {
            position: absolute;
            width: 100%; height: 100%;
            display: none;
        }

        /* INTERFAZ DE USUARIO */
        #ui-layer {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            z-index: 50;
            pointer-events: none;
        }

        #time-display {
            position: absolute;
            top: 20px; right: 40px;
            font-size: 2.5rem;
            font-weight: bold;
        }

        #night-display {
            position: absolute;
            top: 60px; right: 40px;
            font-size: 1.5rem;
        }

        #power-display {
            position: absolute;
            bottom: 20px; left: 40px;
            font-size: 1.5rem;
        }

        #usage-display {
            position: absolute;
            bottom: 20px; left: 250px;
            font-size: 1.5rem;
            display: flex;
            align-items: center;
        }
        
        .usage-bar {
            width: 15px; height: 25px;
            background: #333;
            margin-left: 5px;
            display: inline-block;
        }
        .usage-bar.active-1 { background: #00ff00; }
        .usage-bar.active-2 { background: #ffff00; }
        .usage-bar.active-3 { background: #ffaa00; }
        .usage-bar.active-4 { background: #ff0000; }

        #camera-toggle {
            position: absolute;
            bottom: 20px; right: 20px;
            width: 400px; height: 50px;
            background: rgba(255, 255, 255, 0.2);
            border: 2px solid #fff;
            color: #fff;
            font-size: 1.5rem;
            text-align: center;
            line-height: 50px;
            cursor: pointer;
            pointer-events: auto;
            transition: 0.2s;
        }
        #camera-toggle:hover { background: rgba(255, 255, 255, 0.4); }

        /* LA OFICINA */
        #office {
            position: absolute;
            width: 100%; height: 100%;
            background: radial-gradient(circle at center, #2a2a2a 0%, #050505 100%);
            perspective: 1000px;
            display: flex;
            justify-content: space-between;
        }

        /* PAREDES Y PUERTAS */
        .hallway {
            width: 25%;
            height: 100%;
            background: #0a0a0a;
            position: relative;
            border-left: 20px solid #111;
            border-right: 20px solid #111;
            box-sizing: border-box;
            box-shadow: inset 0 0 100px #000;
            overflow: hidden;
            transition: background 0.1s;
        }

        .door {
            position: absolute;
            top: -100%; /* Abierta por defecto */
            left: 0;
            width: 100%;
            height: 100%;
            background: repeating-linear-gradient(0deg, #333, #333 10px, #222 10px, #222 20px);
            border: 5px solid #111;
            box-sizing: border-box;
            transition: top 0.3s ease-in-out;
            z-index: 5;
        }

        .door.closed { top: 0; }

        /* BOTONES DE PUERTA */
        .door-controls {
            position: absolute;
            top: 40%;
            width: 60px;
            height: 140px;
            background: #222;
            border: 2px solid #111;
            display: flex;
            flex-direction: column;
            justify-content: space-around;
            align-items: center;
            z-index: 10;
        }
        #left-controls { right: -80px; }
        #right-controls { left: -80px; }

        .ctrl-btn {
            width: 40px; height: 40px;
            border-radius: 50%;
            cursor: pointer;
            border: 2px solid #000;
            box-shadow: inset 0 0 10px rgba(0,0,0,0.8);
            pointer-events: auto;
        }
        .btn-door { background: var(--red-btn); }
        .btn-door.active { background: #ff5555; box-shadow: 0 0 15px #ff0000; }
        .btn-light { background: var(--white-btn); }
        .btn-light.active { background: #ffffff; box-shadow: 0 0 15px #ffffff; }

        /* ESCRITORIO Y CENTRO */
        .office-center {
            width: 50%;
            height: 100%;
            position: relative;
            display: flex;
            flex-direction: column;
            justify-content: flex-end;
            align-items: center;
            z-index: 2;
        }

        .desk {
            width: 90%;
            height: 30%;
            background: linear-gradient(to bottom, #444, #111);
            border-top: 10px solid #222;
            border-radius: 10px 10px 0 0;
            position: relative;
        }

        .fan {
            position: absolute;
            top: -80px;
            left: 50%;
            transform: translateX(-50%);
            width: 60px; height: 60px;
            border: 5px solid #333;
            border-radius: 50%;
            background: #111;
        }
        .fan-blades {
            width: 100%; height: 100%;
            background: conic-gradient(from 0deg, transparent 0deg 40deg, #555 40deg 120deg, transparent 120deg 160deg, #555 160deg 240deg, transparent 240deg 280deg, #555 280deg 360deg);
            border-radius: 50%;
            animation: spin 0.2s linear infinite;
        }
        @keyframes spin { 100% { transform: rotate(360deg); } }

        /* MONSTRUOS EN EL PASILLO (SILUETAS) */
        .hallway-monster {
            position: absolute;
            bottom: 10%;
            left: 50%;
            transform: translateX(-50%);
            width: 150px;
            height: 300px;
            opacity: 0;
            transition: opacity 0.1s;
            pointer-events: none;
            z-index: 1;
        }

        /* SISTEMA DE CÁMARAS */
        #camera-system {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: #000;
            display: none;
            z-index: 40;
            pointer-events: auto;
        }

        #camera-feed {
            width: 100%; height: 100%;
            object-fit: cover;
        }

        #camera-static {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            opacity: 0.3;
            pointer-events: none;
        }

        .cam-name {
            position: absolute;
            top: 30px; left: 40px;
            font-size: 2rem;
            border: 2px solid #fff;
            padding: 5px 15px;
            background: rgba(0,0,0,0.5);
        }

        .rec-dot {
            position: absolute;
            top: 30px; left: 300px;
            width: 20px; height: 20px;
            background: red;
            border-radius: 50%;
            animation: blink 1s infinite;
        }
        @keyframes blink { 0%, 49% { opacity: 1; } 50%, 100% { opacity: 0; } }

        /* MAPA DE CÁMARAS */
        #map-container {
            position: absolute;
            bottom: 80px; right: 40px;
            width: 350px; height: 350px;
            border: 2px solid #fff;
            background: rgba(0,0,0,0.7);
        }

        .map-room {
            position: absolute;
            border: 1px solid #fff;
            background: transparent;
        }
        
        .cam-btn {
            position: absolute;
            background: #222;
            color: #fff;
            border: 1px solid #fff;
            padding: 2px 5px;
            font-size: 0.8rem;
            cursor: pointer;
        }
        .cam-btn.active { background: #00aa00; }

        /* PANTALLAS DE FIN DE JUEGO / VICTORIA */
        .overlay-screen {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: #000;
            display: none;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 300;
        }
        .overlay-screen h1 { font-size: 5rem; color: #fff; }
        
        /* JUMPSCARE */
        #jumpscare-container {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: #000;
            display: none;
            z-index: 250;
            overflow: hidden;
        }
        #jumpscare-canvas {
            width: 100%; height: 100%;
            animation: shake 0.05s infinite;
        }
        @keyframes shake {
            0% { transform: translate(10px, 10px) scale(1.1); }
            25% { transform: translate(-10px, -10px) scale(1.1); }
            50% { transform: translate(-10px, 10px) scale(1.2); }
            75% { transform: translate(10px, -10px) scale(1.1); }
            100% { transform: translate(0px, 0px) scale(1.2); }
        }

    </style>
</head>
<body>

    <!-- Filtro CRT general -->
    <div class="crt-overlay"></div>

    <!-- PANTALLA DE INICIO -->
    <div id="main-menu">
        <h1>Five Nights at Freddy's</h1>
        <p>Sobrevive desde las 12 AM hasta las 6 AM.</p>
        <p>Vigila las cámaras, cierra las puertas sólo si es necesario. La energía es limitada.</p>
        <button class="btn-start" onclick="startGame()">Nueva Partida</button>
    </div>

    <!-- JUEGO PRINCIPAL -->
    <div id="game-container">
        <!-- UI -->
        <div id="ui-layer">
            <div id="time-display">12 AM</div>
            <div id="night-display">Noche 1</div>
            <div id="power-display">Energía: 100%</div>
            <div id="usage-display">
                Uso: 
                <div class="usage-bar active-1"></div>
                <div class="usage-bar" id="u2"></div>
                <div class="usage-bar" id="u3"></div>
                <div class="usage-bar" id="u4"></div>
            </div>
            <div id="camera-toggle" onclick="toggleCamera()">ABRIR MONITOR</div>
        </div>

        <!-- OFICINA -->
        <div id="office">
            <!-- Pasillo Izquierdo (Bonnie/Foxy) -->
            <div class="hallway" id="hall-left">
                <canvas id="monster-left" class="hallway-monster"></canvas>
                <div class="door" id="door-left"></div>
                <div class="door-controls" id="left-controls">
                    <div class="ctrl-btn btn-door" onclick="toggleDoor('left')"></div>
                    <div class="ctrl-btn btn-light" onclick="toggleLight('left')"></div>
                </div>
            </div>

            <!-- Centro -->
            <div class="office-center">
                <div class="desk">
                    <div class="fan"><div class="fan-blades"></div></div>
                </div>
            </div>

            <!-- Pasillo Derecho (Chica/Freddy) -->
            <div class="hallway" id="hall-right">
                <canvas id="monster-right" class="hallway-monster"></canvas>
                <div class="door" id="door-right"></div>
                <div class="door-controls" id="right-controls">
                    <div class="ctrl-btn btn-door" onclick="toggleDoor('right')"></div>
                    <div class="ctrl-btn btn-light" onclick="toggleLight('right')"></div>
                </div>
            </div>
        </div>

        <!-- SISTEMA DE CÁMARAS -->
        <div id="camera-system">
            <canvas id="camera-feed"></canvas>
            <canvas id="camera-static"></canvas>
            <div class="cam-name" id="cam-name-display">CAM 1A - Show Stage</div>
            <div class="rec-dot"></div>
            
            <div id="map-container">
                <!-- Estructura del mapa generada dinámicamente -->
            </div>
        </div>

        <!-- JUMPSCARE -->
        <div id="jumpscare-container">
            <canvas id="jumpscare-canvas"></canvas>
        </div>

        <!-- PANTALLAS DE FIN -->
        <div id="game-over" class="overlay-screen">
            <canvas id="static-end" style="position:absolute; top:0; left:0; width:100%; height:100%; opacity:0.3;"></canvas>
            <h1 style="z-index:10; font-family:serif; color:red;">GAME OVER</h1>
            <button class="btn-start" style="z-index:10; margin-top:30px;" onclick="location.reload()">Intentar de nuevo</button>
        </div>

        <div id="win-screen" class="overlay-screen">
            <h1>5:59 AM</h1>
            <h1 id="win-text" style="display:none; color:yellow; margin-top:20px;">6:00 AM</h1>
            <p style="color:#aaa; margin-top:20px;">¡Has sobrevivido la noche!</p>
            <button class="btn-start" style="margin-top:30px;" onclick="location.reload()">Jugar Noche 2 (Recargar)</button>
        </div>
    </div>

    <!-- CÓDIGO DEL JUEGO -->
    <script>
        // --- ESTADO DEL JUEGO ---
        let gameActive = false;
        let power = 100;
        let timeHour = 0; // 0 = 12AM, 1 = 1AM... 6 = 6AM
        let usage = 1;
        let powerDraining;
        let timeTicking;
        let aiTicking;
        
        let state = {
            doorLeft: false,
            doorRight: false,
            lightLeft: false,
            lightRight: false,
            cameraUp: false,
            currentCam: '1A',
            powerOut: false
        };

        // --- SISTEMA DE AUDIO PROCEDIMENTAL (Web Audio API) ---
        // Evita depender de archivos externos y genera sonidos retro/de terror
        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        let humOscillator = null;

        function playSound(type) {
            if (!audioCtx) return;
            if (audioCtx.state === 'suspended') audioCtx.resume();
            
            const osc = audioCtx.createOscillator();
            const gainNode = audioCtx.createGain();
            osc.connect(gainNode);
            gainNode.connect(audioCtx.destination);

            const now = audioCtx.currentTime;

            if (type === 'blip') {
                // Sonido de botón de UI
                osc.type = 'sine';
                osc.frequency.setValueAtTime(600, now);
                osc.frequency.exponentialRampToValueAtTime(800, now + 0.1);
                gainNode.gain.setValueAtTime(0.3, now);
                gainNode.gain.exponentialRampToValueAtTime(0.01, now + 0.1);
                osc.start(now);
                osc.stop(now + 0.1);
            } else if (type === 'door') {
                // Golpe metálico
                osc.type = 'square';
                osc.frequency.setValueAtTime(150, now);
                osc.frequency.exponentialRampToValueAtTime(40, now + 0.3);
                gainNode.gain.setValueAtTime(0.8, now);
                gainNode.gain.exponentialRampToValueAtTime(0.01, now + 0.3);
                osc.start(now);
                osc.stop(now + 0.3);
            } else if (type === 'light') {
                // Zumbido eléctrico
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(60, now);
                gainNode.gain.setValueAtTime(0.1, now);
                gainNode.gain.linearRampToValueAtTime(0, now + 0.5);
                osc.start(now);
                osc.stop(now + 0.5);
            } else if (type === 'jumpscare') {
                // Alarido digital aterrador
                const osc2 = audioCtx.createOscillator();
                osc.type = 'sawtooth';
                osc2.type = 'square';
                osc.frequency.setValueAtTime(100, now);
                osc.frequency.linearRampToValueAtTime(600, now + 0.2);
                osc2.frequency.setValueAtTime(150, now);
                osc2.frequency.linearRampToValueAtTime(800, now + 0.2);
                
                // Efecto de distorsión básica
                gainNode.gain.setValueAtTime(1, now);
                gainNode.gain.linearRampToValueAtTime(0.8, now + 1.5);
                gainNode.gain.linearRampToValueAtTime(0.01, now + 2);

                osc2.connect(gainNode);
                osc.start(now);
                osc2.start(now);
                osc.stop(now + 2);
                osc2.stop(now + 2);
            } else if (type === 'powerdown') {
                // Apagón
                osc.type = 'sine';
                osc.frequency.setValueAtTime(300, now);
                osc.frequency.exponentialRampToValueAtTime(20, now + 3);
                gainNode.gain.setValueAtTime(0.5, now);
                gainNode.gain.linearRampToValueAtTime(0.01, now + 3);
                osc.start(now);
                osc.stop(now + 3);
            }
        }

        function startHum() {
            if (!audioCtx) return;
            humOscillator = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            humOscillator.type = 'triangle';
            humOscillator.frequency.value = 55; // Tono grave
            gain.gain.value = 0.2;
            humOscillator.connect(gain);
            gain.connect(audioCtx.destination);
            humOscillator.start();
        }

        function stopHum() {
            if (humOscillator) {
                humOscillator.stop();
                humOscillator = null;
            }
        }

        // --- MAPA DE CÁMARAS Y UBICACIONES ---
        const cameras = {
            '1A': { name: 'Show Stage', x: 150, y: 20 },
            '1B': { name: 'Dining Area', x: 100, y: 100 },
            '1C': { name: 'Pirate Cove', x: 20, y: 140 },
            '5':  { name: 'Backstage', x: 10, y: 200 },
            '2A': { name: 'West Hall', x: 80, y: 240 },
            '2B': { name: 'W. Hall Corner', x: 80, y: 300 },
            '3':  { name: 'Supply Closet', x: 30, y: 270 },
            '4A': { name: 'East Hall', x: 220, y: 240 },
            '4B': { name: 'E. Hall Corner', x: 220, y: 300 }
        };

        // --- INTELIGENCIA ARTIFICIAL (ANIMATRÓNICOS) ---
        // Rutas y dificultad básica (Escala 1-20)
        let animatronics = {
            bonnie: { loc: '1A', path: ['1A', '1B', '5', '2A', '3', '2B', 'left_door'], ai: 4 },
            chica:  { loc: '1A', path: ['1A', '1B', '4A', '4B', 'right_door'], ai: 3 },
            foxy:   { loc: '1C', stage: 0, ai: 2 }, // Stage 0: cerrado, 3: corre, 4: ataque
            freddy: { loc: '1A', path: ['1A', '1B', '4A', '4B', 'right_door'], ai: 1 } // Solo ataca a niveles bajos de energía o avanzado el juego
        };

        // --- INICIALIZACIÓN Y BUCLE PRINCIPAL ---
        function startGame() {
            document.getElementById('main-menu').style.display = 'none';
            document.getElementById('game-container').style.display = 'block';
            
            if (audioCtx.state === 'suspended') audioCtx.resume();
            startHum();
            buildMap();
            
            gameActive = true;
            power = 100;
            timeHour = 0;
            updateUI();
            
            // Bucles de tiempo y energía
            timeTicking = setInterval(advanceTime, 60000); // 1 hora in-game = 60 segs reales (Total 6 min)
            powerDraining = setInterval(drainPower, 1000); 
            aiTicking = setInterval(updateAI, 4900); // Movimiento de la IA cada ~5 segs
            
            // Loop de dibujo para la estática de cámaras
            requestAnimationFrame(renderCameras);
        }

        // --- TIEMPO Y ENERGÍA ---
        function advanceTime() {
            if (!gameActive) return;
            timeHour++;
            document.getElementById('time-display').innerText = timeHour === 0 ? '12 AM' : timeHour + ' AM';
            
            // Aumentar dificultad
            if (timeHour === 2) { animatronics.bonnie.ai++; animatronics.chica.ai++; }
            if (timeHour === 3) { animatronics.bonnie.ai++; animatronics.foxy.ai++; }
            if (timeHour === 4) { animatronics.chica.ai++; animatronics.freddy.ai++; }

            if (timeHour >= 6) {
                winGame();
            }
        }

        function drainPower() {
            if (!gameActive || state.powerOut) return;
            // Calcular uso actual
            usage = 1;
            if (state.doorLeft) usage++;
            if (state.doorRight) usage++;
            if (state.lightLeft || state.lightRight) usage++;
            if (state.cameraUp) usage++;
            if (usage > 4) usage = 4;

            // Restar energía basándose en el uso (ajustado para partidas de 6 mins)
            let drainAmount = usage * 0.15; 
            power -= drainAmount;

            if (power <= 0) {
                power = 0;
                triggerPowerOut();
            }
            updateUI();
        }

        function updateUI() {
            document.getElementById('power-display').innerText = `Energía: ${Math.floor(power)}%`;
            for(let i=1; i<=4; i++) {
                let bar = document.getElementById(i === 1 ? 'u1' : 'u'+i);
                if(bar) {
                    if (i <= usage) bar.classList.add(`active-${i}`);
                    else bar.className = 'usage-bar';
                }
            }
        }

        function triggerPowerOut() {
            state.powerOut = true;
            stopHum();
            playSound('powerdown');
            
            // Abrir todo
            if (state.doorLeft) toggleDoor('left');
            if (state.doorRight) toggleDoor('right');
            if (state.cameraUp) toggleCamera();
            state.lightLeft = false; state.lightRight = false;
            
            document.getElementById('office').style.background = '#000';
            document.getElementById('ui-layer').style.display = 'none';
            
            // Freddy ataca en la oscuridad después de unos segundos
            setTimeout(() => {
                triggerJumpscare('freddy');
            }, Math.random() * 10000 + 5000);
        }

        // --- CONTROLES DE LA OFICINA ---
        function toggleDoor(side) {
            if (state.powerOut || !gameActive) return;
            playSound('door');
            let door = document.getElementById(`door-${side}`);
            let btn = document.querySelector(`#${side}-controls .btn-door`);
            
            if (side === 'left') {
                state.doorLeft = !state.doorLeft;
                state.doorLeft ? door.classList.add('closed') : door.classList.remove('closed');
                state.doorLeft ? btn.classList.add('active') : btn.classList.remove('active');
            } else {
                state.doorRight = !state.doorRight;
                state.doorRight ? door.classList.add('closed') : door.classList.remove('closed');
                state.doorRight ? btn.classList.add('active') : btn.classList.remove('active');
            }
            updatePowerUsageImmediate();
        }

        function toggleLight(side) {
            if (state.powerOut || !gameActive || state.cameraUp) return;
            
            // Apagar la otra
            if (side === 'left' && state.lightRight) toggleLight('right');
            if (side === 'right' && state.lightLeft) toggleLight('left');

            let hallway = document.getElementById(`hall-${side}`);
            let btn = document.querySelector(`#${side}-controls .btn-light`);
            let monsterCanvas = document.getElementById(`monster-${side}`);
            
            if (side === 'left') {
                state.lightLeft = !state.lightLeft;
                if(state.lightLeft) {
                    playSound('light');
                    btn.classList.add('active');
                    hallway.style.background = '#556677'; // Luz blanca/azul
                    // Comprobar si hay monstruo
                    if (animatronics.bonnie.loc === 'left_door' || animatronics.foxy.loc === 'left_door') {
                        monsterCanvas.style.opacity = 1;
                        drawSilhouette(monsterCanvas, animatronics.bonnie.loc === 'left_door' ? 'bonnie' : 'foxy');
                        playSound('blip'); // Sonido de asombro
                    }
                } else {
                    btn.classList.remove('active');
                    hallway.style.background = '#0a0a0a';
                    monsterCanvas.style.opacity = 0;
                }
            } else {
                state.lightRight = !state.lightRight;
                if(state.lightRight) {
                    playSound('light');
                    btn.classList.add('active');
                    hallway.style.background = '#777755'; // Luz amarillenta
                    if (animatronics.chica.loc === 'right_door' || animatronics.freddy.loc === 'right_door') {
                        monsterCanvas.style.opacity = 1;
                        drawSilhouette(monsterCanvas, animatronics.chica.loc === 'right_door' ? 'chica' : 'freddy');
                        playSound('blip');
                    }
                } else {
                    btn.classList.remove('active');
                    hallway.style.background = '#0a0a0a';
                    monsterCanvas.style.opacity = 0;
                }
            }
            updatePowerUsageImmediate();
        }

        function updatePowerUsageImmediate() {
            let u = 1;
            if (state.doorLeft) u++;
            if (state.doorRight) u++;
            if (state.lightLeft || state.lightRight) u++;
            if (state.cameraUp) u++;
            usage = u > 4 ? 4 : u;
            updateUI();
        }

        // --- SISTEMA DE CÁMARAS ---
        function toggleCamera() {
            if (state.powerOut || !gameActive) return;
            playSound('door'); // Usar sonido metálico como sustituto de abrir monitor
            state.cameraUp = !state.cameraUp;
            
            // Apagar luces al usar cámara
            if (state.lightLeft) toggleLight('left');
            if (state.lightRight) toggleLight('right');

            const camSys = document.getElementById('camera-system');
            camSys.style.display = state.cameraUp ? 'block' : 'none';
            document.getElementById('camera-toggle').innerText = state.cameraUp ? "BAJAR MONITOR" : "ABRIR MONITOR";
            
            if (state.cameraUp) renderCamFrame(); // Forzar dibujo inmediato
            updatePowerUsageImmediate();
        }

        function buildMap() {
            const container = document.getElementById('map-container');
            for (let id in cameras) {
                let btn = document.createElement('button');
                btn.className = 'cam-btn';
                btn.id = 'btn-cam-' + id;
                btn.style.left = cameras[id].x + 'px';
                btn.style.top = cameras[id].y + 'px';
                btn.innerText = 'CAM ' + id;
                btn.onclick = () => switchCamera(id);
                container.appendChild(btn);
            }
            document.getElementById('btn-cam-1A').classList.add('active');
        }

        function switchCamera(id) {
            playSound('blip');
            document.querySelectorAll('.cam-btn').forEach(b => b.classList.remove('active'));
            document.getElementById('btn-cam-' + id).classList.add('active');
            state.currentCam = id;
            document.getElementById('cam-name-display').innerText = `CAM ${id} - ${cameras[id].name}`;
            renderCamFrame(); // Redibujar escena actual
        }

        // --- RENDERIZADO PROCEDIMENTAL (CANVAS) ---
        function drawSilhouette(canvas, type) {
            const ctx = canvas.getContext('2d');
            canvas.width = 150; canvas.height = 300;
            ctx.clearRect(0,0, canvas.width, canvas.height);
            ctx.fillStyle = '#000'; // Silueta oscura
            
            if (type === 'bonnie') {
                // Cabeza
                ctx.beginPath(); ctx.ellipse(75, 150, 40, 50, 0, 0, Math.PI*2); ctx.fill();
                // Orejas largas
                ctx.beginPath(); ctx.ellipse(55, 60, 15, 60, -0.2, 0, Math.PI*2); ctx.fill();
                ctx.beginPath(); ctx.ellipse(95, 60, 15, 60, 0.2, 0, Math.PI*2); ctx.fill();
                // Ojos brillantes
                ctx.fillStyle = '#fff';
                ctx.beginPath(); ctx.arc(60, 140, 5, 0, Math.PI*2); ctx.fill();
                ctx.beginPath(); ctx.arc(90, 140, 5, 0, Math.PI*2); ctx.fill();
            } else if (type === 'chica') {
                // Cabeza redondeada
                ctx.beginPath(); ctx.ellipse(75, 160, 45, 45, 0, 0, Math.PI*2); ctx.fill();
                // Plumas
                ctx.fillRect(70, 100, 10, 20);
                // Pico
                ctx.fillStyle = '#ff8800'; // Naranja
                ctx.beginPath(); ctx.ellipse(75, 180, 25, 15, 0, 0, Math.PI*2); ctx.fill();
                // Ojos
                ctx.fillStyle = '#fff';
                ctx.beginPath(); ctx.arc(60, 150, 6, 0, Math.PI*2); ctx.fill();
                ctx.beginPath(); ctx.arc(90, 150, 6, 0, Math.PI*2); ctx.fill();
            } else if (type === 'foxy') {
                // Hocico agudo
                ctx.beginPath(); ctx.moveTo(75, 180); ctx.lineTo(30, 140); ctx.lineTo(120, 140); ctx.fill();
                // Parche
                ctx.fillStyle = '#111';
                ctx.beginPath(); ctx.arc(90, 130, 10, 0, Math.PI*2); ctx.fill();
                ctx.fillRect(75, 125, 30, 5);
                // Ojo descubierto
                ctx.fillStyle = '#ffaa00';
                ctx.beginPath(); ctx.arc(60, 130, 6, 0, Math.PI*2); ctx.fill();
            } else if (type === 'freddy') {
                // Cabeza ancha
                ctx.beginPath(); ctx.ellipse(75, 150, 55, 45, 0, 0, Math.PI*2); ctx.fill();
                // Sombrero
                ctx.fillRect(55, 60, 40, 45);
                ctx.fillRect(40, 100, 70, 10);
                // Ojos brillando
                ctx.fillStyle = '#fff';
                ctx.beginPath(); ctx.arc(55, 140, 4, 0, Math.PI*2); ctx.fill();
                ctx.beginPath(); ctx.arc(95, 140, 4, 0, Math.PI*2); ctx.fill();
            }
        }

        // Renderiza el fondo de la cámara y los animatrónicos presentes
        function renderCamFrame() {
            const canvas = document.getElementById('camera-feed');
            const ctx = canvas.getContext('2d');
            canvas.width = 800; canvas.height = 600;
            
            // Dibujar fondo de habitación (geométrico oscuro)
            ctx.fillStyle = '#1a1a1a';
            ctx.fillRect(0,0, canvas.width, canvas.height);
            
            // Perspectiva simple para dar sensación de habitación 3D
            ctx.strokeStyle = '#333';
            ctx.lineWidth = 5;
            ctx.beginPath(); ctx.moveTo(0,0); ctx.lineTo(200, 150); ctx.stroke();
            ctx.beginPath(); ctx.moveTo(800,0); ctx.lineTo(600, 150); ctx.stroke();
            ctx.beginPath(); ctx.moveTo(0,600); ctx.lineTo(200, 450); ctx.stroke();
            ctx.beginPath(); ctx.moveTo(800,600); ctx.lineTo(600, 450); ctx.stroke();
            ctx.strokeRect(200, 150, 400, 300);

            let hasAnimatronic = false;
            
            // Dibujar a los animatrónicos en sus posiciones relativas a la cámara
            if (animatronics.bonnie.loc === state.currentCam) {
                drawCamMonster(ctx, 'bonnie', 300, 300);
                hasAnimatronic = true;
            }
            if (animatronics.chica.loc === state.currentCam) {
                drawCamMonster(ctx, 'chica', 500, 320);
                hasAnimatronic = true;
            }
            if (animatronics.freddy.loc === state.currentCam && (animatronics.freddy.loc !== '1A' || hasAnimatronic)) {
                // Freddy se esconde en las sombras
                ctx.globalAlpha = 0.5;
                drawCamMonster(ctx, 'freddy', 400, 250);
                ctx.globalAlpha = 1.0;
                hasAnimatronic = true;
            }

            // Foxy especial en Pirate Cove
            if (state.currentCam === '1C') {
                ctx.fillStyle = '#300'; // Cortinas rojas
                ctx.fillRect(100, 100, 200, 400); // Izq
                ctx.fillRect(500, 100, 200, 400); // Der
                
                if (animatronics.foxy.stage === 1) drawCamMonster(ctx, 'foxy', 400, 350); // Asomando
                else if (animatronics.foxy.stage === 2) drawCamMonster(ctx, 'foxy', 400, 400); // Fuera
            }
        }

        // Dibuja una versión más grande/detallada para las cámaras
        function drawCamMonster(ctx, type, x, y) {
            ctx.save();
            ctx.translate(x - 75, y - 150);
            
            if (type === 'bonnie') {
                ctx.fillStyle = '#3a4a6e'; // Azul violáceo
                ctx.beginPath(); ctx.ellipse(75, 150, 40, 50, 0, 0, Math.PI*2); ctx.fill();
                ctx.beginPath(); ctx.ellipse(55, 60, 15, 60, -0.2, 0, Math.PI*2); ctx.fill();
                ctx.beginPath(); ctx.ellipse(95, 60, 15, 60, 0.2, 0, Math.PI*2); ctx.fill();
                ctx.fillStyle = '#fff';
                ctx.beginPath(); ctx.arc(60, 140, 5, 0, Math.PI*2); ctx.fill();
                ctx.beginPath(); ctx.arc(90, 140, 5, 0, Math.PI*2); ctx.fill();
            } else if (type === 'chica') {
                ctx.fillStyle = '#bfa82a'; // Amarillo
                ctx.beginPath(); ctx.ellipse(75, 160, 45, 45, 0, 0, Math.PI*2); ctx.fill();
                ctx.fillStyle = '#d66f1c'; // Pico naranja
                ctx.beginPath(); ctx.ellipse(75, 180, 25, 15, 0, 0, Math.PI*2); ctx.fill();
                ctx.fillStyle = '#fff';
                ctx.beginPath(); ctx.arc(60, 150, 6, 0, Math.PI*2); ctx.fill();
                ctx.beginPath(); ctx.arc(90, 150, 6, 0, Math.PI*2); ctx.fill();
            } else if (type === 'freddy') {
                ctx.fillStyle = '#4a2f1d'; // Marrón
                ctx.beginPath(); ctx.ellipse(75, 150, 55, 45, 0, 0, Math.PI*2); ctx.fill();
                ctx.fillStyle = '#111'; // Sombrero
                ctx.fillRect(55, 60, 40, 45);
                ctx.fillRect(40, 100, 70, 10);
                ctx.fillStyle = '#fff';
                ctx.beginPath(); ctx.arc(55, 140, 4, 0, Math.PI*2); ctx.fill();
                ctx.beginPath(); ctx.arc(95, 140, 4, 0, Math.PI*2); ctx.fill();
            } else if (type === 'foxy') {
                ctx.fillStyle = '#8a2b2b'; // Rojo oscuro
                ctx.beginPath(); ctx.moveTo(75, 180); ctx.lineTo(30, 140); ctx.lineTo(120, 140); ctx.fill();
                ctx.fillStyle = '#111'; // Parche
                ctx.beginPath(); ctx.arc(90, 130, 10, 0, Math.PI*2); ctx.fill();
                ctx.fillRect(75, 125, 30, 5);
                ctx.fillStyle = '#ffaa00';
                ctx.beginPath(); ctx.arc(60, 130, 6, 0, Math.PI*2); ctx.fill();
                // Garfio
                ctx.strokeStyle = '#aaa'; ctx.lineWidth = 5;
                ctx.beginPath(); ctx.arc(20, 200, 20, Math.PI, 0); ctx.stroke();
            }
            ctx.restore();
        }

        // Bucle continuo para la estática
        function renderCameras() {
            if (!gameActive) return;
            if (state.cameraUp) {
                const staticCanvas = document.getElementById('camera-static');
                const ctx = staticCanvas.getContext('2d');
                staticCanvas.width = 400; staticCanvas.height = 300;
                
                // Generar ruido blanco en un canvas superpuesto
                const imgData = ctx.createImageData(staticCanvas.width, staticCanvas.height);
                const data = imgData.data;
                for (let i = 0; i < data.length; i += 4) {
                    const val = Math.random() * 255;
                    data[i] = val;     // red
                    data[i+1] = val;   // green
                    data[i+2] = val;   // blue
                    data[i+3] = 100;   // alpha
                }
                ctx.putImageData(imgData, 0, 0);
            }
            requestAnimationFrame(renderCameras);
        }

        // --- LÓGICA DE MOVIMIENTO E IA ---
        function updateAI() {
            if (!gameActive || state.powerOut) return;
            
            // Oportunidad de movimiento basada en nivel AI
            const tryMove = (aiLevel) => (Math.random() * 20) <= aiLevel;

            // BONNIE
            if (tryMove(animatronics.bonnie.ai)) {
                moveAnimatronic('bonnie');
            }

            // CHICA
            if (tryMove(animatronics.chica.ai)) {
                moveAnimatronic('chica');
            }

            // FOXY
            if (tryMove(animatronics.foxy.ai)) {
                // Si la cámara actual NO es 1C (Pirate Cove) o las cámaras están apagadas, Foxy avanza
                if (!state.cameraUp || state.currentCam !== '1C') {
                    animatronics.foxy.stage++;
                    if (animatronics.foxy.stage >= 3) {
                        animatronics.foxy.loc = 'left_door';
                        // Corre por el pasillo. ¡Debe cerrarse la puerta!
                        setTimeout(checkFoxyAttack, 2000); // 2 segundos para cerrar puerta
                    }
                } else if (state.cameraUp && state.currentCam === '1C') {
                    // Vigilar a Foxy lo frena o retrocede levemente
                    if(Math.random() > 0.5 && animatronics.foxy.stage > 0) animatronics.foxy.stage--;
                }
            }

            // FREDDY
            if (tryMove(animatronics.freddy.ai)) {
                // Freddy solo se mueve si no se le está mirando directamente en cámaras
                if (!state.cameraUp || state.currentCam !== animatronics.freddy.loc) {
                   moveAnimatronic('freddy');
                }
            }

            // Actualizar vista de cámara si es necesario
            if (state.cameraUp) renderCamFrame();
            
            // Jumpscare si están en la puerta y el jugador abre cámaras
            if (state.cameraUp) {
                if (animatronics.bonnie.loc === 'left_door' && !state.doorLeft) triggerJumpscare('bonnie');
                if (animatronics.chica.loc === 'right_door' && !state.doorRight) triggerJumpscare('chica');
                if (animatronics.freddy.loc === 'right_door' && !state.doorRight) triggerJumpscare('freddy');
            }
        }

        function moveAnimatronic(name) {
            let anim = animatronics[name];
            let currentIdx = anim.path.indexOf(anim.loc);
            
            // Movimiento hacia adelante o hacia atrás aleatorio
            if (currentIdx < anim.path.length - 1) {
                // 80% avanzar, 20% retroceder
                if (Math.random() > 0.2 || currentIdx === 0) {
                    anim.loc = anim.path[currentIdx + 1];
                } else {
                    anim.loc = anim.path[currentIdx - 1];
                }
            } else if (anim.loc.includes('door')) {
                // Ya están en la puerta. ¿Está cerrada?
                let doorClosed = (name === 'bonnie' && state.doorLeft) || 
                                 (name === 'chica' && state.doorRight) || 
                                 (name === 'freddy' && state.doorRight);
                                 
                if (doorClosed) {
                    // Si está cerrada, se rinden y vuelven al comedor u otra zona
                    anim.loc = '1B'; 
                } else {
                    // Si está abierta y el jugador no ve las cámaras (se comprobó arriba si las veía),
                    // existe la chance de ataque aleatorio, o esperan a que abras cámaras.
                    if (Math.random() > 0.5) triggerJumpscare(name);
                }
            }
        }

        function checkFoxyAttack() {
            if (!gameActive) return;
            if (state.doorLeft) {
                // Golpe en la puerta, quita poder
                playSound('door');
                setTimeout(() => playSound('door'), 300);
                setTimeout(() => playSound('door'), 600);
                power -= 10; // Penalización por Foxy
                if (power < 0) power = 0;
                updateUI();
                animatronics.foxy.stage = 0;
                animatronics.foxy.loc = '1C';
            } else {
                // Puerta abierta = GAME OVER
                triggerJumpscare('foxy');
            }
        }

        // --- EVENTOS FINALES ---
        function triggerJumpscare(type) {
            if (!gameActive) return;
            gameActive = false;
            clearInterval(timeTicking);
            clearInterval(powerDraining);
            clearInterval(aiTicking);
            
            // Bajar cámara
            document.getElementById('camera-system').style.display = 'none';
            document.getElementById('office').style.display = 'none';
            document.getElementById('ui-layer').style.display = 'none';

            // Mostrar y animar Jumpscare
            const jContainer = document.getElementById('jumpscare-container');
            const jCanvas = document.getElementById('jumpscare-canvas');
            const ctx = jCanvas.getContext('2d');
            jCanvas.width = window.innerWidth;
            jCanvas.height = window.innerHeight;
            
            jContainer.style.display = 'block';
            playSound('jumpscare');

            // Dibujo aterrador al centro que escala
            let scale = 1;
            let scareInterval = setInterval(() => {
                ctx.fillStyle = '#000';
                ctx.fillRect(0,0, jCanvas.width, jCanvas.height);
                
                ctx.save();
                ctx.translate(jCanvas.width/2, jCanvas.height/2);
                ctx.scale(scale, scale);
                
                // Dibujar versión hiper dimensionada para dar miedo
                drawCamMonster(ctx, type, 0, 0); 
                
                // Efecto de ruido/estática estroboscópica sobre el modelo
                if(Math.random() > 0.5) {
                    ctx.fillStyle = 'rgba(255,255,255,0.2)';
                    ctx.fillRect(-jCanvas.width, -jCanvas.height, jCanvas.width*2, jCanvas.height*2);
                }
                
                ctx.restore();
                scale += 0.5; // Acercamiento brutal
                
                if (scale > 10) {
                    clearInterval(scareInterval);
                    showGameOver();
                }
            }, 50);
        }

        function showGameOver() {
            document.getElementById('jumpscare-container').style.display = 'none';
            document.getElementById('game-over').style.display = 'flex';
            
            // Animación de estática de Game Over
            const sCanvas = document.getElementById('static-end');
            const ctx = sCanvas.getContext('2d');
            sCanvas.width = 400; sCanvas.height = 300;
            setInterval(() => {
                const imgData = ctx.createImageData(400, 300);
                const data = imgData.data;
                for (let i = 0; i < data.length; i += 4) {
                    const val = Math.random() * 255;
                    data[i] = val; data[i+1] = val; data[i+2] = val; data[i+3] = 255;
                }
                ctx.putImageData(imgData, 0, 0);
            }, 30);
        }

        function winGame() {
            if (!gameActive) return;
            gameActive = false;
            clearInterval(timeTicking);
            clearInterval(powerDraining);
            clearInterval(aiTicking);
            stopHum();
            
            document.getElementById('office').style.display = 'none';
            document.getElementById('camera-system').style.display = 'none';
            document.getElementById('ui-layer').style.display = 'none';
            
            const winScreen = document.getElementById('win-screen');
            winScreen.style.display = 'flex';
            playSound('blip');
            
            // Efecto de reloj cambiando de 5:59 a 6:00
            setTimeout(() => {
                winScreen.querySelector('h1:nth-child(1)').style.display = 'none';
                document.getElementById('win-text').style.display = 'block';
                // Campanada improvisada
                playSound('door'); 
            }, 2000);
        }

    </script>
</body>
</html>
