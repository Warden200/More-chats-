<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>P2P Мессенджер - Улучшенная версия</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif;
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
            max-width: 1300px;
            height: 90vh;
            display: flex;
            overflow: hidden;
        }

        .sidebar {
            width: 320px;
            background: #f8f9fa;
            border-right: 1px solid #e0e0e0;
            display: flex;
            flex-direction: column;
            overflow-y: auto;
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

        input, select {
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
            transition: all 0.2s;
        }

        button:hover {
            background: #5a67d8;
            transform: translateY(-1px);
        }

        .status-badge {
            display: inline-block;
            padding: 4px 8px;
            border-radius: 12px;
            font-size: 11px;
            font-weight: 600;
            margin-top: 8px;
        }

        .status-connected { background: #48bb78; color: white; }
        .status-connecting { background: #ed8936; color: white; }
        .status-failed { background: #e53e3e; color: white; }

        .connections-list {
            flex: 1;
            padding: 20px;
        }

        .connection-item {
            background: white;
            padding: 12px;
            border-radius: 10px;
            margin-bottom: 10px;
            cursor: pointer;
            border: 2px solid transparent;
            transition: all 0.2s;
        }

        .connection-item.active {
            border-color: #667eea;
            background: #f0f4ff;
        }

        .chat-area {
            flex: 1;
            display: flex;
            flex-direction: column;
        }

        .chat-header {
            padding: 20px;
            background: white;
            border-bottom: 1px solid #e0e0e0;
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
            max-width: 70%;
            display: flex;
            flex-direction: column;
        }

        .message.sent { align-self: flex-end; }
        .message.received { align-self: flex-start; }

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

        .message-input-area {
            padding: 20px;
            border-top: 1px solid #e0e0e0;
            display: flex;
            gap: 10px;
        }

        .message-input-area input {
            margin-bottom: 0;
        }

        .message-input-area button {
            width: auto;
            padding: 10px 20px;
        }

        .debug-panel {
            background: #2d3748;
            color: #a0aec0;
            padding: 10px;
            font-size: 11px;
            font-family: monospace;
            border-top: 1px solid #4a5568;
            max-height: 150px;
            overflow-y: auto;
        }

        .empty-state {
            text-align: center;
            color: #999;
            padding: 40px;
        }

        @media (max-width: 768px) {
            .sidebar { width: 280px; }
            .message { max-width: 85%; }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="sidebar">
            <div class="user-info">
                <strong>🆔 Ваш ID:</strong>
                <div class="user-id" id="myUserId">Загрузка...</div>
                <button id="copyIdBtn" style="background: #48bb78; margin-top: 8px;">📋 Копировать</button>
                <div id="connectionStatus"></div>
            </div>
            
            <div class="connection-panel">
                <input type="text" id="remoteUserId" placeholder="ID собеседника">
                <select id="signalingMethod">
                    <option value="auto">🔄 Авто (рекомендуется)</option>
                    <option value="local">💻 Локальный (BroadcastChannel)</option>
                    <option value="manual">📋 Ручной обмен (копировать/вставить)</option>
                </select>
                <button id="connectBtn">🔗 Подключиться</button>
                <button id="retryBtn" style="background: #ed8936; margin-top: 5px;">🔄 Повторить подключение</button>
            </div>
            
            <div class="connections-list">
                <div style="font-weight: 600; margin-bottom: 10px;">💬 Активные чаты:</div>
                <div id="connectionsList"></div>
            </div>
        </div>
        
        <div class="chat-area">
            <div class="chat-header">
                <h3 id="currentChatTitle">📡 Выберите чат</h3>
            </div>
            <div class="messages" id="messages">
                <div class="empty-state">💡 Совет: используйте ручной режим, если автоматическое подключение не работает</div>
            </div>
            <div class="message-input-area">
                <input type="text" id="messageInput" placeholder="Сообщение..." disabled>
                <button id="sendBtn" disabled>📤 Отправить</button>
            </div>
            <div class="debug-panel" id="debugPanel">
                [Система] Мессенджер запущен. Ваш ID сгенерирован.<br>
            </div>
        </div>
    </div>

    <script>
        // Расширенная конфигурация ICE с множеством серверов
        const configuration = {
            iceServers: [
                { urls: 'stun:stun.l.google.com:19302' },
                { urls: 'stun:stun1.l.google.com:19302' },
                { urls: 'stun:stun2.l.google.com:19302' },
                { urls: 'stun:stun3.l.google.com:19302' },
                { urls: 'stun:stun4.l.google.com:19302' },
                { urls: 'stun:stun.stunprotocol.org:3478' },
                {
                    urls: 'turn:openrelay.metered.ca:80',
                    username: 'openrelayproject',
                    credential: 'openrelayproject'
                },
                {
                    urls: 'turn:openrelay.metered.ca:443',
                    username: 'openrelayproject',
                    credential: 'openrelayproject'
                }
            ],
            iceCandidatePoolSize: 10
        };
        
        let myUserId = localStorage.getItem('p2p_user_id_v2');
        if (!myUserId) {
            myUserId = 'user_' + Math.random().toString(36).substr(2, 9) + '_' + Date.now();
            localStorage.setItem('p2p_user_id_v2', myUserId);
        }
        
        document.getElementById('myUserId').textContent = myUserId;
        
        // Логирование
        function addDebugLog(message, type = 'info') {
            const debugPanel = document.getElementById('debugPanel');
            const time = new Date().toLocaleTimeString();
            const colors = { info: '#a0aec0', success: '#48bb78', error: '#f56565', warning: '#ed8936' };
            debugPanel.innerHTML += `<div style="color: ${colors[type] || colors.info}">[${time}] ${message}</div>`;
            debugPanel.scrollTop = debugPanel.scrollHeight;
            console.log(`[${type}]`, message);
        }
        
        // Копирование ID
        document.getElementById('copyIdBtn').onclick = () => {
            navigator.clipboard.writeText(myUserId);
            addDebugLog('✅ ID скопирован в буфер обмена', 'success');
        };
        
        // Хранилище соединений
        const connections = new Map();
        let activeChat = null;
        
        // Сигналинг через разные методы
        let signalingMethod = 'auto';
        let pendingOffers = new Map(); // для ручного режима
        
        // Обновление статуса подключения в UI
        function updateConnectionStatus(userId, status, message = '') {
            const conn = connections.get(userId);
            if (conn) {
                conn.status = status;
                const statusDiv = document.getElementById(`status_${userId}`);
                if (statusDiv) {
                    const statusText = status === 'connected' ? '🟢 Онлайн' : (status === 'connecting' ? '🟡 Подключение...' : '🔴 Ошибка');
                    statusDiv.innerHTML = statusText;
                    if (message) statusDiv.title = message;
                }
                updateConnectionsList();
                
                // Обновляем активный чат
                if (activeChat === userId && status === 'connected') {
                    document.getElementById('messageInput').disabled = false;
                    document.getElementById('sendBtn').disabled = false;
                } else if (activeChat === userId && status !== 'connected') {
                    document.getElementById('messageInput').disabled = true;
                    document.getElementById('sendBtn').disabled = true;
                }
            }
        }
        
        // Создание PeerConnection
        async function createPeerConnection(remoteUserId, isInitiator = true) {
            addDebugLog(`🔧 Создание соединения с ${remoteUserId} (инициатор: ${isInitiator})`, 'info');
            
            const peerConnection = new RTCPeerConnection(configuration);
            let dataChannel = null;
            
            if (isInitiator) {
                dataChannel = peerConnection.createDataChannel('chat');
                setupDataChannel(dataChannel, remoteUserId);
            }
            
            peerConnection.ondatachannel = (event) => {
                addDebugLog(`📡 Получен data channel от ${remoteUserId}`, 'success');
                dataChannel = event.channel;
                setupDataChannel(dataChannel, remoteUserId);
            };
            
            peerConnection.onicecandidate = (event) => {
                if (event.candidate) {
                    addDebugLog(`🔌 ICE кандидат для ${remoteUserId}: ${event.candidate.candidate.substring(0, 50)}...`, 'info');
                    sendSignal(remoteUserId, {
                        type: 'candidate',
                        candidate: event.candidate
                    });
                } else {
                    addDebugLog(`✅ ICE gathering завершён для ${remoteUserId}`, 'success');
                }
            };
            
            peerConnection.oniceconnectionstatechange = () => {
                addDebugLog(`🔄 ICE состояние для ${remoteUserId}: ${peerConnection.iceConnectionState}`, 'info');
                if (peerConnection.iceConnectionState === 'connected') {
                    updateConnectionStatus(remoteUserId, 'connected');
                    addDebugLog(`🎉 Соединение с ${remoteUserId} установлено!`, 'success');
                } else if (peerConnection.iceConnectionState === 'failed') {
                    updateConnectionStatus(remoteUserId, 'failed', 'ICE соединение не удалось');
                    addDebugLog(`❌ Ошибка ICE для ${remoteUserId}`, 'error');
                } else if (peerConnection.iceConnectionState === 'disconnected') {
                    updateConnectionStatus(remoteUserId, 'connecting');
                }
            };
            
            function setupDataChannel(channel, userId) {
                channel.onopen = () => {
                    addDebugLog(`🔓 Data channel открыт с ${userId}`, 'success');
                    updateConnectionStatus(userId, 'connected');
                };
                channel.onclose = () => {
                    addDebugLog(`🔒 Data channel закрыт с ${userId}`, 'warning');
                    updateConnectionStatus(userId, 'connecting');
                };
                channel.onmessage = (event) => {
                    const message = JSON.parse(event.data);
                    receiveMessage(userId, message);
                };
            }
            
            return { peerConnection, dataChannel };
        }
        
        // Отправка сигнала (разные методы)
        function sendSignal(targetUserId, signal) {
            const method = document.getElementById('signalingMethod').value;
            
            if (method === 'manual') {
                // Ручной режим - показываем SDP для копирования
                if (signal.type === 'offer' || signal.type === 'answer') {
                    const sdp = JSON.stringify({ from: myUserId, to: targetUserId, signal });
                    addDebugLog(`📋 Скопируйте этот SDP и отправьте собеседнику:`, 'warning');
                    addDebugLog(`${sdp.substring(0, 200)}...`, 'info');
                    
                    // Сохраняем для ручного обмена
                    const manualData = {
                        type: signal.type,
                        sdp: signal.sdp,
                        from: myUserId
                    };
                    localStorage.setItem('p2p_manual_signal', JSON.stringify(manualData));
                    addDebugLog(`💾 Данные сохранены в localStorage (ключ: p2p_manual_signal)`, 'info');
                }
            } else {
                // Автоматический режим через BroadcastChannel
                const signalData = { from: myUserId, to: targetUserId, signal, timestamp: Date.now() };
                const channel = new BroadcastChannel('p2p_signaling_v2');
                channel.postMessage(signalData);
                channel.close();
            }
        }
        
        // Инициирование соединения
        async function initiateConnection(remoteUserId) {
            if (connections.has(remoteUserId)) {
                addDebugLog(`⚠️ Соединение с ${remoteUserId} уже существует`, 'warning');
                return;
            }
            
            addDebugLog(`🚀 Инициируем подключение к ${remoteUserId}...`, 'info');
            
            const { peerConnection, dataChannel } = await createPeerConnection(remoteUserId, true);
            connections.set(remoteUserId, {
                peerConnection,
                dataChannel,
                status: 'connecting',
                messages: []
            });
            updateConnectionsList();
            
            // Создаём offer
            const offer = await peerConnection.createOffer();
            await peerConnection.setLocalDescription(offer);
            sendSignal(remoteUserId, offer);
            addDebugLog(`📤 Offer отправлен для ${remoteUserId}`, 'success');
            
            // Таймаут подключения
            setTimeout(() => {
                const conn = connections.get(remoteUserId);
                if (conn && conn.status !== 'connected') {
                    addDebugLog(`⏰ Таймаут подключения к ${remoteUserId}`, 'error');
                    updateConnectionStatus(remoteUserId, 'failed', 'Таймаут подключения');
                }
            }, 30000);
        }
        
        // Обработка входящих сигналов
        async function handleIncomingSignal(fromUserId, signal) {
            addDebugLog(`📨 Получен сигнал от ${fromUserId}: ${signal.type}`, 'info');
            
            let connection = connections.get(fromUserId);
            
            if (!connection && signal.type === 'offer') {
                addDebugLog(`🔄 Создаём ответное соединение для ${fromUserId}`, 'info');
                const { peerConnection, dataChannel } = await createPeerConnection(fromUserId, false);
                connection = { peerConnection, dataChannel, status: 'connecting', messages: [] };
                connections.set(fromUserId, connection);
                updateConnectionsList();
            }
            
            if (!connection) {
                addDebugLog(`❌ Нет соединения для ${fromUserId}`, 'error');
                return;
            }
            
            try {
                if (signal.type === 'offer') {
                    await connection.peerConnection.setRemoteDescription(new RTCSessionDescription(signal));
                    const answer = await connection.peerConnection.createAnswer();
                    await connection.peerConnection.setLocalDescription(answer);
                    sendSignal(fromUserId, answer);
                    addDebugLog(`📤 Answer отправлен для ${fromUserId}`, 'success');
                } else if (signal.type === 'answer') {
                    await connection.peerConnection.setRemoteDescription(new RTCSessionDescription(signal));
                    addDebugLog(`✅ Remote description установлен для ${fromUserId}`, 'success');
                } else if (signal.type === 'candidate') {
                    await connection.peerConnection.addIceCandidate(new RTCIceCandidate(signal.candidate));
                    addDebugLog(`🧊 ICE кандидат добавлен для ${fromUserId}`, 'success');
                }
            } catch (error) {
                addDebugLog(`❌ Ошибка обработки сигнала: ${error.message}`, 'error');
            }
        }
        
        // Слушаем сигналы
        function startSignalingListener() {
            const channel = new BroadcastChannel('p2p_signaling_v2');
            channel.onmessage = (event) => {
                const { from, to, signal } = event.data;
                if (to === myUserId) {
                    handleIncomingSignal(from, signal);
                }
            };
            addDebugLog(`📡 Слушатель сигналов запущен`, 'success');
        }
        
        // Ручной режим - вставка SDP
        function setupManualMode() {
            const manualInput = document.createElement('textarea');
            manualInput.placeholder = 'Вставьте SDP от собеседника здесь...';
            manualInput.style.width = '100%';
            manualInput.style.marginTop = '10px';
            manualInput.style.padding = '8px';
            manualInput.style.fontSize = '11px';
            manualInput.style.fontFamily = 'monospace';
            manualInput.rows = 3;
            
            const manualBtn = document.createElement('button');
            manualBtn.textContent = '📥 Применить SDP';
            manualBtn.style.marginTop = '5px';
            manualBtn.style.background = '#4299e1';
            
            const panel = document.querySelector('.connection-panel');
            panel.appendChild(manualInput);
            panel.appendChild(manualBtn);
            
            manualBtn.onclick = async () => {
                try {
                    const data = JSON.parse(manualInput.value);
                    if (data.from && data.signal) {
                        await handleIncomingSignal(data.from, data.signal);
                        addDebugLog(`✅ SDP применён успешно`, 'success');
                        manualInput.value = '';
                    } else {
                        addDebugLog(`❌ Неверный формат SDP`, 'error');
                    }
                } catch (e) {
                    addDebugLog(`❌ Ошибка парсинга SDP: ${e.message}`, 'error');
                }
            };
        }
        
        // Кнопки
        document.getElementById('connectBtn').onclick = async () => {
            const remoteUserId = document.getElementById('remoteUserId').value.trim();
            if (!remoteUserId || remoteUserId === myUserId) {
                addDebugLog(`❌ Введите корректный ID (не свой)`, 'error');
                return;
            }
            await initiateConnection(remoteUserId);
        };
        
        document.getElementById('retryBtn').onclick = async () => {
            const remoteUserId = document.getElementById('remoteUserId').value.trim();
            if (!remoteUserId) {
                addDebugLog(`❌ Введите ID для повторного подключения`, 'error');
                return;
            }
            
            const existing = connections.get(remoteUserId);
            if (existing) {
                existing.peerConnection.close();
                connections.delete(remoteUserId);
                addDebugLog(`🔄 Старое соединение закрыто`, 'info');
            }
            await initiateConnection(remoteUserId);
        };
        
        // Обновление списка
        function updateConnectionsList() {
            const container = document.getElementById('connectionsList');
            if (connections.size === 0) {
                container.innerHTML = '<div style="color: #999; text-align: center;">Нет активных чатов<br>Подключитесь к кому-нибудь</div>';
                return;
            }
            
            let html = '';
            for (const [userId, conn] of connections) {
                const statusText = conn.status === 'connected' ? '🟢 Онлайн' : (conn.status === 'connecting' ? '🟡 Подключение...' : '🔴 Ошибка');
                const activeClass = activeChat === userId ? 'active' : '';
                html += `
                    <div class="connection-item ${activeClass}" onclick="window.setActiveChat('${userId}')">
                        <div class="connection-name">${userId}</div>
                        <div id="status_${userId}" class="status-badge status-${conn.status === 'connected' ? 'connected' : (conn.status === 'connecting' ? 'connecting' : 'failed')}">${statusText}</div>
                    </div>
                `;
            }
            container.innerHTML = html;
        }
        
        window.setActiveChat = function(userId) {
            activeChat = userId;
            updateConnectionsList();
            document.getElementById('currentChatTitle').textContent = `💬 Чат с: ${userId.substring(0, 20)}...`;
            
            const conn = connections.get(userId);
            if (conn) {
                displayMessages(userId, conn.messages);
                const canSend = conn.status === 'connected';
                document.getElementById('messageInput').disabled = !canSend;
                document.getElementById('sendBtn').disabled = !canSend;
            }
        };
        
        function displayMessages(userId, messages) {
            const container = document.getElementById('messages');
            if (!messages || messages.length === 0) {
                container.innerHTML = '<div class="empty-state">💬 Нет сообщений<br>Напишите что-нибудь</div>';
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
            container.innerHTML = html;
            container.scrollTop = container.scrollHeight;
        }
        
        function receiveMessage(fromUserId, message) {
            const conn = connections.get(fromUserId);
            if (conn) {
                conn.messages.push(message);
                if (activeChat === fromUserId) {
                    displayMessages(fromUserId, conn.messages);
                }
                addDebugLog(`💬 Новое сообщение от ${fromUserId}: ${message.text.substring(0, 50)}`, 'info');
            }
        }
        
        document.getElementById('sendBtn').onclick = sendMessage;
        document.getElementById('messageInput').onkeypress = (e) => {
            if (e.key === 'Enter') sendMessage();
        };
        
        function sendMessage() {
            if (!activeChat) return;
            const conn = connections.get(activeChat);
            if (!conn || conn.status !== 'connected' || !conn.dataChannel || conn.dataChannel.readyState !== 'open') {
                addDebugLog(`❌ Канал не готов для отправки`, 'error');
                return;
            }
            
            const input = document.getElementById('messageInput');
            const text = input.value.trim();
            if (!text) return;
            
            const message = { text, sender: myUserId, timestamp: Date.now() };
            conn.dataChannel.send(JSON.stringify(message));
            conn.messages.push(message);
            displayMessages(activeChat, conn.messages);
            input.value = '';
            addDebugLog(`📤 Сообщение отправлено ${activeChat}`, 'success');
        }
        
        function escapeHtml(text) {
            const div = document.createElement('div');
            div.textContent = text;
            return div.innerHTML;
        }
        
        // Инициализация
        startSignalingListener();
        setupManualMode();
        
        // Проверка соединения
        setInterval(() => {
            for (const [userId, conn] of connections) {
                if (conn.dataChannel && conn.dataChannel.readyState === 'open' && conn.status !== 'connected') {
                    updateConnectionStatus(userId, 'connected');
                } else if (conn.dataChannel && conn.dataChannel.readyState !== 'open' && conn.status === 'connected') {
                    updateConnectionStatus(userId, 'connecting');
                }
            }
        }, 3000);
        
        addDebugLog(`✅ Мессенджер запущен! Ваш ID: ${myUserId}`, 'success');
        addDebugLog(`💡 Если подключение зависает - переключитесь на "Ручной обмен"`, 'warning');
    </script>
</body>
</html>
