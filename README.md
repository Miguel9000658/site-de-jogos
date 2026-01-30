<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Classroom Chat</title>
<style>
  body { margin:0; font-family: Arial, sans-serif; background:#e5ddd5; display:flex; justify-content:center; align-items:center; height:100vh; }
  .chat-container { width: 450px; max-width: 95vw; background:#f0f0f0; border-radius:12px; display:flex; flex-direction:column; overflow:hidden; box-shadow:0 5px 20px rgba(0,0,0,0.2); }
  .messages { flex:1; padding:10px; overflow-y:auto; background:#d6e9ff; min-height:200px; }
  .message { margin-bottom:6px; max-width:75%; padding:8px 12px; border-radius:18px; word-wrap: break-word; }
  .message.you { background:#25d366; color:#fff; margin-left:auto; border-bottom-right-radius:2px; }
  .message.friend { background:#fff; color:#000; margin-right:auto; border-bottom-left-radius:2px; }
  .input-area { display:flex; border-top:1px solid #ccc; background:#f0f0f0; }
  .input-area input { flex:1; padding:10px; border:none; outline:none; border-radius:0; }
  .input-area button { padding:10px 14px; border:none; background:#34b7f1; color:#fff; cursor:pointer; font-weight:bold; }
  .top-controls { display:flex; gap:4px; padding:8px; background:#d0e0ff; }
  .top-controls input { padding:6px; flex:1; }
</style>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
</head>
<body>
<div class="chat-container">
  <div class="top-controls">
    <input type="text" id="username" placeholder="Your name" />
    <input type="text" id="room" placeholder="Room name" />
    <button onclick="joinRoom()">Join/Create Room</button>
  </div>
  <div class="messages" id="messages"></div>
  <div class="input-area">
    <input type="text" id="msg" placeholder="Type a message" />
    <button onclick="sendMessage()">Send</button>
  </div>
</div>
<script>
// ===== Firebase config =====
const firebaseConfig = {
  apiKey: "AIzaSyATZXvVFiObc_e12PZzA0k0qeEjblfOe2Y",
  authDomain: "littlezap-7ac2d.firebaseapp.com",
  databaseURL: "https://littlezap-7ac2d.firebaseio.com",
  projectId: "littlezap-7ac2d",
  storageBucket: "littlezap-7ac2d.appspot.com",
  messagingSenderId: "206009524289",
  appId: "1:206009524289:web:f5dc8325817b0e9ba666c0"
};

// Initialize Firebase
const app = firebase.initializeApp(firebaseConfig);
const db = firebase.database();

let currentRoom = 'General';
const messagesDiv = document.getElementById('messages');

// ===== Join or create room =====
function joinRoom(){
    const roomInput = document.getElementById('room').value.trim();
    if(roomInput===''){ alert('Enter a room name!'); return; }
    currentRoom = roomInput;
    messagesDiv.innerHTML = '';

    // Listen to messages in this room
    db.ref(`chat/${currentRoom}`).off();
    db.ref(`chat/${currentRoom}`).on('child_added', snapshot=>{
        const data = snapshot.val();
        const div = document.createElement('div');
        div.classList.add('message');
