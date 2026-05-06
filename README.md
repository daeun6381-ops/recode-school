<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>내 주머니 속 생기부</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://cdn.jsdelivr.net/gh/orioncactus/pretendard/dist/web/static/pretendard.css');
        
        body {
            font-family: 'Pretendard', 'Malgun Gothic', '맑은 고딕', sans-serif;
            background-color: #d1d5db; 
        }

        .paper-container {
            background-color: white;
            margin: 2rem auto;
            padding: 3rem 2rem;
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
            min-height: 297mm; 
            max-width: 5xl;
        }

        .neis-table {
            width: 100%;
            border-collapse: collapse;
            margin-bottom: 2.5rem;
            border-top: 2px solid #000;
            border-bottom: 2px solid #000;
            font-size: 0.95rem;
        }
        .neis-table th, .neis-table td {
            border: 1px solid #9ca3af; 
            padding: 0.5rem;
            vertical-align: middle;
        }
        .neis-table th {
            background-color: #f3f4f6; 
            font-weight: bold;
            text-align: center;
            color: #111827;
        }
        
        .section-title {
            font-size: 1.1rem;
            font-weight: bold;
            color: #000;
            margin-bottom: 0.5rem;
            display: flex;
            justify-content: space-between;
            align-items: flex-end;
        }

        .smart-textarea-container {
            position: relative;
            width: 100%;
            height: 180px;
            background-color: white;
            overflow: hidden;
        }

        .smart-textarea-container:focus-within {
            outline: 2px solid rgba(59, 130, 246, 0.5);
            outline-offset: -2px;
        }

        .highlight-backdrop, .highlight-textarea {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            padding: 0.75rem;
            font-family: inherit;
            font-size: 0.95rem;
            line-height: 1.6;
            border: none;
            outline: none;
            white-space: pre-wrap;
            word-wrap: break-word;
            overflow-y: auto;
            resize: none;
            margin: 0;
        }

        .highlight-backdrop {
            z-index: 1;
            color: transparent;
            pointer-events: none;
        }

        .highlight-textarea {
            z-index: 2;
            background-color: transparent;
            color: #111827;
        }

        .highlight-backdrop mark {
            background-color: #bae6fd;
            color: transparent;
            border-radius: 2px;
        }

        .highlight-textarea::-webkit-scrollbar, .highlight-backdrop::-webkit-scrollbar {
            width: 8px;
        }
        .highlight-textarea::-webkit-scrollbar-thumb {
            background-color: #d1d5db;
            border-radius: 4px;
        }

        @media print {
            body { background-color: white; }
            .paper-container { margin: 0; padding: 0; box-shadow: none; max-width: 100%; }
            .no-print { display: none !important; }
            .neis-table { page-break-inside: avoid; }
        }
    </style>
