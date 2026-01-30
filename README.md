
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Chat de Sala Online</title>
<style>
  body { margin:0; font-family: Arial, sans-serif; background:#e5ddd5; display:flex; justify-content:center; align-items:center; height:100vh; }
  .chat-container { width: 400px; max-width: 95vw; background:#f0f0f0; border-radius:12px; display:flex; flex-direction:column; overflow:hidden; box-shadow: 0 5px 20px rgba(0,0,0,0.2); }
  .messages { flex:1; padding:10px; overflow-y:auto; background:#d6e9ff; }
  .message { margin-bottom:6px; max-width:75%; padding:8px 12px; border-radius:18px; word-wrap: break-word; }
  .message.you { background:#25d366; color:#fff; margin-left:auto; border-bottom-right-radius:2px; }
  .message.friend { background:#fff; color:#000; margin-right:auto; border-bottom-left-radius:2px; }
  .input-area { display:flex; border-top:1px solid #ccc; background:#f0f0f0; }
  .input-area input { flex:1; padding:10px; border:none; outline:none; border-radius:0; }
  .input-area button { padding:10px 14px; border:none; background:#34b7f1; color:#fff; cursor:pointer; font-weight:bold; }
  #username, #room { padding:10px; border:none; width:120px; margin-right:4px; }
</style>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
</head>
<body>
<div class="chat-container">
  <div class="messages" id="messages"></div>
  <div class="input-area">
    <input type="text" id="username" placeholder="Seu nome" />
    <input type="text" id="room" placeholder="Sala" />
    <input type="text" id="msg" placeholder="Mensagem" />
    <button onclick="sendMessage()">Enviar</button>
  </div>
</div>
<script>
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

  function sendMessage() {
    const user = document.getElementById('username').value.trim() || 'Anon';
    const room = document.getElementById('room').value.trim() || 'Geral';
    const msg = document.getElementById('msg').value.trim();
    if(!msg) return;
    db.ref(`chat/${room}`).push({username:user, message:msg, timestamp:Date.now()});
    document.getElementById('msg').value='';
  }

  function joinRoom(room) {
    db.ref(`chat/${room}`).off();
    messagesDiv.innerHTML = '';
    db.ref(`chat/${room}`).on('child_added', snapshot => {
      const data = snapshot.val();
      const div = document.createElement('div');
      div.classList.add('message');
      if(data.username === document.getElementById('username').value.trim()) div.classList.add('you');
      else div.classList.add('friend');
      div.innerHTML = `<strong>${data.username}:</strong> ${data.message}`;
      messagesDiv.appendChild(div);
      messagesDiv.scrollTop = messagesDiv.scrollHeight;
    });
  }

  // Mudar de sala ao digitar o nome da sala
  document.getElementById('room').addEventListener('change', e => {
    const room = e.target.value.trim() || 'Geral';
    joinRoom(room);
  });

  // Inicializa na sala padrão 'Geral'
  joinRoom('Geral');
</script>
</body>
</html>
