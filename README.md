<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Geometry Dash Lite Fan-Made</title>
  <style>
    :root {
      --bg: #0b0f1a;
      --neon: #5ef2ff;
      --accent: #7cff6b;
      --danger: #ff5e7a;
      --text: #e8ebff;
    }
    * { box-sizing: border-box; }
    body {
      margin: 0; font-family: system-ui, sans-serif;
      background: linear-gradient(180deg, #0b0f1a, #090c1f);
      display: grid; place-items: center; min-height: 100vh; color: var(--text);
    }
    .game-container { background: #12172a; border-radius: 12px; padding: 16px; box-shadow: 0 20px 60px rgba(0,0,0,.5); }
    canvas { display: block; background: linear-gradient(180deg, #0e1330, #090d1f); border-radius: 10px; }
    .hud { display: flex; justify-content: space-between; margin-top: 8px; font-size: 1rem; }
    .controls { display: flex; gap: 8px; margin-bottom: 8px; }
    .color { width: 24px; height: 24px; border-radius: 4px; cursor: pointer; border: 2px solid rgba(255,255,255,.2); }
    .selected { border: 2px solid #fff; }
    button { padding: 8px 14px; border-radius: 8px; border: none; cursor: pointer; background: var(--neon); color: #000; font-weight: bold; }
  </style>
</head>
<body>
  <div class="game-container">
    <div class="controls" id="colorPicker"></div>
    <canvas id="game" width="800" height="360"></canvas>
    <div class="hud">
      <div>Pontuação: <span id="score">0</span></div>
      <div>Recorde: <span id="best">0</span></div>
      <button onclick="resetGame()">Reiniciar</button>
      <button onclick="startGame()">Jogar</button>
    </div>
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

    const groundY = canvas.height - 40;
    const gravity = 0.6;
    const player = { x: 80, y: groundY - 24, w: 24, h: 24, vy: 0, jump: -12, color: '#5ef2ff' };
    let speed = 4;

    // Obstáculos estilo Geometry Dash Lite (nível fixo)
    const level = [
      { x: 400, y: groundY - 24, w: 24, h: 24 },
      { x: 600, y: groundY - 48, w: 24, h: 48 },
      { x: 800, y: groundY - 24, w: 24, h: 24 },
      { x: 1000, y: groundY - 72, w: 24, h: 72 },
      { x: 1200, y: groundY - 24, w: 24, h: 24 },
      { x: 1400, y: groundY - 48, w: 24, h: 48 },
    ];
    let obstacles = level.map(o => ({ ...o }));

    function startGame() {
      if (!running) {
        running = true;
        obstacles = level.map(o => ({ ...o }));
        score = 0; speed = 4;
        requestAnimationFrame(loop);
      }
    }

    function resetGame() {
      running = false;
      score = 0; speed = 4; obstacles = level.map(o => ({ ...o }));
      player.y = groundY - player.h; player.vy = 0;
      scoreEl.textContent = score;
      draw();
    }

    function collide(a, b) {
      return a.x < b.x + b.w && a.x + a.w > b.x && a.y < b.y + b.h && a.y + a.h > b.y;
    }

    function update() {
      player.vy += gravity;
      player.y += player.vy;
      if (player.y > groundY - player.h) player.y = groundY - player.h, player.vy = 0;

      for (const o of obstacles) o.x -= speed;
      if (obstacles[0] && obstacles[0].x + obstacles[0].w < 0) obstacles.shift();

      // Colisão
      for (const o of obstacles) {
        if (collide(player, o)) {
          running = false;
          best = Math.max(best, score);
          localStorage.setItem('best-run', best);
          bestEl.textContent = best;
        }
      }

      score++; speed += 0.0015;
      scoreEl.textContent = score;
    }

    function draw() {
      ctx.clearRect(0,0,canvas.width,canvas.height);
      ctx.fillStyle = '#0a0f26';
      ctx.fillRect(0, groundY, canvas.width, 40);
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

    // Personalização do cubo
    const colors = ['#5ef2ff','#ff5e7a','#7cff6b','#ffd700','#ff8c00','#ff69b4'];
    const colorPicker = document.getElementById('colorPicker');
    colors.forEach(c => {
      const div = document.createElement('div');
      div.classList.add('color');
      div.style.background = c;
      div.addEventListener('click', () => {
        player.color = c;
        document.querySelectorAll('.color').forEach(d => d.classList.remove('selected'));
        div.classList.add('selected');
      });
      colorPicker.appendChild(div);
    });
    document.querySelector('.color').classList.add('selected');

    draw();
  </script>
</body>
</html>
