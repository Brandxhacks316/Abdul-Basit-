<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ABDUL BASIT CHAT PORTAL</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Courier New', Courier, monospace; }
        body { background-color: #050505; color: #ff0033; overflow: hidden; height: 100vh; }
        canvas#matrix { position: fixed; top: 0; left: 0; width: 100%; height: 100%; z-index: 0; opacity: 0.4; }
        
        .screen { position: absolute; top: 0; left: 0; width: 100%; height: 100%; z-index: 10; display: none; flex-direction: column; align-items: center; justify-content: center; background: rgba(0, 0, 0, 0.85); }
        .active { display: flex !important; }

        .owner-tag { color: #00ffcc; font-size: 1.5rem; margin-bottom: 10px; text-shadow: 0 0 10px #00ffcc; animation: flicker 2s infinite; }
        .title-3d { font-size: 3.5rem; font-weight: 900; color: #fff; text-shadow: 0 5px 0 #8a001d, 0 10px 0 #ff0033, 0 0 20px #ff0033; margin-bottom: 40px; text-align: center; text-transform: uppercase; }
        .login-btn { padding: 15px 30px; background: transparent; border: 2px solid #ff0033; color: #ff0033; font-size: 1.2rem; cursor: pointer; text-transform: uppercase; font-weight: bold; transition: 0.3s; }
        .login-btn:hover { background: #ff0033; color: #000; box-shadow: 0 0 20px #ff0033; }

        .chat-container { width: 95%; max-width: 600px; height: 85vh; border: 1px solid #ff0033; background: rgba(10, 10, 10, 0.95); display: flex; flex-direction: column; position: relative; box-shadow: 0 0 15px rgba(255, 0, 51, 0.3); }
        .chat-header { padding: 15px; border-bottom: 1px solid #ff0033; text-align: center; font-weight: bold; background: #111; color: #fff; display: flex; justify-content: space-between; align-items: center; }
        .logout-btn { background: none; border: 1px solid #ff0033; color: #ff0033; padding: 5px 10px; cursor: pointer; }
        
        .messages { flex: 1; padding: 15px; overflow-y: auto; display: flex; flex-direction: column; gap: 10px; }
        .msg { padding: 10px; border-radius: 5px; max-width: 80%; word-wrap: break-word; }
        .msg.my-msg { background: rgba(255, 0, 51, 0.2); border: 1px solid #ff0033; align-self: flex-end; color: #fff; }
        .msg.other-msg { background: rgba(50, 50, 50, 0.5); border: 1px solid #555; align-self: flex-start; color: #ccc; }
        
        .input-area { display: flex; padding: 10px; border-top: 1px solid #ff0033; background: #0a0a0a; }
        .input-area input { flex: 1; padding: 10px; background: transparent; border: 1px solid #ff0033; color: #00ffcc; outline: none; }
        .input-area button { padding: 10px 20px; background: #ff0033; color: #000; border: none; cursor: pointer; font-weight: bold; margin-left: 10px; }

        .admin-layout { flex-direction: row; max-width: 1000px; }
        .sidebar { width: 30%; border-right: 1px solid #ff0033; overflow-y: auto; background: #111; }
        .customer-item { padding: 15px; border-bottom: 1px dashed #333; cursor: pointer; color: #ccc; }
        .customer-item:hover { background: #ff0033; color: #000; }
        .admin-chat-area { width: 70%; display: flex; flex-direction: column; }
        
        @keyframes flicker { 0%, 100% { opacity: 1; } 50% { opacity: 0.5; } }
    </style>
</head>
<body>
    <canvas id="matrix"></canvas>

    <!-- 1. LOGIN SCREEN -->
    <div id="login-screen" class="screen active">
        <div class="owner-tag">( Owner Magisk )</div>
        <div class="title-3d">ABDUL BASIT CHAT PORTAL</div>
        <button id="loginBtn" class="login-btn">> INITIATE GOOGLE LOGIN</button>
    </div>

    <!-- 2. CUSTOMER CHAT SCREEN -->
    <div id="customer-screen" class="screen">
        <div class="chat-container">
            <div class="chat-header">
                <span>SECURE CHAT</span>
                <button id="custLogoutBtn" class="logout-btn">DISCONNECT</button>
            </div>
            <div id="customerMessages" class="messages"></div>
            <div class="input-area">
                <input type="text" id="custMsgInput" placeholder="Type your message...">
                <button id="custSendBtn">SEND</button>
            </div>
        </div>
    </div>

    <!-- 3. ADMIN DASHBOARD -->
    <div id="admin-screen" class="screen">
        <div class="chat-container admin-layout">
            <div class="sidebar">
                <div class="chat-header">CUSTOMERS <button id="adminLogoutBtn" class="logout-btn">X</button></div>
                <div id="customerList"></div>
            </div>
            <div class="admin-chat-area">
                <div class="chat-header" id="adminActiveChatHeader">SELECT A CUSTOMER</div>
                <div id="adminMessages" class="messages"></div>
                <div class="input-area">
                    <input type="text" id="adminMsgInput" placeholder="Type to customer..." disabled>
                    <button id="adminSendBtn" disabled>SEND</button>
                </div>
            </div>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.5.0/firebase-app.js";
        import { getAuth, signInWithPopup, GoogleAuthProvider, onAuthStateChanged, signOut } from "https://www.gstatic.com/firebasejs/10.5.0/firebase-auth.js";
        import { getFirestore, collection, addDoc, query, where, orderBy, onSnapshot, setDoc, doc, serverTimestamp } from "https://www.gstatic.com/firebasejs/10.5.0/firebase-firestore.js";

        // YAHAN APKA ADMIN EMAIL HAI
        const ADMIN_EMAIL = "khanji223353@gmail.com"; 
        
        // APKI ASLI FIREBASE KEYS (REAL BACKEND)
        const firebaseConfig = {
            apiKey: "AIzaSyD5odl3uHyob5GDwK88B9RHaQ1nDMmRrV0",
            authDomain: "abdulbasit8t1.firebaseapp.com",
            projectId: "abdulbasit8t1",
            storageBucket: "abdulbasit8t1.firebasestorage.app",
            messagingSenderId: "595773588592",
            appId: "1:595773588592:web:261ebaed28dd8dc15b60b5",
            measurementId: "G-P7XNBB3RGY"
        };

        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const provider = new GoogleAuthProvider();

        let currentUser = null;
        let activeAdminChatUid = null;
        let unsubscribeMessages = null;

        function showScreen(screenId) {
            document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
            document.getElementById(screenId).classList.add('active');
        }

        // ASLI GOOGLE LOGIN
        document.getElementById('loginBtn').addEventListener('click', () => {
            signInWithPopup(auth, provider).catch(error => alert("Login Failed: " + error.message));
        });

        const logout = () => signOut(auth);
        document.getElementById('custLogoutBtn').addEventListener('click', logout);
        document.getElementById('adminLogoutBtn').addEventListener('click', logout);

        onAuthStateChanged(auth, async (user) => {
            if (user) {
                currentUser = user;
                if (user.email === ADMIN_EMAIL) {
                    showScreen('admin-screen');
                    loadAdminDashboard();
                } else {
                    await setDoc(doc(db, "customers", user.uid), {
                        uid: user.uid, name: user.displayName, email: user.email, lastLogin: serverTimestamp()
                    }, { merge: true });
                    showScreen('customer-screen');
                    loadCustomerChat(user.uid);
                }
            } else {
                currentUser = null;
                if (unsubscribeMessages) unsubscribeMessages();
                showScreen('login-screen');
            }
        });

        async function sendMessage(text, chatRoomId) {
            if (!text.trim()) return;
            await addDoc(collection(db, "messages"), {
                chatRoomId: chatRoomId, senderId: currentUser.uid, senderName: currentUser.displayName, text: text, timestamp: serverTimestamp()
            });
        }

        document.getElementById('custSendBtn').addEventListener('click', () => {
            const input = document.getElementById('custMsgInput');
            sendMessage(input.value, currentUser.uid);
            input.value = "";
        });

        document.getElementById('adminSendBtn').addEventListener('click', () => {
            const input = document.getElementById('adminMsgInput');
            if (activeAdminChatUid) {
                sendMessage(input.value, activeAdminChatUid);
                input.value = "";
            }
        });

        function loadCustomerChat(uid) {
            const q = query(collection(db, "messages"), where("chatRoomId", "==", uid), orderBy("timestamp", "asc"));
            if (unsubscribeMessages) unsubscribeMessages();
            
            unsubscribeMessages = onSnapshot(q, (snapshot) => {
                const box = document.getElementById('customerMessages');
                box.innerHTML = '';
                snapshot.forEach((doc) => {
                    const data = doc.data();
                    const isMe = data.senderId === currentUser.uid;
                    box.innerHTML += `<div class="msg ${isMe ? 'my-msg' : 'other-msg'}"><b>${isMe ? 'You' : 'Admin'}:</b> ${data.text}</div>`;
                });
                box.scrollTop = box.scrollHeight;
            });
        }

        function loadAdminDashboard() {
            const q = query(collection(db, "customers"), orderBy("lastLogin", "desc"));
            onSnapshot(q, (snapshot) => {
                const list = document.getElementById('customerList');
                list.innerHTML = '';
                snapshot.forEach((docSnap) => {
                    const cust = docSnap.data();
                    const div = document.createElement('div');
                    div.className = "customer-item";
                    div.innerHTML = `> ${cust.name}`;
                    div.onclick = () => openAdminChat(cust.uid, cust.name);
                    list.appendChild(div);
                });
            });
        }

        function openAdminChat(custUid, custName) {
            activeAdminChatUid = custUid;
            document.getElementById('adminActiveChatHeader').innerText = "CHATTING WITH: " + custName;
            document.getElementById('adminMsgInput').disabled = false;
            document.getElementById('adminSendBtn').disabled = false;

            const q = query(collection(db, "messages"), where("chatRoomId", "==", custUid), orderBy("timestamp", "asc"));
            if (unsubscribeMessages) unsubscribeMessages();
            
            unsubscribeMessages = onSnapshot(q, (snapshot) => {
                const box = document.getElementById('adminMessages');
                box.innerHTML = '';
                snapshot.forEach((doc) => {
                    const data = doc.data();
                    const isMe = data.senderId === currentUser.uid;
                    box.innerHTML += `<div class="msg ${isMe ? 'my-msg' : 'other-msg'}"><b>${isMe ? 'Admin' : custName}:</b> ${data.text}</div>`;
                });
                box.scrollTop = box.scrollHeight;
            });
        }

        const canvas = document.getElementById('matrix');
        const ctx = canvas.getContext('2d');
        canvas.width = window.innerWidth; canvas.height = window.innerHeight;
        const letters = "01010101 ABDUL BASIT X Y Z".split("");
        const fontSize = 16; const columns = canvas.width / fontSize;
        const drops = []; for (let x = 0; x < columns; x++) drops[x] = 1;
        
        function drawMatrix() {
            ctx.fillStyle = "rgba(0, 0, 0, 0.05)";
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = "#ff0033"; ctx.font = fontSize + "px monospace";
            for (let i = 0; i < drops.length; i++) {
                const text = letters[Math.floor(Math.random() * letters.length)];
                ctx.fillText(text, i * fontSize, drops[i] * fontSize);
                if (drops[i] * fontSize > canvas.height && Math.random() > 0.975) drops[i] = 0;
                drops[i]++;
            }
        }
        setInterval(drawMatrix, 50);
    </script>
</body>
</html>
