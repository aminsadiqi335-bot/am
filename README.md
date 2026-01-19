<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chatbot Premium</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
        }

        /* Botón flotante con pulso */
        #chat-bubble {
            position: fixed;
            bottom: 24px;
            right: 24px;
            width: 70px;
            height: 70px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            border-radius: 50%;
            cursor: pointer;
            box-shadow: 0 8px 32px rgba(102, 126, 234, 0.4);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 10000;
            transition: all 0.3s cubic-bezier(0.68, -0.55, 0.265, 1.55);
            animation: pulse 2s infinite;
        }

        #chat-bubble:hover {
            transform: scale(1.1);
            box-shadow: 0 12px 40px rgba(102, 126, 234, 0.6);
        }

        #chat-bubble.active {
            transform: scale(0.9);
        }

        @keyframes pulse {
            0%, 100% {
                box-shadow: 0 8px 32px rgba(102, 126, 234, 0.4);
            }
            50% {
                box-shadow: 0 8px 32px rgba(102, 126, 234, 0.8);
            }
        }

        #chat-bubble svg {
            width: 32px;
            height: 32px;
            fill: white;
            transition: transform 0.3s ease;
        }

        #chat-bubble.active svg {
            transform: rotate(90deg);
        }

        /* Notificación de nuevo mensaje */
        .notification-badge {
            position: absolute;
            top: -4px;
            right: -4px;
            width: 24px;
            height: 24px;
            background: #ff4757;
            border-radius: 50%;
            display: none;
            align-items: center;
            justify-content: center;
            font-size: 12px;
            font-weight: bold;
            color: white;
            animation: bounce 0.5s ease;
        }

        @keyframes bounce {
            0%, 100% { transform: scale(1); }
            50% { transform: scale(1.2); }
        }

        /* Ventana del chat */
        #chat-container {
            position: fixed;
            bottom: 110px;
            right: 24px;
            width: 420px;
            height: 680px;
            background: #ffffff;
            border-radius: 24px;
            box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
            display: none;
            flex-direction: column;
            overflow: hidden;
            z-index: 9999;
            animation: slideUp 0.4s cubic-bezier(0.68, -0.55, 0.265, 1.55);
        }

        #chat-container.active {
            display: flex;
        }

        @keyframes slideUp {
            from {
                opacity: 0;
                transform: translateY(30px) scale(0.95);
            }
            to {
                opacity: 1;
                transform: translateY(0) scale(1);
            }
        }

        /* Header con glassmorphism */
        #chat-header {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            padding: 24px;
            position: relative;
            overflow: hidden;
        }

        #chat-header::before {
            content: '';
            position: absolute;
            top: -50%;
            right: -50%;
            width: 200%;
            height: 200%;
            background: radial-gradient(circle, rgba(255,255,255,0.1) 0%, transparent 70%);
            animation: rotate 20s linear infinite;
        }

        @keyframes rotate {
            from { transform: rotate(0deg); }
            to { transform: rotate(360deg); }
        }

        #chat-header-content {
            position: relative;
            z-index: 1;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .header-info {
            display: flex;
            align-items: center;
            gap: 12px;
        }

        .bot-avatar {
            width: 48px;
            height: 48px;
            background: rgba(255, 255, 255, 0.2);
            backdrop-filter: blur(10px);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            border: 2px solid rgba(255, 255, 255, 0.3);
        }

        .bot-avatar svg {
            width: 24px;
            height: 24px;
            fill: white;
        }

        .header-text h3 {
            color: white;
            font-size: 18px;
            font-weight: 600;
            margin-bottom: 4px;
        }

        .status {
            display: flex;
            align-items: center;
            gap: 6px;
            color: rgba(255, 255, 255, 0.9);
            font-size: 13px;
        }

        .status-dot {
            width: 8px;
            height: 8px;
            background: #00ff88;
            border-radius: 50%;
            animation: blink 1.5s infinite;
        }

        @keyframes blink {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.3; }
        }

        #close-chat {
            background: rgba(255, 255, 255, 0.2);
            backdrop-filter: blur(10px);
            border: none;
            width: 36px;
            height: 36px;
            border-radius: 50%;
            color: white;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: all 0.3s ease;
        }

        #close-chat:hover {
            background: rgba(255, 255, 255, 0.3);
            transform: rotate(90deg);
        }

        /* Mensajes */
        #chat-messages {
            flex: 1;
            overflow-y: auto;
            padding: 24px;
            background: linear-gradient(to bottom, #f8f9fa 0%, #e9ecef 100%);
            position: relative;
        }

        /* Scrollbar personalizado */
        #chat-messages::-webkit-scrollbar {
            width: 6px;
        }

        #chat-messages::-webkit-scrollbar-track {
            background: transparent;
        }

        #chat-messages::-webkit-scrollbar-thumb {
            background: rgba(102, 126, 234, 0.3);
            border-radius: 3px;
        }

        #chat-messages::-webkit-scrollbar-thumb:hover {
            background: rgba(102, 126, 234, 0.5);
        }

        .message {
            display: flex;
            margin-bottom: 20px;
            animation: fadeIn 0.3s ease;
        }

        @keyframes fadeIn {
            from {
                opacity: 0;
                transform: translateY(10px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        .message.bot {
            justify-content: flex-start;
        }

        .message.user {
            justify-content: flex-end;
        }

        .message-avatar {
            width: 36px;
            height: 36px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            flex-shrink: 0;
            margin-right: 12px;
            box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
        }

        .message-avatar svg {
            width: 18px;
            height: 18px;
            fill: white;
        }

        .message-content {
            max-width: 75%;
        }

        .message-bubble {
            padding: 14px 18px;
            border-radius: 18px;
            font-size: 15px;
            line-height: 1.5;
            position: relative;
        }

        .message.bot .message-bubble {
            background: white;
            color: #2d3748;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
            border-bottom-left-radius: 4px;
        }

        .message.user .message-bubble {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
            border-bottom-right-radius: 4px;
        }

        .message-time {
            font-size: 11px;
            opacity: 0.6;
            margin-top: 6px;
        }

        /* Typing indicator */
        .typing-indicator {
            display: none;
            padding: 14px 18px;
            background: white;
            border-radius: 18px;
            border-bottom-left-radius: 4px;
            width: fit-content;
            box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
        }

        .typing-indicator.active {
            display: block;
            animation: fadeIn 0.3s ease;
        }

        .typing-indicator span {
            height: 10px;
            width: 10px;
            background: #cbd5e0;
            border-radius: 50%;
            display: inline-block;
            margin: 0 2px;
            animation: typing 1.4s infinite;
        }

        .typing-indicator span:nth-child(2) {
            animation-delay: 0.2s;
        }

        .typing-indicator span:nth-child(3) {
            animation-delay: 0.4s;
        }

        @keyframes typing {
            0%, 60%, 100% {
                transform: translateY(0);
                background: #cbd5e0;
            }
            30% {
                transform: translateY(-10px);
                background: #667eea;
            }
        }

        /* Sugerencias rápidas */
        .quick-replies {
            display: flex;
            gap: 8px;
            padding: 0 24px 16px;
            overflow-x: auto;
            scrollbar-width: none;
        }

        .quick-replies::-webkit-scrollbar {
            display: none;
        }

        .quick-reply-btn {
            background: white;
            border: 2px solid #e2e8f0;
            padding: 10px 16px;
            border-radius: 20px;
            font-size: 13px;
            color: #4a5568;
            cursor: pointer;
            white-space: nowrap;
            transition: all 0.3s ease;
            font-weight: 500;
        }

        .quick-reply-btn:hover {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border-color: transparent;
            transform: translateY(-2px);
            box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
        }

        /* Input area */
        #chat-input-container {
            padding: 20px 24px;
            background: white;
            border-top: 1px solid #e2e8f0;
            display: flex;
            gap: 12px;
            align-items: center;
        }

        #chat-input {
            flex: 1;
            border: 2px solid #e2e8f0;
            border-radius: 24px;
            padding: 14px 20px;
            font-size: 15px;
            outline: none;
            transition: all 0.3s ease;
            font-family: inherit;
        }

        #chat-input:focus {
            border-color: #667eea;
            box-shadow: 0 0 0 4px rgba(102, 126, 234, 0.1);
        }

        #send-button {
            width: 48px;
            height: 48px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            border: none;
            border-radius: 50%;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: all 0.3s ease;
            box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
        }

        #send-button:hover {
            transform: scale(1.1);
            box-shadow: 0 6px 20px rgba(102, 126, 234, 0.4);
        }

        #send-button:active {
            transform: scale(0.95);
        }

        #send-button:disabled {
            opacity: 0.5;
            cursor: not-allowed;
            transform: scale(1);
        }

        #send-button svg {
            width: 20px;
            height: 20px;
            fill: white;
        }

        /* Responsive */
        @media (max-width: 480px) {
            #chat-container {
                width: calc(100vw - 32px);
                height: calc(100vh - 140px);
                right: 16px;
                bottom: 100px;
                border-radius: 20px;
            }

            #chat-bubble {
                width: 60px;
                height: 60px;
                right: 16px;
                bottom: 16px;
            }
        }

        /* Animación de envío */
        @keyframes sendPulse {
            0% {
                transform: scale(1);
            }
            50% {
                transform: scale(0.9);
            }
            100% {
                transform: scale(1);
            }
        }

        .sending {
            animation: sendPulse 0.3s ease;
        }
    </style>
