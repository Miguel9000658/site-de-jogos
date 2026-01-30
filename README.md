<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Geometry Run — Jogo Web</title>
  <style>
    :root {
      --bg: #0b0f1a;
      --panel: #12172a;
      --neon: #5ef2ff;
      --accent: #7cff6b;
      --danger: #ff5e7a;
      --text: #e8ebff;
    }
    * { box-sizing: border-box; }
    body {
      margin: 0;
      font-family: system-ui, -apple-system, Segoe UI, Roboto, Ubuntu, Cantarell, Noto Sans, Arial, sans-serif;
      background: radial-gradient(1200px 600px at 50% -10%, #1b2250, var(--bg));
      color: var(--text);
      min-height: 100vh;
      display: grid;
      place-items: center;
    }
    .app {
      width: min(1100px, 96vw);
      background: linear-gradient(180deg, rgba(255,255,255,.04), rgba(255,255,255,.02));
      border: 1px solid rgba(255,255,255,.08);
      border-radius: 18px;
      box-shadow: 0 30px 80px rgba(0,0,0,.45);
      overflow: hidden;
    }
    header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 16px 20px;
      background: linear-gradient(180deg, rgba(94,242,255,.15), rgba(94,242,255,0));
      border-bottom: 1px solid rgba(255,255,255,.08);
    }
    .brand {
      display: flex; gap: 12px; align-items: center;
    }
    .logo {
      width: 36px; height: 36px; border-radius: 8px;
      background: linear-gradient(135deg, var(--neon), var(--accent));
      box-shadow: 0 0 24px rgba(94,242,255,.6);
    }
    .brand h1 { font-size: 1.1rem; margin: 0; letter-spacing: .5px; }
    .actions button {
      background: var(--panel);
      color: var(--text);
      border: 1px solid rgba(255,255,255,.12);
      border-radius: 10px;
      padding: 10px 14px;
      cursor: pointer;
    }
    .actions button.primary {
      background: linear-gradient(135deg, var(--neon), #7aa2ff);
      color: #001;
      border: none;
      font-weight: 700;
    }
    main {
      display: grid;
      grid-template-columns: 1fr 320px;
      gap: 16px;
      padding: 16px;
    }
    @media (max-width: 900px) { main { grid-template-columns: 1fr; } }

    .game-wrap {
      background: var(--panel);
      border-radius: 14px;
      border: 1px solid rgba(255,255,255,.08);
      padding: 12px;
    }
    canvas {
      width: 100%;
      height: 360px;
      background: linear-gradient(180deg, #0e1330, #090d1f);
      border-radius: 10px;
      display: block;
    }
    .hud {
      display: flex; justify-content: space-between; align-items: center; margin-top: 10px;
      font-size: .9rem; opacity: .9;
    }
    .panel {
      background: var(--panel);
      border-radius: 14px;
      border: 1px solid rgba(255,255,255,.08);
      padding: 14px;
    }
    .panel h3 { margin: 0 0 8px; }
    .kbd { padding: 2px 8px; border-radius: 6px; background: #0a0f26; border: 1px solid rgba(255,255,255,.12); }
    footer { padding: 10px 16px; opacity: .7; font-size: .8rem; }
  </style>
</head>
<body>
  <div class="app">
    <header>
      <div class="brand">
        <div class="logo"></div>
        <h1>Geometry Run <small style="opacity:.6">(fan‑made)</small></h1>
      </div>
      <div class="actions">
        <button onclick="resetGame()">Reiniciar</button>
        <button class="primary" onclick="startGame()">Jogar</button>
      </div>
    </header>

    <main>
      <section class="game-wrap">
        <canvas id="game" width="900" height="360"></canvas>
        <div class="hud">
          <div>Pontuação: <strong id="score">0</strong></div>
          <div>Recorde: <strong id="best">0</strong></div>
        </div>
      </section>
      <aside class="panel">
        <h3>Como jogar</h3>
        <p>Pule os obstáculos no ritmo. Um toque = um pulo.</p>
        <p><span class="kbd">Espaço</span> ou <span class="kbd">Clique</span></p>
        <hr style="border-color: rgba(255,255,255,.08)" />
        <h3>Dicas</h3>
        <ul>
          <li>Antecipe os espinhos</li>
          <li>Não pule cedo demais</li>
          <li>Concentre no ritmo</li>
        </ul>
      </aside>
    </main>

    <footer>
      Jogo web inspirado em plataformas rítmicas. Arte e código originais.
    </footer>
  </div>

  <script>
    const canvas = document.getElementById('game');
    const ctx = canvas.getContext('2d');
    const scoreEl = document.getElementById('score');
    const bestEl = document.getElementById('best');

    let running = false;
    let score = 0;
    let best = Number(localStorage.getItem('best-run')) || 0;
    bestEl.textContent = best;

    const groundY = canvas.height - 48;
    const player = { x: 80, y: groundY - 24, w: 24, h: 24, vy: 0, jump: -10 };
    let obstacles = [];
    let speed = 5;
    let gravity = 0.6;
    let spawnTimer = 0;

    function startGame() {
      if (!running) {
        running = true;
        requestAnimationFrame(loop);
      }
    }
    function resetGame() {
      running = false;
      score = 0; speed = 5; obstacles = []; spawnTimer = 0;
      player.y = groundY - player.h; player.vy = 0;
      scoreEl.textContent = score;
      draw();
    }

    function spawnObstacle() {
      const size = 24 + Math.random() * 16;
      obstacles.push({ x: canvas.width + 20, y: groundY - size, w: size, h: size });
    }

    function collide(a, b) {
      return a.x < b.x + b.w && a.x + a.w > b.x && a.y < b.y + b.h && a.y + a.h > b.y;
    }

    function update() {
      player.vy += gravity;
      player.y += player.vy;
      if (player.y > groundY - player.h) { player.y = groundY - player.h; player.vy = 0; }

      spawnTimer--;
      if (spawnTimer <= 0) { spawnObstacle(); spawnTimer = 60 + Math.random() * 40; }

      for (const o of obstacles) o.x -= speed;
      obstacles = obstacles.filter(o => o.x + o.w > -20);

      for (const o of obstacles) {
        if (collide(player, o)) {
          running = false;
          best = Math.max(best, score);
          localStorage.setItem('best-run', best);
          bestEl.textContent = best;
        }
      }

      score++;
      speed += 0.0015;
      scoreEl.textContent = score;
    }

    function draw() {
      ctx.clearRect(0,0,canvas.width,canvas.height);
      // ground
      ctx.fillStyle = '#0a0f26';
      ctx.fillRect(0, groundY, canvas.width, 48);

      // player
      ctx.fillStyle = '#5ef2ff';
      ctx.fillRect(player.x, player.y, player.w, player.h);

      // obstacles
      ctx.fillStyle = '#ff5e7a';
      for (const o of obstacles) ctx.fillRect(o.x, o.y, o.w, o.h);
    }

    function loop() {
      if (!running) return;
      update(); draw();
      requestAnimationFrame(loop);
    }

    function jump() {
      if (player.y >= groundY - player.h - 0.1) player.vy = player.jump;
    }

    window.addEventListener('keydown', e => { if (e.code === 'Space') jump(); });
    canvas.addEventListener('mousedown', jump);

    draw();
  </script>
</body>
</html>
