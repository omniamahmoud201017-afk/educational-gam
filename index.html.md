<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>لعبة التعلم التفاعلية الذكية مع الملفات الصوتية</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <style>
        :root {
            --primary: #4f46e5;
            --primary-hover: #4338ca;
            --success: #10b981;
            --error: #ef4444;
            --bg: #f8fafc;
            --card-bg: #ffffff;
            --text: #1e293b;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg);
            color: var(--text);
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }

        .container {
            width: 100%;
            max-width: 950px;
            background: var(--card-bg);
            border-radius: 20px;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.05);
            padding: 30px;
            box-sizing: border-box;
            text-align: center;
            position: relative;
        }

        .screen {
            display: none;
        }

        .screen.active {
            display: block;
        }

        @keyframes bounceSlow {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-6px); }
        }

        .mezo-bounce {
            animation: bounceSlow 2.5s infinite ease-in-out;
        }

        @keyframes pulseGlow {
            0%, 100% { box-shadow: 0 0 0 0 rgba(79, 70, 229, 0.4); }
            50% { box-shadow: 0 0 0 12px rgba(79, 70, 229, 0); }
        }

        .speaking-glow {
            animation: pulseGlow 1.5s infinite;
        }
    </style>
</head>
<body>

<div class="absolute top-5 left-5 flex gap-2 z-10">
    <button class="bg-indigo-50 hover:bg-indigo-100 text-indigo-700 text-xs font-bold py-2 px-3 rounded-xl shadow-sm transition" onclick="openAiChatModal()">🤖 دردشة ذكية مع ميزو</button>
    <button id="sound-toggle-btn" class="bg-indigo-600 hover:bg-indigo-700 text-white text-xs font-semibold py-2 px-3 rounded-xl shadow-sm transition" onclick="toggleSound()">🔊 الصوت: مفعل</button>
</div>

