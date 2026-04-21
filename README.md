<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>More chats!</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }

        .container {
            background: white;
            border-radius: 20px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
            width: 100%;
            max-width: 1200px;
            height: 90vh;
            display: flex;
            overflow: hidden;
        }

        /* Sidebar */
        .sidebar {
            width: 300px;
            background: #f8f9fa;
            border-right: 1px solid #e0e0e0;
            display: flex;
            flex-direction: column;
        }

        .user-info {
            padding: 20px;
            background: white;
            border-bottom: 1px solid #e0e0e0;
        }

        .user-id {
            font-size: 12px;
            color: #667eea;
            word-break: break-all;
            background: #f0f0f0;
            padding: 8px;
            border-radius: 8px;
            margin-top: 8px;
            font-family: monospace;
        }

        .connection-panel {
            padding: 20px;
            background: white;
            border-bottom: 1px solid #e0e0e0;
        }

        input {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 8px;
            font-size: 14px;
            margin-bottom: 10px;
        }

        button {
            width: 100%;
            padding: 10px;
            background: #667eea;
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-size: 14px;
            font-weight: 600;
            transition: transform 0.2s, background 0.2s;
        }

        button:hover {
            background: #5a67d8;
            transform: translateY(-1px);
        }

        button:active {
            transform: translateY(0);
        }

        button.danger {
            background: #e53e3e;
        }

        button.danger:hover {
            background: #c53030;
        }

        .connections-list {
            flex: 1;
            padding: 20px;
            overflow-y: auto;
        }

        .connection-item {
            background: white;
            padding: 12px;
            border-radius: 10px;
            margin-bottom: 10px;
            cursor: pointer;
            transition: all 0.2s;
            border: 2px solid transparent;
        }

        .connection-item:hover {
            transform: translateX(5px);
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
        }

        .connection-item.active {
            border-color: #667eea;
            background: #f0f4ff;
        }

        .connection-name {
            font-weight: 600;
            color: #333;
            word-break: break-all;
            font-size: 13px;
        }

        .connection-status {
            font-size: 11px;
            color: #48bb78;
            margin-top: 4px;
        }

        /* Chat area */
        .chat-area {
            flex: 1;
            display: flex;
            flex-direction: column;
            background: #ffffff;
        }

        .chat-header {
            padding: 20px;
            background: white;
            border-bottom: 1px solid #e0e0e0;
        }

        .chat-header h3 {
            color: #333;
            font-size: 18px;
        }

        .messages {
            flex: 1;
            overflow-y: auto;
            padding: 20px;
            display: flex;
            flex-direction: column;
            gap: 12px;
        }

        .message {
            display: flex;
            flex-direction: column;
            max-width: 70%;
        }

        .message.sent {
            align-self: flex-end;
        }

        .message.received {
            align-self: flex-start;
        }

        .message-bubble {
            padding: 10px 15px;
            border-radius: 18px;
            word-wrap: break-word;
        }

        .message.sent .message-bubble {
            background: #667eea;
            color: white;
            border-bottom-right-radius: 4px;
        }

        .message.received .message-bubble {
            background: #f0f0f0;
            color: #333;
            border-bottom-left-radius: 4px;
        }

        .message-time {
            font-size: 10px;
            color: #999;
            margin-top: 4px;
            margin-left: 10px;
        }

        .message.sent .message-time {
            text-align: right;
        }

        .message-input-area {
            padding: 20px;
            border-top: 1px solid #e0e0e0;
            display: flex;
            gap: 10px;
        }

        .message-input-area input {
            flex: 1;
            margin-bottom: 0;
        }

        .message-input-area button {
            width: auto;
            padding: 10px 20px;
        }

        .empty-state {
            text-align: center;
            color: #999;
            padding: 40px;
        }

        @media (max-width: 768px) {
            .sidebar {
                width: 250px;
            }
            
            .message {
                max-width: 85%;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="sidebar">
            <div class="user-info">
                <strong>Ваш ID:</strong>
                <div class="user-id" id="myUserId">Загрузка...</div>
                <button id="copyIdBtn" style="margin-top: 10px; background: #48bb78;">📋 Скопировать ID</button>
            </div>
            
            <div class="connection-panel">
                <input type="text" id="remoteUserId" placeholder="Введите ID собеседника">
                <button id="connectBtn">🔗 Подключиться</button>
            </div>
            
            <div class="connections-list">
                <div style="font-weight: 600; margin-bottom: 10px; color: #666;">📱 Активные чаты:</div>
                <div id="connectionsList"></div>
            </div>
        </div>
        
        <div class="chat-area">
            <div class="chat-header">
                <h3 id="currentChatTitle">Выберите чат</h3>
            </div>
            <div class="messages" id="messages">
                <div class="empty-state">💬 Выберите собеседника для начала общения</div>
            </div>
            <div class="message-input-area">
                <input type="text" id="messageInput" placeholder="Введите сообщение..." disabled>
                <button id="sendBtn" disabled>📤 Отправить</button>
            </div>
        </div>
    </div>

    <script>
        // Конфигурация Signaling сервера (публичный бесплатный сервер для обмена сигналами)
        // Это единственная зависимость - сервер для обмена метаданными WebRTC
        // Сам обмен сообщениями идет напрямую P2P
        const configuration = {
            iceServers: [
                { urls: 'stun:stun.l.google.com:19302' },
                { urls: 'stun:stun1.l.google.com:19302' },
                { urls: 'stun:stun2.l.google.com:19302' }
            ]
        };
        
        // Генерируем уникальный ID пользователя
        function generateUserId() {
            return 'user_' + Math.random().toString(36).substr(2, 9) + '_' + Date.now();
        }
        
        let myUserId = localStorage.getItem('p2p_user_id');
        if (!myUserId) {
            myUserId = generateUserId();
            localStorage.setItem('p2p_user_id', myUserId);
        }
        
        document.getElementById('myUserId').textContent = myUserId;
        
        // Копирование ID
        document.getElementById('copyIdBtn').onclick = () => {
            navigator.clipboard.writeText(myUserId);
            alert('ID скопирован!');
        };
        
        // Хранилище соединений
        const connections = new Map(); // key: userId, value: { peerConnection, dataChannel, status }
        let activeChat = null;
        let messageHandlers = new Map();
        
        // Signaling через BroadcastChannel (для одного домена/вкладки)
        // Для разных пользователей используем localStorage события
        // Это позволяет работать без внешнего сервера в пределах одного домена
        const signalingChannel = new BroadcastChannel('p2p_signaling');
        
        // Отправка сигнала
        function sendSignal(targetUserId, signal) {
            const signalData = {
                from: myUserId,
                to: targetUserId,
                signal: signal,
                timestamp: Date.now()
            };
            signalingChannel.postMessage(signalData);
            
            // Также сохраняем в localStorage для обнаружения новых пользователей
            const pendingSignals = JSON.parse(localStorage.getItem('p2p_signals') || '{}');
            if (!pendingSignals[targetUserId]) pendingSignals[targetUserId] = [];
            pendingSignals[targetUserId].push(signalData);
            localStorage.setItem('p2p_signals', JSON.stringify(pendingSignals));
        }
        
        // Обработка входящих сигналов
        signalingChannel.onmessage = (event) => {
            const { from, to, signal } = event.data;
            if (to === myUserId) {
                handleSignal(from, signal);
            }
        };
        
        // Проверка непрочитанных сигналов при загрузке
        function checkPendingSignals() {
            const pendingSignals = JSON.parse(localStorage.getItem('p2p_signals') || '{}');
            for (const [fromUserId, signals] of Object.entries(pendingSignals)) {
                signals.forEach(signalData => {
                    if (signalData.to === myUserId) {
                        handleSignal(fromUserId, signalData.signal);
                    }
                });
            }
            localStorage.removeItem('p2p_signals');
        }
        
        // Обработка сигналов WebRTC
        async function handleSignal(fromUserId, signal) {
            let connection = connections.get(fromUserId);
            
            if (!connection) {
                // Создаем новое соединение
                connection = await createPeerConnection(fromUserId);
                connections.set(fromUserId, connection);
                updateConnectionsList();
            }
            
            if (signal.type === 'offer') {
                await connection.peerConnection.setRemoteDescription(new RTCSessionDescription(signal));
                const answer = await connection.peerConnection.createAnswer();
                await connection.peerConnection.setLocalDescription(answer);
                sendSignal(fromUserId, connection.peerConnection.localDescription);
            } else if (signal.type === 'answer') {
                await connection.peerConnection.setRemoteDescription(new RTCSessionDescription(signal));
            } else if (signal.type === 'candidate') {
                await connection.peerConnection.addIceCandidate(new RTCIceCandidate(signal.candidate));
            }
        }
        
        // Создание peer connection
        async function createPeerConnection(remoteUserId) {
            const peerConnection = new RTCPeerConnection(configuration);
            const dataChannel = peerConnection.createDataChannel('chat');
            
            dataChannel.onopen = () => {
                console.log('Data channel opened with', remoteUserId);
                const conn = connections.get(remoteUserId);
                if (conn) {
                    conn.status = 'connected';
                    updateConnectionsList();
                    
                    // Автоматически открываем чат при подключении
                    if (!activeChat) {
                        setActiveChat(remoteUserId);
                    }
                }
            };
            
            dataChannel.onmessage = (event) => {
                const message = JSON.parse(event.data);
                receiveMessage(remoteUserId, message);
            };
            
            dataChannel.onclose = () => {
                console.log('Data channel closed with', remoteUserId);
                const conn = connections.get(remoteUserId);
                if (conn) {
                    conn.status = 'disconnected';
                    updateConnectionsList();
                }
            };
            
            peerConnection.ondatachannel = (event) => {
                const channel = event.channel;
                channel.onmessage = (e) => {
                    const message = JSON.parse(e.data);
                    receiveMessage(remoteUserId, message);
                };
                channel.onopen = () => {
                    const conn = connections.get(remoteUserId);
                    if (conn) {
                        conn.status = 'connected';
                        updateConnectionsList();
                    }
                };
                connections.get(remoteUserId).dataChannel = channel;
            };
            
            peerConnection.onicecandidate = (event) => {
                if (event.candidate) {
                    sendSignal(remoteUserId, {
                        type: 'candidate',
                        candidate: event.candidate
                    });
                }
            };
            
            return {
                peerConnection,
                dataChannel,
                status: 'connecting',
                messages: []
            };
        }
        
        // Инициирование соединения
        document.getElementById('connectBtn').onclick = async () => {
            const remoteUserId = document.getElementById('remoteUserId').value.trim();
            if (!remoteUserId || remoteUserId === myUserId) {
                alert('Введите корректный ID собеседника (не свой собственный)');
                return;
            }
            
            if (connections.has(remoteUserId)) {
                alert('Соединение уже существует или устанавливается');
                return;
            }
            
            const connection = await createPeerConnection(remoteUserId);
            connections.set(remoteUserId, connection);
            
            // Создаем offer
            const offer = await connection.peerConnection.createOffer();
            await connection.peerConnection.setLocalDescription(offer);
            sendSignal(remoteUserId, offer);
            
            updateConnectionsList();
        };
        
        // Обновление списка соединений в UI
        function updateConnectionsList() {
            const container = document.getElementById('connectionsList');
            if (connections.size === 0) {
                container.innerHTML = '<div style="color: #999; text-align: center;">Нет активных чатов<br>Подключитесь к кому-нибудь</div>';
                return;
            }
            
            let html = '';
            for (const [userId, conn] of connections) {
                const statusText = conn.status === 'connected' ? '🟢 Онлайн' : '🟡 Подключение...';
                const activeClass = activeChat === userId ? 'active' : '';
                html += `
                    <div class="connection-item ${activeClass}" onclick="setActiveChat('${userId}')">
                        <div class="connection-name">${userId}</div>
                        <div class="connection-status">${statusText}</div>
                    </div>
                `;
            }
            container.innerHTML = html;
        }
        
        // Установка активного чата
        window.setActiveChat = function(userId) {
            activeChat = userId;
            updateConnectionsList();
            
            const conn = connections.get(userId);
            if (conn) {
                document.getElementById('currentChatTitle').textContent = `Чат с: ${userId}`;
                const messageInput = document.getElementById('messageInput');
                const sendBtn = document.getElementById('sendBtn');
                
                if (conn.status === 'connected') {
                    messageInput.disabled = false;
                    sendBtn.disabled = false;
                } else {
                    messageInput.disabled = true;
                    sendBtn.disabled = true;
                }
                
                // Отображаем историю сообщений
                displayMessages(userId, conn.messages);
            }
        };
        
        // Отображение сообщений
        function displayMessages(userId, messages) {
            const messagesContainer = document.getElementById('messages');
            if (!messages || messages.length === 0) {
                messagesContainer.innerHTML = '<div class="empty-state">💬 Нет сообщений<br>Напишите что-нибудь</div>';
                return;
            }
            
            let html = '';
            messages.forEach(msg => {
                const isSent = msg.sender === myUserId;
                const time = new Date(msg.timestamp).toLocaleTimeString();
                html += `
                    <div class="message ${isSent ? 'sent' : 'received'}">
                        <div class="message-bubble">${escapeHtml(msg.text)}</div>
                        <div class="message-time">${time}</div>
                    </div>
                `;
            });
            messagesContainer.innerHTML = html;
            messagesContainer.scrollTop = messagesContainer.scrollHeight;
        }
        
        // Получение сообщения
        function receiveMessage(fromUserId, message) {
            const conn = connections.get(fromUserId);
            if (conn) {
                conn.messages.push({
                    text: message.text,
                    sender: fromUserId,
                    timestamp: message.timestamp
                });
                
                if (activeChat === fromUserId) {
                    displayMessages(fromUserId, conn.messages);
                }
                
                // Уведомление
                if (activeChat !== fromUserId && Notification.permission === 'granted') {
                    new Notification('Новое сообщение', {
                        body: `От ${fromUserId}: ${message.text}`,
                        icon: 'https://cdn-icons-png.flaticon.com/512/565/565422.png'
                    });
                }
            }
        }
        
        // Отправка сообщения
        document.getElementById('sendBtn').onclick = () => {
            sendMessage();
        };
        
        document.getElementById('messageInput').onkeypress = (e) => {
            if (e.key === 'Enter') {
                sendMessage();
            }
        };
        
        function sendMessage() {
            if (!activeChat) {
                alert('Выберите чат');
                return;
            }
            
            const conn = connections.get(activeChat);
            if (!conn || conn.status !== 'connected') {
                alert('Соединение не установлено');
                return;
            }
            
            const messageInput = document.getElementById('messageInput');
            const text = messageInput.value.trim();
            if (!text) return;
            
            const message = {
                text: text,
                sender: myUserId,
                timestamp: Date.now()
            };
            
            // Отправляем через data channel
            if (conn.dataChannel && conn.dataChannel.readyState === 'open') {
                conn.dataChannel.send(JSON.stringify(message));
                
                // Сохраняем в локальной истории
                conn.messages.push({
                    text: text,
                    sender: myUserId,
                    timestamp: message.timestamp
                });
                
                displayMessages(activeChat, conn.messages);
                messageInput.value = '';
            } else {
                alert('Канал не готов');
            }
        }
        
        // Функция для экранирования HTML
        function escapeHtml(text) {
            const div = document.createElement('div');
            div.textContent = text;
            return div.innerHTML;
        }
        
        // Запрос уведомлений
        if ('Notification' in window && Notification.permission !== 'granted' && Notification.permission !== 'denied') {
            Notification.requestPermission();
        }
        
        // Проверяем непрочитанные сигналы при загрузке
        checkPendingSignals();
        
        // Периодически очищаем старые сигналы
        setInterval(() => {
            const pendingSignals = JSON.parse(localStorage.getItem('p2p_signals') || '{}');
            let changed = false;
            for (const [userId, signals] of Object.entries(pendingSignals)) {
                // Удаляем сигналы старше 5 минут
                const filtered = signals.filter(s => Date.now() - s.timestamp < 300000);
                if (filtered.length !== signals.length) {
                    pendingSignals[userId] = filtered;
                    changed = true;
                }
            }
            if (changed) {
                localStorage.setItem('p2p_signals', JSON.stringify(pendingSignals));
            }
        }, 60000);
        
        // Отображение инструкции
        console.log('P2P Мессенджер запущен! Ваш ID:', myUserId);
    </script>
</body>
</html>
