<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>익명</title>
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-database-compat.js"></script>
    <style>
        /* iMessage 스타일 말풍선 꼬리 및 스크롤바 숨김 */
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
    </style>
</head>
<body class="bg-gray-100 h-screen flex flex-col items-center justify-center font-sans">

    <div id="lobby-view" class="bg-white p-8 rounded-2xl shadow-md max-w-sm w-full text-center hidden">
        <h1 class="text-2xl font-bold mb-6 text-gray-800">익명 대화방 생성</h1>
        <button id="btn-create" class="w-full bg-blue-500 hover:bg-blue-600 text-white font-semibold py-3 px-4 rounded-xl transition">
            새로운 익명 대화방 만들기
        </button>
    </div>

    <div id="chat-view" class="bg-white w-full max-w-md h-full sm:h-[85vh] sm:rounded-2xl shadow-xl flex flex-col overflow-hidden hidden">
        <div class="bg-white border-b border-gray-200 p-4 flex flex-col items-center justify-center relative">
            <div id="timer" class="text-xs font-semibold text-red-500 tracking-wider mb-1">15:00</div>
            <div class="text-lg font-bold text-gray-800">익명</div>
            <button id="btn-share" class="absolute right-4 top-5 text-sm bg-gray-100 hover:bg-gray-200 text-gray-600 px-3 py-1 rounded-full font-medium">
                공유
            </button>
        </div>

        <div id="message-container" class="flex-1 overflow-y-auto p-4 space-y-3 bg-white no-scrollbar">
            </div>

        <div class="p-4 border-t border-gray-200 bg-white">
            <div class="flex items-center bg-gray-100 rounded-full px-4 py-2">
                <input id="input-msg" type="text" placeholder="메시지 보내기..." class="flex-1 bg-transparent outline-none text-sm text-gray-800" autocomplete="off">
                <button id="btn-send" class="text-blue-500 font-semibold text-sm ml-2 hover:text-blue-700">보내기</button>
            </div>
        </div>
    </div>

    <script>
        // ⚠️ 본인의 Firebase 설정값으로 교체해야 정상 작동합니다.
        const firebaseConfig = {
            apiKey: "YOUR_API_KEY",
            authDomain: "YOUR_PROJECT.firebaseapp.com",
            databaseURL: "https://YOUR_PROJECT-default-rtdb.firebaseio.com",
            projectId: "YOUR_PROJECT",
            storageBucket: "YOUR_PROJECT.appspot.com",
            messagingSenderId: "YOUR_SENDER_ID",
            appId: "YOUR_APP_ID"
        };
        firebase.initializeApp(firebaseConfig);
        const db = firebase.database();

        // 익명 유저 식별자 생성 (세션 내 유지)
        if (!sessionStorage.getItem('userId')) {
            sessionStorage.setItem('userId', 'user_' + Math.random().toString(36).substr(2, 9));
        }
        const myId = sessionStorage.getItem('userId');

        // URL 파라미터 파싱
        const urlParams = new URLSearchParams(window.location.search);
        const roomId = urlParams.get('room');

        const lobbyView = document.getElementById('lobby-view');
        const chatView = document.getElementById('chat-view');

        // 라우팅 처리
        if (!roomId) {
            lobbyView.classList.remove('hidden');
        } else {
            chatView.classList.remove('hidden');
            initChatRoom(roomId);
        }

        // 방 생성 로직
        document.getElementById('btn-create').addEventListener('click', () => {
            const newRoomId = db.ref().child('rooms').push().key;
            const now = Date.now();
            
            db.ref('rooms/' + newRoomId).set({
                createdAt: now,
                status: 'active'
            }).then(() => {
                window.location.search = `?room=${newRoomId}`;
            });
        });

        // 채팅방 활성화 및 실시간 바인딩
        function initChatRoom(id) {
            const roomRef = db.ref('rooms/' + id);
            
            // 1. 타이머 관리 (15분 = 900초)
            roomRef.child('createdAt').once('value', snapshot => {
                const createdAt = snapshot.val();
                if (!createdAt) return;

                const timerInterval = setInterval(() => {
                    const elapsed = Math.floor((Date.now() - createdAt) / 1000);
                    const remaining = 900 - elapsed;

                    if (remaining <= 0) {
                        clearInterval(timerInterval);
                        document.getElementById('timer').innerText = "종료된 대화방";
                        document.getElementById('input-msg').disabled = true;
                        document.getElementById('input-msg').placeholder = "15분이 지나 대화가 종료되었습니다.";
                        document.getElementById('btn-send').disabled = true;
                    } else {
                        const min = String(Math.floor(remaining / 60)).padStart(2, '0');
                        const sec = String(remaining % 60).padStart(2, '0');
                        document.getElementById('timer').innerText = `${min}:${sec}`;
                    }
                }, 1000);
            });

            // 2. 실시간 메시지 수신
            const container = document.getElementById('message-container');
            roomRef.child('messages').on('child_added', snapshot => {
                const data = snapshot.val();
                const isMe = data.sender === myId;

                const wrapper = document.createElement('div');
                wrapper.className = `flex w-full ${isMe ? 'justify-end' : 'justify-start'}`;

                // 인스타/iMessage 스타일 (좌 회색, 우 파란색, 글자 길이에 맞춰 조절)
                wrapper.innerHTML = `
                    <div class="max-w-[70%] rounded-2xl px-4 py-2 text-sm break-words shadow-sm
                        ${isMe ? 'bg-blue-500 text-white rounded-tr-none' : 'bg-gray-200 text-gray-800 rounded-tl-none'}">
                        ${escapeHtml(data.text)}
                    </div>
                `;
                container.appendChild(wrapper);
                container.scrollTop = container.scrollHeight; // 자동 스크롤
            });

            // 3. 메시지 전송 로직
            const input = document.getElementById('input-msg');
            const sendBtn = document.getElementById('btn-send');

            function sendMessage() {
                const text = input.value.trim();
                if (!text) return;
                
                roomRef.child('messages').push({
                    sender: myId,
                    text: text,
                    timestamp: Date.now()
                });
                input.value = '';
            }

            sendBtn.addEventListener('click', sendMessage);
            input.addEventListener('keypress', (e) => { if(e.key === 'Enter') sendMessage(); });

            // 4. 링크 공유 버튼
            document.getElementById('btn-share').addEventListener('click', () => {
                navigator.clipboard.writeText(window.location.href);
                alert('대화방 링크가 복사되었습니다. 상대방에게 공유하세요!');
            });
        }

        // XSS 방지용 에스케이프 함수
        function escapeHtml(str) {
            return str.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/"/g, "&quot;").replace(/'/g, "&#039;");
        }
    </script>
</body>
</html># test0001
