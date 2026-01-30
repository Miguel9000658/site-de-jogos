<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Chat com Amigos</title>
<style>
  body { margin:0; font-family: Arial, sans-serif; background:#12172a; color:#fff; display:flex; justify-content:center; align-items:center; height:100vh; }
  .chat-container { width: 400px; max-width: 95vw; background:#1f2535; border-radius:12px; display:flex; flex-direction:column; overflow:hidden; }
  .messages { flex:1; padding:10px; overflow-y:auto; }
  .message { margin-bottom:6px; }
  .message strong { color:#5ef2ff; }
  .input-area { display:flex; border-top:1px solid #333; }
  .input-area input { flex:1; padding:10px; border:none; outline:none; background:#2a2f44; color:#fff; }
  .input-area button { padding:10px 14px; border:none; background:#5ef2ff; color:#000; cursor:pointer; }
</style>
<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
</head>
<body>
<div class="chat-container">
  <div class="messages" id="messages"></div>
  <div class="input-area">
    <input type="text" id="username" placeholder="Seu nome" />
    <input type="text" id="msg" placeholder="Digite uma mensagem" />
    <button onclick="sendMessage()">Enviar</button>
  </div>
</div>
<script>
  // Configuração Firebase (substitua pelos seus dados do projeto Firebase)
  const firebaseConfig = {
    apiKey: "SUA_API_KEY",
    authDomain: "SEU_PROJECT_ID.firebaseapp.com",
    databaseURL: "https://SEU_PROJECT_ID.firebaseio.com",
    projectId: "SEU_PROJECT_ID",
    storageBucket: "SEU_PROJECT_ID.appspot.com",
    messagingSenderId: "SUA_MESSAGING_ID",
    appId: "SEU_APP_ID"
  };

  const app = firebase.initializeApp(firebaseConfig);
  const db = firebase.database();

  const messagesDiv = document.getElementById('messages');

  // Enviar mensagem
  function sendMessage() {
    const user = document.getElementById('username').value.trim() || 'Anon';
    const msg = document.getElementById('msg').value.trim();
    if(msg==='') return;
    db.ref('chat').push({username:user, message:msg, timestamp:Date.now()});
    document.getElementById('msg').value='';
  }

  // Ouvir novas mensagens
  db.ref('chat').on('child_added', snapshot => {
    const data = snapshot.val();
    const div = document.createElement('div');
    div.classList.add('message');
    div.innerHTML = `<strong>${data.username}:</strong> ${data.message}`;
    messagesDiv.appendChild(div);
    messagesDiv.scrollTop = messagesDiv.scrollHeight;
  });
</script>
</body>
</html>
