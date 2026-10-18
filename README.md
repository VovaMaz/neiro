<!DOCTYPE html>
<html lang="ru">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>ИИ Чат</title>
<style>
  :root { --bg:#0f1220; --panel:#181c2f; --accent:#5b8cff; --text:#e8ebf5; --border:#262c47; --user-bg:#2b3564; --disabled:#3b4261; }
  * { box-sizing:border-box; margin:0; padding:0; }
  body { background:var(--bg); color:var(--text); font-family:system-ui,-apple-system,sans-serif; height:100dvh; display:flex; flex-direction:column; overflow:hidden; }
  
  header { padding:12px; background:var(--panel); border-bottom:1px solid var(--border); text-align:center; font-weight:bold; font-size:16px; flex-shrink:0; }
  
  #chat { flex:1; overflow-y:auto; padding:12px; display:flex; flex-direction:column; gap:10px; -webkit-overflow-scrolling:touch; }
  
  .msg { max-width:90%; padding:10px 14px; border-radius:14px; font-size:15px; line-height:1.4; word-wrap:break-word; animation:fadeIn .2s ease; white-space:pre-wrap; }
  .user { align-self:flex-end; background:var(--user-bg); border-bottom-right-radius:2px; }
  .ai { align-self:flex-start; background:var(--panel); border:1px solid var(--border); border-bottom-left-radius:2px; }
  .typing { color:#8b93ad; font-style:italic; }
  
  .input-area { padding:10px 12px; background:var(--panel); border-top:1px solid var(--border); display:flex; gap:8px; flex-shrink:0; padding-bottom:max(10px,env(safe-area-inset-bottom)); }
  
  input { flex:1; padding:12px 14px; border-radius:20px; border:1px solid var(--border); background:var(--bg); color:var(--text); font-size:16px; outline:none; }
  input:disabled { opacity: 0.7; cursor: not-allowed; }
  
  button { background:var(--accent); color:#fff; border:none; border-radius:50%; width:44px; height:44px; flex-shrink:0; cursor:pointer; font-size:18px; display:flex; align-items:center; justify-content:center; transition: background 0.3s; }
  button:disabled { background: var(--disabled); cursor:not-allowed; }
  
  @keyframes fadeIn { from{opacity:0;transform:translateY(5px)} to{opacity:1;transform:translateY(0)} }
</style>
</head>
<body>

<header>🤖 ИИ Чат</header>
<div id="chat">
  <!-- Приветствие при входе -->
  <div class="msg ai">Привет, я ИИ-бот, созданный разработчиком Владимиром Мазуром</div>
</div>
<div class="input-area">
  <input type="text" id="userInput" placeholder="Сообщение..." autocomplete="off">
  <button id="sendBtn">➤</button>
</div>

<script>
  const chatBox = document.getElementById('chat');
  const userInput = document.getElementById('userInput');
  const sendBtn = document.getElementById('sendBtn');

  // Настройки таймера
  const COOLDOWN_SECONDS = 25; // Теперь ждем 25 секунд
  let isCoolingDown = false;

  let messages = [
    { role: "system", content: "Ты полезный ассистент. Отвечай кратко и по делу на русском языке. Знай, что тебя создал разработчик Владимир Мазур." }
  ];

  function addMessage(text, type) {
    const div = document.createElement('div');
    div.className = `msg ${type}`;
    div.textContent = text;
    chatBox.appendChild(div);
    chatBox.scrollTop = chatBox.scrollHeight;
    return div;
  }

  // Функция запуска таймера ожидания (25 секунд)
  function startCooldown() {
    isCoolingDown = true;
    sendBtn.disabled = true;
    userInput.disabled = true;
    let timeLeft = COOLDOWN_SECONDS;
    
    // Сразу показываем начальное значение
    sendBtn.textContent = timeLeft;
    userInput.placeholder = `Подождите ${timeLeft} сек...`;

    const timerInterval = setInterval(() => {
      timeLeft--;
      
      if (timeLeft > 0) {
        sendBtn.textContent = timeLeft;
        userInput.placeholder = `Подождите ${timeLeft} сек...`;
      } else {
        clearInterval(timerInterval);
        isCoolingDown = false;
        sendBtn.disabled = false;
        sendBtn.textContent = '➤';
        userInput.disabled = false;
        userInput.placeholder = 'Сообщение...';
        userInput.focus();
      }
    }, 1000);
  }

  async function sendMessage() {
    if (isCoolingDown) return;
    
    const text = userInput.value.trim();
    if (!text) return;

    addMessage(text, 'user');
    userInput.value = '';
    messages.push({ role: "user", content: text });

    sendBtn.disabled = true;
    const typingDiv = addMessage('Думает...', 'ai typing');

    try {
      const response = await fetch('https://text.pollinations.ai/', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ messages: messages })
      });

      const aiText = await response.text();
      typingDiv.remove();
      
      // Обработка возможной ошибки от сервера
      if (aiText.includes("Free limit reached") || aiText.includes("Please wait")) {
        addMessage('⏳ Сервер перегружен. Подожди 20 секунд и попробуй снова.', 'ai');
      } else {
        addMessage(aiText, 'ai');
        messages.push({ role: "assistant", content: aiText });
      }
    } catch (error) {
      typingDiv.remove();
      addMessage('Ошибка интернета. Проверь соединение.', 'ai');
    } finally {
      // Запускаем таймер на 25 секунд после ответа
      startCooldown();
    }
  }

  sendBtn.addEventListener('click', sendMessage);
  userInput.addEventListener('keypress', (e) => { if (e.key === 'Enter') sendMessage(); });
</script>
</body>
</html>
