<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ระบบเช็คชื่อกิจกรรมยุคดิจิทัล (Strict Privacy Control)</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
    <!-- face-api.js for Face Detection -->
    <script defer src="https://cdn.jsdelivr.net/npm/@vladmandic/face-api/dist/face-api.js"></script>
</head>
<body class="bg-slate-100 font-sans min-h-screen pb-12">

    <!-- Header & Auth Status Bar -->
    <header class="bg-indigo-700 text-white p-6 shadow-lg">
        <div class="max-w-5xl mx-auto flex flex-col md:flex-row justify-between items-center gap-4">
            <div>
                <h1 class="text-2xl font-bold flex items-center gap-2">
                    <i class="fa-solid fa-user-check"></i> ระบบลงทะเบียนเข้าร่วมกิจกรรม
                </h1>
                <p class="text-indigo-200 text-sm">ต้องเข้าสู่ระบบก่อนทำรายการ • ข้อมูลสรุปและประวัติเข้าถึงได้เฉพาะครูเวร</p>
            </div>
            
            <!-- Auth Buttons Container -->
            <div id="authContainer" class="flex flex-wrap items-center gap-2">
                <!-- Guest State -->
                <div id="guestBtns" class="flex gap-2">
                    <button onclick="openStudentLoginModal()" class="bg-emerald-500 hover:bg-emerald-600 text-white font-bold px-3.5 py-2 rounded-xl text-xs transition flex items-center gap-1.5 shadow">
                        <i class="fa-solid fa-user-graduate"></i> นักเรียนเข้าสู่ระบบ
                    </button>
                    <button onclick="openTeacherLoginModal()" class="bg-amber-500 hover:bg-amber-600 text-slate-900 font-bold px-3.5 py-2 rounded-xl text-xs transition flex items-center gap-1.5 shadow">
                        <i class="fa-solid fa-user-shield"></i> ครูเวรเข้าสู่ระบบ
                    </button>
                </div>

                <!-- Logged In State Badge -->
                <div id="userProfile" class="hidden flex items-center gap-2 bg-indigo-800 border border-indigo-500 px-3 py-1.5 rounded-xl text-xs">
                    <i id="roleIcon" class="fa-solid text-amber-400"></i>
                    <span id="userNameDisplay" class="font-bold text-amber-300">ผู้ใช้งาน</span>
                    <button onclick="logout()" class="ml-2 bg-rose-600 hover:bg-rose-700 text-white px-2 py-1 rounded-lg text-[10px] font-bold transition">
                        ออกจากระบบ
                    </button>
                </div>
            </div>
        </div>
    </header>

    <main class="max-w-5xl mx-auto mt-6 px-4 space-y-6">

        <!-- Status Timeline -->
        <section class="bg-white p-6 rounded-2xl shadow-md border border-slate-200">
            <h2 class="text-lg font-bold text-slate-800 mb-4 flex items-center gap-2">
                <i class="fa-solid fa-clock-rotate-left text-indigo-600"></i> เกณฑ์เวลาการเข้าร่วมกิจกรรม
            </h2>
            <div class="relative w-full bg-slate-200 h-5 rounded-full overflow-hidden flex shadow-inner">
                <div class="w-1/2 bg-emerald-500 h-full flex items-center justify-center text-[11px] text-white font-bold">ปกติ (< 08:30)</div>
                <div class="w-1/6 bg-amber-500 h-full flex items-center justify-center text-[11px] text-white font-bold">สาย (08:30-09:00)</div>
                <div class="w-1/3 bg-rose-500 h-full flex items-center justify-center text-[11px] text-white font-bold">ไม่เข้าร่วม (> 09:00)</div>
            </div>
            <div id="statusBadge" class="mt-4 p-3 rounded-xl text-center font-bold text-sm bg-slate-100 text-slate-700 border border-slate-200">
                สถานะปัจจุบันหากลงทะเบียนตอนนี้: <span id="currentRuleStatus" class="underline">กำลังคำวณ...</span>
            </div>
        </section>

        <!-- Form Check-in (With Lock Screen when not logged in) -->
        <section class="bg-white p-6 rounded-2xl shadow-md border border-slate-200 relative overflow-hidden">
            
            <!-- Auth Lock Overlay -->
            <div id="formLockOverlay" class="absolute inset-0 bg-slate-900/40 backdrop-blur-[2px] z-20 flex flex-col items-center justify-center p-6 text-center transition-all duration-300">
                <div class="bg-white p-6 rounded-2xl shadow-2xl max-w-sm w-full space-y-3 border border-slate-100">
                    <div class="w-12 h-12 bg-rose-100 text-rose-600 rounded-full flex items-center justify-center mx-auto text-xl">
                        <i class="fa-solid fa-lock"></i>
                    </div>
                    <h3 class="text-base font-bold text-slate-800">กรุณาเข้าสู่ระบบก่อนเช็คชื่อ</h3>
                    <p class="text-xs text-slate-500">นักเรียนหรือคุณครูเวร ต้องเข้าสู่ระบบด้านบนก่อน จึงจะสามารถบันทึกเวลาเข้าร่วมกิจกรรมได้</p>
                    <div class="flex gap-2 pt-2">
                        <button onclick="openStudentLoginModal()" class="w-1/2 bg-emerald-600 hover:bg-emerald-700 text-white font-bold py-2 rounded-xl text-xs transition">นักเรียน Login</button>
                        <button onclick="openTeacherLoginModal()" class="w-1/2 bg-amber-500 hover:bg-amber-600 text-slate-900 font-bold py-2 rounded-xl text-xs transition">ครูเวร Login</button>
                    </div>
                </div>
            </div>

            <div class="flex justify-between items-center mb-4">
                <h2 class="text-lg font-bold text-slate-800 flex items-center gap-2">
                    <i class="fa-solid fa-pen-to-square text-indigo-600"></i> แบบฟอร์มลงทะเบียนเช็คชื่อ
                </h2>
                <span id="studentLockNotice" class="hidden text-xs bg-emerald-100 text-emerald-800 font-bold px-2.5 py-1 rounded-lg border border-emerald-300">
                    <i class="fa-solid fa-user-check"></i> เข้าสู่ระบบเรียบร้อยแล้ว
                </span>
            </div>

            <form id="checkinForm" class="space-y-4">
                
                <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
                    <div class="md:col-span-2">
                        <label class="block text-sm font-medium text-slate-700 mb-1">ชื่อ - นามสกุล <span class="text-rose-500">*</span></label>
                        <input type="text" id="fullname" required disabled placeholder="กรุณาเข้าสู่ระบบก่อน..." class="w-full px-4 py-2.5 border border-slate-300 rounded-xl focus:ring-2 focus:ring-indigo-500 focus:outline-none disabled:bg-slate-100 disabled:cursor-not-allowed">
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-slate-700 mb-1">ระดับชั้น <span class="text-rose-500">*</span></label>
                        <input type="text" id="studentClass" required disabled placeholder="เช่น ม.1/1" class="w-full px-4 py-2.5 border border-slate-300 rounded-xl focus:ring-2 focus:ring-indigo-500 focus:outline-none disabled:bg-slate-100 disabled:cursor-not-allowed">
                    </div>
                    <div>
                        <label class="block text-sm font-medium text-slate-700 mb-1">เลขที่ <span class="text-rose-500">*</span></label>
                        <input type="number" id="studentNo" min="1" max="90" required disabled placeholder="เช่น 15" class="w-full px-4 py-2.5 border border-slate-300 rounded-xl focus:ring-2 focus:ring-indigo-500 focus:outline-none disabled:bg-slate-100 disabled:cursor-not-allowed">
                    </div>
                </div>

                <!-- Camera / Photo Input -->
                <div>
                    <label class="block text-sm font-medium text-slate-700 mb-1">ถ่ายรูปโปรไฟล์ (ตรวจจับใบหน้า) <span class="text-rose-500">*</span></label>
                    <input type="file" id="photo" accept="image/*" capture="user" required disabled class="w-full text-sm text-slate-500 file:mr-4 file:py-2.5 file:px-4 file:rounded-xl file:border-0 file:text-sm file:font-semibold file:bg-indigo-50 file:text-indigo-700 hover:file:bg-indigo-100 cursor-pointer disabled:opacity-50 disabled:cursor-not-allowed">
                    
                    <div id="faceStatus" class="mt-2 text-xs font-bold hidden"></div>
                    <div id="imagePreview" class="mt-2 hidden relative w-36 h-36">
                        <img id="previewImg" src="" alt="Preview" class="w-full h-full rounded-xl border border-slate-300 object-cover shadow-sm">
                    </div>
                </div>

                <!-- GPS Location -->
                <div>
                    <div class="flex justify-between items-center mb-1">
                        <label class="block text-sm font-medium text-slate-700">พิกัดตำแหน่ง GPS <span class="text-rose-500">*</span></label>
                        <span id="geoStatus" class="text-xs font-bold px-2 py-0.5 rounded-full bg-slate-100 text-slate-500">รอตรวจสอบ</span>
                    </div>
                    <div class="flex gap-2">
                        <input type="text" id="location" readonly placeholder="กดปุ่มเพื่อตรวจสอบพิกัด GPS" class="w-full px-4 py-2.5 bg-slate-50 border border-slate-300 rounded-xl text-sm text-slate-600" required>
                        <button type="button" id="getGpsBtn" onclick="getLocation()" disabled class="bg-slate-800 text-white px-5 py-2.5 rounded-xl text-sm hover:bg-slate-700 transition flex items-center whitespace-nowrap gap-1 disabled:bg-slate-400 disabled:cursor-not-allowed">
                            <i class="fa-solid fa-location-crosshairs"></i> ดึงพิกัด GPS
                        </button>
                    </div>
                </div>

                <button type="submit" id="submitBtn" disabled class="w-full bg-indigo-600 text-white font-bold py-3.5 rounded-xl shadow-md hover:bg-indigo-700 active:scale-[0.99] transition duration-200 text-base flex items-center