</head>
<body class="text-gray-900 antialiased min-h-screen">

    <!-- 로딩 화면 -->
    <div id="loading-screen" class="min-h-screen flex flex-col items-center justify-center p-4 bg-gray-100 fixed inset-0 z-[200]">
        <div class="animate-spin rounded-full h-16 w-16 border-t-4 border-b-4 border-blue-800 mb-4"></div>
        <h2 class="text-xl font-bold text-gray-800">서버 통신 중...</h2>
        <p class="text-gray-500 text-sm mt-2">나만의 생기부 저장소를 준비하고 있습니다.</p>
    </div>

    <!-- 로그인/회원가입 화면 (개인 서버 코드를 넣으면 활성화됨) -->
    <div id="auth-screen" class="hidden min-h-screen flex items-center justify-center p-4 bg-gray-100 relative">
        <div class="bg-white p-10 rounded-lg shadow-xl max-w-sm w-full text-center border-t-4 border-gray-800">
            <h1 class="text-2xl font-extrabold mb-1">내 주머니 속 생기부</h1>
            <p class="text-gray-500 mb-8 text-sm">어디서든 내 계정으로 로그인하세요</p>
            
            <input type="email" id="email-input" placeholder="이메일 (아이디)" class="w-full mb-3 p-3 border border-gray-300 rounded focus:outline-none focus:border-blue-500 text-sm">
            <input type="password" id="pwd-input" placeholder="비밀번호 (6자 이상)" class="w-full mb-6 p-3 border border-gray-300 rounded focus:outline-none focus:border-blue-500 text-sm">
            
            <div class="flex gap-2 mb-6">
                <button onclick="handleLogin()" class="w-1/2 py-3 bg-gray-800 hover:bg-gray-700 text-white font-bold rounded transition shadow-sm">로그인</button>
                <button onclick="handleSignup()" class="w-1/2 py-3 bg-blue-600 hover:bg-blue-500 text-white font-bold rounded transition shadow-sm">가입하기</button>
            </div>

            <div class="border-t border-gray-200 mt-2 pt-4">
                <button onclick="handleGuestLogin()" class="text-xs text-gray-400 hover:text-gray-600 font-medium">로그인 없이 기기에 임시저장 (게스트)</button>
            </div>
        </div>
    </div>

    <!-- 학년 선택 화면 -->
    <div id="intro-screen" class="hidden min-h-screen flex items-center justify-center p-4 bg-gray-100 relative">
        <div class="absolute top-4 right-4 text-right">
            <!-- 개인 서버 연동 시 나타나는 유저 정보 -->
            <div id="user-info-display" class="hidden">
                <div class="text-xs font-bold text-gray-700 mb-1">👤 <span id="user-email-display"></span></div>
                <button onclick="handleLogout()" class="text-xs bg-gray-200 hover:bg-gray-300 text-gray-800 px-3 py-1 rounded font-bold transition">로그아웃</button>
            </div>
            <!-- 채팅창 기본 환경 뱃지 -->
            <div id="auto-cloud-badge" class="bg-blue-100 text-blue-800 text-xs font-bold px-3 py-1.5 rounded-full shadow-sm">
                ☁️ 내 계정 클라우드 연동 완료
            </div>
        </div>
        
        <div class="bg-white p-10 rounded-lg shadow-xl max-w-lg w-full text-center border-t-4 border-gray-800">
            <h1 class="text-3xl font-extrabold mb-2 tracking-tight">학교생활기록부</h1>
            <p class="text-gray-600 mb-8 font-medium">관리용 생기부 에디터 (자동 클라우드 저장)</p>
            
            <h2 class="text-lg font-bold mb-6 text-gray-800">작성할 학년을 선택하십시오.</h2>
            <div class="grid grid-cols-1 gap-3 sm:grid-cols-3">
                <button onclick="startApp(1)" class="py-3 px-6 bg-white border-2 border-gray-300 hover:border-gray-800 hover:bg-gray-50 font-bold rounded transition">1학년</button>
                <button onclick="startApp(2)" class="py-3 px-6 bg-white border-2 border-gray-300 hover:border-gray-800 hover:bg-gray-50 font-bold rounded transition">2학년</button>
                <button onclick="startApp(3)" class="py-3 px-6 bg-white border-2 border-gray-300 hover:border-gray-800 hover:bg-gray-50 font-bold rounded transition">3학년</button>
            </div>
        </div>
    </div>

    <!-- 메인 에디터 화면 -->
    <div id="editor-screen" class="hidden">
        
        <!-- 상단 고정 컨트롤 바 -->
        <div class="sticky top-0 z-50 bg-gray-800 text-white p-3 flex justify-between items-center no-print shadow-md">
            <div class="flex items-center">
                <span class="font-bold mr-4 whitespace-nowrap">내 주머니 속 생기부</span>
                <span id="grade-badge" class="bg-gray-600 text-xs px-2 py-1 rounded whitespace-nowrap"></span>
                <span class="ml-4 text-xs text-blue-300 hidden md:inline font-bold">☁️ 변경사항 실시간 저장 중...</span>
            </div>
            <div class="flex items-center space-x-2">
                <button onclick="resetApp()" class="px-3 py-1.5 bg-gray-700 hover:bg-gray-600 text-xs font-bold rounded transition whitespace-nowrap">학년 다시선택</button>
                <button id="export-btn" onclick="exportToDocument()" class="px-3 py-1.5 bg-green-600 hover:bg-green-500 text-xs font-bold rounded transition inline-flex items-center gap-1 whitespace-nowrap">
                    <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"></path></svg>
                    <span class="hidden sm:inline">문서 다운로드 (한글/워드)</span>
                    <span class="sm:hidden">문서 다운로드</span>
                </button>
                <button onclick="saveData()" class="px-3 py-1.5 bg-blue-600 hover:bg-blue-500 text-xs font-bold rounded transition whitespace-nowrap">클라우드 저장</button>
                <button id="logout-btn" onclick="handleLogout()" class="hidden px-3 py-1.5 bg-red-600 hover:bg-red-500 text-xs font-bold rounded transition whitespace-nowrap">로그아웃</button>
            </div>
        </div>

        <div class="paper-container relative mx-auto max-w-5xl" id="document-content">
            
            <h1 class="text-3xl font-extrabold text-center mb-10 tracking-widest" style="font-family: 'Batang', '바탕', serif;">학교생활기록부</h1>

            <!-- 1. 수상경력 -->
            <section>
                <div class="section-title">
                    <span>▣ 4. 수상경력</span>
                    <button onclick="addAward()" class="text-xs font-normal bg-white border border-gray-400 px-2 py-1 hover:bg-gray-50 no-print">+ 수상 추가</button>
                </div>
                <table class="neis-table">
                    <colgroup>
                        <col style="width: 35%">
                        <col style="width: 30%">
                        <col style="width: 20%">
                        <col style="width: 15%" class="no-print">
                    </colgroup>
                    <thead>
                        <tr>
                            <th>수상명</th>
                            <th>수여기관</th>
                            <th>연월일</th>
                            <th class="no-print">관리</th>
                        </tr>
                    </thead>
                    <tbody id="awards-list">
                        <!-- 동적 추가 -->
                    </tbody>
                </table>
            </section>

            <!-- 2. 창의적 체험활동상황 -->
            <section>
                <div class="section-title">
                    <span>▣ 7. 창의적 체험활동상황</span>
                </div>
                <table class="neis-table">
                    <colgroup>
                        <col style="width: 15%">
                        <col style="width: 85%">
                    </colgroup>
                    <thead>
                        <tr>
                            <th>영역</th>
                            <th>특기사항</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td class="font-bold text-center bg-gray-50">자율활동</td>
                            <td class="p-0 relative align-top">
                                <div class="flex justify-end p-1 bg-gray-50 border-b border-gray-200 no-print">
                                    <span class="text-xs font-medium text-gray-500" id="counter-act-1">0 / 500자</span>
                                </div>
                                <div id="container-act-1" class="smart-textarea-container" data-limit="500"></div>
                            </td>
                        </tr>
                        <tr>
                            <td class="font-bold text-center bg-gray-50">동아리활동</td>
                            <td class="p-0 relative align-top">
                                <div class="flex justify-end p-1 bg-gray-50 border-b border-gray-200 no-print">
                                    <span class="text-xs font-medium text-gray-500" id="counter-act-2">0 / 500자</span>
                                </div>
                                <div id="container-act-2" class="smart-textarea-container" data-limit="500"></div>
                            </td>
                        </tr>
                        <tr>
                            <td class="font-bold text-center bg-gray-50">진로활동</td>
                            <td class="p-0 relative align-top">
                                <div class="flex justify-end p-1 bg-gray-50 border-b border-gray-200 no-print">
                                    <span class="text-xs font-medium text-gray-500" id="counter-act-3">0 / 700자</span>
                                </div>
                                <div id="container-act-3" class="smart-textarea-container" data-limit="700"></div>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </section>

            <!-- 3. 교과학습발달상황 -->
            <section>
                <div class="section-title">
                    <span>▣ 8. 교과학습발달상황 <span class="text-sm font-normal text-gray-600 ml-2">(세부능력 및 특기사항)</span></span>
                    <button onclick="addSubject()" class="text-xs font-normal bg-white border border-gray-400 px-2 py-1 hover:bg-gray-50 no-print">+ 과목 추가</button>
                </div>
                <table class="neis-table">
                    <colgroup>
                        <col style="width: 15%">
                        <col style="width: 77%">
                        <col style="width: 8%" class="no-print">
                    </colgroup>
                    <thead>
                        <tr>
                            <th>과목</th>
                            <th>세부능력 및 특기사항</th>
                            <th class="no-print">관리</th>
                        </tr>
                    </thead>
                    <tbody id="subjects-list">
                        <!-- 동적 추가 -->
                    </tbody>
                </table>
            </section>

            <!-- 4. 행동특성 및 종합의견 -->
            <section>
                <div class="section-title">
                    <span>▣ 10. 행동특성 및 종합의견</span>
                </div>
                <table class="neis-table">
                    <tbody>
                        <tr>
                            <td class="p-0 relative align-top">
                                <div class="flex justify-end p-1 bg-gray-50 border-b border-gray-200 no-print">
                                    <span class="text-xs font-medium text-gray-500" id="counter-behavior">0 / 500자</span>
                                </div>
                                <div id="container-behavior" class="smart-textarea-container" data-limit="500" style="height: 250px;"></div>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </section>

        </div>
    </div>

    <!-- 토스트 알림 -->
    <div id="toast" class="fixed bottom-4 left-1/2 transform -translate-x-1/2 bg-gray-800 text-white px-4 py-2 rounded shadow-lg opacity-0 pointer-events-none transition-opacity duration-300 z-50">
        저장되었습니다.
    </div>

    <!-- 학년 변경 모달 -->
    <div id="confirm-modal" class="fixed inset-0 bg-black bg-opacity-50 z-[100] hidden items-center justify-center">
        <div class="bg-white p-6 rounded-lg shadow-xl max-w-sm w-full text-center">
            <h3 class="text-lg font-bold mb-4">학년 변경</h3>
            <p class="text-gray-600 mb-6 text-sm">학년을 변경하시겠습니까?<br>현재 화면의 내용은 자동으로 클라우드에 저장됩니다.</p>
            <div class="flex justify-center space-x-3">
                <button onclick="closeConfirmModal()" class="px-4 py-2 bg-gray-200 hover:bg-gray-300 rounded font-bold text-sm transition text-gray-800">취소</button>
                <button onclick="executeResetApp()" class="px-4 py-2 bg-gray-800 hover:bg-gray-700 text-white rounded font-bold text-sm transition">확인</button>
            </div>
        </div>
    </div>

    <!-- 과목 삭제 모달 -->
    <div id="delete-modal" class="fixed inset-0 bg-black bg-opacity-50 z-[100] hidden items-center justify-center">
        <div class="bg-white p-6 rounded-lg shadow-xl max-w-sm w-full text-center">
            <h3 class="text-lg font-bold mb-4">과목 삭제</h3>
            <p class="text-gray-600 mb-6 text-sm">이 과목을 삭제하시겠습니까?</p>
            <div class="flex justify-center space-x-3">
                <button onclick="closeDeleteModal()" class="px-4 py-2 bg-gray-200 hover:bg-gray-300 rounded font-bold text-sm transition text-gray-800">취소</button>
                <button id="confirm-delete-btn" class="px-4 py-2 bg-red-600 hover:bg-red-700 text-white rounded font-bold text-sm transition">삭제</button>
            </div>
        </div>
    </div>

    <!-- Firebase 및 비즈니스 로직 -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInWithEmailAndPassword, createUserWithEmailAndPassword, signInWithCustomToken, signInAnonymously, onAuthStateChanged, signOut } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, getDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // ============================================================================
        // 💡 [나만의 파이어베이스 설정 영역]
        // ============================================================================
        const MY_FIREBASE_CONFIG = {
            apiKey: "AIzaSyA30iujeXRiP1iJW1db_2WIttKJ3sO4SAI",
            authDomain: "space1-b521a.firebaseapp.com",
            projectId: "space1-b521a",
            storageBucket: "space1-b521a.firebasestorage.app",
            messagingSenderId: "235223237323",
            appId: "1:235223237323:web:a195263242c149e265514e",
            measurementId: "G-HJ34NT7FDV"
        };

        let currentGrade = null;
        let textareasMap = new Map();
        let itemToDelete = null;
        
        let app, auth, db, currentUser;
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        
        const defaultSubjects = {
            1: ['국어', '수학', '영어', '한국사', '통합사회', '통합과학', '과학탐구실험'],
            2: ['문학', '독서', '수학I', '수학II', '영어I', '영어II', '물리학I', '화학I', '생명과학I'],
            3: ['화법과 작문', '미적분', '확률과 통계', '심화국어', '영어독해와 작문', '사회·문화', '지구과학II']
        };

        // 1. Firebase 초기화 (자동 로그인 및 개인 서버 판별)
        async function initFirebase() {
            try {
                let configToUse = null;
                let isCanvasEnv = false;

                // 개인 설정 코드가 있으면 개인 서버 사용, 없으면 채팅창 기본 서버 사용
                if (MY_FIREBASE_CONFIG) {
                    configToUse = MY_FIREBASE_CONFIG;
                } else if (typeof __firebase_config !== 'undefined') {
                    configToUse = JSON.parse(__firebase_config);
                    isCanvasEnv = true;
                }

                if (configToUse) {
                    app = initializeApp(configToUse);
                    auth = getAuth(app);
                    db = getFirestore(app);

                    // 채팅창 기본 서버일 경우에만 AI가 알아서 자동 로그인 시켜줌
                    if (isCanvasEnv) {
                        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                            await signInWithCustomToken(auth, __initial_auth_token);
                        } else {
                            await signInAnonymously(auth);
                        }
                    }

                    // 로그인(인증) 상태 감지
                    onAuthStateChanged(auth, (user) => {
                        document.getElementById('loading-screen').classList.add('hidden');
                        
                        if (user) {
                            currentUser = user;
                            document.getElementById('auth-screen').classList.add('hidden');
                            
                            // 개인 서버 연동 시에는 우측 상단에 내 이메일과 로그아웃 버튼 표시
                            if (!isCanvasEnv) {
                                document.getElementById('auto-cloud-badge').classList.add('hidden');
                                document.getElementById('user-info-display').classList.remove('hidden');
                                document.getElementById('logout-btn').classList.remove('hidden');
                                document.getElementById('user-email-display').textContent = user.isAnonymous ? '게스트' : user.email;
                            }

                            if (document.getElementById('editor-screen').classList.contains('hidden')) {
                                document.getElementById('intro-screen').classList.remove('hidden');
                            }
                        } else {
                            // 로그아웃 상태 (개인 서버일 때 진입하면 로그인 화면 띄움)
                            currentUser = null;
                            document.getElementById('intro-screen').classList.add('hidden');
                            document.getElementById('editor-screen').classList.add('hidden');
                            document.getElementById('auth-screen').classList.remove('hidden');
                        }
                    });
                } else {
                    // 환경변수가 없는 로컬 테스트용 폴백
                    currentUser = { uid: 'local-tester' };
                    document.getElementById('loading-screen').classList.add('hidden');
                    document.getElementById('intro-screen').classList.remove('hidden');
                }
            } catch (error) {
                console.error("Firebase 초기화 에러:", error);
                document.getElementById('loading-screen').innerHTML = "<p class='text-red-600 font-bold'>서버 연결에 실패했습니다. 새로고침 해주세요.</p>";
            }
        }

        // ================= Auth 핸들링 (개인 서버 연동 시 작동) ================= 
        async function handleLogin() {
            const email = document.getElementById('email-input').value;
            const pwd = document.getElementById('pwd-input').value;
            if(!email || !pwd) return showToast("이메일과 비밀번호를 모두 입력해주세요.");
            try {
                document.getElementById('loading-screen').classList.remove('hidden');
                await signInWithEmailAndPassword(auth, email, pwd);
                showToast("로그인 성공! 환영합니다.");
            } catch(e) {
                document.getElementById('loading-screen').classList.add('hidden');
                showToast("로그인 실패: 아이디나 비밀번호가 틀렸습니다.");
            }
        }

        async function handleSignup() {
            const email = document.getElementById('email-input').value;
            const pwd = document.getElementById('pwd-input').value;
            if(!email || !pwd) return showToast("사용할 이메일과 비밀번호를 입력해주세요.");
            if(pwd.length < 6) return showToast("비밀번호는 최소 6자 이상이어야 합니다.");
            try {
                document.getElementById('loading-screen').classList.remove('hidden');
                await createUserWithEmailAndPassword(auth, email, pwd);
                showToast("회원가입 완료! 내 생기부 작성을 시작하세요.");
            } catch(e) {
                document.getElementById('loading-screen').classList.add('hidden');
                showToast("가입 실패: 이미 사용중이거나 형식이 잘못된 이메일입니다.");
            }
        }

        async function handleGuestLogin() {
            try {
                document.getElementById('loading-screen').classList.remove('hidden');
                await signInAnonymously(auth);
            } catch(e) {
                document.getElementById('loading-screen').classList.add('hidden');
                showToast("게스트 로그인에 실패했습니다.");
            }
        }

        async function handleLogout() {
            try {
                await signOut(auth);
                currentGrade = null;
                document.getElementById('email-input').value = '';
                document.getElementById('pwd-input').value = '';
                showToast("안전하게 로그아웃 되었습니다.");
            } catch(e) {
                showToast("로그아웃 중 오류가 발생했습니다.");
            }
        }

        // ================= 생기부 기능 로직 ================= 
        async function startApp(grade) {
            currentGrade = grade;
            document.getElementById('intro-screen').classList.add('hidden');
            document.getElementById('editor-screen').classList.remove('hidden');
            document.getElementById('grade-badge').textContent = `${grade}학년`;

            initSmartTextareas();
            await loadData(grade);
        }

        function resetApp() {
            document.getElementById('confirm-modal').classList.remove('hidden');
            document.getElementById('confirm-modal').classList.add('flex');
        }

        function closeConfirmModal() {
            document.getElementById('confirm-modal').classList.add('hidden');
            document.getElementById('confirm-modal').classList.remove('flex');
        }

        async function executeResetApp() {
            closeConfirmModal();
            await saveData();
            document.getElementById('editor-screen').classList.add('hidden');
            document.getElementById('intro-screen').classList.remove('hidden');
            
            document.getElementById('awards-list').innerHTML = '';
            document.getElementById('subjects-list').innerHTML = '';
            textareasMap.clear();
        }

        function escapeHtml(text) {
            return text.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/"/g, "&quot;").replace(/'/g, "&#039;");
        }

        function createSmartTextarea(containerId, limit, initialText = "", counterId = null) {
            const container = document.getElementById(containerId);
            if (!container) return;

            container.innerHTML = '';
            const backdrop = document.createElement('div');
            backdrop.className = 'highlight-backdrop';
            const textarea = document.createElement('textarea');
            textarea.className = 'highlight-textarea';
            textarea.placeholder = "내용을 입력하세요...";
            textarea.value = initialText;
            textarea.setAttribute('spellcheck', 'false');

            container.appendChild(backdrop);
            container.appendChild(textarea);

            const counterEl = counterId ? document.getElementById(counterId) : null;

            const updateHighlight = () => {
                let text = textarea.value;
                let renderText = text;
                if (renderText.endsWith('\n')) renderText += ' '; 

                if (text.length <= limit) {
                    backdrop.innerHTML = escapeHtml(renderText);
                } else {
                    const safeUnder = escapeHtml(renderText.substring(0, limit));
                    const safeOver = escapeHtml(renderText.substring(limit));
                    backdrop.innerHTML = safeUnder + '<mark>' + safeOver + '</mark>';
                }

                if (counterEl) {
                    counterEl.textContent = `${text.length} / ${limit}자`;
                    if (text.length > limit) {
                        counterEl.classList.add('text-red-600', 'font-bold');
                        counterEl.classList.remove('text-gray-500');
                    } else {
                        counterEl.classList.add('text-gray-500');
                        counterEl.classList.remove('text-red-600', 'font-bold');
                    }
                }
                textareasMap.set(containerId, text);
            };

            textarea.addEventListener('input', () => {
                updateHighlight();
                debouncedSave();
            });
            
            textarea.addEventListener('scroll', () => {
                backdrop.scrollTop = textarea.scrollTop;
                backdrop.scrollLeft = textarea.scrollLeft;
            });

            updateHighlight();
            return textarea;
        }

        function initSmartTextareas() {
            createSmartTextarea('container-act-1', 500, "", 'counter-act-1');
            createSmartTextarea('container-act-2', 500, "", 'counter-act-2');
            createSmartTextarea('container-act-3', 700, "", 'counter-act-3');
            createSmartTextarea('container-behavior', 500, "", 'counter-behavior');
        }

        function addAward(name = "", date = "", agency = "") {
            const list = document.getElementById('awards-list');
            const id = `award-${Date.now()}`;
            
            const tr = document.createElement('tr');
            tr.id = id;
            tr.innerHTML = `
                <td class="p-1"><input type="text" placeholder="수상명" value="${name}" class="award-name w-full bg-transparent border border-transparent hover:border-gray-300 focus:border-blue-500 p-1 text-sm outline-none transition-colors"></td>
                <td class="p-1"><input type="text" placeholder="수여기관" value="${agency}" class="award-agency w-full text-center bg-transparent border border-transparent hover:border-gray-300 focus:border-blue-500 p-1 text-sm outline-none transition-colors"></td>
                <td class="p-1"><input type="date" value="${date}" class="award-date w-full text-center bg-transparent border border-transparent hover:border-gray-300 focus:border-blue-500 p-1 text-sm outline-none transition-colors"></td>
                <td class="p-1 text-center no-print"><button onclick="document.getElementById('${id}').remove(); window.debouncedSave();" class="text-xs text-red-600 hover:underline">삭제</button></td>
            `;
            list.appendChild(tr);
            tr.querySelectorAll('input').forEach(input => input.addEventListener('input', debouncedSave));
        }

        function addSubject(subjectName = "", content = "") {
            const list = document.getElementById('subjects-list');
            const uniqueId = Date.now() + Math.floor(Math.random() * 1000);
            const containerId = `subject-container-${uniqueId}`;
            const counterId = `subject-counter-${uniqueId}`;
            const wrapId = `subject-wrap-${uniqueId}`;

            const tr = document.createElement('tr');
            tr.id = wrapId;
            tr.className = 'subject-item';
            tr.innerHTML = `
                <td class="bg-gray-50 text-center p-2 align-top">
                    <input type="text" value="${subjectName}" placeholder="과목명" class="subject-name w-full text-center font-bold bg-transparent border-b border-gray-300 focus:border-blue-500 p-1 outline-none mb-1">
                    <div class="text-xs text-gray-500 mt-2 no-print" id="${counterId}">0 / 500자</div>
                </td>
                <td class="p-0 relative align-top">
                    <div id="${containerId}" class="smart-textarea-container" data-limit="500"></div>
                </td>
                <td class="p-2 text-center align-middle no-print">
                    <button onclick="removeSubject('${wrapId}')" class="text-xs text-red-600 hover:underline">삭제</button>
                </td>
            `;
            list.appendChild(tr);
            createSmartTextarea(containerId, 500, content, counterId);
            tr.querySelector('.subject-name').addEventListener('input', debouncedSave);
        }

        function removeSubject(wrapId) {
            itemToDelete = wrapId;
            document.getElementById('delete-modal').classList.remove('hidden');
            document.getElementById('delete-modal').classList.add('flex');
            document.getElementById('confirm-delete-btn').onclick = executeRemoveSubject;
        }

        function closeDeleteModal() {
            document.getElementById('delete-modal').classList.add('hidden');
            document.getElementById('delete-modal').classList.remove('flex');
            itemToDelete = null;
        }

        function executeRemoveSubject() {
            if (itemToDelete) {
                document.getElementById(itemToDelete).remove();
                debouncedSave();
                closeDeleteModal();
            }
        }

        // Firebase 데이터 클라우드 저장
        async function saveData() {
            if (!currentGrade || !currentUser) return;

            const data = {
                awards: [],
                activities: {
                    act1: textareasMap.get('container-act-1') || "",
                    act2: textareasMap.get('container-act-2') || "",
                    act3: textareasMap.get('container-act-3') || ""
                },
                subjects: [],
                behavior: textareasMap.get('container-behavior') || ""
            };

            document.querySelectorAll('#awards-list > tr').forEach(tr => {
                data.awards.push({
                    name: tr.querySelector('.award-name').value,
                    agency: tr.querySelector('.award-agency').value,
                    date: tr.querySelector('.award-date').value
                });
            });

            document.querySelectorAll('.subject-item').forEach(tr => {
                const name = tr.querySelector('.subject-name').value;
                const containerId = tr.querySelector('.smart-textarea-container').id;
                const content = textareasMap.get(containerId) || "";
                data.subjects.push({ name, content });
            });

            try {
                if(db) {
                    const docRef = doc(db, 'artifacts', appId, 'users', currentUser.uid, 'sgb_grades', String(currentGrade));
                    await setDoc(docRef, data);
                    showToast("☁️ 클라우드에 안전하게 저장되었습니다.");
                }
            } catch (e) {
                console.error("클라우드 저장 에러:", e);
                localStorage.setItem(`pocket-sgb-${currentUser.uid}-grade-${currentGrade}`, JSON.stringify(data));
                showToast("기기에 임시 저장되었습니다.");
            }
        }

        let debounceTimer;
        function debouncedSave() {
            clearTimeout(debounceTimer);
            debounceTimer = setTimeout(saveData, 1500); 
        }

        // Firebase 데이터 클라우드 로드
        async function loadData(grade) {
            document.getElementById('awards-list').innerHTML = '';
            document.getElementById('subjects-list').innerHTML = '';
            
            let data = null;

            if (db && currentUser) {
                try {
                    const docRef = doc(db, 'artifacts', appId, 'users', currentUser.uid, 'sgb_grades', String(grade));
                    const docSnap = await getDoc(docRef);
                    if (docSnap.exists()) {
                        data = docSnap.data();
                    }
                } catch (e) {
                    console.error("클라우드 로드 에러:", e);
                    const localData = localStorage.getItem(`pocket-sgb-${currentUser.uid}-grade-${grade}`);
                    if(localData) data = JSON.parse(localData);
                }
            }
            
            if (data) {
                data.awards.forEach(award => addAward(award.name, award.date, award.agency));
                createSmartTextarea('container-act-1', 500, data.activities.act1, 'counter-act-1');
                createSmartTextarea('container-act-2', 500, data.activities.act2, 'counter-act-2');
                createSmartTextarea('container-act-3', 700, data.activities.act3, 'counter-act-3');
                
                if (data.subjects && data.subjects.length > 0) {
                    data.subjects.forEach(sub => addSubject(sub.name, sub.content));
                } else {
                    defaultSubjects[grade].forEach(subjectName => addSubject(subjectName, ""));
                }
                createSmartTextarea('container-behavior', 500, data.behavior, 'counter-behavior');
            } else {
                createSmartTextarea('container-act-1', 500, "", 'counter-act-1');
                createSmartTextarea('container-act-2', 500, "", 'counter-act-2');
                createSmartTextarea('container-act-3', 700, "", 'counter-act-3');
                createSmartTextarea('container-behavior', 500, "", 'counter-behavior');
                defaultSubjects[grade].forEach(subjectName => addSubject(subjectName, ""));
            }
        }

        function showToast(msg) {
            const toast = document.getElementById('toast');
            toast.textContent = msg;
            toast.classList.remove('opacity-0');
            setTimeout(() => toast.classList.add('opacity-0'), 2000);
        }

        function exportToDocument() {
            const contentClone = document.getElementById('document-content').cloneNode(true);
            contentClone.querySelectorAll('.no-print').forEach(el => el.remove());

            const originalInputs = document.getElementById('document-content').querySelectorAll('input');
            const clonedInputs = contentClone.querySelectorAll('input');
            originalInputs.forEach((input, index) => {
                const span = document.createElement('span');
                span.textContent = input.value;
                clonedInputs[index].parentNode.replaceChild(span, clonedInputs[index]);
            });

            contentClone.querySelectorAll('.smart-textarea-container').forEach(container => {
                const text = textareasMap.get(container.id) || "";
                const div = document.createElement('div');
                div.style.padding = '8px';
                div.style.lineHeight = '1.6';
                div.style.whiteSpace = 'pre-wrap';
                div.innerHTML = escapeHtml(text).replace(/\n/g, '<br>');
                container.parentNode.replaceChild(div, container);
            });

            contentClone.querySelectorAll('table').forEach(table => {
                table.style.borderCollapse = 'collapse';
                table.style.width = '100%';
                table.style.marginBottom = '20px';
                table.style.border = '1px solid black';
            });
            contentClone.querySelectorAll('th, td').forEach(cell => {
                cell.style.border = '1px solid black';
                cell.style.padding = '8px';
                cell.style.verticalAlign = 'middle';
            });
            contentClone.querySelectorAll('th').forEach(th => {
                th.style.backgroundColor = '#f3f4f6';
                th.style.fontWeight = 'bold';
                th.style.textAlign = 'center';
            });
            contentClone.querySelectorAll('td.bg-gray-50').forEach(td => {
                td.style.backgroundColor = '#f3f4f6';
                td.style.fontWeight = 'bold';
            });

            const htmlHeader = `
                <html xmlns:o="urn:schemas-microsoft-com:office:office" xmlns:w="urn:schemas-microsoft-com:office:word" xmlns="http://www.w3.org/TR/REC-html40">
                <head>
                    <meta charset="utf-8">
                    <title>학교생활기록부</title>
                    <style>
                        body { font-family: 'Malgun Gothic', '맑은 고딕', sans-serif; }
                        h1 { text-align: center; font-size: 24pt; margin-bottom: 20px; }
                        .section-title { font-weight: bold; font-size: 14pt; margin-bottom: 10px; margin-top: 20px; }
                    </style>
                </head>
                <body>
            `;
            const htmlFooter = `</body></html>`;
            const fullHtmlContent = htmlHeader + contentClone.innerHTML + htmlFooter;

            const blob = new Blob(['\ufeff', fullHtmlContent], { type: 'application/msword' });
            const url = URL.createObjectURL(blob);
            const downloadLink = document.createElement('a');
            downloadLink.href = url;
            downloadLink.download = `학교생활기록부_${currentGrade}학년.doc`;
            
            document.body.appendChild(downloadLink);
            downloadLink.click();
            document.body.removeChild(downloadLink);
            URL.revokeObjectURL(url);

            showToast("문서 파일 다운로드가 완료되었습니다.");
        }

        // HTML 태그에서 사용할 수 있도록 함수들을 window 객체에 연결
        window.handleLogin = handleLogin;
        window.handleSignup = handleSignup;
        window.handleGuestLogin = handleGuestLogin;
        window.handleLogout = handleLogout;

        window.startApp = startApp;
        window.resetApp = resetApp;
        window.executeResetApp = executeResetApp;
        window.closeConfirmModal = closeConfirmModal;
        window.addAward = addAward;
        window.addSubject = addSubject;
        window.removeSubject = removeSubject;
        window.closeDeleteModal = closeDeleteModal;
        window.saveData = saveData;
        window.debouncedSave = debouncedSave;
        window.exportToDocument = exportToDocument;

        // 웹사이트 로드 시 Firebase 연동 시작
        window.onload = initFirebase;
    </script>
</body>
</html>
