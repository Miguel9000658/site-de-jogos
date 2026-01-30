<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Geometry Dash Fan-Made</title>
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
    .brand { display: flex; gap: 12px; align-items: center; }
    .logo { width: 36px; height: 36px; border-radius: 8px; background: linear-gradient(135deg, var(--neon), var(--accent)); box-shadow: 0 0 24px rgba(94,242,255,.6); }
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
    .game-wrap { background: var(--panel); border-radius: 14px; border: 1px solid rgba(255,255,255,.08); padding: 12px; }
    canvas { width: 100%; height: 360px; background: linear-gradient(180deg, #0e1330, #090d1f); border-radius: 10px; display: block; }
    .hud { display: flex; justify-content: space-between; align-items: center; margin-top: 10px; font-size: .9rem; opacity: .9; }
    .panel { background: var(--panel); border-radius: 14px; border: 1px solid rgba(255,255,255,.08); padding: 14px; }
    .panel h3 { margin: 0 0 8px; }
    .kbd { padding: 2px 8px; border-radius: 6px; background: #0a0f26; border: 1px solid rgba(255,255,255,.12); }
    footer { padding: 10px 16px; opacity: .7; font-size: .8rem; }
    .color-picker { display: flex; gap: 6px; margin-bottom: 8px; }
    .color-picker div { width: 24px; height: 24px; border-radius: 4px; cursor: pointer; border: 2px solid rgba(255,255,255,.2); }
    .selected { border: 2px solid #fff; }
  </style>
</head>
<body>
  <div class="app">
    <header>
      <div class="brand">
        <div class="logo"></div>
        <h1>Geometry Dash Fan-Made</h1>
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
        <h3>Escolha seu cubo</h3>
        <div class="color-picker" id="colors"></div>
        <h3>Como jogar</h3>
        <p>Pule os obstáculos no ritmo. Um toque = um pulo.</p>
        <p><span class="kbd">Espaço</span> ou <span class="kbd">Clique</span></p>
      </aside>
    </main>

    <footer>
      Jogo web fan-made inspirado no Geometry Dash Lite. Arte original.
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
    const player = { x: 80, y: groundY - 24, w: 24, h: 24, vy: 0, jump: -10, color: '#5ef2ff' };
    let obstacles = [];
    let speed = 5;
    let gravity = 0.6;
    let spawnTimer = 0;

    // Mapa estilo Geometry Dash Lite (predefinido)
    const map = [
      { x: 400, y: groundY-24, w: 24, h: 24 },
      { x: 600, y: groundY-48, w: 24, h: 48 },
      { x: 850, y: groundY-24, w: 24, h: 24 },
      { x: 1050, y: groundY-72, w: 24, h: 72 }
    ];

    function startGame() {
      if (!running) {
        running = true;
        obstacles = map.map(o => ({...o}));
        requestAnimationFrame(loop);
      }
    }

    function resetGame() {
      running = false;
      score = 0; speed = 5; obstacles = map.map(o => ({...o})); player.y = groundY - player.h; player.vy = 0;
      scoreEl.textContent = score;
      draw();
    }

    function collide(a, b) {
      return a.x < b.x + b.w && a.x + a.w > b.x && a.y < b.y + b.h && a.y + a.h > b.y;
    }

    function update() {
      player.vy += gravity;
      player.y += player.vy;
      if (player.y > groundY - player.h) { player.y = groundY - player.h; player.vy = 0; }

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
      ctx.fillStyle = '#0a0f26';
      ctx.fillRect(0, groundY, canvas.width, 48);
      ctx.fillStyle = player.color;
      ctx.fillRect(player.x, player.y, player.w, player.h);
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

    // Personalização de cores
    const colors = ['#5ef2ff','#ff5e7a','#7cff6b','#ffd700','#ff8c00','#ff69b4'];
    const colorsDiv = document.getElementById('colors');
    colors.forEach(c => {
      const div = document.createElement('div');
      div.style.background = c;
      div.addEventListener('click', () => {
        player.color = c;
        document.querySelectorAll('#colors div').forEach(d => d.classList.remove('selected'));
        div.classList.add('selected');
      });
      colorsDiv.appendChild(div);
    });
    document.querySelector('#colors div').classList.add('selected');

    draw();
  </script>
</body>
</html>