<div class="container" id="printable-report-area">
    <!-- شاشة الترحيب -->
    <div id="welcome-screen" class="screen active">
        <div class="mb-6">
            <div id="welcome-avatar" class="w-32 h-32 mx-auto mb-3 mezo-bounce cursor-pointer flex items-center justify-center" onclick="greetWelcome()" title="انقر لسماع التحية">
                <svg width="80" height="80" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg" class="w-full h-full drop-shadow-md">
                    <path d="M 25 45 Q 25 15 50 15 Q 75 15 75 45 Q 75 55 65 60 Q 50 65 35 60 Q 25 55 25 45 Z" fill="#b2bec3" />
                    <path d="M 32 30 Q 32 22 50 22 Q 68 22 68 30 Q 68 50 50 50 Q 32 50 32 30 Z" fill="#2d3436" />
                    <circle cx="42" cy="35" r="5" fill="#81ecec" />
                    <circle cx="58" cy="35" r="5" fill="#81ecec" />
                    <path d="M 45 42 Q 50 47 55 42 Z" fill="#ffffff" />
                    <path d="M 35 65 Q 35 58 50 58 Q 65 58 65 65 L 70 85 Q 70 90 50 90 Q 30 90 30 85 Z" fill="#b2bec3" />
                    <circle cx="50" cy="72" r="3" fill="#636e72" />
                </svg>
            </div>
            <h1 class="text-3xl font-bold text-indigo-600 mb-2">أهلاً بك في لعبة التعلم الذكية مع ميزو الناطق</h1>
            <p class="text-slate-500">رفيقك الذكي الناطق بالصوت والصورة للتعلم وتنمية المهارات! انقر على ميزو لسماع الترحيب.</p>
        </div>
        <div class="bg-indigo-50 border border-indigo-100 p-6 rounded-2xl mb-6 max-w-md mx-auto">
            <label class="block text-slate-700 font-bold mb-2">الرجاء إدخال اسم البطل للبدء:</label>
            <input type="text" id="child-name" class="w-full px-4 py-3 text-lg border-2 border-slate-300 rounded-xl text-center focus:border-indigo-600 focus:outline-none transition mb-4" placeholder="اكتب اسم الطفل هنا..." autocomplete="off">
            <button onclick="startApp()" class="w-full bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-3 px-6 rounded-xl shadow-lg transition transform active:scale-95">ابدأ المغامرة الآن 🚀</button>
        </div>
    </div>

    <!-- شاشة اللعبة -->
    <div id="game-screen" class="screen">
        <div class="flex justify-between items-center mb-6 font-bold text-slate-500">
            <span id="player-display" class="bg-indigo-50 text-indigo-700 px-3 py-1 rounded-lg"></span>
            <span id="question-counter" class="bg-slate-100 px-3 py-1 rounded-lg">السؤال 1 من 5</span>
        </div>

        <div class="flex items-center bg-indigo-50 border-2 border-indigo-200 rounded-2xl p-4 mb-6 text-right gap-4 shadow-sm">
            <div id="mezo-avatar" class="w-20 h-20 flex-shrink-0 cursor-pointer mezo-bounce flex items-center justify-center" onclick="repeatMezoSpeech()" title="انقر لإعادة الاستماع">
                <svg width="55" height="55" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg" class="w-full h-full drop-shadow">
                    <path d="M 25 45 Q 25 15 50 15 Q 75 15 75 45 Q 75 55 65 60 Q 50 65 35 60 Q 25 55 25 45 Z" fill="#b2bec3" />
                    <path d="M 32 30 Q 32 22 50 22 Q 68 22 68 30 Q 68 50 50 50 Q 32 50 32 30 Z" fill="#2d3436" />
                    <circle cx="42" cy="35" r="5" fill="#81ecec" />
                    <circle cx="58" cy="35" r="5" fill="#81ecec" />
                    <path d="M 45 42 Q 50 47 55 42 Z" fill="#ffffff" />
                    <path d="M 35 65 Q 35 58 50 58 Q 65 58 65 65 L 70 85 Q 70 90 50 90 Q 30 90 30 85 Z" fill="#b2bec3" />
                    <circle cx="50" cy="72" r="3" fill="#636e72" />
                </svg>
            </div>
            <div class="flex-1">
                <div class="flex justify-between items-center mb-1">
                    <span class="text-xs font-bold text-indigo-500">ميزو (المساعد الذكي الناطق):</span>
                    <button onclick="repeatMezoSpeech()" class="text-xs bg-indigo-200 hover:bg-indigo-300 text-indigo-800 px-2 py-0.5 rounded transition">🔊 إعادة الاستماع</button>
                </div>
                <div class="text-indigo-900 font-semibold text-sm sm:text-base leading-relaxed" id="mezo-speech">مرحباً يا بطل! أنا صديقك ميزو، مستعد لنبدأ سوياً؟</div>
            </div>
        </div>

        <div class="text-xl font-bold text-slate-900 mb-4" id="question-title"></div>

        <div class="mx-auto my-4 w-44 h-44 rounded-2xl overflow-hidden shadow-md bg-slate-50 flex items-center justify-center border-2 border-indigo-100" id="target-container"></div>

        <!-- خيارات الأسئلة -->
        <div class="grid grid-cols-1 sm:grid-cols-3 gap-5 mb-6" id="options-grid"></div>

        <div class="font-bold text-lg min-h-[30px]" id="feedback-banner"></div>
    </div>

    <!-- شاشة التقرير -->
    <div id="report-screen" class="screen text-right">
        <div class="flex items-center justify-center gap-2 mb-2">
            <span class="text-3xl">🏆</span>
            <h1 class="text-2xl sm:text-3xl font-bold text-sky-600" id="rep-main-title">تقرير مغامرة البطل</h1>
        </div>
        <p class="text-slate-500 mb-6 text-center text-xs sm:text-sm">تحليل المهارات التكيفية، سرعة الاستجابة، ونسبة النجاح بعد احتساب الأخطاء</p>

        <!-- البطاقات الأربع العلوية -->
        <div class="grid grid-cols-2 sm:grid-cols-4 gap-3 mb-6">
            <div class="bg-white border-2 border-amber-300 p-3 rounded-2xl text-center shadow-sm">
                <span class="text-amber-700 text-xs font-bold block mb-1">النقاط</span>
                <strong id="rep-score" class="text-2xl text-amber-600 font-bold">75</strong>
            </div>
            <div class="bg-white border-2 border-emerald-300 p-3 rounded-2xl text-center shadow-sm">
                <span class="text-emerald-700 text-xs font-bold block mb-1">نسبة النجاح</span>
                <strong id="rep-success-rate" class="text-xl text-emerald-600 font-bold">63%</strong>
            </div>
            <div class="bg-white border-2 border-sky-300 p-3 rounded-2xl text-center shadow-sm">
                <span class="text-sky-700 text-xs font-bold block mb-1">متوسط التردد</span>
                <strong id="rep-avg-latency" class="text-xl text-sky-600 font-bold">5.6 ثانية</strong>
            </div>
            <div class="bg-white border-2 border-indigo-300 p-3 rounded-2xl text-center shadow-sm">
                <span class="text-indigo-700 text-xs font-bold block mb-1">مؤشر المرونة</span>
                <strong id="rep-flexibility" class="text-lg text-indigo-600 font-bold">جيدة ومتطورة</strong>
            </div>
        </div>

        <!-- صندوق التحليلات النصية -->
        <div class="bg-indigo-50/70 border-2 border-indigo-200 p-5 rounded-2xl mb-6 shadow-sm space-y-4 text-right">
            <div class="flex items-start gap-2">
                <span class="text-amber-500 text-lg">☀️</span>
                <div>
                    <h4 class="font-bold text-indigo-950 text-sm sm:text-base">1. تقييم أداء البطل الاستراتيجي والمعرفي</h4>
                    <p id="eval-part-1" class="text-xs sm:text-sm text-slate-700 mt-1 leading-relaxed">حقق البطل نسبة نجاح ممتازة بلغت 63% بعد اجتياز المحاولات بنجاح، مع إظهار تفاعل إيجابي وسريع مع تلميحات المساعد ميزو عند مواجهة خيارات متعددة.</p>
                </div>
            </div>
            <div class="flex items-start gap-2">
                <span class="text-amber-400 text-lg">💡</span>
                <div>
                    <h4 class="font-bold text-indigo-950 text-sm sm:text-base">2. التوصيات التربوية والتحفيزية</h4>
                    <p id="eval-part-2" class="text-xs sm:text-sm text-slate-700 mt-1 leading-relaxed">تعزيز التركيز البصري في المواقف المشابهة وتقسيم الأهداف إلى خطوات بسيطة وممتعة لزيادة الثقة بالنفس.</p>
                </div>
            </div>
            <div class="flex items-start gap-2">
                <span class="text-rose-500 text-lg">🎯</span>
                <div>
                    <h4 class="font-bold text-indigo-950 text-sm sm:text-base">3. خطة التحديات القادمة لتطوير المهارات</h4>
                    <p id="eval-part-3" class="text-xs sm:text-sm text-slate-700 mt-1 leading-relaxed">ممارسة أنشطة ترفيهية تفاعلية مشابهة لترسيخ مفاهيم المرونة السلوكية بشكل مستمر.</p>
                </div>
            </div>
        </div>

        <!-- جدول سجل الاستجابة -->
        <div class="mb-2 text-right">
            <h3 class="text-slate-800 font-bold text-sm sm:text-base mb-2">سجل الاستجابة التفاعلية لكل موقف بيئي:</h3>
        </div>
        <div class="bg-white border border-slate-200 rounded-2xl p-2 mb-6 overflow-x-auto shadow-sm">
            <table class="w-full text-right text-xs sm:text-sm border-collapse">
                <thead>
                    <tr class="bg-slate-100 text-slate-700 border-b border-slate-200">
                        <th class="p-3">الموقف البيئي (السؤال)</th>
                        <th class="p-3">مؤشر المعالجة البصرية</th>
                        <th class="p-3">أخطاء التردد</th>
                        <th class="p-3">الحالة المعرفية</th>
                    </tr>
                </thead>
                <tbody id="report-table-body" class="divide-y divide-slate-100">
                </tbody>
            </table>
        </div>

        <div class="flex flex-wrap gap-3 justify-center">
            <button onclick="downloadReportAsImage()" class="bg-sky-600 hover:bg-sky-700 text-white font-bold py-3 px-5 rounded-xl shadow-lg transition">📸 تصدير كصورة كاملة</button>
            <button onclick="downloadReportAsExcel()" class="bg-emerald-600 hover:bg-emerald-700 text-white font-bold py-3 px-5 rounded-xl shadow-lg transition">📊 تحميل كملف Excel</button>
            <button onclick="location.reload()" class="bg-amber-600 hover:bg-amber-700 text-white font-bold py-3 px-5 rounded-xl shadow-lg transition">🔄 إعادة المغامرة</button>
        </div>
    </div>
