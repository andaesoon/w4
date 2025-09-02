# w4
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>나만의 걸음 기록 웹앱</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
            color: #1f2937;
        }
        .container {
            max-width: 600px;
        }
        .progress-circle {
            position: relative;
            width: 150px;
            height: 150px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 1.5rem;
            font-weight: bold;
            color: #1f2937;
            background: conic-gradient(#10b981 0%, #d1d5db 0%);
            transition: background 0.5s ease-in-out;
        }
        .progress-circle::before {
            content: '';
            position: absolute;
            width: 80%;
            height: 80%;
            border-radius: 50%;
            background-color: white;
        }
        .progress-circle-inner-text {
            position: relative;
            z-index: 10;
        }
    </style>
</head>
<body class="flex items-center justify-center min-h-screen p-4">

    <!-- 모달 창 -->
    <div id="modal" class="fixed inset-0 z-50 hidden items-center justify-center bg-gray-900 bg-opacity-50">
        <div class="bg-white p-6 rounded-2xl shadow-xl w-80 text-center">
            <p id="modal-text" class="text-gray-800 text-lg mb-4"></p>
            <button onclick="closeModal()" class="bg-green-500 text-white px-4 py-2 rounded-xl hover:bg-green-600 transition-colors">확인</button>
        </div>
    </div>

    <!-- 로딩 스피너 -->
    <div id="loading" class="fixed inset-0 z-50 hidden items-center justify-center bg-gray-900 bg-opacity-50">
        <div class="spinner-border animate-spin inline-block w-8 h-8 rounded-full border-4 border-t-transparent border-white"></div>
    </div>
    
    <div class="container bg-white rounded-3xl shadow-lg p-6 w-full">
        <h1 class="text-3xl font-bold text-center mb-6">나만의 걸음 기록</h1>
        <div id="auth-status" class="text-center text-sm text-gray-500 mb-4">
            <!-- 사용자 ID가 더 이상 표시되지 않습니다 -->
        </div>

        <!-- 목표 설정 및 달성률 -->
        <div class="bg-gray-100 p-6 rounded-2xl mb-6">
            <h2 class="text-xl font-bold mb-4">목표 설정 및 현황</h2>
            <div class="flex flex-col sm:flex-row items-center justify-between mb-4">
                <div class="flex items-center mb-2 sm:mb-0 w-full sm:w-1/2">
                    <label for="goal-input" class="text-gray-600 font-semibold mr-2 whitespace-nowrap">목표 걸음 수:</label>
                    <input type="number" id="goal-input" class="w-full p-2 border border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-green-500" placeholder="10000">
                </div>
                <button id="set-goal-btn" class="bg-blue-500 text-white px-4 py-2 rounded-xl hover:bg-blue-600 transition-colors mt-2 sm:mt-0 w-full sm:w-auto">목표 설정</button>
            </div>
            
            <div class="flex items-center justify-center relative my-6">
                <div id="goal-progress-circle" class="progress-circle" style="background: conic-gradient(#10b981 0%, #d1d5db 0%);">
                    <div class="progress-circle-inner-text">
                        <span id="progress-text">0%</span>
                        <div class="text-sm font-normal text-gray-500 text-center">달성률</div>
                    </div>
                </div>
            </div>
            
            <p class="text-center text-gray-600">오늘 걸음 수: <span id="daily-steps" class="font-bold text-lg">0</span> / 목표: <span id="goal-display" class="font-bold text-lg">0</span></p>
            <p id="achievement-status" class="text-center mt-2 text-lg font-bold"></p>
        </div>

        <!-- 걸음 수 기록 -->
        <div class="bg-gray-100 p-6 rounded-2xl mb-6">
            <h2 class="text-xl font-bold mb-4">걸음 수 기록</h2>
            <div class="flex flex-col sm:flex-row items-center space-y-2 sm:space-y-0 sm:space-x-2">
                <input type="number" id="steps-input" class="w-full p-2 border border-gray-300 rounded-xl focus:outline-none focus:ring-2 focus:ring-green-500" placeholder="오늘 걸음 수를 입력하세요">
                <button id="record-steps-btn" class="bg-green-500 text-white px-4 py-2 rounded-xl hover:bg-green-600 transition-colors w-full sm:w-auto">기록</button>
            </div>
        </div>

        <!-- 누적 걸음 그래프 -->
        <div class="bg-gray-100 p-6 rounded-2xl mb-6">
            <h2 class="text-xl font-bold mb-4">누적 걸음 그래프</h2>
            <div class="flex justify-center space-x-2 mb-4">
                <button id="chart-daily" class="px-4 py-2 rounded-xl bg-gray-300 hover:bg-gray-400 transition-colors">일간</button>
                <button id="chart-weekly" class="px-4 py-2 rounded-xl bg-gray-300 hover:bg-gray-400 transition-colors">주간</button>
                <button id="chart-monthly" class="px-4 py-2 rounded-xl bg-gray-300 hover:bg-gray-400 transition-colors">월간</button>
            </div>
            <div class="relative w-full h-64">
                <canvas id="step-chart"></canvas>
            </div>
        </div>

        <!-- 사진 분석 -->
        <div class="bg-gray-100 p-6 rounded-2xl">
            <h2 class="text-xl font-bold mb-4">사진 분석</h2>
            <p class="text-gray-600 mb-2 text-sm">걸음 수나 풍경이 담긴 사진을 업로드하고 분석을 요청하세요.</p>
            <input type="file" id="image-upload" accept="image/*" class="mb-4">
            <button id="analyze-photo-btn" class="bg-purple-500 text-white px-4 py-2 rounded-xl hover:bg-purple-600 transition-colors w-full">사진 분석 요청</button>
            <div id="analysis-result" class="mt-4 p-4 bg-white rounded-xl shadow hidden">
                <p id="result-text" class="text-gray-800"></p>
            </div>
        </div>
    </div>

    <script type="module">
        let dailySteps = 0;
        let goal = 0;
        let stepChart;
        let walkingData = [];

        // UI 요소
        const stepsInput = document.getElementById('steps-input');
        const recordStepsBtn = document.getElementById('record-steps-btn');
        const dailyStepsEl = document.getElementById('daily-steps');
        const goalInput = document.getElementById('goal-input');
        const setGoalBtn = document.getElementById('set-goal-btn');
        const goalDisplayEl = document.getElementById('goal-display');
        const progressCircle = document.getElementById('goal-progress-circle');
        const progressText = document.getElementById('progress-text');
        const modal = document.getElementById('modal');
        const modalText = document.getElementById('modal-text');
        const loadingSpinner = document.getElementById('loading');
        const achievementStatusEl = document.getElementById('achievement-status');

        // 메시지 모달을 표시하는 함수 (alert() 대체)
        function showModal(message) {
            modalText.textContent = message;
            modal.style.display = 'flex';
        }

        // 메시지 모달을 닫는 함수
        window.closeModal = function() {
            modal.style.display = 'none';
        }

        // Chart.js 그래프 생성 함수
        function createChart(labels, data) {
            const ctx = document.getElementById('step-chart').getContext('2d');
            if (stepChart) {
                stepChart.destroy();
            }
            stepChart = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: labels,
                    datasets: [{
                        label: '걸음 수',
                        data: data,
                        backgroundColor: 'rgba(16, 185, 129, 0.8)',
                        borderColor: 'rgba(16, 185, 129, 1)',
                        borderWidth: 1
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: {
                            beginAtZero: true
                        }
                    }
                }
            });
        }

        // 데이터 가져와서 그래프 업데이트 (로컬 데이터 사용)
        async function updateChart(timePeriod) {
            let labels = [];
            let data = [];
            let groupedData = {};

            if (timePeriod === 'daily') {
                labels = walkingData.map(d => d.date.toLocaleTimeString('ko-KR', { hour: '2-digit', minute: '2-digit' }));
                data = walkingData.map(d => d.steps);
            } else if (timePeriod === 'weekly') {
                const oneWeekAgo = new Date();
                oneWeekAgo.setDate(oneWeekAgo.getDate() - 7);
                const filteredData = walkingData.filter(d => d.date >= oneWeekAgo);
                
                filteredData.forEach(item => {
                    const day = item.date.toISOString().slice(0, 10);
                    if (!groupedData[day]) {
                        groupedData[day] = 0;
                    }
                    groupedData[day] += item.steps;
                });
                labels = Object.keys(groupedData).sort();
                data = labels.map(day => groupedData[day]);
            } else if (timePeriod === 'monthly') {
                const oneMonthAgo = new Date();
                oneMonthAgo.setMonth(oneMonthAgo.getMonth() - 1);
                const filteredData = walkingData.filter(d => d.date >= oneMonthAgo);

                filteredData.forEach(item => {
                    const week = `Week ${Math.ceil(item.date.getDate() / 7)}`;
                    if (!groupedData[week]) {
                        groupedData[week] = 0;
                    }
                    groupedData[week] += item.steps;
                });
                labels = Object.keys(groupedData).sort();
                data = labels.map(week => groupedData[week]);
            }
            
            createChart(labels, data);
        }

        document.getElementById('chart-daily').addEventListener('click', () => updateChart('daily'));
        document.getElementById('chart-weekly').addEventListener('click', () => updateChart('weekly'));
        document.getElementById('chart-monthly').addEventListener('click', () => updateChart('monthly'));

        // UI 업데이트
        function updateUI() {
            dailyStepsEl.textContent = dailySteps;
            goalDisplayEl.textContent = goal;

            let percentage = 0;
            if (goal > 0) {
                percentage = Math.min((dailySteps / goal) * 100, 100);
            }
            progressText.textContent = `${Math.round(percentage)}%`;

            const color = percentage >= 100 ? '#10b981' : '#f59e0b';
            progressCircle.style.background = `conic-gradient(${color} ${percentage}%, #d1d5db ${percentage}%)`;

            if (dailySteps >= goal && goal > 0) {
                achievementStatusEl.textContent = '목표 달성 성공! 🎉';
                achievementStatusEl.className = 'text-center mt-2 text-lg font-bold text-green-600';
            } else {
                achievementStatusEl.textContent = '';
            }
        }
        
        // 데이터 저장 (localStorage)
        function saveData() {
            const dataToSave = {
                dailySteps,
                goal,
                walkingData: walkingData.map(item => ({ steps: item.steps, date: item.date.toISOString() }))
            };
            localStorage.setItem('walkingApp', JSON.stringify(dataToSave));
        }

        // 데이터 불러오기 (localStorage)
        function loadData() {
            const savedData = localStorage.getItem('walkingApp');
            if (savedData) {
                const parsedData = JSON.parse(savedData);
                dailySteps = parsedData.dailySteps || 0;
                goal = parsedData.goal || 0;
                walkingData = parsedData.walkingData.map(item => ({ steps: item.steps, date: new Date(item.date) }));
            }
        }

        // 초기 로딩 함수
        window.onload = async function() {
            loadData();
            updateChart('weekly');
            updateUI();
        };
        
        // 걸음 기록 버튼 클릭 이벤트 (로컬 데이터 사용)
        recordStepsBtn.addEventListener('click', () => {
            const steps = parseInt(stepsInput.value);
            if (isNaN(steps) || steps <= 0) {
                showModal('유효한 걸음 수를 입력하세요.');
                return;
            }

            dailySteps += steps;
            walkingData.push({ date: new Date(), steps: steps });
            
            updateUI();
            updateChart('weekly');
            saveData();
            showModal('걸음 수가 성공적으로 기록되었습니다!');
            stepsInput.value = '';
        });

        // 목표 설정 버튼 클릭 이벤트 (로컬 데이터 사용)
        setGoalBtn.addEventListener('click', () => {
            const newGoal = parseInt(goalInput.value);
            if (isNaN(newGoal) || newGoal <= 0) {
                showModal('유효한 목표 걸음 수를 입력하세요.');
                return;
            }
            
            goal = newGoal;
            updateUI();
            saveData();
            showModal('목표 걸음 수가 설정되었습니다!');
        });

        // 사진 분석 기능
        document.getElementById('analyze-photo-btn').addEventListener('click', async () => {
            const fileInput = document.getElementById('image-upload');
            if (fileInput.files.length === 0) {
                showModal("먼저 사진 파일을 선택해주세요.");
                return;
            }

            loadingSpinner.style.display = 'flex';
            const file = fileInput.files[0];
            const reader = new FileReader();

            reader.onloadend = async () => {
                const base64Data = reader.result.split(',')[1];
                const analysisResultEl = document.getElementById('analysis-result');
                const resultTextEl = document.getElementById('result-text');

                const systemPrompt = "이미지 분석가 역할을 수행합니다. 이미지에서 걸음 수(예: 스마트워치, 만보기 화면)를 찾으세요. 걸음 수가 있다면 숫자만 추출하여 반환하세요. 다른 정보는 포함하지 마세요. 걸음 수가 없으면 '없음'이라고 반환하세요.";
                const userQuery = "이 이미지에 대해 분석해 주세요.";
                const apiKey = "AIzaSyCWpt1uxNSdV2B_QKV5B1G7Vd6osT3xCIc";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;

                const payload = {
                    contents: [
                        {
                            role: "user",
                            parts: [
                                { text: userQuery },
                                {
                                    inlineData: {
                                        mimeType: file.type,
                                        data: base64Data
                                    }
                                }
                            ]
                        }
                    ],
                    systemInstruction: {
                        parts: [{ text: systemPrompt }]
                    },
                };

                try {
                    const response = await fetch(apiUrl, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify(payload)
                    });
                    const result = await response.json();
                    
                    const text = result?.candidates?.[0]?.content?.parts?.[0]?.text || "분석 결과를 불러오는 데 실패했습니다.";
                    
                    const parsedSteps = parseInt(text);

                    if (!isNaN(parsedSteps)) {
                        stepsInput.value = parsedSteps;
                        showModal("사진에서 걸음 수를 파악했어요! 기록 버튼을 눌러주세요.");
                        resultTextEl.textContent = `사진에서 ${parsedSteps} 걸음을 찾았습니다.`;
                    } else {
                        showModal("사진에서 걸음 수를 찾을 수 없었습니다. 풍경을 분석해 드릴게요.");
                        const genericPrompt = "이미지 분석가 역할을 수행합니다. 이미지에 대해 자세히 설명해 주세요.";
                        const genericPayload = {
                            contents: [
                                {
                                    role: "user",
                                    parts: [
                                        { text: "이 이미지에 대해 설명해 주세요." },
                                        {
                                            inlineData: {
                                                mimeType: file.type,
                                                data: base64Data
                                            }
                                        }
                                    ]
                                }
                            ],
                            systemInstruction: {
                                parts: [{ text: genericPrompt }]
                            },
                        };

                        const genericResponse = await fetch(apiUrl, {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/json' },
                            body: JSON.stringify(genericPayload)
                        });
                        const genericResult = await genericResponse.json();
                        const genericText = genericResult?.candidates?.[0]?.content?.parts?.[0]?.text || "추가 분석 결과를 불러오는 데 실패했습니다.";
                        resultTextEl.textContent = genericText;
                    }

                    analysisResultEl.style.display = 'block';

                } catch (error) {
                    console.error("사진 분석 오류:", error);
                    resultTextEl.textContent = '사진 분석 중 오류가 발생했습니다.';
                    analysisResultEl.style.display = 'block';
                } finally {
                    loadingSpinner.style.display = 'none';
                }
            };
            reader.readAsDataURL(file);
        });
    </script>
</body>
</html>
