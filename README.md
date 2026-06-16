<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>익명 메시지</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  :root {
    --bg: #ffffff;
    --bg2: #f5f5f7;
    --bg3: #ebebeb;
    --text: #1c1c1e;
    --text2: #6e6e73;
    --blue: #007aff;
    --blue-dark: #0062cc;
    --bubble-in: #e9e9eb;
    --bubble-out: #007aff;
    --border: #d1d1d6;
    --radius: 18px;
    --input-h: 44px;
  }
  body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif; background: var(--bg); color: var(--text); min-height: 100vh; }

  /* ─── PAGES ─── */
  .page { display: none; min-height: 100vh; }
  .page.active { display: flex; flex-direction: column; }

  /* ─── HOME ─── */
  #home { align-items: center; justify-content: center; background: var(--bg); }
  .home-inner { width: 100%; max-width: 400px; padding: 40px 24px; display: flex; flex-direction: column; align-items: center; gap: 28px; }
  .logo-area { display: flex; flex-direction: column; align-items: center; gap: 10px; }
  .logo-bubble { display: flex; gap: 8px; }
  .lbub { width: 18px; height: 18px; border-radius: 50%; }
  .lbub.l { background: var(--bubble-in); }
  .lbub.r { background: var(--bubble-out); }
  .logo-area h1 { font-size: 26px; font-weight: 700; letter-spacing: -0.5px; }
  .logo-area p { font-size: 14px; color: var(--text2); text-align: center; line-height: 1.5; }
  .home-btn { width: 100%; padding: 0 20px; height: 50px; border-radius: 14px; border: none; cursor: pointer; font-size: 16px; font-weight: 600; transition: all .15s; }
  .home-btn.primary { background: var(--blue); color: #fff; }
  .home-btn.primary:hover { background: var(--blue-dark); }
  .home-btn.secondary { background: var(--bg3); color: var(--text); }
  .home-btn.secondary:hover { background: #d8d8da; }
  .home-btn.ghost { background: transparent; color: var(--blue); border: 1.5px solid var(--blue); }
  .home-btn.ghost:hover { background: #f0f7ff; }
  .divider { display: flex; align-items: center; gap: 12px; width: 100%; }
  .divider span { font-size: 13px; color: var(--text2); white-space: nowrap; }
  .divider::before, .divider::after { content: ''; flex: 1; height: 1px; background: var(--border); }
  .join-row { display: flex; gap: 10px; width: 100%; }
  .join-row input { flex: 1; height: 50px; border-radius: 14px; border: 1.5px solid var(--border); padding: 0 16px; font-size: 15px; outline: none; }
  .join-row input:focus { border-color: var(--blue); }
  .join-row button { height: 50px; padding: 0 20px; border-radius: 14px; border: none; background: var(--blue); color: #fff; font-size: 15px; font-weight: 600; cursor: pointer; white-space: nowrap; }
  .admin-link { font-size: 13px; color: var(--text2); cursor: pointer; text-decoration: underline; }

  /* ─── CHAT ─── */
  #chat { background: var(--bg); }
  .chat-header { background: var(--bg); border-bottom: 1px solid var(--border); padding: 12px 16px 10px; display: flex; align-items: center; gap: 12px; position: sticky; top: 0; z-index: 10; }
  .chat-header .back-btn { background: none; border: none; color: var(--blue); font-size: 15px; cursor: pointer; display: flex; align-items: center; gap: 4px; padding: 4px 0; white-space: nowrap; }
  .chat-header .back-btn svg { width: 10px; height: 16px; }
  .chat-title { flex: 1; text-align: center; }
  .chat-title .room-name { font-size: 14px; font-weight: 600; }
  .chat-title .room-id { font-size: 11px; color: var(--text2); margin-top: 1px; }
  .share-btn { background: none; border: none; color: var(--blue); font-size: 13px; cursor: pointer; white-space: nowrap; padding: 4px 0; }

  /* timer */
  .timer-bar { background: var(--bg2); padding: 6px 16px; display: flex; align-items: center; justify-content: center; gap: 8px; font-size: 13px; }
  .timer-bar .t-label { color: var(--text2); }
  .timer-bar .t-count { font-weight: 700; font-variant-numeric: tabular-nums; color: var(--text); min-width: 42px; display: inline-block; text-align: center; }
  .timer-bar.warning .t-count { color: #ff3b30; }
  .timer-bar .t-dot { width: 7px; height: 7px; border-radius: 50%; background: #34c759; }
  .timer-bar.warning .t-dot { background: #ff3b30; }

  .messages-wrap { flex: 1; overflow-y: auto; padding: 16px 0 4px; }
  .messages { display: flex; flex-direction: column; gap: 4px; max-width: 740px; margin: 0 auto; padding: 0 16px; }

  /* date separator */
  .date-sep { text-align: center; font-size: 11px; color: var(--text2); margin: 10px 0 4px; }

  /* bubbles */
  .msg-row { display: flex; flex-direction: column; max-width: 72%; }
  .msg-row.out { align-self: flex-end; align-items: flex-end; }
  .msg-row.in { align-self: flex-start; align-items: flex-start; }
  .sender-label { font-size: 11px; color: var(--text2); margin-bottom: 2px; padding: 0 4px; }
  .bubble { padding: 10px 14px; border-radius: var(--radius); font-size: 16px; line-height: 1.45; word-break: break-word; max-width: 100%; display: inline-block; }
  .msg-row.out .bubble { background: var(--bubble-out); color: #fff; border-bottom-right-radius: 4px; }
  .msg-row.in .bubble { background: var(--bubble-in); color: var(--text); border-bottom-left-radius: 4px; }
  .bubble-time { font-size: 11px; color: var(--text2); margin-top: 2px; padding: 0 4px; }

  /* stacked bubbles same sender */
  .msg-row.out + .msg-row.out { margin-top: -2px; }
  .msg-row.out + .msg-row.out .bubble { border-top-right-radius: 6px; }
  .msg-row.in + .msg-row.in { margin-top: -2px; }
  .msg-row.in + .msg-row.in .bubble { border-top-left-radius: 6px; }

  /* input */
  .input-area { border-top: 1px solid var(--border); padding: 10px 12px 12px; background: var(--bg); }
  .input-inner { display: flex; align-items: flex-end; gap: 8px; max-width: 740px; margin: 0 auto; }
  .input-inner textarea { flex: 1; min-height: var(--input-h); max-height: 120px; border: 1.5px solid var(--border); border-radius: 22px; padding: 10px 16px; font-size: 16px; line-height: 1.4; resize: none; outline: none; font-family: inherit; background: var(--bg2); transition: border-color .15s; }
  .input-inner textarea:focus { border-color: var(--blue); background: var(--bg); }
  .send-btn { width: 36px; height: 36px; border-radius: 50%; background: var(--blue); border: none; cursor: pointer; display: flex; align-items: center; justify-content: center; flex-shrink: 0; margin-bottom: 4px; transition: background .15s; }
  .send-btn:disabled { background: var(--border); cursor: default; }
  .send-btn svg { width: 16px; height: 16px; fill: #fff; }

  /* ─── EXPIRED ─── */
  .expired-banner { background: #fff3cd; border-bottom: 1px solid #ffc107; padding: 8px 16px; text-align: center; font-size: 13px; color: #856404; }

  /* ─── ADMIN ─── */
  #admin { background: var(--bg); }
  .admin-header { background: var(--bg); border-bottom: 1px solid var(--border); padding: 16px 20px; display: flex; align-items: center; justify-content: space-between; position: sticky; top: 0; z-index: 10; }
  .admin-header h2 { font-size: 20px; font-weight: 700; }
  .admin-header button { background: none; border: none; color: var(--blue); cursor: pointer; font-size: 14px; }
  .admin-body { padding: 16px 20px; max-width: 900px; margin: 0 auto; }
  .admin-stats { display: flex; gap: 12px; margin-bottom: 20px; flex-wrap: wrap; }
  .stat-card { background: var(--bg2); border-radius: 12px; padding: 12px 18px; flex: 1; min-width: 100px; }
  .stat-card .sc-label { font-size: 12px; color: var(--text2); }
  .stat-card .sc-val { font-size: 22px; font-weight: 700; margin-top: 2px; }
  .rooms-table { width: 100%; border-collapse: collapse; }
  .rooms-table th { text-align: left; font-size: 13px; color: var(--text2); font-weight: 500; padding: 8px 12px; border-bottom: 1.5px solid var(--border); }
  .rooms-table td { padding: 10px 12px; font-size: 14px; border-bottom: 1px solid var(--border); vertical-align: middle; }
  .rooms-table tr:last-child td { border-bottom: none; }
  .rooms-table tr:hover td { background: var(--bg2); }
  .badge { display: inline-block; font-size: 11px; padding: 3px 9px; border-radius: 20px; font-weight: 500; }
  .badge.active { background: #d1fae5; color: #065f46; }
  .badge.expired { background: #fee2e2; color: #991b1b; }
  .badge.extended { background: #dbeafe; color: #1e40af; }
  .act-btn { background: none; border: 1px solid var(--border); border-radius: 7px; padding: 4px 10px; font-size: 12px; cursor: pointer; color: var(--text); margin-right: 4px; }
  .act-btn:hover { background: var(--bg2); }
  .act-btn.danger { border-color: #fca5a5; color: #dc2626; }
  .act-btn.danger:hover { background: #fef2f2; }
  .empty-state { text-align: center; padding: 60px 20px; color: var(--text2); font-size: 15px; }

  /* ─── MODAL ─── */
  .modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,.4); display: none; align-items: center; justify-content: center; z-index: 100; }
  .modal-overlay.open { display: flex; }
  .modal { background: var(--bg); border-radius: 16px; padding: 28px 24px; width: 90%; max-width: 360px; }
  .modal h3 { font-size: 18px; font-weight: 700; margin-bottom: 8px; }
  .modal p { font-size: 14px; color: var(--text2); margin-bottom: 20px; line-height: 1.5; }
  .modal input { width: 100%; height: 46px; border: 1.5px solid var(--border); border-radius: 12px; padding: 0 14px; font-size: 15px; outline: none; margin-bottom: 14px; }
  .modal input:focus { border-color: var(--blue); }
  .modal .modal-btns { display: flex; gap: 10px; }
  .modal .modal-btns button { flex: 1; height: 46px; border-radius: 12px; border: none; font-size: 15px; font-weight: 600; cursor: pointer; }
  .modal .modal-btns .m-cancel { background: var(--bg3); color: var(--text); }
  .modal .modal-btns .m-confirm { background: var(--blue); color: #fff; }
  .modal .modal-btns .m-danger { background: #dc2626; color: #fff; }

  /* ─── TOAST ─── */
  .toast { position: fixed; bottom: 32px; left: 50%; transform: translateX(-50%) translateY(20px); background: rgba(28,28,30,.88); color: #fff; font-size: 14px; padding: 10px 20px; border-radius: 20px; z-index: 200; opacity: 0; transition: all .25s; pointer-events: none; white-space: nowrap; }
  .toast.show { opacity: 1; transform: translateX(-50%) translateY(0); }

  /* ─── SHARE MODAL ─── */
  .share-code-box { background: var(--bg2); border-radius: 12px; padding: 16px; text-align: center; margin-bottom: 14px; }
  .share-code-box .code { font-size: 32px; font-weight: 800; letter-spacing: 6px; color: var(--text); font-variant-numeric: tabular-nums; }
  .share-code-box .code-hint { font-size: 12px; color: var(--text2); margin-top: 4px; }
  .share-url { width: 100%; background: var(--bg2); border: 1px solid var(--border); border-radius: 10px; padding: 10px 12px; font-size: 13px; color: var(--text2); word-break: break-all; margin-bottom: 14px; }

  @media (max-width: 500px) {
    .bubble { font-size: 15px; padding: 9px 13px; }
  }
</style>
</head>
<body>

<!-- TOAST -->
<div class="toast" id="toast"></div>

<!-- MODAL: admin login -->
<div class="modal-overlay" id="modal-admin">
  <div class="modal">
    <h3>관리자 로그인</h3>
    <p>관리자 비밀번호를 입력하세요.</p>
    <input type="password" id="admin-pw-input" placeholder="비밀번호" />
    <div class="modal-btns">
      <button class="m-cancel" onclick="closeModal('modal-admin')">취소</button>
      <button class="m-confirm" onclick="tryAdminLogin()">로그인</button>
    </div>
  </div>
</div>

<!-- MODAL: share -->
<div class="modal-overlay" id="modal-share">
  <div class="modal">
    <h3>채팅방 공유</h3>
    <div class="share-code-box">
      <div class="code" id="share-code-display"></div>
      <div class="code-hint">입장 코드</div>
    </div>
    <div class="share-url" id="share-url-display"></div>
    <div class="modal-btns">
      <button class="m-cancel" onclick="closeModal('modal-share')">닫기</button>
      <button class="m-confirm" onclick="copyShareLink()">링크 복사</button>
    </div>
  </div>
</div>

<!-- MODAL: delete confirm -->
<div class="modal-overlay" id="modal-delete">
  <div class="modal">
    <h3>채팅방 삭제</h3>
    <p id="delete-msg">이 채팅방을 삭제하면 모든 대화 내용이 사라집니다. 계속하시겠습니까?</p>
    <div class="modal-btns">
      <button class="m-cancel" onclick="closeModal('modal-delete')">취소</button>
      <button class="m-danger" onclick="confirmDelete()">삭제</button>
    </div>
  </div>
</div>

<!-- MODAL: extend timer -->
<div class="modal-overlay" id="modal-extend">
  <div class="modal">
    <h3>타이머 연장</h3>
    <p>이 채팅방의 타이머를 15분 연장합니다.</p>
    <div class="modal-btns">
      <button class="m-cancel" onclick="closeModal('modal-extend')">취소</button>
      <button class="m-confirm" onclick="confirmExtend()">연장</button>
    </div>
  </div>
</div>

<!-- PAGE: HOME -->
<div class="page active" id="home">
  <div class="home-inner">
    <div class="logo-area">
      <div class="logo-bubble">
        <div class="lbub l"></div>
        <div class="lbub r"></div>
        <div class="lbub l"></div>
        <div class="lbub r"></div>
      </div>
      <h1>익명 메시지</h1>
      <p>회원가입 없이 익명으로 대화<br>15분 타이머 · 자동 대화 보관</p>
    </div>
    <button class="home-btn primary" onclick="createRoom()">새 채팅방 만들기</button>
    <div class="divider"><span>또는 코드로 입장</span></div>
    <div class="join-row">
      <input id="join-code-input" placeholder="6자리 입장 코드" maxlength="8" oninput="this.value=this.value.toUpperCase()" onkeydown="if(event.key==='Enter')joinRoom()" />
      <button onclick="joinRoom()">입장</button>
    </div>
    <span class="admin-link" onclick="openAdminLogin()">관리자 페이지</span>
  </div>
</div>

<!-- PAGE: CHAT -->
<div class="page" id="chat">
  <div class="chat-header">
    <button class="back-btn" onclick="goHome()">
      <svg viewBox="0 0 10 16" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M8 1L2 8L8 15" stroke="#007aff" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>
      홈
    </button>
    <div class="chat-title">
      <div class="room-name">익명 채팅</div>
      <div class="room-id" id="chat-room-id"></div>
    </div>
    <button class="share-btn" onclick="openShareModal()">공유</button>
  </div>
  <div class="timer-bar" id="timer-bar">
    <div class="t-dot"></div>
    <span class="t-label">남은 시간</span>
    <span class="t-count" id="timer-display">15:00</span>
  </div>
  <div id="expired-banner" style="display:none" class="expired-banner">⏰ 타이머가 만료되었습니다. 대화 내용은 보관됩니다.</div>
  <div class="messages-wrap" id="messages-wrap">
    <div class="messages" id="messages"></div>
  </div>
  <div class="input-area" id="input-area">
    <div class="input-inner">
      <textarea id="msg-input" placeholder="메시지 입력..." rows="1" onkeydown="handleKey(event)" oninput="autoResize(this);toggleSendBtn()"></textarea>
      <button class="send-btn" id="send-btn" onclick="sendMessage()" disabled>
        <svg viewBox="0 0 16 16" xmlns="http://www.w3.org/2000/svg"><path d="M2 14L14 8L2 2V6.5L10 8L2 9.5V14Z"/></svg>
      </button>
    </div>
  </div>
</div>

<!-- PAGE: ADMIN -->
<div class="page" id="admin">
  <div class="admin-header">
    <h2>관리자</h2>
    <button onclick="goHome()">← 홈으로</button>
  </div>
  <div class="admin-body">
    <div class="admin-stats" id="admin-stats"></div>
    <div id="rooms-container"></div>
  </div>
</div>

<script>
// ─── STORAGE KEY PREFIX ───
const S = 'anonchat_';
const TIMER_MINS = 15;
const ADMIN_PW = 'admin1234'; // 기본 관리자 비밀번호

// ─── STATE ───
let currentRoomId = null;
let myNickname = null;
let timerInterval = null;
let pollInterval = null;
let adminDeleteTarget = null;
let adminExtendTarget = null;
let bc = null; // BroadcastChannel

// ─── UTILS ───
function randId(len = 6) {
  const chars = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789';
  return Array.from({length: len}, () => chars[Math.floor(Math.random() * chars.length)]).join('');
}
function genNickname() {
  const adj = ['빠른','조용한','밝은','따뜻한','차가운','예리한','부드러운','용감한','현명한','신비한','활발한','느긋한'];
  const noun = ['참새','고양이','여우','늑대','독수리','펭귄','토끼','곰','호랑이','돌고래','사슴','너구리'];
  return adj[Math.floor(Math.random()*adj.length)] + noun[Math.floor(Math.random()*noun.length)];
}
function getOrSetNickname(roomId) {
  const key = S + 'nick_' + roomId;
  let nick = sessionStorage.getItem(key);
  if (!nick) { nick = genNickname(); sessionStorage.setItem(key, nick); }
  return nick;
}
function nowMs() { return Date.now(); }
function fmtTime(ts) {
  const d = new Date(ts);
  const h = d.getHours(), m = d.getMinutes();
  const ampm = h < 12 ? '오전' : '오후';
  return `${ampm} ${h % 12 || 12}:${String(m).padStart(2,'0')}`;
}
function fmtDate(ts) {
  const d = new Date(ts);
  return `${d.getFullYear()}년 ${d.getMonth()+1}월 ${d.getDate()}일`;
}

// ─── ROOM DATA ───
function getRooms() {
  try { return JSON.parse(localStorage.getItem(S + 'rooms') || '{}'); } catch { return {}; }
}
function saveRooms(rooms) { localStorage.setItem(S + 'rooms', JSON.stringify(rooms)); }
function getRoom(id) { return getRooms()[id] || null; }
function saveRoom(id, data) {
  const rooms = getRooms();
  rooms[id] = data;
  saveRooms(rooms);
}
function createRoomData(id) {
  return { id, created: nowMs(), expiresAt: nowMs() + TIMER_MINS * 60 * 1000, messages: [], extended: false };
}
function getMessages(roomId) {
  const r = getRoom(roomId);
  return r ? r.messages || [] : [];
}
function addMessage(roomId, msg) {
  const rooms = getRooms();
  if (!rooms[roomId]) return;
  rooms[roomId].messages = rooms[roomId].messages || [];
  rooms[roomId].messages.push(msg);
  saveRooms(rooms);
  try { const ch = new BroadcastChannel('anonchat_' + roomId); ch.postMessage({type:'new_msg', msg}); ch.close(); } catch {}
}

// ─── PAGES ───
function showPage(id) {
  document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
  document.getElementById(id).classList.add('active');
}
function goHome() {
  stopTimerInterval();
  stopPoll();
  if (bc) { try { bc.close(); } catch {} bc = null; }
  currentRoomId = null;
  showPage('home');
  document.getElementById('join-code-input').value = '';
}

// ─── HOME ───
function createRoom() {
  const id = randId(6);
  const roomData = createRoomData(id);
  saveRoom(id, roomData);
  enterRoom(id);
}
function joinRoom() {
  const code = document.getElementById('join-code-input').value.trim().toUpperCase();
  if (!code) { toast('코드를 입력해 주세요.'); return; }
  const r = getRoom(code);
  if (!r) { toast('존재하지 않는 채팅방입니다.'); return; }
  enterRoom(code);
}

// ─── ENTER ROOM ───
function enterRoom(id) {
  currentRoomId = id;
  myNickname = getOrSetNickname(id);
  document.getElementById('chat-room-id').textContent = '코드: ' + id;
  showPage('chat');
  renderMessages();
  startTimer();
  startPoll();
  // BroadcastChannel for same-browser real-time sync
  try {
    if (bc) bc.close();
    bc = new BroadcastChannel('anonchat_' + id);
    bc.onmessage = (e) => { if (e.data && e.data.type === 'new_msg') renderMessages(); };
  } catch {}
  const el = document.getElementById('msg-input');
  el.value = ''; el.style.height = ''; el.disabled = false;
  document.getElementById('send-btn').disabled = true;
  updateInputState();
}

// ─── RENDER MESSAGES ───
function renderMessages() {
  const msgs = getMessages(currentRoomId);
  const container = document.getElementById('messages');
  container.innerHTML = '';
  let lastDate = null;
  msgs.forEach((msg, i) => {
    const d = fmtDate(msg.ts);
    if (d !== lastDate) {
      const sep = document.createElement('div');
      sep.className = 'date-sep';
      sep.textContent = d;
      container.appendChild(sep);
      lastDate = d;
    }
    const isOut = msg.nick === myNickname;
    const row = document.createElement('div');
    row.className = 'msg-row ' + (isOut ? 'out' : 'in');
    // show sender label for incoming
    if (!isOut) {
      const sl = document.createElement('div');
      sl.className = 'sender-label';
      sl.textContent = msg.nick;
      row.appendChild(sl);
    }
    const bub = document.createElement('div');
    bub.className = 'bubble';
    bub.textContent = msg.text;
    row.appendChild(bub);
    const bt = document.createElement('div');
    bt.className = 'bubble-time';
    bt.textContent = fmtTime(msg.ts);
    row.appendChild(bt);
    container.appendChild(row);
  });
  const wrap = document.getElementById('messages-wrap');
  wrap.scrollTop = wrap.scrollHeight;
}

// ─── SEND MESSAGE ───
function sendMessage() {
  const input = document.getElementById('msg-input');
  const text = input.value.trim();
  if (!text || !currentRoomId) return;
  const r = getRoom(currentRoomId);
  if (!r) return;
  if (nowMs() > r.expiresAt) { toast('타이머가 만료되어 메시지를 보낼 수 없습니다.'); return; }
  const msg = { nick: myNickname, text, ts: nowMs() };
  addMessage(currentRoomId, msg);
  input.value = '';
  input.style.height = '';
  document.getElementById('send-btn').disabled = true;
  renderMessages();
}
function handleKey(e) {
  if (e.key === 'Enter' && !e.shiftKey) { e.preventDefault(); sendMessage(); }
}
function autoResize(el) {
  el.style.height = 'auto';
  el.style.height = Math.min(el.scrollHeight, 120) + 'px';
}
function toggleSendBtn() {
  const v = document.getElementById('msg-input').value.trim();
  document.getElementById('send-btn').disabled = !v;
}

// ─── TIMER ───
function startTimer() {
  stopTimerInterval();
  updateTimerDisplay();
  timerInterval = setInterval(updateTimerDisplay, 1000);
}
function stopTimerInterval() { if (timerInterval) { clearInterval(timerInterval); timerInterval = null; } }
function updateTimerDisplay() {
  if (!currentRoomId) return;
  const r = getRoom(currentRoomId);
  if (!r) return;
  const left = r.expiresAt - nowMs();
  const bar = document.getElementById('timer-bar');
  const disp = document.getElementById('timer-display');
  if (left <= 0) {
    disp.textContent = '00:00';
    bar.classList.add('warning');
    stopTimerInterval();
    document.getElementById('expired-banner').style.display = '';
    updateInputState();
    return;
  }
  const mins = Math.floor(left / 60000);
  const secs = Math.floor((left % 60000) / 1000);
  disp.textContent = String(mins).padStart(2,'0') + ':' + String(secs).padStart(2,'0');
  if (left < 3 * 60 * 1000) bar.classList.add('warning'); else bar.classList.remove('warning');
}
function updateInputState() {
  const r = currentRoomId ? getRoom(currentRoomId) : null;
  const expired = !r || (nowMs() > r.expiresAt);
  const input = document.getElementById('msg-input');
  input.disabled = expired;
  input.placeholder = expired ? '타이머가 만료되었습니다.' : '메시지 입력...';
}

// ─── POLL (cross-tab sync fallback) ───
function startPoll() {
  stopPoll();
  pollInterval = setInterval(() => { renderMessages(); updateTimerDisplay(); }, 2000);
}
function stopPoll() { if (pollInterval) { clearInterval(pollInterval); pollInterval = null; } }

// ─── SHARE ───
function openShareModal() {
  document.getElementById('share-code-display').textContent = currentRoomId;
  const url = location.origin + location.pathname + '?room=' + currentRoomId;
  document.getElementById('share-url-display').textContent = url;
  openModal('modal-share');
}
function copyShareLink() {
  const url = location.origin + location.pathname + '?room=' + currentRoomId;
  navigator.clipboard.writeText(url).then(() => toast('링크가 복사되었습니다!')).catch(() => toast('링크: ' + url));
  closeModal('modal-share');
}

// ─── ADMIN ───
function openAdminLogin() { openModal('modal-admin'); document.getElementById('admin-pw-input').value = ''; }
function tryAdminLogin() {
  const pw = document.getElementById('admin-pw-input').value;
  if (pw === ADMIN_PW) { closeModal('modal-admin'); renderAdmin(); showPage('admin'); }
  else { toast('비밀번호가 올바르지 않습니다.'); }
}
function renderAdmin() {
  const rooms = getRooms();
  const ids = Object.keys(rooms);
  // stats
  const now = nowMs();
  const active = ids.filter(id => rooms[id].expiresAt > now).length;
  const totalMsgs = ids.reduce((s, id) => s + (rooms[id].messages||[]).length, 0);
  document.getElementById('admin-stats').innerHTML = `
    <div class="stat-card"><div class="sc-label">전체 채팅방</div><div class="sc-val">${ids.length}</div></div>
    <div class="stat-card"><div class="sc-label">활성 채팅방</div><div class="sc-val">${active}</div></div>
    <div class="stat-card"><div class="sc-label">만료 채팅방</div><div class="sc-val">${ids.length - active}</div></div>
    <div class="stat-card"><div class="sc-label">총 메시지 수</div><div class="sc-val">${totalMsgs}</div></div>
  `;
  // rooms list
  const container = document.getElementById('rooms-container');
  if (ids.length === 0) {
    container.innerHTML = '<div class="empty-state">생성된 채팅방이 없습니다.</div>';
    return;
  }
  const sorted = ids.sort((a, b) => rooms[b].created - rooms[a].created);
  let html = `<table class="rooms-table">
    <thead><tr>
      <th>코드</th><th>생성시간</th><th>만료시간</th><th>메시지</th><th>상태</th><th>관리</th>
    </tr></thead><tbody>`;
  sorted.forEach(id => {
    const r = rooms[id];
    const exp = r.expiresAt > now ? '<span class="badge active">활성</span>' : '<span class="badge expired">만료</span>';
    const extended = r.extended ? '<span class="badge extended" style="margin-left:4px">연장됨</span>' : '';
    const created = new Date(r.created).toLocaleString('ko-KR');
    const expires = new Date(r.expiresAt).toLocaleString('ko-KR');
    const msgCnt = (r.messages||[]).length;
    html += `<tr>
      <td><strong>${id}</strong></td>
      <td style="font-size:13px;color:var(--text2)">${created}</td>
      <td style="font-size:13px;color:var(--text2)">${expires}</td>
      <td>${msgCnt}</td>
      <td>${exp}${extended}</td>
      <td>
        <button class="act-btn" onclick="adminViewRoom('${id}')">보기</button>
        <button class="act-btn" onclick="adminExtend('${id}')">연장</button>
        <button class="act-btn danger" onclick="adminDelete('${id}')">삭제</button>
      </td>
    </tr>`;
  });
  html += '</tbody></table>';
  container.innerHTML = html;
}
function adminViewRoom(id) {
  enterRoom(id);
}
function adminDelete(id) {
  adminDeleteTarget = id;
  document.getElementById('delete-msg').textContent = `채팅방 [${id}]을 삭제합니다. 모든 대화 내용이 사라집니다.`;
  openModal('modal-delete');
}
function confirmDelete() {
  if (!adminDeleteTarget) return;
  const rooms = getRooms();
  delete rooms[adminDeleteTarget];
  saveRooms(rooms);
  adminDeleteTarget = null;
  closeModal('modal-delete');
  toast('채팅방이 삭제되었습니다.');
  renderAdmin();
}
function adminExtend(id) {
  adminExtendTarget = id;
  openModal('modal-extend');
}
function confirmExtend() {
  if (!adminExtendTarget) return;
  const rooms = getRooms();
  if (rooms[adminExtendTarget]) {
    const now = nowMs();
    const base = Math.max(rooms[adminExtendTarget].expiresAt, now);
    rooms[adminExtendTarget].expiresAt = base + TIMER_MINS * 60 * 1000;
    rooms[adminExtendTarget].extended = true;
    saveRooms(rooms);
    toast('타이머가 15분 연장되었습니다.');
    if (currentRoomId === adminExtendTarget) {
      document.getElementById('expired-banner').style.display = 'none';
      updateInputState();
      startTimer();
    }
  }
  adminExtendTarget = null;
  closeModal('modal-extend');
  renderAdmin();
}

// ─── MODAL ───
function openModal(id) { document.getElementById(id).classList.add('open'); }
function closeModal(id) { document.getElementById(id).classList.remove('open'); }

// ─── TOAST ───
function toast(msg) {
  const t = document.getElementById('toast');
  t.textContent = msg;
  t.classList.add('show');
  setTimeout(() => t.classList.remove('show'), 2600);
}

// ─── URL PARAM AUTO-JOIN ───
(function init() {
  const params = new URLSearchParams(location.search);
  const roomParam = params.get('room');
  if (roomParam) {
    const r = getRoom(roomParam.toUpperCase());
    if (r) { enterRoom(roomParam.toUpperCase()); }
    else { toast('존재하지 않는 채팅방입니다.'); }
  }
})();
</script>
</body>
</html>
