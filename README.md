<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>익명 - 관리자 창</title>
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/10.8.0/firebase-database-compat.js"></script>
</head>
<body class="bg-gray-900 text-gray-100 min-h-screen p-8">
    <div class="max-w-6xl mx-auto">
        <h1 class="text-3xl font-bold mb-8 border-b border-gray-700 pb-4 text-blue-400">Master 관리자 대시보드</h1>
        
        <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
            <div class="lg:col-span-1 bg-gray-800 rounded-xl p-4 shadow-lg border border-gray-700">
                <h2 class="text-lg font-semibold mb-4 text-gray-300">생성된 대화방 목록</h2>
                <div id="room-list" class="space-y-2 h-[70vh] overflow-y-auto pr-2">
                    </div>
            </div>

            <div class="lg:col-span-2 bg-gray-800 rounded-xl p-6 shadow-lg border border-gray-700 flex flex-col h-[78vh]">
                <div class="flex justify-between items-center mb-4 border-b border-gray-700 pb-3">
                    <h2 id="current-room-title" class="text-lg font-semibold text-gray-300">대화방을 선택하세요</h2>
                    <button id="btn-delete-room" class="bg-red-600 hover:bg-red-700 text-white text-sm px-4 py-2 rounded-lg hidden transition">
                        이 방 전체 삭제 (DB 파기)
                    </button>
                </div>
                
                <div id="admin-chat-view" class="flex-1 overflow-y-auto space-y-3 mb-4 p-4 bg-gray-950 rounded-xl">
                    <p class="text-gray-500 text-center mt-10">선택된 기록이 없습니다.</p>
                </div>
            </div>
        </div>
    </div>

    <script>
        // index.html과 동일한 Firebase 설정을 입력하세요
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

        let activeAdminRoomId = null;

        // 전체 방 리스트 실시간 로드 (일렬 배치)
        db.ref('rooms').on('value', snapshot => {
            const listContainer = document.getElementById('room-list');
            listContainer.innerHTML = '';
            
            const rooms = snapshot.val();
            if(!rooms) {
                listContainer.innerHTML = '<p class="text-gray-500 text-sm">생성된 방이 없습니다.</p>';
                return;
            }

            Object.keys(rooms).forEach(id => {
                const room = rooms[id];
                const date = new Date(room.createdAt).toLocaleString('ko-KR', { timeStyle: 'short', dateStyle: 'short' });
                const msgCount = room.messages ? Object.keys(room.messages).length : 0;

                const item = document.createElement('div');
                item.className = `p-3 rounded-lg cursor-pointer transition flex justify-between items-center ${activeAdminRoomId === id ? 'bg-blue-600 text-white' : 'bg-gray-700 hover:bg-gray-600'}`;
                item.innerHTML = `
                    <div>
                        <div class="font-mono text-xs opacity-70">ID: ${id.substr(1, 6)}...</div>
                        <div class="text-xs mt-1 opacity-50">${date}</div>
                    </div>
                    <span class="bg-gray-900 text-gray-300 text-xs px-2 py-1 rounded-full">${msgCount} 톡</span>
                `;
                
                item.addEventListener('click', () => selectRoom(id));
                listContainer.appendChild(item);
            });
        });

        // 특정 방 선택 시 기록 로드 및 관리 활성화
        function selectRoom(id) {
            activeAdminRoomId = id;
            document.getElementById('btn-delete-room').classList.remove('hidden');
            document.getElementById('current-room-title').innerText = `대화방 관리 기록 (${id})`;
            
            const chatView = document.getElementById('admin-chat-view');
            
            // 실시간 리스너 바인딩 (사용자가 안 써도 데이터는 보관되어 출력됨)
            db.ref(`rooms/${id}/messages`).on('value', snapshot => {
                chatView.innerHTML = '';
                const messages = snapshot.val();
                
                if(!messages) {
                    chatView.innerHTML = '<p class="text-gray-500 text-center mt-4">대화 기록이 비어있습니다.</p>';
                    return;
                }

                Object.keys(messages).forEach(msgKey => {
                    const msg = messages[msgKey];
                    const div = document.createElement('div');
                    div.className = "bg-gray-900 p-2 rounded border border-gray-800 flex justify-between items-center";
                    div.innerHTML = `
                        <div class="flex-1">
                            <span class="text-xs text-yellow-500 font-mono">[${msg.sender.substr(5, 4)}]</span>
                            <span class="text-sm ml-2 text-gray-200" id="text-${msgKey}">${escapeHtml(msg.text)}</span>
                        </div>
                        <div class="flex space-x-2 ml-4">
                            <button onclick="editMessage('${id}', '${msgKey}')" class="text-xs bg-yellow-600 hover:bg-yellow-700 px-2 py-1 rounded text-white">수정</button>
                            <button onclick="deleteMessage('${id}', '${msgKey}')" class="text-xs bg-red-600 hover:bg-red-700 px-2 py-1 rounded text-white">삭제</button>
                        </div>
                    `;
                    chatView.appendChild(div);
                });
            });
        }

        // 관리자 기능: 개별 메시지 수정
        window.editMessage = function(roomId, msgKey) {
            const currentText = document.getElementById(`text-${msgKey}`).innerText;
            const newText = prompt("수정할 내용을 입력하세요:", currentText);
            if (newText !== null && newText.trim() !== "") {
                db.ref(`rooms/${roomId}/messages/${msgKey}`).update({ text: newText });
            }
        };

        // 관리자 기능: 개별 메시지 삭제
        window.deleteMessage = function(roomId, msgKey) {
            if(confirm("이 메시지를 삭제하시겠습니까?")) {
                db.ref(`rooms/${roomId}/messages/${msgKey}`).remove();
            }
        };

        // 관리자 기능: 방 전체 파기
        document.getElementById('btn-delete-room').addEventListener('click', () => {
            if(activeAdminRoomId && confirm("이 방의 모든 데이터와 대화 기록을 완벽히 삭제합니까? 복구 불가능합니다.")) {
                db.ref(`rooms/${activeAdminRoomId}`).remove().then(() => {
                    activeAdminRoomId = null;
                    document.getElementById('admin-chat-view').innerHTML = '<p class="text-gray-500 text-center mt-10">방이 완전히 삭제되었습니다.</p>';
                    document.getElementById('current-room-title').innerText = '대화방을 선택하세요';
                    document.getElementById('btn-delete-room').classList.add('hidden');
                });
            }
        });

        function escapeHtml(str) {
            return str.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/"/g, "&quot;").replace(/'/g, "&#039;");
        }
    </script>
</body>
</html>