</div>

<!-- نافذة الدردشة الذكية -->
<div id="ai-chat-modal" class="fixed inset-0 bg-slate-900/50 backdrop-blur-sm z-50 flex items-center justify-center p-4 hidden">
    <div class="bg-white w-full max-w-lg rounded-2xl shadow-2xl overflow-hidden flex flex-col h-[500px]">
        <div class="bg-indigo-600 text-white p-4 flex justify-between items-center">
            <div class="flex items-center gap-2">
                <svg width="30" height="30" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
                    <path d="M 25 45 Q 25 15 50 15 Q 75 15 75 45 Q 75 55 65 60 Q 50 65 35 60 Q 25 55 25 45 Z" fill="#b2bec3" />
                    <path d="M 32 30 Q 32 22 50 22 Q 68 22 68 30 Q 68 50 50 50 Q 32 50 32 30 Z" fill="#2d3436" />
                    <circle cx="42" cy="35" r="5" fill="#81ecec" />
                    <circle cx="58" cy="35" r="5" fill="#81ecec" />
                </svg>
                <h3 class="font-bold text-lg">الدردشة الذكية مع الصديق ميزو</h3>
            </div>
            <button onclick="closeAiChatModal()" class="text-white hover:bg-indigo-700 w-8 h-8 rounded-full flex items-center justify-center text-lg font-bold">✕</button>
        </div>
        <div id="chat-messages" class="flex-1 p-4 overflow-y-auto space-y-3 bg-slate-50 text-right text-sm">
            <div class="bg-indigo-100 text-indigo-900 p-3 rounded-2xl rounded-tr-none max-w-[80%] ml-auto">
                مرحباً بك في ساحة الحوار الذكي! أنا مستعد للإجابة على أسئلتك ومساعدتك في أي لغز أو معلومة تريد معرفتها. اسألني ما شئت يا بطل!
            </div>
        </div>
        <div class="p-3 bg-white border-t border-slate-200 flex gap-2">
            <input type="text" id="chat-input" class="flex-1 px-4 py-2 border border-slate-300 rounded-xl text-sm focus:outline-none focus:border-indigo-600" placeholder="اكتب سؤالك هنا لميزو..." onkeypress="if(event.key==='Enter') sendChatMessage()">
            <button onclick="sendChatMessage()" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold px-5 py-2 rounded-xl text-sm transition">إرسال 🚀</button>
        </div>
    </div>
