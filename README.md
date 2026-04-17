
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/drawing_utils/drawing_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/pose/pose.js" crossorigin="anonymous"></script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700;900&display=swap');

        :root {
            --bg: #02000a;
            --accent: #00ffcc;
            --accent2: #ff00aa;
            --glass: rgba(10, 5, 30, 0.5);
            --border: rgba(0, 255, 204, 0.2);
        }

        * { margin: 0; padding: 0; box-sizing: border-box; }

        body {
            background: var(--bg);
            overflow: hidden;
            font-family: 'Orbitron', monospace;
            color: white;
        }

        .video-container {
            position: fixed;
            inset: 0;
        }

        video {
            position: absolute;
            width: 100%; height: 100%;
            object-fit: cover;
            transform: scaleX(-1);
            filter: brightness(0.5) saturate(1.2);
            z-index: 0;
        }

        canvas {
            position: absolute;
            inset: 0;
            width: 100%; height: 100%;
            transform: scaleX(-1);
            z-index: 2;
        }

        /* HUD */
        #hud {
            position: fixed;
            top: 24px; left: 24px;
            z-index: 20;
            display: flex;
            flex-direction: column;
            gap: 12px;
            pointer-events: none;
        }

        .panel {
            background: var(--glass);
            backdrop-filter: blur(20px);
            border: 1px solid var(--border);
            border-radius: 10px;
            padding: 12px 18px;
            box-shadow: 0 0 20px rgba(0,255,204,0.08), inset 0 0 20px rgba(0,255,204,0.03);
        }

        .panel-title {
            font-size: 0.55rem;
            letter-spacing: 4px;
            color: var(--accent);
            opacity: 0.7;
            margin-bottom: 8px;
            text-transform: uppercase;
        }

        .stat {
            display: flex;
            justify-content: space-between;
            gap: 24px;
            font-size: 0.7rem;
            color: rgba(255,255,255,0.6);
            margin-bottom: 5px;
        }
        .stat:last-child { margin-bottom: 0; }
        .val { color: var(--accent); font-weight: 700; }

        /* Themes Bar */
        #themes {
            position: fixed;
            bottom: 24px;
            left: 50%;
            transform: translateX(-50%);
            z-index: 20;
            display: flex;
            gap: 8px;
            background: var(--glass);
            backdrop-filter: blur(20px);
            border: 1px solid var(--border);
            border-radius: 50px;
            padding: 6px;
        }

        .theme-btn {
            background: transparent;
            border: 1px solid transparent;
            color: rgba(255,255,255,0.5);
            padding: 7px 16px;
            border-radius: 50px;
            cursor: pointer;
            font-family: 'Orbitron', monospace;
            font-size: 0.6rem;
            letter-spacing: 1px;
            transition: all 0.25s;
            pointer-events: all;
        }

        .theme-btn:hover { color: white; }
        .theme-btn.active {
            background: rgba(0,255,204,0.12);
            border-color: rgba(0,255,204,0.4);
            color: var(--accent);
            text-shadow: 0 0 10px var(--accent);
        }

        /* Start Overlay */
        #overlay {
            position: fixed;
            inset: 0;
            z-index: 100;
            background: radial-gradient(ellipse at center, #0d0020 0%, #020008 100%);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            transition: opacity 0.8s ease;
        }

        #overlay.gone {
            opacity: 0;
            pointer-events: none;
        }

        .title {
            font-size: clamp(2rem, 6vw, 4rem);
            font-weight: 900;
            letter-spacing: 8px;
            text-transform: uppercase;
            background: linear-gradient(135deg, #00ffcc, #ff00aa, #7b00ff);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            background-clip: text;
            margin-bottom: 12px;
            text-align: center;
        }

        .subtitle {
            font-size: 0.65rem;
            letter-spacing: 5px;
            color: rgba(255,255,255,0.3);
            margin-bottom: 60px;
            text-align: center;
        }

        .start-btn {
            padding: 16px 48px;
            font-family: 'Orbitron', monospace;
            font-size: 0.8rem;
            font-weight: 700;
            letter-spacing: 4px;
            text-transform: uppercase;
            background: transparent;
            border: 1px solid rgba(0,255,204,0.5);
            color: var(--accent);
            border-radius: 50px;
            cursor: pointer;
            position: relative;
            overflow: hidden;
            transition: all 0.3s;
            box-shadow: 0 0 30px rgba(0,255,204,0.15), inset 0 0 30px rgba(0,255,204,0.05);
        }

        .start-btn::before {
            content: '';
            position: absolute;
            inset: 0;
            background: linear-gradient(135deg, rgba(0,255,204,0.1), rgba(255,0,170,0.1));
            opacity: 0;
            transition: opacity 0.3s;
        }

        .start-btn:hover {
            box-shadow: 0 0 60px rgba(0,255,204,0.3), inset 0 0 30px rgba(0,255,204,0.1);
            transform: scale(1.03);
        }

        .start-btn:hover::before { opacity: 1; }

        .features {
            margin-top: 50px;
            display: flex;
            gap: 30px;
            opacity: 0.4;
            font-size: 0.55rem;
            letter-spacing: 3px;
            color: white;
        }

        .hidden { display: none; }
    </style>
</head>
<body>

<div id="overlay">
    <div class="title">NEON BODY AR</div>
    <div class="subtitle">FULL BODY TRACKING · REAL-TIME EFFECTS</div>
    <button class="start-btn" id="startBtn">ENTER</button>
    <div class="features">
        <span>✦ SKELETON</span>
        <span>✦ PARTICLES</span>
        <span>✦ AURA</span>
        <span>✦ LIGHTNING</span>
    </div>
</div>

<div class="video-container">
    <video id="video" autoplay playsinline></video>
    <canvas id="canvas"></canvas>
</div>

<div id="hud" class="hidden">
    <div class="panel">
        <div class="panel-title">System</div>
        <div class="stat">FPS <span class="val" id="ui-fps">—</span></div>
        <div class="stat">Body <span class="val" id="ui-body">Not Detected</span></div>
    </div>
    <div class="panel">
        <div class="panel-title">Pose</div>
        <div class="stat">Pose <span class="val" id="ui-pose">—</span></div>
        <div class="stat">Energy <span class="val" id="ui-energy">—</span></div>
    </div>
</div>

<div id="themes" class="hidden">
    <button class="theme-btn active" data-theme="Neon">Neon</button>
    <button class="theme-btn" data-theme="Plasma">Plasma</button>
    <button class="theme-btn" data-theme="Inferno">Inferno</button>
    <button class="theme-btn" data-theme="Void">Void</button>
    <button class="theme-btn" data-theme="Aurora">Aurora</button>
</div>

<script>
// ─── GLOBALS ────────────────────────────────────────────────────────────────
const video   = document.getElementById('video');
const canvas  = document.getElementById('canvas');
const ctx     = canvas.getContext('2d');

let W = window.innerWidth, H = window.innerHeight;
let time = 0, lastTime = performance.now();
let fps = 0, fpsFrames = 0, fpsLast = performance.now();
let currentLandmarks = null;
let prevLandmarks    = null;
let energy = 0; // body movement energy

// ─── THEMES ─────────────────────────────────────────────────────────────────
let theme = 'Neon';
const THEMES = {
    Neon:    { joint: '#00ffcc', bone: '#00ffcc', aura: '#00ffcc', particle: () => `hsl(${160 + Math.random()*40},100%,70%)` },
    Plasma:  { joint: '#ff00ff', bone: '#aa00ff', aura: '#ff00ff', particle: () => `hsl(${270 + Math.random()*60},100%,70%)` },
    Inferno: { joint: '#ff6600', bone: '#ff2200', aura: '#ff4400', particle: () => `hsl(${10 + Math.random()*30},100%,60%)` },
    Void:    { joint: '#ffffff', bone: '#8888ff', aura: '#4444ff', particle: () => `hsl(${220 + Math.random()*40},80%,80%)` },
    Aurora:  { joint: '#00ff88', bone: '#00aaff', aura: '#00ddbb', particle: () => `hsl(${150 + Math.random()*80},100%,65%)` },
};

// ─── MEDIAPIPE POSE CONNECTIONS ──────────────────────────────────────────────
// Full body: torso, arms, legs, face
const CONNECTIONS = [
    // Face
    [0,1],[1,2],[2,3],[3,7],[0,4],[4,5],[5,6],[6,8],
    // Shoulders & torso
    [11,12],[11,23],[12,24],[23,24],
    // Left arm
    [11,13],[13,15],[15,17],[15,19],[15,21],[17,19],
    // Right arm
    [12,14],[14,16],[16,18],[16,20],[16,22],[18,20],
    // Left leg
    [23,25],[25,27],[27,29],[27,31],[29,31],
    // Right leg
    [24,26],[26,28],[28,30],[28,32],[30,32],
];

// Key joints for extra glow
const KEY_JOINTS = [0, 11, 12, 13, 14, 15, 16, 23, 24, 25, 26, 27, 28];
// Fingertip equivalents for sparks
const SPARK_JOINTS = [15, 16, 27, 28, 31, 32, 29, 30];

// ─── PARTICLES ───────────────────────────────────────────────────────────────
let particles = [];

function spawnParticle(x, y, color, speed = 4) {
    const angle = Math.random() * Math.PI * 2;
    particles.push({
        x, y,
        vx: Math.cos(angle) * speed * Math.random(),
        vy: Math.sin(angle) * speed * Math.random() - 1,
        life: 1.0,
        decay: 0.015 + Math.random() * 0.03,
        size: 1.5 + Math.random() * 3,
        color,
    });
}

// ─── TRAIL / AURA DATA ────────────────────────────────────────────────────────
let trailFrames = []; // stores last N landmark arrays
const TRAIL_LEN = 8;

// ─── RESIZE ──────────────────────────────────────────────────────────────────
function resize() {
    W = window.innerWidth; H = window.innerHeight;
    canvas.width = W; canvas.height = H;
}
window.addEventListener('resize', resize);
resize();

// ─── MAP LANDMARK → CANVAS ───────────────────────────────────────────────────
function lm(landmark) {
    return { x: landmark.x * W, y: landmark.y * H, z: landmark.z || 0 };
}

// ─── THEME SWITCHER ──────────────────────────────────────────────────────────
document.querySelectorAll('.theme-btn').forEach(btn => {
    btn.addEventListener('click', e => {
        document.querySelectorAll('.theme-btn').forEach(b => b.classList.remove('active'));
        e.target.classList.add('active');
        theme = e.target.dataset.theme;
    });
});

// ─── POSE DETECTION LOGIC ────────────────────────────────────────────────────
function detectPose(lms) {
    if (!lms) return '—';
    const ls = lms[11], rs = lms[12];
    const lh = lms[15], rh = lms[16];
    const lk = lms[25], rk = lms[26];

    const leftArmUp  = lh && ls && lh.y < ls.y;
    const rightArmUp = rh && rs && rh.y < rs.y;
    const bothArmsUp = leftArmUp && rightArmUp;
    const kneesBent  = lk && ls && rk && rs && lk.y < ls.y + 0.1 && rk.y < rs.y + 0.1;

    if (bothArmsUp) return 'ARMS RAISED';
    if (leftArmUp)  return 'LEFT ARM UP';
    if (rightArmUp) return 'RIGHT ARM UP';
    if (kneesBent)  return 'CROUCHING';
    return 'STANDING';
}

function calcEnergy(curr, prev) {
    if (!curr || !prev) return 0;
    let total = 0;
    for (let i = 0; i < Math.min(curr.length, prev.length); i++) {
        total += Math.hypot(curr[i].x - prev[i].x, curr[i].y - prev[i].y);
    }
    return Math.min(total / curr.length, 1);
}

// ─── DRAW ROUTINES ───────────────────────────────────────────────────────────

function drawAuraTrails(lms) {
    const t = THEMES[theme];
    trailFrames.forEach((frame, fi) => {
        const alpha = (fi / trailFrames.length) * 0.18;
        CONNECTIONS.forEach(([a, b]) => {
            if (!frame[a] || !frame[b]) return;
            const pa = lm(frame[a]), pb = lm(frame[b]);
            ctx.beginPath();
            ctx.moveTo(pa.x, pa.y);
            ctx.lineTo(pb.x, pb.y);
            ctx.strokeStyle = t.aura;
            ctx.globalAlpha = alpha;
            ctx.lineWidth = 12 - fi * 0.8;
            ctx.stroke();
        });
    });
    ctx.globalAlpha = 1;
}

function drawSkeleton(lms) {
    const t = THEMES[theme];

    // Glow bones
    CONNECTIONS.forEach(([a, b]) => {
        if (!lms[a] || !lms[b]) return;
        const pa = lm(lms[a]), pb = lm(lms[b]);

        // Outer glow
        ctx.beginPath();
        ctx.moveTo(pa.x, pa.y);
        ctx.lineTo(pb.x, pb.y);
        ctx.strokeStyle = t.bone;
        ctx.shadowBlur = 24;
        ctx.shadowColor = t.bone;
        ctx.lineWidth = 3;
        ctx.globalAlpha = 0.5;
        ctx.stroke();

        // Inner bright line
        ctx.beginPath();
        ctx.moveTo(pa.x, pa.y);
        ctx.lineTo(pb.x, pb.y);
        ctx.strokeStyle = '#ffffff';
        ctx.shadowBlur = 6;
        ctx.shadowColor = t.bone;
        ctx.lineWidth = 1.5;
        ctx.globalAlpha = 0.9;
        ctx.stroke();
    });

    // Draw joints
    lms.forEach((pt, i) => {
        if (!pt) return;
        const p = lm(pt);
        const isKey = KEY_JOINTS.includes(i);
        const r = isKey ? 7 : 4;

        ctx.beginPath();
        ctx.arc(p.x, p.y, r, 0, Math.PI * 2);
        ctx.fillStyle = t.joint;
        ctx.shadowBlur = 30;
        ctx.shadowColor = t.joint;
        ctx.globalAlpha = 1.0;
        ctx.fill();

        // White center
        ctx.beginPath();
        ctx.arc(p.x, p.y, r * 0.4, 0, Math.PI * 2);
        ctx.fillStyle = '#ffffff';
        ctx.shadowBlur = 0;
        ctx.fill();
    });

    ctx.shadowBlur = 0;
    ctx.globalAlpha = 1;
}

function drawBodyAura(lms) {
    if (!lms[11] || !lms[12] || !lms[23] || !lms[24]) return;
    const t = THEMES[theme];

    // Build body silhouette path for aura fill
    const ls  = lm(lms[11]), rs  = lm(lms[12]);
    const lh  = lm(lms[23]), rh  = lm(lms[24]);
    const nose = lm(lms[0]);

    const cx = (ls.x + rs.x + lh.x + rh.x) / 4;
    const cy = (ls.y + rs.y + lh.y + rh.y) / 4;
    const radius = Math.hypot(ls.x - rh.x, ls.y - rh.y) * 0.75;

    // Radial pulse aura
    const pulse = 0.5 + 0.5 * Math.sin(time * 3);
    const grad = ctx.createRadialGradient(cx, cy, 0, cx, cy, radius * (1 + pulse * 0.3));
    grad.addColorStop(0,   `${t.aura}22`);
    grad.addColorStop(0.5, `${t.aura}11`);
    grad.addColorStop(1,   `${t.aura}00`);

    ctx.beginPath();
    ctx.ellipse(cx, cy, radius * (1 + pulse * 0.15), radius * 1.6, 0, 0, Math.PI * 2);
    ctx.fillStyle = grad;
    ctx.globalAlpha = 0.8;
    ctx.fill();
    ctx.globalAlpha = 1;
}

function spawnBodyParticles(lms) {
    const t = THEMES[theme];
    SPARK_JOINTS.forEach(i => {
        if (!lms[i]) return;
        const p = lm(lms[i]);
        if (Math.random() > (0.6 - energy * 0.3)) {
            spawnParticle(p.x, p.y, t.particle(), 3 + energy * 6);
        }
    });

    // Extra burst along bones proportional to energy
    if (energy > 0.015) {
        CONNECTIONS.forEach(([a, b]) => {
            if (!lms[a] || !lms[b] || Math.random() > 0.15) return;
            const pa = lm(lms[a]), pb = lm(lms[b]);
            const tx = pa.x + (pb.x - pa.x) * Math.random();
            const ty = pa.y + (pb.y - pa.y) * Math.random();
            spawnParticle(tx, ty, t.particle(), 2 + energy * 4);
        });
    }
}

function drawParticles() {
    for (let i = particles.length - 1; i >= 0; i--) {
        const p = particles[i];
        p.x  += p.vx;
        p.y  += p.vy;
        p.vy += 0.08;
        p.life -= p.decay;

        if (p.life <= 0) { particles.splice(i, 1); continue; }

        ctx.beginPath();
        ctx.arc(p.x, p.y, p.size * p.life, 0, Math.PI * 2);
        ctx.fillStyle = p.color;
        ctx.globalAlpha = p.life;
        ctx.shadowBlur = 8;
        ctx.shadowColor = p.color;
        ctx.fill();
    }
    ctx.globalAlpha = 1;
    ctx.shadowBlur = 0;
}

function drawChakraNodes(lms) {
    // Draw glowing chakra-style orbs at major body nodes
    const CHAKRAS = [0, 11, 12, 23, 24, 15, 16, 27, 28];
    const t = THEMES[theme];

    CHAKRAS.forEach((i, ci) => {
        if (!lms[i]) return;
        const p = lm(lms[i]);
        const hue = (ci * 40 + time * 60) % 360;
        const color = theme === 'Neon' ? `hsl(${hue},100%,65%)` : t.joint;
        const pulse = 1 + 0.4 * Math.sin(time * 4 + ci * 1.2);

        ctx.beginPath();
        ctx.arc(p.x, p.y, 12 * pulse, 0, Math.PI * 2);
        ctx.strokeStyle = color;
        ctx.lineWidth = 2;
        ctx.shadowBlur = 30;
        ctx.shadowColor = color;
        ctx.globalAlpha = 0.6;
        ctx.stroke();

        ctx.beginPath();
        ctx.arc(p.x, p.y, 5, 0, Math.PI * 2);
        ctx.fillStyle = color;
        ctx.shadowBlur = 20;
        ctx.globalAlpha = 1;
        ctx.fill();
    });
    ctx.shadowBlur = 0;
    ctx.globalAlpha = 1;
}

function drawLightningBetweenHands(lms) {
    if (!lms[15] || !lms[16]) return;
    const lh = lm(lms[15]), rh = lm(lms[16]);
    const dist = Math.hypot(lh.x - rh.x, lh.y - rh.y);
    if (dist > W * 0.45) return; // only when hands are close-ish

    const t = THEMES[theme];
    const steps = 6;
    ctx.beginPath();
    ctx.moveTo(lh.x, lh.y);

    for (let i = 1; i < steps; i++) {
        const tx = lh.x + (rh.x - lh.x) * (i / steps) + (Math.random() - 0.5) * 60;
        const ty = lh.y + (rh.y - lh.y) * (i / steps) + (Math.random() - 0.5) * 60;
        ctx.lineTo(tx, ty);
    }
    ctx.lineTo(rh.x, rh.y);

    ctx.strokeStyle = '#ffffff';
    ctx.lineWidth = 2;
    ctx.shadowBlur = 25;
    ctx.shadowColor = t.joint;
    ctx.globalAlpha = 0.5 + Math.random() * 0.5;
    ctx.stroke();
    ctx.globalAlpha = 1;
    ctx.shadowBlur = 0;
}

// ─── MAIN RENDER LOOP ─────────────────────────────────────────────────────────
function render(timestamp) {
    requestAnimationFrame(render);

    const dt = (timestamp - lastTime) / 1000;
    lastTime = timestamp;
    time += dt;

    fpsFrames++;
    if (timestamp - fpsLast > 1000) {
        fps = fpsFrames;
        fpsFrames = 0;
        fpsLast = timestamp;
        document.getElementById('ui-fps').textContent = fps;
    }

    // Fade canvas
    ctx.globalCompositeOperation = 'destination-out';
    ctx.fillStyle = 'rgba(0,0,0,0.35)';
    ctx.fillRect(0, 0, W, H);
    ctx.globalCompositeOperation = 'source-over';

    const lms = currentLandmarks;

    if (lms) {
        // Trail
        drawAuraTrails(lms);
        // Body aura glow
        ctx.globalCompositeOperation = 'screen';
        drawBodyAura(lms);
        ctx.globalCompositeOperation = 'source-over';
        // Lightning
        drawLightningBetweenHands(lms);
        // Skeleton
        drawSkeleton(lms);
        // Chakra nodes
        drawChakraNodes(lms);
        // Particles
        spawnBodyParticles(lms);

        document.getElementById('ui-body').textContent   = 'TRACKED';
        document.getElementById('ui-pose').textContent   = detectPose(lms);
        document.getElementById('ui-energy').textContent = Math.round(energy * 1000) / 10 + '%';
    } else {
        document.getElementById('ui-body').textContent   = 'NOT DETECTED';
        document.getElementById('ui-pose').textContent   = '—';
        document.getElementById('ui-energy').textContent = '—';
    }

    drawParticles();
}

// ─── MEDIAPIPE SETUP ──────────────────────────────────────────────────────────
function initMediaPipe() {
    const pose = new Pose({
        locateFile: file => `https://cdn.jsdelivr.net/npm/@mediapipe/pose/${file}`
    });

    pose.setOptions({
        modelComplexity: 1,
        smoothLandmarks: true,
        enableSegmentation: false,
        minDetectionConfidence: 0.5,
        minTrackingConfidence: 0.5,
    });

    pose.onResults(results => {
        prevLandmarks    = currentLandmarks;
        currentLandmarks = results.poseLandmarks || null;

        if (currentLandmarks && prevLandmarks) {
            energy = calcEnergy(currentLandmarks, prevLandmarks);
        }

        if (currentLandmarks) {
            // Keep trail
            trailFrames.push([...currentLandmarks]);
            if (trailFrames.length > TRAIL_LEN) trailFrames.shift();
        }
    });

    const camera = new Camera(video, {
        onFrame: async () => { await pose.send({ image: video }); },
        width: 1280, height: 720
    });
    camera.start();
}

// ─── START ────────────────────────────────────────────────────────────────────
document.getElementById('startBtn').addEventListener('click', () => {
    document.getElementById('overlay').classList.add('gone');
    document.getElementById('hud').classList.remove('hidden');
    document.getElementById('themes').classList.remove('hidden');
    initMediaPipe();
    requestAnimationFrame(render);
});
</script>
</body>
</html>
