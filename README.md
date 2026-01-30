# site-de-jogos<body>
  <header>🎮 Meu Site de Jogos</header>
  <div class="container">
    <div class="card">
      <h2>🧠 Clique Rápido</h2>
      <p>Teste sua velocidade</p>
      <button onclick="jogoClique()">Jogar</button>
    </div>
    ...
  </div>
</body>
<script>
  function jogoClique() { alert("Clique OK! Você é rápido ⚡"); }
  function jogoNumero() { 
    let segredo = Math.floor(Math.random() * 10) + 1; 
    let tentativa = prompt("Adivinhe o número de 1 a 10"); 
    if (tentativa == segredo) { alert("🎉 Acertou!"); } 
    else { alert("❌ Errou! O número era " + segredo); } 
  }
</script>
