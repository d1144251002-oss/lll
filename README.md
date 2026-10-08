<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>互動式課堂問答系統</title>
    <!-- Tailwind CSS (樣式庫) -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome (圖示庫) -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Chart.js (圖表庫) -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body class="bg-slate-100 font-sans min-h-screen text-slate-800">

    <!-- 頂部導覽列 -->
    <nav class="bg-indigo-600 text-white shadow-md">
        <div class="max-w-6xl mx-auto px-4 py-3 flex justify-between items-center">
            <div class="flex items-center space-x-2">
                <i class="fa-solid fa-graduation-cap text-2xl"></i>
                <span class="font-bold text-xl tracking-wide">ClassPoll 課堂即時問答</span>
            </div>
            <div id="userBadge" class="hidden text-sm bg-indigo-700 px-3 py-1 rounded-full flex items-center gap-2">
                <span id="roleLabel" class="font-semibold"></span>
                <span id="nameLabel" class="text-indigo-200"></span>
            </div>
        </div>
    </nav>

    <!-- 主要內容區 -->
    <div class="max-w-4xl mx-auto p-4 md:p-6">

        <!-- 角色選擇頁面 (初始) -->
        <div id="roleSelectScreen" class="bg-white p-6 md:p-10 rounded-2xl shadow-lg text-center mt-8">
            <h1 class="text-3xl font-extrabold text-slate-800 mb-3">歡迎使用即時問答系統</h1>
            <p class="text-slate-500 mb-8">請選擇您的身分以開始使用</p>
            
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6 max-w-lg mx-auto">
                <button onclick="selectRole('teacher')" class="p-6 border-2 border-indigo-200 hover:border-indigo-500 rounded-xl bg-indigo-50 hover:bg-indigo-100 transition duration-200 text-indigo-700 font-bold text-lg flex flex-col items-center gap-3">
                    <i class="fa-solid fa-chalkboard-user text-4xl"></i>
                    我是教師 (建立與主持問答)
                </button>
                <button onclick="selectRole('student')" class="p-6 border-2 border-emerald-200 hover:border-emerald-500 rounded-xl bg-emerald-50 hover:bg-emerald-100 transition duration-200 text-emerald-700 font-bold text-lg flex flex-col items-center gap-3">
                    <i class="fa-solid fa-user-graduate text-4xl"></i>
                    我是學生 (加入與回答問題)
                </button>
            </div>
        </div>

        <!-- 登入與教室碼輸入視窗 -->
        <div id="loginScreen" class="hidden bg-white p-6 md:p-8 rounded-2xl shadow-lg max-w-md mx-auto mt-8">
            <h2 id="loginTitle" class="text-2xl font-bold mb-6 text-center text-slate-800">登入系統</h2>
            <div class="space-y-4">
                <div>
                    <label class="block text-sm font-medium text-slate-700 mb-1">您的名稱</label>
                    <input type="text" id="usernameInput" placeholder="例如: 王小明 或 張老師" class="w-full px-4 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:outline-none">
                </div>
                <div>
                    <label class="block text-sm font-medium text-slate-700 mb-1">教室代碼 (Room Code)</label>
                    <input type="text" id="roomCodeInput" placeholder="例如: 888" value="888" class="w-full px-4 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:outline-none uppercase font-mono tracking-wider">
                </div>
                <button onclick="enterRoom()" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 rounded-lg shadow-md transition duration-200">
                    進入教室
                </button>
                <button onclick="backToRoleSelect()" class="w-full text-slate-500 hover:text-slate-700 text-sm py-2">
                    ← 返回選擇角色
                </button>
            </div>
        </div>

        <!-- 【教師控制台】 -->
        <div id="teacherDashboard" class="hidden space-y-6">
            <!-- 教室資訊卡片 -->
            <div class="bg-indigo-900 text-white p-6 rounded-2xl shadow-md flex flex-wrap justify-between items-center gap-4">
                <div>
                    <span class="text-indigo-300 text-sm font-medium">當前教室代碼</span>
                    <div class="text-3xl font-black font-mono tracking-widest text-amber-300" id="teacherRoomCodeDisplay">---</div>
                </div>
                <div class="flex items-center gap-3">
                    <button onclick="generateBotAnswers()" class="bg-indigo-700 hover:bg-indigo-600 text-indigo-100 text-sm px-4 py-2 rounded-lg transition border border-indigo-500">
                        <i class="fa-solid fa-robot mr-1"></i> 模擬 5 位學生回答
                    </button>
                    <button onclick="clearCurrentAnswers()" class="bg-rose-600 hover:bg-rose-700 text-white text-sm px-4 py-2 rounded-lg transition">
                        <i class="fa-solid fa-trash mr-1"></i> 清除本次回答
                    </button>
                </div>
            </div>

            <!-- 出題區域 -->
            <div class="bg-white p-6 rounded-2xl shadow-md">
                <h3 class="text-lg font-bold mb-4 text-slate-800 border-b pb-2">
                    <i class="fa-solid fa-pen-to-square text-indigo-600 mr-2"></i>發布新題目
                </h3>
                <div class="space-y-4">
                    <div>
                        <label class="block text-sm font-medium text-slate-700 mb-1">題目內容</label>
                        <input type="text" id="questionInput" value="請問下列何者是台灣最高的山？" class="w-full px-4 py-2 border border-slate-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:outline-none">
                    </div>

                    <div class="grid grid-cols-1 md:grid-cols-2 gap-3">
                        <div>
                            <label class="block text-xs text-slate-500 mb-1">選項 A</label>
                            <input type="text" id="optA" value="玉山" class="w-full px-3 py-2 border rounded-lg text-sm">
                        </div>
                        <div>
                            <label class="block text-xs text-slate-500 mb-1">選項 B</label>
                            <input type="text" id="optB" value="雪山" class="w-full px-3 py-2 border rounded-lg text-sm">
                        </div>
                        <div>
                            <label class="block text-xs text-slate-500 mb-1">選項 C</label>
                            <input type="text" id="optC" value="陽明山" class="w-full px-3 py-2 border rounded-lg text-sm">
                        </div>
                        <div>
                            <label class="block text-xs text-slate-500 mb-1">選項 D</label>
                            <input type="text" id="optD" value="阿里山" class="w-full px-3 py-2 border rounded-lg text-sm">
                        </div>
                    </div>

                    <button onclick="publishQuestion()" class="w-full bg-emerald-600 hover:bg-emerald-700 text-white font-bold py-3 rounded-lg shadow-md transition">
                        <i class="fa-solid fa-paper-plane mr-2"></i>推播題目給學生
                    </button>
                </div>
            </div>

            <!-- 實時統計數據 -->
            <div class="bg-white p-6 rounded-2xl shadow-md">
                <div class="flex justify-between items-center mb-4 border-b pb-2">
                    <h3 class="text-lg font-bold text-slate-800">
                        <i class="fa-solid fa-chart-column text-indigo-600 mr-2"></i>實時答題統計
                    </h3>
                    <span class="text-sm bg-slate-100 text-slate-600 px-3 py-1 rounded-full font-bold">
                        已回答人數: <span id="totalRespondedCount" class="text-indigo-600 text-base">0</span> 人
                    </span>
                </div>

                <div class="w-full max-w-md mx-auto my-4">
                    <canvas id="pollChart"></canvas>
                </div>

                <!-- 答題名單列表 -->
                <div class="mt-6">
                    <h4 class="text-sm font-semibold text-slate-600 mb-2">已提交答案的學生明細：</h4>
                    <div id="studentResponsesList" class="max-h-40 overflow-y-auto space-y-1 bg-slate-50 p-3 rounded-lg border