</head>
<body>
    <!-- Tu contenido normal aquí -->
    <div style="max-width: 1200px; margin: 0 auto; padding: 40px 20px;">
        <h1>Ejemplo de Página Web</h1>
        <p>El chatbot aparecerá en la esquina inferior derecha. Haz clic para probarlo.</p>
    </div>

    <!-- Chatbot Widget -->
    <div id="chat-bubble">
        <span class="notification-badge">1</span>
        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
            <path d="M20 2H4c-1.1 0-1.99.9-1.99 2L2 22l4-4h14c1.1 0 2-.9 2-2V4c0-1.1-.9-2-2-2zM6 9h12v2H6V9zm8 5H6v-2h8v2zm4-6H6V6h12v2z"/>
        </svg>
    </div>

    <div id="chat-container">
        <!-- Header -->
        <div id="chat-header">
            <div id="chat-header-content">
                <div class="header-info">
                    <div class="bot-avatar">
                        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
                            <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 3c1.66 0 3 1.34 3 3s-1.34 3-3 3-3-1.34-3-3 1.34-3 3-3zm0 14.2c-2.5 0-4.71-1.28-6-3.22.03-1.99 4-3.08 6-3.08 1.99 0 5.97 1.09 6 3.08-1.29 1.94-3.5 3.22-6 3.22z"/>
                        </svg>
                    </div>
                    <div class="header-text">
                        <h3>Asistente Virtual</h3>
                        <div class="status">
                            <span class="status-dot"></span>
                            Online ahora
                        </div>
                    </div>
                </div>
                <button id="close-chat">✕</button>
            </div>
        </div>

        <!-- Messages -->
        <div id="chat-messages">
            <!-- Los mensajes se añadirán aquí -->
        </div>

        <!-- Quick Replies -->
        <div class="quick-replies" id="quick-replies">
            <button class="quick-reply-btn" data-message="¿Cuáles son tus horarios?">⏰ Horarios</button>
            <button class="quick-reply-btn" data-message="¿Cuánto cuesta?">💰 Precios</button>
            <button class="quick-reply-btn" data-message="Quiero hablar con alguien">👤 Contacto</button>
        </div>

        <!-- Typing Indicator -->
        <div class="message bot" style="margin: 0 24px 20px;">
            <div class="message-avatar">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
                    <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 3c1.66 0 3 1.34 3 3s-1.34 3-3 3-3-1.34-3-3 1.34-3 3-3zm0 14.2c-2.5 0-4.71-1.28-6-3.22.03-1.99 4-3.08 6-3.08 1.99 0 5.97 1.09 6 3.08-1.29 1.94-3.5 3.22-6 3.22z"/>
                </svg>
            </div>
            <div class="typing-indicator" id="typing-indicator">
                <span></span>
                <span></span>
                <span></span>
            </div>
        </div>

        <!-- Input -->
        <div id="chat-input-container">
            <input 
                type="text" 
                id="chat-input" 
                placeholder="Escribe tu mensaje..."
                autocomplete="off"
            />
            <button id="send-button">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
                    <path d="M2.01 21L23 12 2.01 3 2 10l15 2-15 2z"/>
                </svg>
            </button>
        </div>
    </div>

    <script>
        // ⚠️ CONFIGURA AQUÍ TU URL DE n8n
        const WEBHOOK_URL = 'PEGA_TU_URL_AQUI';

        // Variables globales
        let userId = localStorage.getItem('chat_user_id') || 'user_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
        localStorage.setItem('chat_user_id', userId);
        let isFirstMessage = true;

        // Elementos del DOM
        const chatBubble = document.getElementById('chat-bubble');
        const chatContainer = document.getElementById('chat-container');
        const closeChat = document.getElementById('close-chat');
        const chatInput = document.getElementById('chat-input');
        const sendButton = document.getElementById('send-button');
        const chatMessages = document.getElementById('chat-messages');
        const typingIndicator = document.getElementById('typing-indicator');
        const notificationBadge = document.querySelector('.notification-badge');

        // Abrir/Cerrar chat
        chatBubble.addEventListener('click', () => {
            chatContainer.classList.add('active');
            chatBubble.classList.add('active');
            notificationBadge.style.display = 'none';
            
            if (isFirstMessage) {
                setTimeout(() => {
                    addMessage('¡Hola! 👋 Soy tu asistente virtual. ¿En qué puedo ayudarte hoy?', 'bot');
                    isFirstMessage = false;
                }, 500);
            }
            
            chatInput.focus();
        });

        closeChat.addEventListener('click', () => {
            chatContainer.classList.remove('active');
            chatBubble.classList.remove('active');
        });

        // Quick replies
        document.querySelectorAll('.quick-reply-btn').forEach(btn => {
            btn.addEventListener('click', () => {
                const message = btn.getAttribute('data-message');
                chatInput.value = message;
                sendMessage();
            });
        });

        // Enviar mensaje
        sendButton.addEventListener('click', sendMessage);
        chatInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter' && !e.shiftKey) {
                e.preventDefault();
                sendMessage();
            }
        });

        async function sendMessage() {
            const message = chatInput.value.trim();
            if (!message) return;

            // Añadir mensaje del usuario
            addMessage(message, 'user');
            chatInput.value = '';
            sendButton.disabled = true;
            sendButton.classList.add('sending');

            // Mostrar typing indicator
            typingIndicator.classList.add('active');
            scrollToBottom();

            try {
                const response = await fetch(WEBHOOK_URL, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({
                        message: message,
                        user_id: userId,
                        session_id: userId,
                        page_url: window.location.href,
                        timestamp: new Date().toISOString()
                    })
                });

                const data = await response.json();

                // Ocultar typing indicator
                typingIndicator.classList.remove('active');

                // Añadir respuesta del bot
                if (data.success && data.message) {
                    setTimeout(() => {
                        addMessage(data.message, 'bot');
                    }, 300);
                } else {
                    addMessage('Lo siento, hubo un error. ¿Puedes intentar de nuevo?', 'bot');
                }

            } catch (error) {
                console.error('Error:', error);
                typingIndicator.classList.remove('active');
                addMessage('No pude conectarme. Por favor intenta más tarde.', 'bot');
            }

            sendButton.disabled = false;
            sendButton.classList.remove('sending');
            scrollToBottom();
        }

        function addMessage(text, sender) {
            const messageDiv = document.createElement('div');
            messageDiv.className = `message ${sender}`;

            const now = new Date();
            const time = now.toLocaleTimeString('es-ES', { hour: '2-digit', minute: '2-digit' });

            if (sender === 'bot') {
                messageDiv.innerHTML = `
                    <div class="message-avatar">
                        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24">
                            <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 3c1.66 0 3 1.34 3 3s-1.34 3-3 3-3-1.34-3-3 1.34-3 3-3zm0 14.2c-2.5 0-4.71-1.28-6-3.22.03-1.99 4-3.08 6-3.08 1.99 0 5.97 1.09 6 3.08-1.29 1.94-3.5 3.22-6 3.22z"/>
                        </svg>
                    </div>
                    <div class="message-content">
                        <div class="message-bubble">${text}</div>
                        <div class="message-time">${time}</div>
                    </div>
                `;
            } else {
                messageDiv.innerHTML = `
                    <div class="message-content">
                        <div class="message-bubble">${text}</div>
                        <div class="message-time" style="text-align: right;">${time}</div>
                    </div>
                `;
            }

            chatMessages.insertBefore(messageDiv, chatMessages.lastElementChild);
            scrollToBottom();

            // Sonido de notificación (opcional)
            if (sender === 'bot' && !chatContainer.classList.contains('active')) {
                notificationBadge.style.display = 'flex';
                // playNotificationSound(); // Descomentar si quieres sonido
            }
        }

        function scrollToBottom() {
            chatMessages.scrollTop = chatMessages.scrollHeight;
        }

        // Auto-resize textarea (opcional)
        chatInput.addEventListener('input', function() {
            this.style.height = 'auto';
            this.style.height = (this.scrollHeight) + 'px';
        });

        // Sonido de notificación (opcional)
        function playNotificationSound() {
            const audio = new Audio('data:audio/wav;base64,UklGRnoGAABXQVZFZm10IBAAAAABAAEAQB8AAEAfAAABAAgAZGF0YQoGAACBhYqFbF1fdJivrJBhNjVgodDbq2EcBj+a2/LDciUFLIHO8tiJNwgZaLvt559NEAxQp+PwtmMcBjiR1/LMeSwFJHfH8N2QQAoUXrTp66hVFApGn+DyvmwhBSuFzvLTjToEGGW67OmlUxALUKbu8LRjHAU2ktjyzHkpBSh+zPLaizsIGGS56+miURAMT6Xj8LNhGwU4k9jyz3otBid6y/LXizsFGWO46uihUBAOUKfu8LRiGwU4lNjy0HouBCR7zPLbizwJGGS56umhURAOUKju8LNiGwU5lNjy0HotBSR7zPLajDsKGGS56umhURAOT6ju8LRiGwU5ldjy0HotBSR7zPLajDsKGGO56umhURAOUKju8LRiGwU5lNjy0HotBSR7zPLajDsKGGO56umhURAOT6ju8LRiGwU5ldjy0HotBSR7zPLajDsKGGO56umhURAOUKju8LRiGwU5lNjy0HotBSR7zPLajDsKGGO56umhURAOUKju8LRiGwU5lNjy0HotBSR7zPLajDsKGGO56umhURAOUKju8LRiGwU5lNjy0HotBSR7zPLajDsKGGO56umhURAOUKju8LRiGwU5lNjy0HotBSR7zPLajDsKGGO56umhURAOUKju8LRiGwU5lNjy0HotBSR7zPLajDsKGGO56umhURAOUKju8LRiGwU5lNjy0HotBSR7zPLajDsKGGO56umhURAOUKju8LRiGwU5lNjy0HotBSR7zPLajDsKGGO56umhURAOUKju8LRiGwU5lNjy0HotBSR7zPLajDsKGGO56umhURAOUKju8LRiGwU5lNjy0HotBSR7zPLajDsKGGO56umhURAOUKju8LRiGwU5lNjy0HotBSR7zPLajDsKGGO56umhURAOUKju8LRiGwU5lNjy0HotBSR7zPLajDsKGGO56umhURAOUKju8LRiGwU5lNjy0HotBSR7zPLajDsKGGO56umhURANU');
            audio.play().catch(() => {}); // Ignore errors
        }
    </script>
</body>
</html>
