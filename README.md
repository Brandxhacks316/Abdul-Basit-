<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Abdul Basit Chat Portal</title>
    <style>
        /* === GLOBAL THEME === */
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Courier New', Courier, monospace; }
        body { background-color: #050505; color: #ff0033; overflow: hidden; height: 100vh; }
        
        /* Matrix Canvas */
        canvas#matrix { position: fixed; top: 0; left: 0; width: 100%; height: 100%; z-index: 0; opacity: 0.35; }
        
        /* Screen Management */
        .screen { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 10; display: none; flex-direction: column; align-items: center; justify-content: center; background: rgba(0, 0, 0, 0.85); }
        .active { display: flex !important; }

        /* === LOGIN SCREEN === */
        .owner-tag {
            color: #00ffff;
            text-shadow: 0 0 10px #00ffff, 0 0 20px #00ffff;
            font-size: 1.2rem;
            margin-bottom: 15px;
            font-weight: bold;
            letter-spacing: 2px;
        }
        
        .title-3d { 
            font-size: 3.5vw; 
            min-font-size: 30px;
            font-weight: 900; 
            color: #fff; 
            font-family: 'Arial Black', sans-serif;
            text-align: center;
            line-height: 1.2;
            margin-bottom: 50px;
            letter-spacing: 3px;
            text-shadow: 
                0 1px 0 #ff0033, 0 2px 0 #e6002e, 0 3px 0 #cc0029, 
                0 4px 0 #b30024, 0 5px 0 #99001f, 0 6px 0 #80001a,
                0 15px 30px rgba(255,0,51,0.6); 
        }

        .btn-group { display: flex; flex-direction: column; gap: 20px; }
        
        .login-btn { 
            padding: 15px 30px; 
            background: rgba(10, 10, 10, 0.8); 
            border: 2px solid #ff0033; 
            color: #ff0033; 
            font-size: 1.1rem; 
            cursor: pointer; 
            text-transform: uppercase; 
            font-weight: bold; 
            transition: 0.3s;
            box-shadow: 0 0 10px rgba(255, 0, 51, 0.2);
        }
        .login-btn:hover { background: #ff0033; color: #000; box-shadow: 0 0 25px #ff0033; }

        /* === CHAT UI (COMMON) === */
        .chat-container { width: 95%; max-width: 900px; height: 85vh; border: 1px solid #ff0033; background: rgba(10, 10, 10, 0.95); display: flex; flex-direction: column; position: relative; box-shadow: 0 0 25px rgba(255, 0, 51, 0.2); }
        .chat-header { padding: 15px; border-bottom: 1px solid #ff0033; text-align: center; font-weight: bold; background: #111; color: #fff; display: flex; justify-content: space-between; align-items: center; letter-spacing: 1px; }
        .logout-btn { background: none; border: 1px solid #ff0033; color: #ff0033; padding: 6px 12px; cursor: pointer; font-weight: bold; transition: 0.3s; }
        .logout-btn:hover { background: #ff0033; color: #000; }
        
        /* Messages Area */
        .messages { flex: 1; padding: 20px; overflow-y: auto; display: flex; flex-direction: column; gap: 12px; }
        .msg { padding: 12px 18px; border-radius: 4px; max-width: 75%; word-wrap: break-word; font-size: 0.95rem; line-height: 1.4; }
        
        /* Message Colors */
        .my-msg { background: rgba(255, 0, 51, 0.2); border: 1px solid #ff0033; align-self: flex-end; color: #fff; }
        .other-msg { background: rgba(40, 40, 40, 0.8); border: 1px solid #444; align-self: flex-start; color: #ccc; }
        
        /* Input Area */
        .input-area { display: flex; padding: 15px; border-top: 1px solid #ff0033; background: #0a0a0a; }
        .input-area input { flex: 1; padding: 12px; background: rgba(0,0,0,0.5); border: 1px solid #ff0033; color: #00ffcc; outline: none; font-size: 1rem; }
        .input-area input::placeholder { color: #555; }
        .input-area button { padding: 12px 25px; background: #ff0033; color: #000; border: none; cursor: pointer; font-weight: bold; margin-left: 10px; transition: 0.3s; }
        .input-area button:hover { background: #fff; color: #ff0033; }

        /* === ADMIN DASHBOARD UI === */
        .admin-layout { flex-direction: row; }
        .sidebar { width: 30%; border-right: 1px solid #ff0033; overflow-y: auto; background: #0c0c0c; display: flex; flex-direction: column; }
        .customer-item { padding: 18px 15px; border-bottom: 1px dashed #333; cursor: pointer; color: #aaa; transition: 0.2s; font-size: 0.9rem; }
        .customer-item:hover, .customer-item.selected { background: rgba(255,0,51,0.2); color: #fff; border-left: 3px solid #ff0033; }
        .admin-chat-area { width: 70%; display: flex; flex-direction: column; }
        
        /* Responsive adjustments */
        @media(max-width: 768px) {
            .title-3d { font-size: 2rem; }
            .admin-layout { flex-direction: column; }
            .sidebar { width: 100%; height: 30%; border-right: none; border-bottom: 1px solid #ff0033; }
            .admin-chat-area { width: 100%; height: 70%; }
        }
    </style>
</head>
<body>

    <!-- Background Animation -->
    <canvas id="matrix"></canvas>

    <!-- 1. LOGIN SCREEN -->
    <div id="login-screen" class="screen active">
        <div class="owner-tag">( Owner Magisk )</div>
        <div class="title-3d">ABDUL BASIT<br>CHAT PORTAL</div>
        <div class="btn-group">
            <button class="login-btn" onclick="switchScreen('customer-screen')">> SIMULATE CUSTOMER LOGIN</button>
            <button class="login-btn" onclick="switchScreen('admin-screen')">> SIMULATE ADMIN LOGIN</button>
        </div>
    </div>

    <!-- 2. CUSTOMER SCREEN -->
    <div id="customer-screen" class="screen">
        <div class="chat-container">
            <div class="chat-header">
                <span>SECURE CHAT: ABDUL BASIT</span>
                <button class="logout-btn" onclick="switchScreen('login-screen')">DISCONNECT</button>
            </div>
            <div class="messages" id="customer-chat">
                <div class="msg my-msg">Hello, I need to complete a P2P transaction.</div>
                <div class="msg other-msg"><b>Admin:</b> Please send your details. I am online.</div>
            </div>
            <div class="input-area">
                <input type="text" id="cust-input" placeholder="Type your message...">
                <button onclick="sendSimulatedMessage('customer-chat', 'cust-input', 'my-msg')">SEND</button>
            </div>
        </div>
    </div>

    <!-- 3. ADMIN SCREEN -->
    <div id="admin-screen" class="screen">
        <div class="chat-container admin-layout">
            
            <!-- Sidebar: Customers List -->
            <div class="sidebar">
                <div class="chat-header">
                    CUSTOMERS 
                    <button class="logout-btn" onclick="switchScreen('login-screen')">X</button>
                </div>
                <div class="customer-item selected">> Client Alpha</div>
                <div class="customer-item">> Client Beta</div>
                <div class="customer-item">> Unknown_IP_99</div>
            </div>
            
            <!-- Main Chat Area -->
            <div class="admin-chat-area">
                <div class="chat-header">CHATTING WITH: Client Alpha</div>
                <div class="messages" id="admin-chat">
                    <div class="msg other-msg"><b>Client Alpha:</b> Hello, I need to complete a P2P transaction.</div>
                    <div class="msg my-msg">Please send your details. I am online.</div>
                </div>
                <div class="input-area">
                    <input type="text" id="admin-input" placeholder="Reply to Client Alpha...">
                    <button onclick="sendSimulatedMessage('admin-chat', 'admin-input', 'my-msg')">SEND</button>
                </div>
            </div>
            
        </div>
    </div>

    <script>
        // Matrix Canvas Animation
        const canvas = document.getElementById('matrix');
        const ctx = canvas.getContext('2d');
        let width = canvas.width = window.innerWidth;
        let height = canvas.height = window.innerHeight;

        const chars = "010101 ABDULBASIT SECURE DATA ENCRYPTED".split("");
        const fontSize = 16;
        const columns = width / fontSize;
        const drops = [];
        
        for (let x = 0; x < columns; x++) {
            drops[x] = Math.random() * 100; // random start position
        }

        function drawMatrix() {
            ctx.fillStyle = "rgba(5, 5, 5, 0.1)"; // Dark fade effect
            ctx.fillRect(0, 0, width, height);
            
            ctx.fillStyle = "#ff0033"; // Red hacker text
            ctx.font = fontSize + "px monospace";

            for (let i = 0; i < drops.length; i++) {
                const text = chars[Math.floor(Math.random() * chars.length)];
                ctx.fillText(text, i * fontSize, drops[i] * fontSize);

                if (drops[i] * fontSize > height && Math.random() > 0.95) {
                    drops[i] = 0;
                }
                drops[i]++;
            }
        }
        setInterval(drawMatrix, 50);
        
        // Handle window resize for canvas
        window.addEventListener('resize', () => {
            width = canvas.width = window.innerWidth;
            height = canvas.height = window.innerHeight;
        });

        // Screen Switching Logic
        function switchScreen(screenId) {
            document.querySelectorAll('.screen').forEach(screen => {
                screen.classList.remove('active');
            });
            document.getElementById(screenId).classList.add('active');
        }

        // Simulated Chat Logic
        function sendSimulatedMessage(chatId, inputId, msgClass) {
            const input = document.getElementById(inputId);
            const text = input.value.trim();
            if (text === "") return;

            const chatBox = document.getElementById(chatId);
            const msgDiv = document.createElement('div');
            msgDiv.className = 'msg ' + msgClass;
            msgDiv.innerText = text;

            chatBox.appendChild(msgDiv);
            
            // Auto-scroll to bottom
            chatBox.scrollTop = chatBox.scrollHeight; 
            
            // Clear input
            input.value = "";
        }
        
        // Add Enter key support for inputs
        document.getElementById('cust-input').addEventListener('keypress', function(e) {
            if (e.key === 'Enter') sendSimulatedMessage('customer-chat', 'cust-input', 'my-msg');
        });
        document.getElementById('admin-input').addEventListener('keypress', function(e) {
            if (e.key === 'Enter') sendSimulatedMessage('admin-chat', 'admin-input', 'my-msg');
        });
    </script>
</body>
</html># Abdul-Basit-
A secure, real-time P2P chat portal designed for private global communications." Yeh ek chota aur behtareen description hai jo aapke project ka sahi maqsad batata hai aur developers par achha impression dalega. Isko paste karne ke baad aap seedha 'Create repository' par click kar ke apni file upload kar sakte hain.