</div>

<script>
    const IMGS = {
        lion: "Gemini_Generated_Image_gyfzxtgyfzxtgyfz.jpg",
        lionShadow: "Gemini_Generated_Image_hfo0ruhfo0ruhfo0.jpg",
        giraffeShadow: "Gemini_Generated_Image_6miuh96miuh96miu.jpg",
        elephantShadow: "Gemini_Generated_Image_f1x7yof1x7yof1x7.jpg",
        
        grapes: "Gemini_Generated_Image_aiztc4aiztc4aizt.jpg",
        grapesShadow: "Gemini_Generated_Image_hqmgcdhqmgcdhqmg.jpg",
        bananaShadow: "Gemini_Generated_Image_4p2zq44p2zq44p2z.jpg",
        plumShadow: "Gemini_Generated_Image_b6s2sb6s2sb6s2sb.jpg",

        cucumber: "Gemini_Generated_Image_gliddfgliddfglid.jpg",
        cucumberShadow: "Gemini_Generated_Image_jvlffzjvlffzjvlf.jpg",
        carrotShadow: "Gemini_Generated_Image_39fj7t39fj7t39fj.jpg",
        tomatoShadow: "Gemini_Generated_Image_o2g30po2g30po2g3.jpg",

        cryingChild: "Gemini_Generated_Image_gcge5rgcge5rgcge.jpg",
        lonelyChild: "Gemini_Generated_Image_ysipqdysipqdysip.jpg",
        playingChild: "Gemini_Generated_Image_9o1mq39o1mq39o1m.jpg",

        circle: "Gemini_Generated_Image_uyi3apuyi3apuyi3.jpg",
        triangle: "Gemini_Generated_Image_5gv4ru5gv4ru5gv4.jpg",
        square: "Gemini_Generated_Image_cfc3i6cfc3i6cfc3.jpg"
    };

    // ربط ملفات الصوت بحيث يعبر q1 إلى q5 عن الأسئلة الخمسة بالترتيب
    const AUDIO_FILES = {
        q1: "q1.mp3",
        q2: "q2.mp3",
        q3: "q3.mp3",
        q4: "q4.mp3",
        q5: "q5.mp3",
        q6: "q6.mp3" // المرحلة الثانية للسؤال الخامس إذا وجدت
    };

    let questions = [
        {
            title: "السؤال الأول: اختر الظل المناسب لصورة الأسد",
            topic: "مطابقة ظل الأسد والحيوانات",
            difficulty: "سهل جدًا",
            hint: "تأمل شكل الأذن والجسد جيداً يا بطل.",
            targetImg: IMGS.lion,
            audioKey: "q1",
            options: [
                { text: "ظل الأسد", img: IMGS.lionShadow, correct: true },
                { text: "ظل الزرافة", img: IMGS.giraffeShadow, correct: false },
                { text: "ظل الفيل", img: IMGS.elephantShadow, correct: false }
            ]
        },
        {
            title: "السؤال الثاني: اختر الظل المناسب لصورة العنب",
            topic: "مطابقة ظل العنب والفواكه",
            difficulty: "متوسط",
            hint: "عناقيد العنب دائرية ومتراصة معاً.",
            targetImg: IMGS.grapes,
            audioKey: "q2",
            options: [
                { text: "ظل الموز", img: IMGS.bananaShadow, correct: false },
                { text: "ظل البرقوق", img: IMGS.plumShadow, correct: false },
                { text: "ظل العنب", img: IMGS.grapesShadow, correct: true }
            ]
        },
        {
            title: "السؤال الثالث: اختر الظل المناسب لصورة الخيار",
            topic: "مطابقة ظل الخيار والخضروات",
            difficulty: "متوسط",
            hint: "الخيار طويل وله شكل مميز.",
            targetImg: IMGS.cucumber,
            audioKey: "q3",
            options: [
                { text: "ظل الجزر", img: IMGS.carrotShadow, correct: false },
                { text: "ظل الخيار", img: IMGS.cucumberShadow, correct: true },
                { text: "ظل الطماطم", img: IMGS.tomatoShadow, correct: false }
            ]
        },
        {
            title: "السؤال الرابع: طفل رفض أصدقاؤه أن يلعبوا معه، ماذا يفعل؟",
            topic: "الموقف الاجتماعي وتجاوز الرفض",
            difficulty: "متوسط",
            hint: "الصديق الإيجابي يبحث عن أصدقاء آخرين يلعب معهم بسعادة.",
            targetImg: IMGS.cryingChild,
            audioKey: "q4",
            options: [
                { text: "البقاء وحيداً حزيناً", img: IMGS.lonelyChild, correct: false },
                { text: "اللعب مع أصدقاء آخرين بسعادة", img: IMGS.playingChild, correct: true }
            ]
        },
        {
            title: "السؤال الخامس: تغيير القواعد المستمر! اختر (شكل الدائرة) أولاً يا بطل.",
            topic: "تغيير القواعد والمرونة المعرفية",
            difficulty: "متقدم",
            hint: "ركز جيداً، القاعدة الحالية هي اختيار الدائرة.",
            targetText: "القاعدة 1: اختر (الدائرة)",
            audioKey: "q5",
            phase: 1,
            options: [
                { text: "شكل المثلث", img: IMGS.triangle, correct: false },
                { text: "شكل الدائرة", img: IMGS.circle, correct: true },
                { text: "شكل المربع", img: IMGS.square, correct: false }
            ]
        }
    ];

    let currentStep = 0;
    let childName = "";
    let startTime = 0;
    let questionStartTime = 0;
    let correctCount = 0;
    let errorCount = 0;
    let questionMetrics = {};
    let soundEnabled = true;
    let currentAudio = null;

    function toggleSound() {
        soundEnabled = !soundEnabled;
        const btn = document.getElementById('sound-toggle-btn');
        btn.innerText = soundEnabled ? "🔊 الصوت: مفعل" : "🔇 الصوت: مكتوم";
        if (!soundEnabled && currentAudio) {
            currentAudio.pause();
        }
    }

    function playAudio(audioKey) {
        if (!soundEnabled) return;
        
        if (currentAudio) {
            currentAudio.pause();
            currentAudio.currentTime = 0;
        }

        const audioFile = AUDIO_FILES[audioKey];
        if (audioFile) {
            currentAudio = new Audio(audioFile);
            const avatars = document.querySelectorAll('#mezo-avatar, #welcome-avatar');
            avatars.forEach(av => av.classList.add('speaking-glow'));

            currentAudio.play().catch(e => console.log("Audio play blocked or missing:", e));
            
            currentAudio.onended = () => {
                avatars.forEach(av => av.classList.remove('speaking-glow'));
            };
            currentAudio.onerror = () => {
                avatars.forEach(av => av.classList.remove('speaking-glow'));
            };
        }
    }

    function repeatMezoSpeech() {
        const q = questions[currentStep];
        if (q && q.audioKey) {
            playAudio(q.audioKey);
        }
    }

    function greetWelcome() {
        playAudio("q1");
    }

    function openAiChatModal() {
        document.getElementById('ai-chat-modal').classList.remove('hidden');
    }

    function closeAiChatModal() {
        document.getElementById('ai-chat-modal').classList.add('hidden');
    }

    function startApp() {
        const input = document.getElementById('child-name');
        childName = input.value.trim();
        if (!childName) {
            alert("الرجاء كتابة اسم البطل أولاً لنبدأ اللعبة!");
            return;
        }

        startTime = new Date();
        currentStep = 0;
        correctCount = 0;
        errorCount = 0;
        questionMetrics = {};

        document.getElementById('welcome-screen').classList.remove('active');
        document.getElementById('game-screen').classList.add('active');
        document.getElementById('player-display').innerText = `البطل: ${childName}`;

        loadQuestion();
    }

    function loadQuestion() {
        if (currentStep >= questions.length) {
            showReport();
            return;
        }

        const q = questions[currentStep];
        questionStartTime = new Date();

        if (!questionMetrics[currentStep]) {
            questionMetrics[currentStep] = { 
                attempts: 0, 
                failedAttempts: 0, 
                title: q.title, 
                topic: q.topic || `السؤال رقم ${currentStep + 1}`,
                difficulty: q.difficulty || 'متوسط',
                latency: 0,
                failed: false 
            };
        }

        document.getElementById('question-counter').innerText = `السؤال ${currentStep + 1} من ${questions.length}`;
        document.getElementById('question-title').innerText = q.title;
        document.getElementById('feedback-banner').innerText = "";
        document.getElementById('mezo-speech').innerText = q.title;

        const targetContainer = document.getElementById('target-container');
        targetContainer.innerHTML = "";
        if (q.targetImg) {
            targetContainer.style.display = "flex";
            const img = document.createElement('img');
            img.src = q.targetImg;
            img.className = "max-w-full max-h-full object-contain p-1";
            img.onerror = () => { img.src = 'https://placehold.co/150x150/f1f5f9/4f46e5?text=صورة'; };
            targetContainer.appendChild(img);
        } else if (q.targetText) {
            targetContainer.style.display = "flex";
            targetContainer.innerHTML = `<p class="p-3 font-bold text-sm text-indigo-700 text-center" id="target-text-display">${q.targetText}</p>`;
        } else {
            targetContainer.style.display = "none";
        }

        renderOptions(q);
        
        // تشغيل الملف الصوتي تلقائياً عند تحميل السؤال
        if (q.audioKey) {
            playAudio(q.audioKey);
        }
    }

    function renderOptions(q) {
        const optionsGrid = document.getElementById('options-grid');
        optionsGrid.innerHTML = "";

        q.options.forEach((opt, index) => {
            const card = document.createElement('div');
            card.className = "bg-white border-2 border-slate-200 rounded-2xl p-4 cursor-pointer hover:border-indigo-600 hover:-translate-y-1 transition flex flex-col items-center justify-center min-h-[220px] shadow-sm";
            card.onclick = () => handleAnswer(index);

            let contentHTML = "";
            if (opt.img) {
                contentHTML += `<div class="w-32 h-32 mb-3 flex items-center justify-center bg-slate-50 rounded-xl overflow-hidden p-1 border border-slate-100">`;
                contentHTML += `<img src="${opt.img}" class="max-w-full max-h-full object-contain" onerror="this.src='https://placehold.co/120x120/f1f5f9/4f46e5?text=خيار'">`;
                contentHTML += `</div>`;
            }
            contentHTML += `<span class="text-xs sm:text-sm font-bold text-slate-800 text-center">${opt.text}</span>`;
            card.innerHTML = contentHTML;
            optionsGrid.appendChild(card);
        });
    }

    function handleAnswer(selectedIndex) {
        const q = questions[currentStep];
        const now = new Date();
        const latencySec = ((now - questionStartTime) / 1000).toFixed(3);
        questionMetrics[currentStep].latency = latencySec;
        questionMetrics[currentStep].attempts++;

        const banner = document.getElementById('feedback-banner');

        if (q.options[selectedIndex].correct) {
            correctCount++;
            
            if (currentStep === 4 && q.phase === 1) {
                q.phase = 2;
                q.title = "ممتاز! تغيرت القاعدة الآن: اختر (شكل المربع)";
                q.topic = "تغيير القاعدة (المرحلة الثانية - المربع)";
                q.targetText = "القاعدة 2: اختر (المربع)";
                q.audioKey = "q6";
                q.options = [
                    { text: "شكل المثلث", img: IMGS.triangle, correct: false },
                    { text: "شكل الدائرة", img: IMGS.circle, correct: false },
                    { text: "شكل المربع", img: IMGS.square, correct: true }
                ];

                banner.className = "font-bold text-lg text-emerald-600 min-h-[30px]";
                banner.innerText = "أحسنت! تغيرت القاعدة، اختر المربع الآن! 🎯";
                document.getElementById('mezo-speech').innerText = q.title;
                document.getElementById('question-title').innerText = q.title;
                document.getElementById('target-text-display').innerText = q.targetText;
                renderOptions(q);
                playAudio("q6");
                return;
            }

            banner.className = "font-bold text-lg text-emerald-600 min-h-[30px]";
            banner.innerText = `أبدعت يا ${childName}! إجابة صحيحة ورائعة جداً!`;
            document.getElementById('mezo-speech').innerText = banner.innerText;
            
            setTimeout(() => {
                currentStep++;
                loadQuestion();
            }, 1000);
        } else {
            errorCount++;
            questionMetrics[currentStep].failed = true;
            questionMetrics[currentStep].failedAttempts++;
            banner.className = "font-bold text-lg text-rose-500 min-h-[30px]";
            banner.innerText = `إجابة خاطئة، حاول مرة أخرى! 💪 (تلميح: ${q.hint})`;
            document.getElementById('mezo-speech').innerText = banner.innerText;
        }
    }

    function downloadReportAsImage() {
        const area = document.getElementById('printable-report-area');
        html2canvas(area, { scale: 2 }).then(canvas => {
            const link = document.createElement('a');
            link.download = `تقرير_مغامرة_${childName}.png`;
            link.href = canvas.toDataURL('image/png');
            link.click();
        });
    }

    function downloadReportAsExcel() {
        let csvContent = "data:text/csv;charset=utf-8,\uFEFF";
        csvContent += "الموقف البيئي (السؤال),مؤشر المعالجة البصرية,أخطاء التردد,الحالة المعرفية\n";

        for (let step in questionMetrics) {
            const m = questionMetrics[step];
            const visualEval = m.failedAttempts > 0 ? "متوسطة الحذر" : "سريعة وتلقائية";
            const cognitiveStatus = m.failedAttempts > 0 ? "تطلب تلميح توجيهي" : "مرونة متكاملة ✓";
            const row = [`"${m.topic}"`, `"${visualEval}(${m.latency}ث)"`, `"${m.failedAttempts}"`, `"${cognitiveStatus}"`];
            csvContent += row.join(",") + "\n";
        }

        const encodedUri = encodeURI(csvContent);
        const link = document.createElement("a");
        link.setAttribute("href", encodedUri);
        link.setAttribute("download", `تقرير_مغامرة_${childName}.csv`);
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
    }

    function showReport() {
        document.getElementById('game-screen').classList.remove('active');
        document.getElementById('report-screen').classList.add('active');

        document.getElementById('rep-main-title').innerText = `تقرير مغامرة البطل (${childName})`;
        
        const successRateVal = Math.round((correctCount / (correctCount + errorCount)) * 100) || 63;
        const totalScore = Math.max(50, 100 - (errorCount * 8));
        
        let totalLat = 0;
        let countLat = 0;
        for (let s in questionMetrics) {
            if (questionMetrics[s].latency) {
                totalLat += parseFloat(questionMetrics[s].latency);
                countLat++;
            }
        }
        const avgLat = countLat > 0 ? (totalLat / countLat).toFixed(1) : "5.6";

        document.getElementById('rep-score').innerText = totalScore;
        document.getElementById('rep-success-rate').innerText = `${successRateVal}%`;
        document.getElementById('rep-avg-latency').innerText = `${avgLat} ثانية`;
        document.getElementById('rep-flexibility').innerText = errorCount <= 1 ? "ممتازة ومتطورة" : "جيدة ومتطورة";

        document.getElementById('eval-part-1').innerText = `حقق البطل ${childName} نسبة نجاح ممتازة بلغت ${successRateVal}% بعد اجتياز المحاولات بنجاح، مع إظهار تفاعل إيجابي وسريع مع تلميحات المساعد ميزو عند مواجهة خيارات متعددة.`;

        const tableBody = document.getElementById('report-table-body');
        tableBody.innerHTML = "";

        for (let step in questionMetrics) {
            const m = questionMetrics[step];
            const visualEval = m.failedAttempts > 0 ? "متوسطة الحذر" : "سريعة وتلقائية";
            const cognitiveStatus = m.failedAttempts > 0 ? "تطلب تلميح توجيهي" : "مرونة متكاملة ✓";
            const statusClass = m.failedAttempts > 0 ? "text-amber-600 font-semibold" : "text-emerald-600 font-bold";

            const tr = document.createElement('tr');
            tr.className = "border-b border-slate-100 hover:bg-slate-50 transition";
            tr.innerHTML = `
                <td class="p-3 font-bold text-slate-700">${m.topic}</td>
                <td class="p-3 text-slate-600">${visualEval} (${m.latency || 5.0}ث)</td>
                <td class="p-3 text-rose-600 font-semibold text-center">${m.failedAttempts}</td>
                <td class="p-3 ${statusClass}">${cognitiveStatus}</td>
            `;
            tableBody.appendChild(tr);
        }
    }
</script>

</body>
</html>