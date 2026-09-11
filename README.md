# copilot-plans
Plans and documentation for Copilot features
Here’s a complete self-contained game. Save it as `index.html` and open it in any modern desktop or mobile browser.

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta
    name="viewport"
    content="width=device-width,initial-scale=1,maximum-scale=1,viewport-fit=cover"
  >
  <meta name="theme-color" content="#050712">
  <title>Neon Coil</title>

  <style>
    :root {
      --bg: #050712;
      --panel: rgba(8, 13, 29, 0.78);
      --panel-strong: rgba(9, 15, 34, 0.94);
      --cyan: #61f6ff;
      --cyan-soft: #33c6df;
      --pink: #ff4fd8;
      --gold: #ffe66b;
      --text: #edfaff;
      --muted: #7892aa;
      --danger: #ff496f;
      --safe-top: env(safe-area-inset-top, 0px);
      --safe-bottom: env(safe-area-inset-bottom, 0px);
    }

    * {
      box-sizing: border-box;
    }

    html,
    body {
      width: 100%;
      height: 100%;
      margin: 0;
      overflow: hidden;
      overscroll-behavior: none;
      background: var(--bg);
      color: var(--text);
      font-family:
        Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont,
        "Segoe UI", sans-serif;
    }

    body {
      min-height: 100dvh;
      touch-action: none;
      user-select: none;
      -webkit-user-select: none;
      -webkit-tap-highlight-color: transparent;
    }

    button {
      border: 0;
      color: inherit;
      font: inherit;
      cursor: pointer;
      touch-action: manipulation;
      -webkit-tap-highlight-color: transparent;
    }

    button:focus-visible {
      outline: 2px solid var(--cyan);
      outline-offset: 4px;
    }

    [hidden] {
      display: none !important;
    }

    #game {
      position: fixed;
      inset: 0;
      width: 100%;
      height: 100%;
      display: block;
      touch-action: none;
    }

    .scanlines {
      position: fixed;
      inset: 0;
      z-index: 2;
      pointer-events: none;
      opacity: 0.13;
      background:
        repeating-linear-gradient(
          to bottom,
          transparent 0,
          transparent 3px,
          rgba(255, 255, 255, 0.025) 4px
        );
      mix-blend-mode: screen;
    }

    .vignette {
      position: fixed;
      inset: 0;
      z-index: 3;
      pointer-events: none;
      background:
        radial-gradient(circle at center, transparent 42%, rgba(0, 0, 0, 0.48) 120%);
    }

    /* HUD */

    .hud {
      position: fixed;
      top: calc(12px + var(--safe-top));
      left: 16px;
      right: 16px;
      z-index: 10;
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 12px;
      opacity: 0;
      pointer-events: none;
      transform: translateY(-12px);
      transition: opacity 180ms ease, transform 180ms ease;
    }

    .hud.visible {
      opacity: 1;
      pointer-events: auto;
      transform: translateY(0);
    }

    .hud-cluster {
      display: flex;
      align-items: center;
      gap: 8px;
      padding: 7px;
      border: 1px solid rgba(97, 246, 255, 0.15);
      border-radius: 16px;
      background: rgba(5, 10, 24, 0.68);
      box-shadow:
        0 12px 40px rgba(0, 0, 0, 0.28),
        inset 0 1px rgba(255, 255, 255, 0.04);
      backdrop-filter: blur(14px);
      -webkit-backdrop-filter: blur(14px);
    }

    .stat {
      min-width: 82px;
      padding: 3px 10px;
    }

    .stat .label {
      display: block;
      margin-bottom: 2px;
      color: var(--muted);
      font-size: 9px;
      font-weight: 800;
      letter-spacing: 0.17em;
      text-transform: uppercase;
    }

    .stat strong {
      display: block;
      color: var(--text);
      font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace;
      font-size: 17px;
      line-height: 1;
      letter-spacing: 0.06em;
      text-shadow: 0 0 12px rgba(97, 246, 255, 0.32);
    }

    #score.bump {
      animation: score-bump 220ms ease;
    }

    @keyframes score-bump {
      40% {
        color: white;
        transform: scale(1.18);
        text-shadow: 0 0 18px var(--cyan);
      }
    }

    .combo-stat strong {
      color: var(--pink);
      text-shadow: 0 0 14px rgba(255, 79, 216, 0.5);
    }

    .combo-track,
    .energy-track {
      width: 88px;
      height: 3px;
      margin-top: 5px;
      overflow: hidden;
      border-radius: 999px;
      background: rgba(255, 255, 255, 0.09);
    }

    .combo-track i,
    .energy-track i {
      display: block;
      width: 100%;
      height: 100%;
      border-radius: inherit;
      transform-origin: left center;
    }

    .combo-track i {
      background: linear-gradient(90deg, var(--pink), #a763ff);
      box-shadow: 0 0 10px var(--pink);
    }

    .energy-track i {
      background: linear-gradient(90deg, var(--cyan), var(--pink));
      box-shadow: 0 0 10px var(--cyan);
    }

    .icon-btn {
      width: 38px;
      height: 38px;
      display: grid;
      place-items: center;
      border: 1px solid rgba(97, 246, 255, 0.16);
      border-radius: 11px;
      background: rgba(10, 18, 38, 0.8);
      color: var(--cyan);
      font-size: 16px;
      font-weight: 900;
      transition:
        transform 100ms ease,
        border-color 100ms ease,
        background 100ms ease;
    }

    .icon-btn:hover {
      border-color: rgba(97, 246, 255, 0.45);
      background: rgba(18, 33, 61, 0.95);
    }

    .icon-btn:active {
      transform: scale(0.92);
    }

    .icon-btn.muted {
      color: var(--danger);
    }

    /* Screens */

    .overlay {
      position: fixed;
      inset: 0;
      z-index: 20;
      display: grid;
      place-items: center;
      padding:
        calc(18px + var(--safe-top))
        18px
        calc(18px + var(--safe-bottom));
      opacity: 0;
      visibility: hidden;
      pointer-events: none;
      transform: scale(1.018);
      transition:
        opacity 180ms ease,
        visibility 180ms ease,
        transform 180ms ease;
    }

    .overlay.active {
      opacity: 1;
      visibility: visible;
      pointer-events: auto;
      transform: scale(1);
    }

    .card {
      width: min(460px, 100%);
      max-height: calc(100dvh - 36px - var(--safe-top) - var(--safe-bottom));
      overflow: auto;
      padding: 30px;
      border: 1px solid rgba(97, 246, 255, 0.19);
      border-radius: 25px;
      background:
        linear-gradient(145deg, rgba(12, 21, 45, 0.94), rgba(6, 9, 23, 0.9));
      box-shadow:
        0 30px 90px rgba(0, 0, 0, 0.54),
        0 0 60px rgba(22, 165, 190, 0.08),
        inset 0 1px rgba(255, 255, 255, 0.06);
      text-align: center;
      backdrop-filter: blur(20px);
      -webkit-backdrop-filter: blur(20px);
      scrollbar-width: none;
    }

    .card::-webkit-scrollbar {
      display: none;
    }

    .eyebrow {
      margin: 0 0 10px;
      color: var(--cyan);
      font-family: ui-monospace, SFMono-Regular, Menlo, monospace;
      font-size: 10px;
      font-weight: 900;
      letter-spacing: 0.28em;
      text-transform: uppercase;
    }

    .logo {
      position: relative;
      margin: 0;
      font-size: clamp(42px, 12vw, 72px);
      font-weight: 950;
      line-height: 0.92;
      letter-spacing: -0.065em;
      text-transform: uppercase;
      background: linear-gradient(110deg, white 5%, var(--cyan) 48%, var(--pink));
      color: transparent;
      -webkit-background-clip: text;
      background-clip: text;
      filter: drop-shadow(0 0 20px rgba(97, 246, 255, 0.25));
    }

    .logo::after {
      content: "NEON COIL";
      position: absolute;
      inset: 0;
      z-index: -1;
      color: var(--pink);
      opacity: 0.18;
      transform: translate(3px, 2px);
      clip-path: inset(52% 0 23% 0);
    }

    h2 {
      margin: 0;
      font-size: clamp(27px, 8vw, 40px);
      line-height: 1;
      letter-spacing: -0.04em;
    }

    .subtitle {
      max-width: 360px;
      margin: 16px auto 20px;
      color: #a9bdcb;
      font-size: 14px;
      line-height: 1.55;
    }

    .control-grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 8px;
      margin: 20px 0;
      text-align: left;
    }

    .control-pill {
      padding: 11px 12px;
      border: 1px solid rgba(255, 255, 255, 0.07);
      border-radius: 13px;
      background: rgba(255, 255, 255, 0.035);
    }

    .control-pill b {
      display: block;
      margin-bottom: 3px;
      color: white;
      font-size: 11px;
      letter-spacing: 0.04em;
    }

    .control-pill span {
      color: var(--muted);
      font-size: 10px;
    }

    .primary-btn,
    .secondary-btn {
      width: 100%;
      min-height: 50px;
      border-radius: 14px;
      font-size: 12px;
      font-weight: 900;
      letter-spacing: 0.15em;
      text-transform: uppercase;
      transition:
        transform 100ms ease,
        filter 100ms ease,
        border-color 100ms ease;
    }

    .primary-btn {
      position: relative;
      overflow: hidden;
      background: linear-gradient(105deg, var(--cyan), #79d8ff 48%, var(--pink));
      color: #031018;
      box-shadow:
        0 12px 30px rgba(41, 215, 234, 0.18),
        0 0 30px rgba(97, 246, 255, 0.12);
    }

    .primary-btn::after {
      content: "";
      position: absolute;
      top: -100%;
      left: -30%;
      width: 28%;
      height: 300%;
      background: rgba(255, 255, 255, 0.5);
      transform: rotate(25deg);
      animation: sheen 3.5s infinite;
    }

    @keyframes sheen {
      0%, 68% { left: -35%; }
      85%, 100% { left: 130%; }
    }

    .secondary-btn {
      margin-top: 9px;
      border: 1px solid rgba(97, 246, 255, 0.18);
      background: rgba(97, 246, 255, 0.055);
      color: var(--cyan);
    }

    .primary-btn:hover,
    .secondary-btn:hover {
      filter: brightness(1.1);
    }

    .primary-btn:active,
    .secondary-btn:active {
      transform: scale(0.975);
    }

    .key-hint {
      margin: 10px 0 0;
      color: #5d7388;
      font-family: ui-monospace, SFMono-Regular, Menlo, monospace;
      font-size: 9px;
      letter-spacing: 0.08em;
      text-transform: uppercase;
    }

    .scores {
      margin-top: 22px;
      padding-top: 17px;
      border-top: 1px solid rgba(255, 255, 255, 0.07);
      text-align: left;
    }

    .scores h3 {
      margin: 0 0 9px;
      color: #70889d;
      font-size: 9px;
      letter-spacing: 0.18em;
      text-transform: uppercase;
    }

    .score-row {
      display: grid;
      grid-template-columns: 26px 1fr auto;
      align-items: center;
      min-height: 28px;
      border-bottom: 1px solid rgba(255, 255, 255, 0.035);
      font-family: ui-monospace, SFMono-Regular, Menlo, monospace;
      font-size: 11px;
    }

    .score-row:last-child {
      border-bottom: 0;
    }

    .score-rank {
      color: #536a7d;
    }

    .score-value {
      color: white;
      font-weight: 800;
    }

    .score-combo {
      color: var(--pink);
    }

    .final-score {
      margin: 14px 0 0;
      color: white;
      font-family: ui-monospace, SFMono-Regular, Menlo, monospace;
      font-size: clamp(42px, 13vw, 68px);
      font-weight: 950;
      line-height: 1;
      letter-spacing: -0.05em;
      text-shadow: 0 0 28px rgba(97, 246, 255, 0.32);
    }

    .new-best {
      display: inline-block;
      margin: 11px 0 0;
      padding: 6px 10px;
      border: 1px solid rgba(255, 230, 107, 0.26);
      border-radius: 999px;
      background: rgba(255, 230, 107, 0.08);
      color: var(--gold);
      font-size: 9px;
      font-weight: 900;
      letter-spacing: 0.17em;
      text-transform: uppercase;
      animation: best-pulse 900ms ease-in-out infinite alternate;
    }

    @keyframes best-pulse {
      to {
        box-shadow: 0 0 18px rgba(255, 230, 107, 0.18);
        transform: scale(1.025);
      }
    }

    .run-stats {
      margin: 13px 0 20px;
      color: #879caf;
      font-family: ui-monospace, SFMono-Regular, Menlo, monospace;
      font-size: 10px;
      line-height: 1.7;
    }

    /* Touch controls */

    .touch-controls {
      display: none;
      position: fixed;
      inset: 0;
      z-index: 12;
      pointer-events: none;
    }

    .dpad {
      position: absolute;
      left: max(16px, env(safe-area-inset-left, 0px));
      bottom: calc(18px + var(--safe-bottom));
      width: 142px;
      height: 142px;
      pointer-events: none;
    }

    .touch-btn {
      position: absolute;
      width: 50px;
      height: 50px;
      display: grid;
      place-items: center;
      border: 1px solid rgba(97, 246, 255, 0.2);
      border-radius: 15px;
      background: rgba(9, 18, 38, 0.64);
      color: rgba(213, 253, 255, 0.92);
      font-size: 20px;
      font-weight: 900;
      box-shadow:
        inset 0 1px rgba(255, 255, 255, 0.05),
        0 8px 30px rgba(0, 0, 0, 0.25);
      pointer-events: auto;
      backdrop-filter: blur(8px);
      -webkit-backdrop-filter: blur(8px);
    }

    .touch-btn:active {
      transform: scale(0.9);
      border-color: var(--cyan);
      background: rgba(25, 74, 92, 0.75);
    }

    .touch-up    { top: 0; left: 46px; }
    .touch-left  { top: 46px; left: 0; }
    .touch-right { top: 46px; right: 0; }
    .touch-down  { bottom: 0; left: 46px; }

    .boost-btn {
      position: absolute;
      right: max(18px, env(safe-area-inset-right, 0px));
      bottom: calc(30px + var(--safe-bottom));
      width: 92px;
      height: 92px;
      border: 1px solid rgba(255, 79, 216, 0.28);
      border-radius: 50%;
      background:
        radial-gradient(circle, rgba(255, 79, 216, 0.17), rgba(10, 15, 35, 0.72) 67%);
      color: var(--pink);
      font-size: 10px;
      font-weight: 950;
      letter-spacing: 0.12em;
      text-transform: uppercase;
      box-shadow:
        0 0 30px rgba(255, 79, 216, 0.09),
        inset 0 0 25px rgba(255, 79, 216, 0.06);
      pointer-events: auto;
    }

    .boost-btn::before {
      content: "";
      position: absolute;
      inset: 8px;
      border: 1px dashed rgba(255, 79, 216, 0.35);
      border-radius: inherit;
      animation: boost-spin 7s linear infinite;
    }

    @keyframes boost-spin {
      to { transform: rotate(360deg); }
    }

    .boost-btn.active {
      background:
        radial-gradient(circle, rgba(255, 255, 255, 0.9), var(--pink) 25%, rgba(255, 79, 216, 0.25) 68%);
      color: #1a0520;
      box-shadow:
        0 0 45px rgba(255, 79, 216, 0.65),
        0 0 80px rgba(97, 246, 255, 0.16);
      transform: scale(0.94);
    }

    @media (pointer: coarse) {
      .touch-controls.ingame {
        display: block;
      }

      .key-hint {
        display: none;
      }
    }

    @media (pointer: coarse) and (orientation: landscape) {
      .dpad {
        bottom: 50%;
        transform: translateY(50%);
      }

      .boost-btn {
        bottom: 50%;
        transform: translateY(50%);
      }

      .boost-btn.active {
        transform: translateY(50%) scale(0.94);
      }
    }

    @media (max-width: 580px) {
      .hud {
        top: calc(7px + var(--safe-top));
        left: 8px;
        right: 8px;
        gap: 5px;
      }

      .hud-cluster {
        gap: 2px;
        padding: 5px;
        border-radius: 13px;
      }

      .stat {
        min-width: 0;
        padding: 3px 7px;
      }

      .stat .label {
        font-size: 7px;
      }

      .stat strong {
        font-size: 14px;
      }

      .level-stat {
        display: none;
      }

      .combo-track,
      .energy-track {
        width: 60px;
      }

      .icon-btn {
        width: 34px;
        height: 34px;
      }

      .card {
        padding: 24px 20px;
        border-radius: 21px;
      }

      .control-grid {
        margin: 16px 0;
      }
    }

    @media (max-height: 690px) and (orientation: portrait) {
      .card {
        padding-top: 20px;
        padding-bottom: 20px;
      }

      .subtitle {
        margin: 10px auto 13px;
      }

      .control-grid {
        margin: 12px 0;
      }

      .scores {
        margin-top: 14px;
        padding-top: 12px;
      }

      .score-row {
        min-height: 23px;
      }
    }

    @media (prefers-reduced-motion: reduce) {
      .primary-btn::after,
      .boost-btn::before,
      .new-best {
        animation: none;
      }

      .overlay,
      .hud,
      button {
        transition-duration: 0ms !important;
      }
    }
  </style>
</head>

<body>
  <canvas id="game" aria-label="Neon Coil game canvas">
    Your browser does not support canvas.
  </canvas>

  <div class="scanlines"></div>
  <div class="vignette"></div>

  <header id="hud" class="hud" aria-label="Game status">
    <div class="hud-cluster">
      <div class="stat">
        <span class="label">Score</span>
        <strong id="score">000000</strong>
      </div>

      <div class="stat combo-stat">
        <span class="label">Combo</span>
        <strong id="combo">×1</strong>
        <div class="combo-track"><i id="comboBar"></i></div>
      </div>

      <div class="stat">
        <span class="label">Overdrive</span>
        <strong id="energyText">75%</strong>
        <div class="energy-track"><i id="energyBar"></i></div>
      </div>

      <div class="stat level-stat">
        <span class="label">Level</span>
        <strong id="level">01</strong>
      </div>
    </div>

    <div class="hud-cluster">
      <button id="muteBtn" class="icon-btn" aria-label="Toggle sound" title="Toggle sound">♪</button>
      <button id="pauseBtn" class="icon-btn" aria-label="Pause game" title="Pause">Ⅱ</button>
    </div>
  </header>

  <section id="startScreen" class="overlay active">
    <div class="card">
      <p class="eyebrow">Arcade transmission // 24×24</p>
      <h1 class="logo">Neon Coil</h1>

      <p class="subtitle">
        Collect signal cores, grow your coil, chain combos and push overdrive.
        The arena wraps—but your own trail and armed mines are lethal.
      </p>

      <div class="control-grid">
        <div class="control-pill">
          <b>Move</b>
          <span>Arrow keys, WASD, swipe or D-pad</span>
        </div>
        <div class="control-pill">
          <b>Overdrive</b>
          <span>Hold Space, Shift or BOOST</span>
        </div>
      </div>

      <button id="startBtn" class="primary-btn">Initialize Run</button>
      <p class="key-hint">Press Enter or Space</p>

      <div class="scores">
        <h3>Local high signals</h3>
        <div id="startScores"></div>
      </div>
    </div>
  </section>

  <section id="pauseScreen" class="overlay">
    <div class="card">
      <p class="eyebrow">Signal suspended</p>
      <h2>Paused</h2>
      <p class="subtitle">Your coil is frozen in phase.</p>
      <button id="resumeBtn" class="primary-btn">Resume</button>
      <button id="quitBtn" class="secondary-btn">Return to Menu</button>
      <p class="key-hint">Press P or Escape</p>
    </div>
  </section>

  <section id="gameOverScreen" class="overlay">
    <div class="card">
      <p id="overEyebrow" class="eyebrow">Connection terminated</p>
      <h2 id="overTitle">Coil Severed</h2>
      <div id="finalScore" class="final-score">0</div>
      <div id="newBest" class="new-best" hidden>New high signal</div>
      <div id="runStats" class="run-stats"></div>

      <button id="restartBtn" class="primary-btn">Instant Restart</button>
      <button id="menuBtn" class="secondary-btn">Return to Menu</button>
      <p class="key-hint">Press R, Enter or Space</p>

      <div class="scores">
        <h3>Local high signals</h3>
        <div id="overScores"></div>
      </div>
    </div>
  </section>

  <div id="touchControls" class="touch-controls" aria-label="Touch controls">
    <div class="dpad">
      <button class="touch-btn touch-up" data-dir="up" aria-label="Move up">▲</button>
      <button class="touch-btn touch-left" data-dir="left" aria-label="Move left">◀</button>
      <button class="touch-btn touch-right" data-dir="right" aria-label="Move right">▶</button>
      <button class="touch-btn touch-down" data-dir="down" aria-label="Move down">▼</button>
    </div>

    <button id="boostBtn" class="boost-btn" aria-label="Hold for overdrive">
      Boost
    </button>
  </div>

  <script>
    (() => {
      "use strict";

      const $ = (selector) => document.querySelector(selector);

      const canvas = $("#game");
      const ctx = canvas.getContext("2d", {
        alpha: false,
        desynchronized: true
      });

      const hud = $("#hud");
      const scoreEl = $("#score");
      const comboEl = $("#combo");
      const comboBar = $("#comboBar");
      const energyBar = $("#energyBar");
      const energyText = $("#energyText");
      const levelEl = $("#level");
      const pauseBtn = $("#pauseBtn");
      const muteBtn = $("#muteBtn");

      const startScreen = $("#startScreen");
      const pauseScreen = $("#pauseScreen");
      const gameOverScreen = $("#gameOverScreen");

      const startBtn = $("#startBtn");
      const resumeBtn = $("#resumeBtn");
      const quitBtn = $("#quitBtn");
      const restartBtn = $("#restartBtn");
      const menuBtn = $("#menuBtn");

      const overEyebrow = $("#overEyebrow");
      const overTitle = $("#overTitle");
      const finalScore = $("#finalScore");
      const newBest = $("#newBest");
      const runStats = $("#runStats");

      const touchControls = $("#touchControls");
      const boostBtn = $("#boostBtn");

      const GRID = 24;
      const COMBO_WINDOW = 3.2;
      const MAX_PARTICLES = 420;
      const SCORE_KEY = "neonCoilScoresV1";

      const COLORS = {
        cyan: "#61f6ff",
        cyanSoft: "#33c6df",
        pink: "#ff4fd8",
        gold: "#ffe66b",
        danger: "#ff496f",
        white: "#f5feff"
      };

      const view = {
        w: innerWidth,
        h: innerHeight,
        dpr: 1,
        boardX: 0,
        boardY: 0,
        side: 400,
        cell: 400 / GRID
      };

      let state = "menu";
      let snake = [];
      let prevSnake = [];
      let direction = { x: 1, y: 0 };
      let inputQueue = [];
      let food = null;
      let mines = [];

      let score = 0;
      let displayScore = 0;
      let combo = 1;
      let maxCombo = 1;
      let comboRemaining = 0;
      let lastEatTime = -Infinity;
      let foodCount = 0;
      let level = 1;
      let baseInterval = 0.125;

      let boostEnergy = 75;
      let isBoosting = false;
      let keyboardBoost = false;
      let pointerBoost = false;
      let boostEmitTimer = 0;

      let accumulator = 0;
      let simTime = 0;
      let globalTime = 0;
      let lastFrame = performance.now();

      let shake = 0;
      let flash = 0;
      let particles = [];
      let popups = [];
      let stars = [];
      let swipe = null;
      let overTimer = 0;
      let scores = loadScores();

      const mod = (value, n) => ((value % n) + n) % n;
      const clamp = (value, min, max) => Math.max(min, Math.min(max, value));
      const random = (min, max) => min + Math.random() * (max - min);

      function roundedRectPath(context, x, y, w, h, radius) {
        const r = Math.min(radius, w / 2, h / 2);
        context.beginPath();
        context.moveTo(x + r, y);
        context.arcTo(x + w, y, x + w, y + h, r);
        context.arcTo(x + w, y + h, x, y + h, r);
        context.arcTo(x, y + h, x, y, r);
        context.arcTo(x, y, x + w, y, r);
        context.closePath();
      }

      function torusDistance(ax, ay, bx, by) {
        const dx = Math.min(Math.abs(ax - bx), GRID - Math.abs(ax - bx));
        const dy = Math.min(Math.abs(ay - by), GRID - Math.abs(ay - by));
        return dx + dy;
      }

      function lerpWrapped(a, b, t) {
        let delta = b - a;

        if (Math.abs(delta) > GRID / 2) {
          delta -= Math.sign(delta) * GRID;
        }

        return a + delta * t;
      }

      function formatScore(value) {
        return Math.max(0, Math.floor(value)).toLocaleString("en-US");
      }

      function resize() {
        const coarse =
          matchMedia("(pointer: coarse)").matches ||
          Math.min(innerWidth, innerHeight) < 600;

        view.w = innerWidth;
        view.h = innerHeight;
        view.dpr = Math.min(devicePixelRatio || 1, 2);

        canvas.width = Math.round(view.w * view.dpr);
        canvas.height = Math.round(view.h * view.dpr);
        canvas.style.width = view.w + "px";
        canvas.style.height = view.h + "px";

        ctx.setTransform(view.dpr, 0, 0, view.dpr, 0, 0);

        if (coarse && view.h >= view.w) {
          view.side = Math.min(view.w - 20, view.h - 250, 720);
          view.side = Math.max(260, view.side);
          view.boardX = (view.w - view.side) / 2;
          view.boardY = Math.max(
            78,
            (view.h - view.side - 170) / 2 + 50
          );
        } else if (coarse) {
          view.side = Math.min(view.h - 28, view.w - 320, 660);
          view.side = Math.max(250, view.side);
          view.boardX = (view.w - view.side) / 2;
          view.boardY = (view.h - view.side) / 2 + 5;
        } else {
          view.side = Math.min(view.w - 40, view.h - 145, 720);
          view.side = Math.max(300, view.side);
          view.boardX = (view.w - view.side) / 2;
          view.boardY = (view.h - view.side) / 2 + 22;
        }

        view.cell = view.side / GRID;

        const starCount = clamp(
          Math.floor((view.w * view.h) / 14500),
          30,
          110
        );

        stars = Array.from({ length: starCount }, () => ({
          x: Math.random() * view.w,
          y: Math.random() * view.h,
          radius: random(0.35, 1.35),
          phase: random(0, Math.PI * 2),
          speed: random(0.35, 1.1)
        }));
      }

      class SoundEngine {
        constructor() {
          this.context = null;
          this.muted = false;

          try {
            this.muted = localStorage.getItem("neonCoilMuted") === "1";
          } catch (_) {}
        }

        ensure() {
          if (!this.context) {
            const AudioContext =
              window.AudioContext || window.webkitAudioContext;

            if (AudioContext) {
              this.context = new AudioContext();
            }
          }

          if (this.context?.state === "suspended") {
            this.context.resume().catch(() => {});
          }
        }

        tone(
          frequency,
          duration = 0.1,
          type = "sine",
          volume = 0.05,
          slide = 0,
          delay = 0
        ) {
          if (this.muted) return;

          this.ensure();
          if (!this.context) return;

          const now = this.context.currentTime + delay;
          const oscillator = this.context.createOscillator();
          const gain = this.context.createGain();

          oscillator.type = type;
          oscillator.frequency.setValueAtTime(frequency, now);

          if (slide) {
            oscillator.frequency.exponentialRampToValueAtTime(
              Math.max(20, frequency + slide),
              now + duration
            );
          }

          gain.gain.setValueAtTime(0.0001, now);
          gain.gain.exponentialRampToValueAtTime(volume, now + 0.012);
          gain.gain.exponentialRampToValueAtTime(
            0.0001,
            now + duration
          );

          oscillator.connect(gain);
          gain.connect(this.context.destination);
          oscillator.start(now);
          oscillator.stop(now + duration + 0.03);
        }

        pickup(currentCombo, prism = false) {
          const root = prism
            ? 620
            : 360 + Math.min(currentCombo, 8) * 38;

          this.tone(root, 0.1, "triangle", 0.052, 150);
          this.tone(root * 1.5, 0.12, "sine", 0.025, 120, 0.035);
        }

        start() {
          this.tone(240, 0.14, "triangle", 0.04, 160);
          this.tone(360, 0.17, "triangle", 0.035, 180, 0.06);
          this.tone(540, 0.2, "sine", 0.03, 150, 0.12);
        }

        boost() {
          this.tone(120, 0.17, "sawtooth", 0.032, 330);
          this.tone(520, 0.08, "sine", 0.022, 120, 0.08);
        }

        crash() {
          if (this.muted) return;

          this.ensure();
          if (!this.context) return;

          const duration = 0.42;
          const length = Math.floor(
            this.context.sampleRate * duration
          );
          const buffer = this.context.createBuffer(
            1,
            length,
            this.context.sampleRate
          );
          const data = buffer.getChannelData(0);

          for (let i = 0; i < length; i++) {
            const falloff = 1 - i / length;
            data[i] =
              (Math.random() * 2 - 1) *
              falloff *
              falloff;
          }

          const source = this.context.createBufferSource();
          const filter = this.context.createBiquadFilter();
          const gain = this.context.createGain();
          const now = this.context.currentTime;

          source.buffer = buffer;
          filter.type = "lowpass";
          filter.frequency.setValueAtTime(1200, now);
          filter.frequency.exponentialRampToValueAtTime(
            90,
            now + duration
          );

          gain.gain.setValueAtTime(0.12, now);
          gain.gain.exponentialRampToValueAtTime(
            0.0001,
            now + duration
          );

          source.connect(filter);
          filter.connect(gain);
          gain.connect(this.context.destination);
          source.start(now);

          this.tone(150, 0.36, "sawtooth", 0.07, -95);
        }

        toggle() {
          this.muted = !this.muted;

          try {
            localStorage.setItem(
              "neonCoilMuted",
              this.muted ? "1" : "0"
            );
          } catch (_) {}

          if (!this.muted) {
            this.tone(520, 0.08, "sine", 0.035, 100);
          }

          updateMuteButton();
        }
      }

      const sfx = new SoundEngine();

      function updateMuteButton() {
        muteBtn.classList.toggle("muted", sfx.muted);
        muteBtn.textContent = sfx.muted ? "×♪" : "♪";
        muteBtn.setAttribute(
          "aria-label",
          sfx.muted ? "Enable sound" : "Mute sound"
        );
      }

      function loadScores() {
        try {
          const parsed = JSON.parse(
            localStorage.getItem(SCORE_KEY) || "[]"
          );

          if (!Array.isArray(parsed)) return [];

          return parsed
            .map((entry) => ({
              score: Math.max(0, Number(entry.score) || 0),
              combo: Math.max(1, Number(entry.combo) || 1),
              cores: Math.max(0, Number(entry.cores) || 0),
              date: Number(entry.date) || Date.now()
            }))
            .sort((a, b) => b.score - a.score || b.combo - a.combo)
            .slice(0, 5);
        } catch (_) {
          return [];
        }
      }

      function saveCurrentScore() {
        if (score <= 0) return false;

        const oldBest = scores[0]?.score || 0;
        const isNewBest = score > oldBest;

        scores.push({
          score,
          combo: maxCombo,
          cores: foodCount,
          date: Date.now()
        });

        scores.sort(
          (a, b) => b.score - a.score || b.combo - a.combo
        );
        scores = scores.slice(0, 5);

        try {
          localStorage.setItem(SCORE_KEY, JSON.stringify(scores));
        } catch (_) {}

        return isNewBest;
      }

      function renderScoreTable(container) {
        if (!scores.length) {
          container.innerHTML = Array.from(
            { length: 5 },
            (_, index) => `
              <div class="score-row">
                <span class="score-rank">${index + 1}</span>
                <span class="score-value">—</span>
                <span class="score-combo">×—</span>
              </div>
            `
          ).join("");
          return;
        }

        container.innerHTML = Array.from(
          { length: 5 },
          (_, index) => {
            const entry = scores[index];

            if (!entry) {
              return `
                <div class="score-row">
                  <span class="score-rank">${index + 1}</span>
                  <span class="score-value">—</span>
                  <span class="score-combo">×—</span>
                </div>
              `;
            }

            return `
              <div class="score-row">
                <span class="score-rank">${index + 1}</span>
                <span class="score-value">${formatScore(entry.score)}</span>
                <span class="score-combo">×${entry.combo}</span>
              </div>
            `;
          }
        ).join("");
      }

      function renderAllScores() {
        renderScoreTable($("#startScores"));
        renderScoreTable($("#overScores"));
      }

      function hideScreens() {
        startScreen.classList.remove("active");
        pauseScreen.classList.remove("active");
        gameOverScreen.classList.remove("active");
      }

      function resetGameData() {
        snake = [
          { x: 10, y: 12 },
          { x: 9, y: 12 },
          { x: 8, y: 12 },
          { x: 7, y: 12 },
          { x: 6, y: 12 },
          { x: 5, y: 12 }
        ];

        prevSnake = snake.map((segment) => ({ ...segment }));
        direction = { x: 1, y: 0 };
        inputQueue = [];

        food = {
          x: 15,
          y: 12,
          type: "core",
          born: 0
        };

        mines = [];
        particles = [];
        popups = [];

        score = 0;
        displayScore = 0;
        combo = 1;
        maxCombo = 1;
        comboRemaining = 0;
        lastEatTime = -Infinity;
        foodCount = 0;
        level = 1;
        baseInterval = 0.125;

        boostEnergy = 75;
        isBoosting = false;
        keyboardBoost = false;
        pointerBoost = false;
        boostEmitTimer = 0;

        accumulator = 0;
        simTime = 0;
        shake = 0;
        flash = 0;
      }

      function startGame() {
        clearTimeout(overTimer);
        sfx.ensure();
        sfx.start();

        resetGameData();
        state = "playing";
        lastFrame = performance.now();

        hideScreens();
        hud.classList.add("visible");
        touchControls.classList.add("ingame");
        pauseBtn.textContent = "Ⅱ";

        updateHud();
      }

      function pauseGame() {
        if (state !== "playing") return;

        state = "paused";
        keyboardBoost = false;
        pointerBoost = false;
        isBoosting = false;

        hideScreens();
        pauseScreen.classList.add("active");
        touchControls.classList.remove("ingame");
        pauseBtn.textContent = "▶";
      }

      function resumeGame() {
        if (state !== "paused") return;

        state = "playing";
        lastFrame = performance.now();

        hideScreens();
        touchControls.classList.add("ingame");
        pauseBtn.textContent = "Ⅱ";
      }

      function goToMenu() {
        clearTimeout(overTimer);

        state = "menu";
        keyboardBoost = false;
        pointerBoost = false;
        isBoosting = false;

        hideScreens();
        startScreen.classList.add("active");
        hud.classList.remove("visible");
        touchControls.classList.remove("ingame");

        renderAllScores();
      }

      function currentTickInterval() {
        return baseInterval * (isBoosting ? 0.58 : 1);
      }

      function isCellOccupied(x, y) {
        if (snake.some((segment) => segment.x === x && segment.y === y)) {
          return true;
        }

        if (mines.some((mine) => mine.x === x && mine.y === y)) {
          return true;
        }

        return false;
      }

      function createFood() {
        const head = snake[0];
        let fallback = null;

        for (let attempt = 0; attempt < 180; attempt++) {
          const x = Math.floor(Math.random() * GRID);
          const y = Math.floor(Math.random() * GRID);

          if (isCellOccupied(x, y)) continue;

          const distance = torusDistance(
            head.x,
            head.y,
            x,
            y
          );

          fallback = { x, y };

          if (distance >= 4 && distance <= 11) {
            return {
              x,
              y,
              type: (foodCount + 1) % 5 === 0 ? "prism" : "core",
              born: simTime
            };
          }
        }

        if (!fallback) {
          outer:
          for (let y = 0; y < GRID; y++) {
            for (let x = 0; x < GRID; x++) {
              if (!isCellOccupied(x, y)) {
                fallback = { x, y };
                break outer;
              }
            }
          }
        }

        return {
          ...(fallback || { x: 2, y: 2 }),
          type: (foodCount + 1) % 5 === 0 ? "prism" : "core",
          born: simTime
        };
      }

      function spawnMine() {
        const maxMines = Math.min(5 + level, 11);
        if (mines.length >= maxMines) return;

        const head = snake[0];

        for (let attempt = 0; attempt < 160; attempt++) {
          const x = Math.floor(Math.random() * GRID);
          const y = Math.floor(Math.random() * GRID);

          if (isCellOccupied(x, y)) continue;
          if (food && food.x === x && food.y === y) continue;
          if (torusDistance(head.x, head.y, x, y) < 5) continue;

          mines.push({
            x,
            y,
            age: 0,
            life: random(21, 28),
            phase: random(0, Math.PI * 2)
          });

          addBurst(
            x + 0.5,
            y + 0.5,
            COLORS.danger,
            10,
            2.8
          );
          return;
        }
      }

      function setDirection(x, y) {
        if (state !== "playing") return;

        const last =
          inputQueue[inputQueue.length - 1] || direction;

        if (last.x === x && last.y === y) return;
        if (last.x + x === 0 && last.y + y === 0) return;

        if (inputQueue.length < 3) {
          inputQueue.push({ x, y });
        }
      }

      function addParticle(particle) {
        if (particles.length >= MAX_PARTICLES) {
          particles.splice(
            0,
            particles.length - MAX_PARTICLES + 1
          );
        }

        particles.push(particle);
      }

      function addBurst(x, y, color, count = 18, speed = 6) {
        for (let i = 0; i < count; i++) {
          const angle = Math.random() * Math.PI * 2;
          const velocity = random(speed * 0.35, speed);
          const life = random(0.3, 0.72);

          addParticle({
            x,
            y,
            vx: Math.cos(angle) * velocity,
            vy: Math.sin(angle) * velocity,
            size: random(0.055, 0.16),
            life,
            maxLife: life,
            color,
            drag: random(1.7, 3.6),
            rot: random(0, Math.PI * 2),
            spin: random(-9, 9)
          });
        }
      }

      function addPopup(x, y, text, color) {
        popups.push({
          x,
          y,
          text,
          color,
          life: 0.9,
          maxLife: 0.9
        });
      }

      function bumpScoreHud() {
        scoreEl.classList.remove("bump");
        void scoreEl.offsetWidth;
        scoreEl.classList.add("bump");
      }

      function collectFood() {
        const prism = food.type === "prism";
        const chained = simTime - lastEatTime < COMBO_WINDOW;

        combo = chained ? Math.min(combo + 1, 9) : 1;
        maxCombo = Math.max(maxCombo, combo);
        comboRemaining = COMBO_WINDOW;
        lastEatTime = simTime;
        foodCount++;

        const basePoints = prism ? 250 : 100;
        const boostMultiplier = isBoosting ? 1.5 : 1;
        const gained = Math.round(
          basePoints * combo * boostMultiplier
        );

        score += gained;
        boostEnergy = Math.min(
          100,
          boostEnergy + (prism ? 48 : 26)
        );

        level = 1 + Math.floor(foodCount / 5);
        baseInterval = Math.max(
          0.078,
          0.125 - (level - 1) * 0.0055
        );

        addBurst(
          food.x + 0.5,
          food.y + 0.5,
          prism ? COLORS.gold : COLORS.cyan,
          prism ? 30 : 20,
          prism ? 8 : 6
        );

        if (prism) {
          addBurst(
            food.x + 0.5,
            food.y + 0.5,
            COLORS.pink,
            16,
            5
          );
        }

        addPopup(
          food.x + 0.5,
          food.y + 0.3,
          `+${gained}`,
          prism ? COLORS.gold : COLORS.white
        );

        shake = Math.max(shake, prism ? 10 : 5);
        flash = prism ? 0.38 : 0.2;

        sfx.pickup(combo, prism);
        bumpScoreHud();

        if (navigator.vibrate) {
          navigator.vibrate(prism ? [18, 20, 28] : 14);
        }

        if (foodCount % 3 === 0) {
          spawnMine();
        }

        food = createFood();
      }

      function endGame(reason, point) {
        if (state !== "playing") return;

        state = "over";
        keyboardBoost = false;
        pointerBoost = false;
        isBoosting = false;

        touchControls.classList.remove("ingame");
        hideScreens();

        shake = 24;
        flash = 0.55;

        addBurst(
          point.x + 0.5,
          point.y + 0.5,
          COLORS.danger,
          52,
          11
        );
        addBurst(
          point.x + 0.5,
          point.y + 0.5,
          COLORS.cyan,
          34,
          8
        );

        sfx.crash();

        if (navigator.vibrate) {
          navigator.vibrate([45, 35, 90]);
        }

        const isBest = saveCurrentScore();
        renderAllScores();

        overEyebrow.textContent =
          reason === "mine"
            ? "Mine collision // signal lost"
            : "Feedback loop // coil collision";

        overTitle.textContent = isBest
          ? "New High Signal"
          : "Coil Severed";

        finalScore.textContent = formatScore(score);
        newBest.hidden = !isBest;

        runStats.innerHTML =
          `${foodCount} CORES &nbsp;•&nbsp; ` +
          `MAX COMBO ×${maxCombo}<br>` +
          `LEVEL ${level} &nbsp;•&nbsp; ` +
          `${snake.length} SEGMENTS`;

        overTimer = setTimeout(() => {
          if (state === "over") {
            gameOverScreen.classList.add("active");
          }
        }, 320);
      }

      function simulationStep() {
        if (inputQueue.length) {
          const next = inputQueue.shift();

          if (
            !(next.x + direction.x === 0 &&
              next.y + direction.y === 0)
          ) {
            direction = next;
          }
        }

        prevSnake = snake.map((segment) => ({ ...segment }));

        const nextHead = {
          x: mod(snake[0].x + direction.x, GRID),
          y: mod(snake[0].y + direction.y, GRID)
        };

        const eating =
          food &&
          nextHead.x === food.x &&
          nextHead.y === food.y;

        const collisionLength =
          snake.length - (eating ? 0 : 1);

        for (let i = 0; i < collisionLength; i++) {
          if (
            snake[i].x === nextHead.x &&
            snake[i].y === nextHead.y
          ) {
            endGame("self", nextHead);
            return;
          }
        }

        const hitMine = mines.some(
          (mine) =>
            mine.age >= 0.75 &&
            mine.x === nextHead.x &&
            mine.y === nextHead.y
        );

        if (hitMine) {
          endGame("mine", nextHead);
          return;
        }

        snake.unshift(nextHead);

        if (eating) {
          collectFood();
        } else {
          snake.pop();
        }
      }

      function updateEffects(dt) {
        for (let i = particles.length - 1; i >= 0; i--) {
          const particle = particles[i];
          particle.life -= dt;

          if (particle.life <= 0) {
            particles.splice(i, 1);
            continue;
          }

          particle.x += particle.vx * dt;
          particle.y += particle.vy * dt;

          const drag = Math.exp(-particle.drag * dt);
          particle.vx *= drag;
          particle.vy *= drag;
          particle.rot += particle.spin * dt;
        }

        for (let i = popups.length - 1; i >= 0; i--) {
          const popup = popups[i];
          popup.life -= dt;
          popup.y -= dt * 1.1;

          if (popup.life <= 0) {
            popups.splice(i, 1);
          }
        }

        shake *= Math.exp(-11 * dt);
        flash = Math.max(0, flash - dt * 1.8);
      }

      function updatePlaying(dt) {
        simTime += dt;

        const wantsBoost = keyboardBoost || pointerBoost;

        if (
          !isBoosting &&
          wantsBoost &&
          boostEnergy > 8
        ) {
          isBoosting = true;
          sfx.boost();
        }

        if (
          isBoosting &&
          (!wantsBoost || boostEnergy <= 0)
        ) {
          isBoosting = false;
        }

        if (isBoosting) {
          boostEnergy = Math.max(
            0,
            boostEnergy - 34 * dt
          );

          boostEmitTimer += dt;

          while (boostEmitTimer >= 0.028) {
            boostEmitTimer -= 0.028;

            const head = snake[0];
            const life = random(0.18, 0.38);

            addParticle({
              x: head.x + 0.5 - direction.x * 0.36,
              y: head.y + 0.5 - direction.y * 0.36,
              vx:
                -direction.x * random(3.5, 6) +
                random(-1.5, 1.5),
              vy:
                -direction.y * random(3.5, 6) +
                random(-1.5, 1.5),
              size: random(0.06, 0.14),
              life,
              maxLife: life,
              color:
                Math.random() > 0.45
                  ? COLORS.pink
                  : COLORS.cyan,
              drag: 4.5,
              rot: random(0, Math.PI),
              spin: random(-8, 8)
            });
          }
        } else {
          boostEnergy = Math.min(
            100,
            boostEnergy + 5.5 * dt
          );
          boostEmitTimer = 0;
        }

        if (comboRemaining > 0) {
          comboRemaining = Math.max(
            0,
            comboRemaining - dt
          );

          if (comboRemaining === 0) {
            combo = 1;
          }
        }

        for (const mine of mines) {
          mine.age += dt;
        }

        mines = mines.filter(
          (mine) => mine.age < mine.life
        );

        accumulator += dt;

        let safety = 0;

        while (
          accumulator >= currentTickInterval() &&
          safety < 12
        ) {
          accumulator -= currentTickInterval();
          simulationStep();
          safety++;

          if (state !== "playing") break;
        }
      }

      function updateHud() {
        displayScore +=
          (score - displayScore) * 0.18;

        if (Math.abs(score - displayScore) < 0.5) {
          displayScore = score;
        }

        scoreEl.textContent = Math.floor(displayScore)
          .toString()
          .padStart(6, "0");

        comboEl.textContent = `×${combo}`;
        comboBar.style.transform =
          `scaleX(${clamp(comboRemaining / COMBO_WINDOW, 0, 1)})`;

        energyBar.style.transform =
          `scaleX(${boostEnergy / 100})`;

        energyText.textContent =
          `${Math.round(boostEnergy)}%`;

        levelEl.textContent =
          String(level).padStart(2, "0");

        boostBtn.classList.toggle("active", isBoosting);
      }

      function drawBackground() {
        const gradient = ctx.createRadialGradient(
          view.w * 0.5,
          view.h * 0.44,
          20,
          view.w * 0.5,
          view.h * 0.5,
          Math.max(view.w, view.h) * 0.78
        );

        gradient.addColorStop(0, "#101a35");
        gradient.addColorStop(0.42, "#090d20");
        gradient.addColorStop(1, "#03050d");

        ctx.fillStyle = gradient;
        ctx.fillRect(0, 0, view.w, view.h);

        ctx.save();

        for (const star of stars) {
          const pulse =
            0.3 +
            Math.sin(
              globalTime * star.speed + star.phase
            ) * 0.22;

          ctx.globalAlpha = pulse;
          ctx.fillStyle = star.radius > 1
            ? COLORS.cyan
            : "#b6d8ed";

          ctx.beginPath();
          ctx.arc(
            star.x,
            star.y,
            star.radius,
            0,
            Math.PI * 2
          );
          ctx.fill();
        }

        ctx.globalAlpha = 0.04;
        ctx.strokeStyle = COLORS.cyan;
        ctx.lineWidth = 1;

        const horizonY = view.h * 0.73;
        const spacing = 70;

        for (
          let x = -view.h;
          x < view.w + view.h;
          x += spacing
        ) {
          ctx.beginPath();
          ctx.moveTo(view.w / 2, horizonY);
          ctx.lineTo(x, view.h);
          ctx.stroke();
        }

        for (let i = 0; i < 7; i++) {
          const t = i / 7;
          const y =
            horizonY +
            Math.pow(t, 1.7) *
              (view.h - horizonY);

          ctx.beginPath();
          ctx.moveTo(0, y);
          ctx.lineTo(view.w, y);
          ctx.stroke();
        }

        ctx.restore();
      }

      function wrappedCopies(x, y, radius, callback) {
        const xs = [x];
        const ys = [y];

        if (x < radius) xs.push(x + view.side);
        if (x > view.side - radius) xs.push(x - view.side);
        if (y < radius) ys.push(y + view.side);
        if (y > view.side - radius) ys.push(y - view.side);

        for (const copyX of xs) {
          for (const copyY of ys) {
            callback(copyX, copyY);
          }
        }
      }

      function drawMines() {
        const cell = view.cell;

        for (const mine of mines) {
          const x = (mine.x + 0.5) * cell;
          const y = (mine.y + 0.5) * cell;
          const armed = mine.age >= 0.75;
          const pulse =
            1 +
            Math.sin(
              globalTime * 6 + mine.phase
            ) * 0.1;

          ctx.save();
          ctx.translate(x, y);
          ctx.scale(pulse, pulse);

          ctx.globalAlpha = armed
            ? 0.96
            : 0.28 + mine.age / 0.75 * 0.45;

          ctx.shadowColor = COLORS.danger;
          ctx.shadowBlur = armed ? cell * 0.7 : 0;
          ctx.fillStyle = armed
            ? COLORS.danger
            : "rgba(255,73,111,0.25)";

          ctx.beginPath();

          const points = 12;

          for (let i = 0; i < points; i++) {
            const angle =
              -Math.PI / 2 +
              i * Math.PI / 6;
            const radius =
              i % 2 === 0
                ? cell * 0.39
                : cell * 0.24;
            const px = Math.cos(angle) * radius;
            const py = Math.sin(angle) * radius;

            if (i === 0) ctx.moveTo(px, py);
            else ctx.lineTo(px, py);
          }

          ctx.closePath();
          ctx.fill();

          ctx.shadowBlur = 0;
          ctx.fillStyle = "#250614";
          ctx.beginPath();
          ctx.arc(0, 0, cell * 0.105, 0, Math.PI * 2);
          ctx.fill();

          if (!armed) {
            ctx.strokeStyle = "rgba(255,255,255,0.75)";
            ctx.lineWidth = Math.max(1, cell * 0.055);
            ctx.beginPath();
            ctx.arc(
              0,
              0,
              cell * 0.48,
              -Math.PI / 2,
              -Math.PI / 2 +
                Math.PI * 2 * clamp(mine.age / 0.75, 0, 1)
            );
            ctx.stroke();
          }

          ctx.restore();
        }
      }

      function drawFood() {
        if (!food) return;

        const cell = view.cell;
        const x = (food.x + 0.5) * cell;
        const y = (food.y + 0.5) * cell;
        const prism = food.type === "prism";
        const color = prism ? COLORS.gold : COLORS.cyan;

        const pulse =
          1 + Math.sin(globalTime * 7) * 0.09;
        const rotation =
          globalTime * (prism ? 2.3 : 1.6);

        ctx.save();
        ctx.translate(x, y);

        ctx.strokeStyle = prism
          ? "rgba(255,230,107,0.45)"
          : "rgba(97,246,255,0.35)";
        ctx.lineWidth = Math.max(1, cell * 0.045);

        ctx.beginPath();
        ctx.arc(
          0,
          0,
          cell *
            (0.57 +
              Math.sin(globalTime * 4) * 0.08),
          0,
          Math.PI * 2
        );
        ctx.stroke();

        ctx.rotate(rotation);
        ctx.scale(pulse, pulse);
        ctx.shadowColor = color;
        ctx.shadowBlur = cell * (prism ? 1.15 : 0.85);
        ctx.fillStyle = color;

        const size = cell * (prism ? 0.34 : 0.29);

        ctx.beginPath();
        ctx.moveTo(0, -size);
        ctx.lineTo(size, 0);
        ctx.lineTo(0, size);
        ctx.lineTo(-size, 0);
        ctx.closePath();
        ctx.fill();

        ctx.shadowBlur = 0;
        ctx.fillStyle = "rgba(255,255,255,0.9)";

        ctx.beginPath();
        ctx.moveTo(0, -size * 0.65);
        ctx.lineTo(size * 0.35, 0);
        ctx.lineTo(0, size * 0.15);
        ctx.lineTo(-size * 0.22, 0);
        ctx.closePath();
        ctx.fill();

        if (prism) {
          ctx.rotate(-rotation * 1.7);
          ctx.strokeStyle = COLORS.pink;
          ctx.globalAlpha = 0.7;
          ctx.strokeRect(
            -cell * 0.43,
            -cell * 0.43,
            cell * 0.86,
            cell * 0.86
          );
        }

        ctx.restore();
      }

      function getSnakePoints(alpha) {
        return snake.map((segment, index) => {
          const previous =
            prevSnake[index] || segment;

          const logicalX = lerpWrapped(
            previous.x,
            segment.x,
            alpha
          );

          const logicalY = lerpWrapped(
            previous.y,
            segment.y,
            alpha
          );

          return {
            x: mod(logicalX + 0.5, GRID) * view.cell,
            y: mod(logicalY + 0.5, GRID) * view.cell
          };
        });
      }

      function drawSnake(alpha) {
        if (!snake.length) return;

        const cell = view.cell;
        const points = getSnakePoints(alpha);

        ctx.save();
        ctx.lineCap = "round";
        ctx.lineJoin = "round";
        ctx.lineWidth = cell * 0.56;
        ctx.strokeStyle = isBoosting
          ? "rgba(255,79,216,0.72)"
          : "rgba(77,225,244,0.62)";
        ctx.shadowColor = isBoosting
          ? COLORS.pink
          : COLORS.cyan;
        ctx.shadowBlur = cell * (isBoosting ? 1.05 : 0.6);

        ctx.beginPath();

        for (let i = points.length - 1; i > 0; i--) {
          const a = points[i];
          const b = points[i - 1];

          let bx = b.x;
          let by = b.y;

          if (Math.abs(bx - a.x) > view.side / 2) {
            bx += bx > a.x ? -view.side : view.side;
          }

          if (Math.abs(by - a.y) > view.side / 2) {
            by += by > a.y ? -view.side : view.side;
          }

          for (const ox of [-view.side, 0, view.side]) {
            for (const oy of [-view.side, 0, view.side]) {
              const ax = a.x + ox;
              const ay = a.y + oy;
              const endX = bx + ox;
              const endY = by + oy;

              if (
                Math.max(ax, endX) < -cell ||
                Math.min(ax, endX) > view.side + cell ||
                Math.max(ay, endY) < -cell ||
                Math.min(ay, endY) > view.side + cell
              ) {
                continue;
              }

              ctx.moveTo(ax, ay);
              ctx.lineTo(endX, endY);
            }
          }
        }

        ctx.stroke();
        ctx.shadowBlur = 0;

        for (let i = points.length - 1; i >= 1; i--) {
          const point = points[i];
          const progress = 1 - i / points.length;
          const radius =
            cell * (0.27 + progress * 0.08);

          const hue =
            188 + progress * 18 +
            (isBoosting ? 75 : 0);
          const lightness = 48 + progress * 18;

          ctx.fillStyle =
            `hsl(${hue} 92% ${lightness}%)`;

          wrappedCopies(
            point.x,
            point.y,
            radius + 1,
            (x, y) => {
              ctx.beginPath();
              ctx.arc(x, y, radius, 0, Math.PI * 2);
              ctx.fill();
            }
          );
        }

        const head = points[0];
        const headRadius = cell * 0.44;
        const angle = Math.atan2(direction.y, direction.x);

        wrappedCopies(
          head.x,
          head.y,
          headRadius + cell * 0.2,
          (x, y) => {
            ctx.save();
            ctx.translate(x, y);
            ctx.rotate(angle);

            if (isBoosting) {
              ctx.strokeStyle = COLORS.pink;
              ctx.lineWidth = cell * 0.08;
              ctx.globalAlpha =
                0.55 + Math.sin(globalTime * 12) * 0.2;
              ctx.beginPath();
              ctx.arc(
                0,
                0,
                cell * 0.56,
                0,
                Math.PI * 2
              );
              ctx.stroke();
              ctx.globalAlpha = 1;
            }

            ctx.shadowColor = isBoosting
              ? COLORS.pink
              : COLORS.cyan;
            ctx.shadowBlur = cell * 1.05;

            const headGradient =
              ctx.createLinearGradient(
                -headRadius,
                0,
                headRadius,
                0
              );

            headGradient.addColorStop(
              0,
              isBoosting ? "#ff4fd8" : "#2abfcf"
            );
            headGradient.addColorStop(0.55, "#efffff");
            headGradient.addColorStop(1, COLORS.cyan);

            ctx.fillStyle = headGradient;
            ctx.beginPath();
            ctx.arc(
              0,
              0,
              headRadius,
              0,
              Math.PI * 2
            );
            ctx.fill();

            ctx.shadowBlur = 0;
            ctx.fillStyle = "#06111b";

            ctx.beginPath();
            ctx.arc(
              cell * 0.17,
              -cell * 0.14,
              cell * 0.055,
              0,
              Math.PI * 2
            );
            ctx.arc(
              cell * 0.17,
              cell * 0.14,
              cell * 0.055,
              0,
              Math.PI * 2
            );
            ctx.fill();

            ctx.fillStyle = "rgba(255,255,255,0.8)";
            ctx.beginPath();
            ctx.moveTo(cell * 0.32, 0);
            ctx.lineTo(cell * 0.12, -cell * 0.065);
            ctx.lineTo(cell * 0.12, cell * 0.065);
            ctx.closePath();
            ctx.fill();

            ctx.restore();
          }
        );

        ctx.restore();
      }

      function drawParticles() {
        const cell = view.cell;

        ctx.save();

        for (const particle of particles) {
          const alpha = clamp(
            particle.life / particle.maxLife,
            0,
            1
          );

          const x = mod(particle.x, GRID) * cell;
          const y = mod(particle.y, GRID) * cell;
          const size = particle.size * cell;

          ctx.globalAlpha = alpha;
          ctx.fillStyle = particle.color;

          wrappedCopies(
            x,
            y,
            size * 2,
            (copyX, copyY) => {
              ctx.save();
              ctx.translate(copyX, copyY);
              ctx.rotate(particle.rot);
              ctx.fillRect(
                -size / 2,
                -size / 2,
                size * 1.7,
                size
              );
              ctx.restore();
            }
          );
        }

        ctx.restore();
      }

      function drawPopups() {
        const cell = view.cell;

        ctx.save();
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";
        ctx.font =
          `900 ${Math.max(11, cell * 0.48)}px ` +
          "ui-monospace, SFMono-Regular, Menlo, monospace";

        for (const popup of popups) {
          const alpha = clamp(
            popup.life / popup.maxLife,
            0,
            1
          );

          const x = mod(popup.x, GRID) * cell;
          const y = mod(popup.y, GRID) * cell;

          ctx.globalAlpha = alpha;
          ctx.fillStyle = popup.color;
          ctx.shadowColor = popup.color;
          ctx.shadowBlur = cell * 0.45;
          ctx.fillText(popup.text, x, y);
        }

        ctx.restore();
      }

      function drawBoard(alpha) {
        const shakeX =
          shake > 0.1 ? random(-shake, shake) : 0;
        const shakeY =
          shake > 0.1 ? random(-shake, shake) : 0;

        ctx.save();
        ctx.translate(
          view.boardX + shakeX,
          view.boardY + shakeY
        );

        ctx.save();
        ctx.shadowColor = "rgba(55,220,246,0.32)";
        ctx.shadowBlur = 34;
        roundedRectPath(
          ctx,
          0,
          0,
          view.side,
          view.side,
          18
        );
        ctx.fillStyle = "rgba(5,11,26,0.92)";
        ctx.fill();
        ctx.restore();

        roundedRectPath(
          ctx,
          0,
          0,
          view.side,
          view.side,
          18
        );
        ctx.fillStyle = "rgba(5,10,23,0.91)";
        ctx.fill();

        ctx.save();
        roundedRectPath(
          ctx,
          0,
          0,
          view.side,
          view.side,
          18
        );
        ctx.clip();

        const innerGlow = ctx.createRadialGradient(
          view.side * 0.5,
          view.side * 0.45,
          0,
          view.side * 0.5,
          view.side * 0.5,
          view.side * 0.72
        );

        innerGlow.addColorStop(
          0,
          "rgba(19,67,87,0.14)"
        );
        innerGlow.addColorStop(
          1,
          "rgba(2,5,15,0.02)"
        );

        ctx.fillStyle = innerGlow;
        ctx.fillRect(0, 0, view.side, view.side);

        ctx.strokeStyle = "rgba(97,246,255,0.047)";
        ctx.lineWidth = 1;

        for (let i = 1; i < GRID; i++) {
          const p = i * view.cell;

          ctx.beginPath();
          ctx.moveTo(p, 0);
          ctx.lineTo(p, view.side);
          ctx.stroke();

          ctx.beginPath();
          ctx.moveTo(0, p);
          ctx.lineTo(view.side, p);
          ctx.stroke();
        }

        ctx.fillStyle = "rgba(97,246,255,0.24)";
        ctx.font =
          `700 ${Math.max(7, view.cell * 0.3)}px ` +
          "ui-monospace, monospace";
        ctx.textAlign = "left";
        ctx.fillText(
          "SECTOR 24 // WRAP ACTIVE",
          view.cell * 0.45,
          view.cell * 0.72
        );

        drawMines();
        drawFood();
        drawParticles();
        drawSnake(alpha);
        drawPopups();

        if (flash > 0) {
          ctx.fillStyle =
            `rgba(97,246,255,${flash})`;
          ctx.fillRect(0, 0, view.side, view.side);
        }

        ctx.restore();

        roundedRectPath(
          ctx,
          0.5,
          0.5,
          view.side - 1,
          view.side - 1,
          18
        );
        ctx.strokeStyle = "rgba(97,246,255,0.35)";
        ctx.lineWidth = 1.2;
        ctx.stroke();

        const bracket = Math.max(10, view.cell * 0.65);
        const inset = 8;

        ctx.strokeStyle = "rgba(255,79,216,0.65)";
        ctx.lineWidth = 2;

        const corners = [
          [inset, inset, 1, 1],
          [view.side - inset, inset, -1, 1],
          [inset, view.side - inset, 1, -1],
          [view.side - inset, view.side - inset, -1, -1]
        ];

        for (const [x, y, sx, sy] of corners) {
          ctx.beginPath();
          ctx.moveTo(x + sx * bracket, y);
          ctx.lineTo(x, y);
          ctx.lineTo(x, y + sy * bracket);
          ctx.stroke();
        }

        ctx.restore();
      }

      function render() {
        ctx.setTransform(
          view.dpr,
          0,
          0,
          view.dpr,
          0,
          0
        );
        ctx.clearRect(0, 0, view.w, view.h);

        drawBackground();

        const interval = currentTickInterval();
        const alpha =
          state === "playing"
            ? clamp(accumulator / interval, 0, 1)
            : 1;

        drawBoard(alpha);
      }

      function frame(now) {
        const dt = Math.min(
          0.05,
          Math.max(0, (now - lastFrame) / 1000)
        );

        lastFrame = now;
        globalTime += dt;

        if (state === "playing") {
          updatePlaying(dt);
          updateEffects(dt);
        } else if (state === "over") {
          updateEffects(dt);
        } else {
          shake *= Math.exp(-10 * dt);
          flash = Math.max(0, flash - dt * 1.8);
        }

        updateHud();
        render();
        requestAnimationFrame(frame);
      }

      function handleKeyDown(event) {
        const key = event.key.toLowerCase();
        const isSpace = event.code === "Space";

        const controlledKeys = [
          "arrowup",
          "arrowdown",
          "arrowleft",
          "arrowright",
          "w",
          "a",
          "s",
          "d",
          "p",
          "escape",
          "shift",
          "enter",
          "r"
        ];

        if (isSpace || controlledKeys.includes(key)) {
          event.preventDefault();
        }

        if (
          state === "menu" &&
          (isSpace || key === "enter")
        ) {
          startGame();
          return;
        }

        if (
          state === "over" &&
          (isSpace || key === "enter" || key === "r")
        ) {
          startGame();
          return;
        }

        if (
          state === "paused" &&
          (key === "p" || key === "escape")
        ) {
          resumeGame();
          return;
        }

        if (state !== "playing") return;

        if (key === "p" || key === "escape") {
          pauseGame();
          return;
        }

        if (isSpace || key === "shift") {
          keyboardBoost = true;
          return;
        }

        switch (key) {
          case "arrowup":
          case "w":
            setDirection(0, -1);
            break;
          case "arrowdown":
          case "s":
            setDirection(0, 1);
            break;
          case "arrowleft":
          case "a":
            setDirection(-1, 0);
            break;
          case "arrowright":
          case "d":
            setDirection(1, 0);
            break;
        }
      }

      function handleKeyUp(event) {
        const key = event.key.toLowerCase();

        if (event.code === "Space" || key === "shift") {
          keyboardBoost = false;
        }
      }

      window.addEventListener("keydown", handleKeyDown, {
        passive: false
      });
      window.addEventListener("keyup", handleKeyUp);
      window.addEventListener("resize", resize);

      document.addEventListener("visibilitychange", () => {
        if (document.hidden && state === "playing") {
          pauseGame();
        }
      });

      startBtn.addEventListener("click", startGame);
      restartBtn.addEventListener("click", startGame);
      resumeBtn.addEventListener("click", resumeGame);
      quitBtn.addEventListener("click", goToMenu);
      menuBtn.addEventListener("click", goToMenu);

      pauseBtn.addEventListener("click", () => {
        if (state === "playing") pauseGame();
        else if (state === "paused") resumeGame();
      });

      muteBtn.addEventListener("click", () => {
        sfx.ensure();
        sfx.toggle();
      });

      document
        .querySelectorAll("[data-dir]")
        .forEach((button) => {
          button.addEventListener(
            "pointerdown",
            (event) => {
              event.preventDefault();
              event.stopPropagation();
              sfx.ensure();

              const dir = button.dataset.dir;

              if (dir === "up") setDirection(0, -1);
              if (dir === "down") setDirection(0, 1);
              if (dir === "left") setDirection(-1, 0);
              if (dir === "right") setDirection(1, 0);
            },
            { passive: false }
          );
        });

      boostBtn.addEventListener(
        "pointerdown",
        (event) => {
          event.preventDefault();
          event.stopPropagation();
          sfx.ensure();
          pointerBoost = true;

          try {
            boostBtn.setPointerCapture(event.pointerId);
          } catch (_) {}
        },
        { passive: false }
      );

      const releaseBoost = () => {
        pointerBoost = false;
      };

      boostBtn.addEventListener("pointerup", releaseBoost);
      boostBtn.addEventListener("pointercancel", releaseBoost);
      boostBtn.addEventListener("lostpointercapture", releaseBoost);

      canvas.addEventListener(
        "pointerdown",
        (event) => {
          if (state !== "playing") return;

          event.preventDefault();
          sfx.ensure();

          swipe = {
            id: event.pointerId,
            x: event.clientX,
            y: event.clientY
          };

          try {
            canvas.setPointerCapture(event.pointerId);
          } catch (_) {}
        },
        { passive: false }
      );

      canvas.addEventListener(
        "pointermove",
        (event) => {
          if (
            !swipe ||
            swipe.id !== event.pointerId ||
            state !== "playing"
          ) {
            return;
          }

          event.preventDefault();

          const dx = event.clientX - swipe.x;
          const dy = event.clientY - swipe.y;
          const threshold = Math.max(15, view.cell * 0.62);

          if (
            Math.abs(dx) < threshold &&
            Math.abs(dy) < threshold
          ) {
            return;
          }

          if (Math.abs(dx) > Math.abs(dy)) {
            setDirection(dx > 0 ? 1 : -1, 0);
          } else {
            setDirection(0, dy > 0 ? 1 : -1);
          }

          swipe.x = event.clientX;
          swipe.y = event.clientY;
        },
        { passive: false }
      );

      const endSwipe = (event) => {
        if (swipe?.id === event.pointerId) {
          swipe = null;
        }
      };

      canvas.addEventListener("pointerup", endSwipe);
      canvas.addEventListener("pointercancel", endSwipe);

      resize();
      resetGameData();
      renderAllScores();
      updateMuteButton();
      updateHud();
      requestAnimationFrame(frame);
    })();
  </script>
</body>
</html>
```
