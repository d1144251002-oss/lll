<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>學生作答系統 - Socket 實時版</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- 載入 Socket.io 客戶端套件 -->
    <script src="https://cdn.socket.io/4.7.2/socket.io.min.js"></script>
</head>
<body class="bg-slate-100 font-sans min-h-screen text-slate-800">

    <nav class="bg-emerald-600 text-white shadow-md">
        <div class="max-w-4xl mx-auto px-4 py-3 flex justify-between items-center">
            <div class="flex items-center space-x-2">
                <i class="fa-solid fa-user-graduate text-2xl"></i>
                <span class="font-bold text-xl tracking-wide">ClassPoll 學生端 (Socket)</span>
            </div>
            <div id="userBadge" class="hidden text-sm bg-emerald-700 px-3 py-1 rounded-full flex items-center gap-2">
                <span id="nameLabel" class="font-semibold"></span>
                <button onclick="leaveRoom()" class="text-xs bg-emerald-800 hover:bg-emerald-900 px-2 py-0.5 rounded ml-2">切換教室</button>
            </div>
        </div>
    </nav>

    <div class="max-w-md mx-auto p-4 md:p-6">

        <!-- 登入卡片 -->
        <div id="loginScreen" class="bg-white p-6 md:p-8 rounded-2xl shadow-lg mt-8">
            <h2 class="text-2xl font-bold mb-6 text-center text-slate-800">加入課堂問答</h2>
            <div class="space-y-4">
                <div>
                    <label class="block text-sm font-medium text-slate-700 mb-1">您的姓名 / 學號</label>
                    <input type="text" id="usernameInput" placeholder="例如: 王小明" class="w-full px-4 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-emerald-500 focus:outline-none">
                </div>
                <div>
                    <label class="block text-sm font-medium text-slate-700 mb-1">教室代碼 (Room Code)</label>
                    <input type="text" id="roomCodeInput" placeholder="例如: 888" value="888" class="w-full px-4 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-emerald-500 focus:outline-none uppercase font-mono tracking-wider">
                </div>
                <button onclick="enterRoom()" class="w-full bg-emerald-600 hover:bg-emerald-700 text-white font-bold py-3 rounded-lg shadow-md transition duration-200">
                    進入教室
                </button>
            </div>
        </div>

        <!-- 學生答題主區域 -->
        <div id="studentDashboard" class="hidden space-y-6 mt-4">
            <div class="bg-emerald-700 text-white p-4 rounded-xl shadow flex justify-between items-center">
                <span>教室代碼: <strong id="studentRoomCodeDisplay" class="font-mono text-xl">---</strong></span>
                <span id="statusBadge" class="text-xs bg-emerald-800 px-2.5 py-1 rounded-full font-bold flex items-center gap-1">
                    <span class="w-2 h-2 rounded-full bg-emerald-300 animate-pulse"></span> Socket 連線中
                </span>
            </div>

            <!-- 等待題目 -->
            <div id="waitingForQuestion" class="bg-white p-8 rounded-2xl shadow-md text-center py-12">
                <i class="fa-solid fa-spinner fa-spin text-4xl text-emerald-500 mb-4"></i>
                <h3 class="text-xl font-bold text-slate-700">等待老師發布題目中...</h3>
                <p class="text-slate-400 text-sm mt-2">Socket 頻道已就緒，題目發布後會立即推送至此。</p>
            </div>

            <!-- 答題卡片 -->
            <div id="studentQuestionCard" class="hidden bg-white p-6 rounded-2xl shadow-md">
                <span class="bg-emerald-100 text-emerald-800 text-xs font-bold px-2.5 py-1 rounded-full">單選題</span>
                <h2 id="displayQuestionText" class="text-xl font-bold text-slate-800 my-4">---</h2>

                <div id="optionsContainer" class="space-y-3"></div>

                <div id="submittedNotice" class="hidden mt-6 bg-emerald-50 border border-emerald-200 text-emerald-800 p-4 rounded-xl text-center">
                    <i class="fa-solid fa-circle-check text-2xl text-emerald-500 mb-1"></i>
                    <p class="font-bold">答案已透過 Socket 即時傳送！</p>
                    <p class="text-xs text-emerald-600 mt-1">您可以點選其他選項修改答案。</p>
                </div>
            </div>
        </div>

    </div>

    <script>
        // 免費 Socket 廣播中繼伺服器 (直接連接)
        const socket = io("https://pie-socket-server.herokuapp.com", {
            transports: ['websocket', 'polling']
        });

        let currentUser = '';
        let currentRoom = '';
        let mySelectedOption = null;

        // 監聽 Socket 事件：接收老師發布的新題目
        socket.on("NEW_QUESTION", (data) => {
            if (data.room === currentRoom) {
                mySelectedOption = null; // 重置答題狀態
                showQuestion(data.questionData);
            }
        });

        // 進入教室
        function enterRoom() {
            const name = document.getElementById('usernameInput').value.trim();
            const room = document.getElementById('roomCodeInput').value.trim().toUpperCase();

            if (!name || !room) {
                alert('請輸入姓名與教室代碼！');
                return;
            }

            currentUser = name;
            currentRoom = room;

            // 告知 Socket 伺服器加入該房間
            socket.emit("JOIN_ROOM", { room: currentRoom, name: currentUser });

            document.getElementById('loginScreen').classList.add('hidden');
            document.getElementById('userBadge').classList.remove('hidden');
            document.getElementById('nameLabel').innerText = currentUser;
            document.getElementById('studentDashboard').classList.remove('hidden');
            document.getElementById('studentRoomCodeDisplay').innerText = currentRoom;
        }

        function leaveRoom() {
            document.getElementById('studentDashboard').classList.add('hidden');
            document.getElementById('userBadge').classList.add('hidden');
            document.getElementById('loginScreen').classList.remove('hidden');
        }

        function showQuestion(questionData) {
            document.getElementById('waitingForQuestion').classList.add('hidden');
            document.getElementById('studentQuestionCard').classList.remove('hidden');
            document.getElementById('displayQuestionText').innerText = questionData.question;

            const container = document.getElementById('optionsContainer');
            container.innerHTML = '';

            questionData.options.forEach((opt, index) => {
                const btn = document.createElement('button');
                const isSelected = mySelectedOption === opt;
                btn.className = `w-full text-left p-4 rounded-xl border-2 transition font-medium flex justify-between items-center ${
                    isSelected 
                    ? 'border-emerald-600 bg-emerald-50 text-emerald-800 font-bold' 
                    : 'border-slate-200 hover:border-emerald-300 hover:bg-slate-50 text-slate-700'
                }`;
                btn.innerHTML = `
                    <span>${String.fromCharCode(65 + index)}. ${opt}</span>
                    ${isSelected ? '<i class="fa-solid fa-circle-check text-emerald-600"></i>' : ''}
                `;
                btn.onclick = () => submitAnswer(opt, questionData);
                container.appendChild(btn);
            });

            if (mySelectedOption) {
                document.getElementById('submittedNotice').classList.remove('hidden');
            } else {
                document.getElementById('submittedNotice').classList.add('hidden');
            }
        }

        function submitAnswer(selectedOption, questionData) {
            mySelectedOption = selectedOption;

            // 透過 Socket 將學生的答案發送給老師
            socket.emit("SUBMIT_ANSWER", {
                room: currentRoom,
                studentName: currentUser,
                option: selectedOption
            });

            showQuestion(questionData);
        }
    </script>
</body>
</html>
