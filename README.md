<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Умный секундомер</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        .timer-circle {
            width: 300px;
            height: 300px;
            border-radius: 50%;
            position: relative;
            display: flex;
            justify-content: center;
            align-items: center;
            transition: all 0.3s ease;
        }
        
        @media (max-width: 640px) {
            .timer-circle {
                width: 250px;
                height: 250px;
            }
        }
        
        .timer-progress {
            position: absolute;
            width: 100%;
            height: 100%;
            border-radius: 50%;
            clip: rect(0, 300px, 300px, 150px);
        }
        
        .progress-bar {
            position: absolute;
            width: 100%;
            height: 100%;
            border-radius: 50%;
            clip: rect(0, 150px, 300px, 0);
            background: #007AFF;
            transform: rotate(0deg);
            transition: transform 0.3s ease;
        }
        
        .dark .progress-bar {
            background: #0A84FF;
        }
        
        .timer-inner {
            width: 90%;
            height: 90%;
            background-color: white;
            border-radius: 50%;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 10;
        }
        
        .dark .timer-inner {
            background-color: #1C1C1E;
        }
        
        .scenario-item.active {
            border-left: 4px solid #007AFF;
        }
        
        .dark .scenario-item.active {
            border-left: 4px solid #0A84FF;
        }
        
        .dragging {
            opacity: 0.5;
            background-color: rgba(0, 122, 255, 0.1);
        }
        
        .dark .dragging {
            background-color: rgba(10, 132, 255, 0.1);
        }
        
        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.05); }
            100% { transform: scale(1); }
        }
        
        .pulse-animation {
            animation: pulse 1.5s infinite;
        }
        
        .stage-time-bar {
            height: 8px;
            border-radius: 4px;
            background-color: #E5E7EB;
        }
        
        .dark .stage-time-bar {
            background-color: #374151;
        }
        
        .stage-time-progress {
            height: 100%;
            border-radius: 4px;
            background-color: #007AFF;
        }
        
        .dark .stage-time-progress {
            background-color: #0A84FF;
        }
        
        .chart-container {
            position: relative;
            height: 200px;
            width: 100%;
        }
        
        .mobile-view .timer-section {
            order: 2;
        }
        
        .mobile-view .scenarios-section {
            order: 1;
        }
        
        .mobile-view .flex-col {
            flex-direction: column;
        }
        
        .stage-selector {
            display: none;
            position: absolute;
            right: 0;
            top: 100%;
            z-index: 20;
            background-color: white;
            border-radius: 8px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            min-width: 200px;
            max-height: 300px;
            overflow-y: auto;
        }
        
        .dark .stage-selector {
            background-color: #1C1C1E;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.3);
        }
        
        .stage-selector.show {
            display: block;
        }
        
        .stage-option {
            padding: 8px 12px;
            cursor: pointer;
            transition: background-color 0.2s;
        }
        
        .stage-option:hover {
            background-color: #F3F4F6;
        }
        
        .dark .stage-option:hover {
            background-color: #2C2C2E;
        }
        
        .delete-btn {
            opacity: 0;
            transition: opacity 0.2s;
        }
        
        .scenario-item:hover .delete-btn {
            opacity: 1;
        }
        
        .tab-close-btn {
            opacity: 0;
            transition: opacity 0.2s;
        }
        
        .scenario-tab:hover .tab-close-btn {
            opacity: 1;
        }
    </style>
</head>
<body class="bg-gray-100 dark:bg-black transition-colors duration-300">
    <div class="container mx-auto px-4 py-8 max-w-4xl">
        <!-- Header -->
        <header class="flex justify-between items-center mb-8">
            <h1 class="text-3xl font-bold text-gray-900 dark:text-white">Умный секундомер</h1>
            <div class="flex items-center space-x-4">
                <button id="theme-toggle" class="p-2 rounded-full bg-gray-200 dark:bg-gray-800 text-gray-700 dark:text-gray-300">
                    <i class="fas fa-moon dark:hidden"></i>
                    <i class="fas fa-sun hidden dark:inline"></i>
                </button>
                <button id="view-toggle" class="p-2 rounded-full bg-gray-200 dark:bg-gray-800 text-gray-700 dark:text-gray-300">
                    <i class="fas fa-mobile"></i>
                </button>
            </div>
        </header>
        
        <!-- Main Content -->
        <div class="flex flex-col md:flex-row gap-8">
            <!-- Timer Section -->
            <div class="timer-section flex-1 flex flex-col items-center">
                <div class="timer-circle bg-gray-200 dark:bg-gray-800 mb-6">
                    <div class="timer-progress">
                        <div class="progress-bar"></div>
                    </div>
                    <div class="timer-inner">
                        <div id="timer-display" class="text-6xl font-mono font-bold text-gray-900 dark:text-white">00:00.00</div>
                        <div id="current-stage" class="text-lg text-gray-600 dark:text-gray-400 mt-2">Не активно</div>
                    </div>
                </div>
                
                <div class="flex space-x-4 mb-8">
                    <button id="start-btn" class="px-6 py-3 bg-green-500 text-white rounded-full font-medium hover:bg-green-600 transition">
                        <i class="fas fa-play mr-2"></i>Старт
                    </button>
                    <button id="select-stage-btn" class="px-6 py-3 bg-blue-500 text-white rounded-full font-medium hover:bg-blue-600 transition relative" disabled>
                        <i class="fas fa-flag mr-2"></i>Выбрать этап
                        <div id="stage-selector" class="stage-selector"></div>
                    </button>
                    <button id="reset-btn" class="px-6 py-3 bg-red-500 text-white rounded-full font-medium hover:bg-red-600 transition">
                        <i class="fas fa-stop mr-2"></i>Стоп
                    </button>
                </div>
                
                <div id="timer-laps" class="w-full max-w-md bg-white dark:bg-gray-800 rounded-lg shadow p-4">
                    <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-3">Результаты</h3>
                    <div class="space-y-2 max-h-60 overflow-y-auto">
                        <div class="text-center text-gray-500 dark:text-gray-400 py-4">Нет данных</div>
                    </div>
                </div>
            </div>
            
            <!-- Scenarios Section -->
            <div class="scenarios-section flex-1 bg-white dark:bg-gray-800 rounded-lg shadow p-6">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-xl font-bold text-gray-900 dark:text-white">Сценарии</h2>
                    <button id="new-scenario-btn" class="px-4 py-2 bg-blue-500 text-white rounded-full text-sm font-medium hover:bg-blue-600 transition">
                        <i class="fas fa-plus mr-1"></i>Новый
                    </button>
                </div>
                
                <div id="scenario-tabs" class="flex border-b border-gray-200 dark:border-gray-700 mb-4 overflow-x-auto">
                    <!-- Tabs will be added here dynamically -->
                </div>
                
                <div id="scenario-details" class="mb-6">
                    <div class="text-center text-gray-500 dark:text-gray-400 py-8">
                        <i class="fas fa-clock text-4xl mb-2"></i>
                        <p>Выберите или создайте сценарий</p>
                    </div>
                </div>
                
                <!-- Scenario Summary -->
                <div id="scenario-summary" class="mb-6 hidden">
                    <div class="flex justify-between items-center mb-2">
                        <h3 class="text-lg font-semibold text-gray-900 dark:text-white">Итоговое время</h3>
                        <span id="total-time" class="font-mono text-gray-700 dark:text-gray-300">00:00.00</span>
                    </div>
                    
                    <div class="chart-container mb-4">
                        <canvas id="time-distribution-chart"></canvas>
                    </div>
                    
                    <div id="stage-times" class="space-y-2">
                        <!-- Stage time bars will be added here dynamically -->
                    </div>
                </div>
                
                <div id="scenario-actions" class="flex flex-wrap gap-2">
                    <button id="copy-scenario-btn" class="px-4 py-2 bg-gray-200 dark:bg-gray-700 text-gray-800 dark:text-white rounded-full text-sm font-medium hover:bg-gray-300 dark:hover:bg-gray-600 transition" disabled>
                        <i class="fas fa-copy mr-1"></i>Копировать
                    </button>
                    <button id="reverse-scenario-btn" class="px-4 py-2 bg-gray-200 dark:bg-gray-700 text-gray-800 dark:text-white rounded-full text-sm font-medium hover:bg-gray-300 dark:hover:bg-gray-600 transition" disabled>
                        <i class="fas fa-exchange-alt mr-1"></i>Обратный порядок
                    </button>
                    <button id="clear-times-btn" class="px-4 py-2 bg-gray-200 dark:bg-gray-700 text-gray-800 dark:text-white rounded-full text-sm font-medium hover:bg-gray-300 dark:hover:bg-gray-600 transition" disabled>
                        <i class="fas fa-eraser mr-1"></i>Очистить время
                    </button>
                    <button id="delete-scenario-btn" class="px-4 py-2 bg-red-500 text-white rounded-full text-sm font-medium hover:bg-red-600 transition" disabled>
                        <i class="fas fa-trash mr-1"></i>Удалить
                    </button>
                    <button id="save-scenario-btn" class="px-4 py-2 bg-blue-500 text-white rounded-full text-sm font-medium hover:bg-blue-600 transition ml-auto" disabled>
                        <i class="fas fa-save mr-1"></i>Сохранить
                    </button>
                </div>
            </div>
        </div>
        
        <!-- Scenario Name Modal -->
        <div id="scenario-name-modal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 hidden">
            <div class="bg-white dark:bg-gray-800 rounded-lg p-6 w-full max-w-md">
                <h3 class="text-xl font-bold text-gray-900 dark:text-white mb-4">Название сценария</h3>
                <input type="text" id="scenario-name-input" class="w-full px-4 py-2 border border-gray-300 dark:border-gray-600 rounded-lg mb-4 dark:bg-gray-700 dark:text-white" placeholder="Введите название">
                <div class="flex justify-end space-x-3">
                    <button id="cancel-scenario-btn" class="px-4 py-2 bg-gray-200 dark:bg-gray-700 text-gray-800 dark:text-white rounded-lg font-medium hover:bg-gray-300 dark:hover:bg-gray-600 transition">
                        Отмена
                    </button>
                    <button id="confirm-scenario-btn" class="px-4 py-2 bg-blue-500 text-white rounded-lg font-medium hover:bg-blue-600 transition">
                        Создать
                    </button>
                </div>
            </div>
        </div>
        
        <!-- Stage Name Modal -->
        <div id="stage-name-modal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 hidden">
            <div class="bg-white dark:bg-gray-800 rounded-lg p-6 w-full max-w-md">
                <h3 class="text-xl font-bold text-gray-900 dark:text-white mb-4">Название этапа</h3>
                <input type="text" id="stage-name-input" class="w-full px-4 py-2 border border-gray-300 dark:border-gray-600 rounded-lg mb-4 dark:bg-gray-700 dark:text-white" placeholder="Введите название">
                <div class="flex justify-end space-x-3">
                    <button id="cancel-stage-btn" class="px-4 py-2 bg-gray-200 dark:bg-gray-700 text-gray-800 dark:text-white rounded-lg font-medium hover:bg-gray-300 dark:hover:bg-gray-600 transition">
                        Отмена
                    </button>
                    <button id="confirm-stage-btn" class="px-4 py-2 bg-blue-500 text-white rounded-lg font-medium hover:bg-blue-600 transition">
                        Сохранить
                    </button>
                </div>
            </div>
        </div>
        
        <!-- Delete Confirmation Modal -->
        <div id="delete-confirm-modal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50 hidden">
            <div class="bg-white dark:bg-gray-800 rounded-lg p-6 w-full max-w-md">
                <h3 class="text-xl font-bold text-gray-900 dark:text-white mb-4">Подтверждение удаления</h3>
                <p class="text-gray-700 dark:text-gray-300 mb-6" id="delete-confirm-text">Вы уверены, что хотите удалить этот сценарий?</p>
                <div class="flex justify-end space-x-3">
                    <button id="cancel-delete-btn" class="px-4 py-2 bg-gray-200 dark:bg-gray-700 text-gray-800 dark:text-white rounded-lg font-medium hover:bg-gray-300 dark:hover:bg-gray-600 transition">
                        Отмена
                    </button>
                    <button id="confirm-delete-btn" class="px-4 py-2 bg-red-500 text-white rounded-lg font-medium hover:bg-red-600 transition">
                        Удалить
                    </button>
                </div>
            </div>
        </div>
    </div>

    <script>
        // Состояние приложения
        const state = {
            timer: {
                running: false,
                startTime: 0,
                elapsed: 0,
                laps: [],
                interval: null,
                currentStageIndex: -1,
                stageStartTime: 0,
                stageElapsed: 0
            },
            scenarios: JSON.parse(localStorage.getItem('scenarios')) || [],
            currentScenarioId: null,
            editingStageIndex: null,
            darkMode: localStorage.getItem('darkMode') === 'true',
            mobileView: false,
            timeChart: null,
            deletingStageIndex: null
        };
        
        // DOM элементы
        const elements = {
            body: document.body,
            timerDisplay: document.getElementById('timer-display'),
            currentStage: document.getElementById('current-stage'),
            startBtn: document.getElementById('start-btn'),
            selectStageBtn: document.getElementById('select-stage-btn'),
            stageSelector: document.getElementById('stage-selector'),
            resetBtn: document.getElementById('reset-btn'),
            timerLaps: document.getElementById('timer-laps'),
            scenarioTabs: document.getElementById('scenario-tabs'),
            scenarioDetails: document.getElementById('scenario-details'),
            scenarioSummary: document.getElementById('scenario-summary'),
            totalTime: document.getElementById('total-time'),
            stageTimes: document.getElementById('stage-times'),
            timeChartCanvas: document.getElementById('time-distribution-chart'),
            newScenarioBtn: document.getElementById('new-scenario-btn'),
            copyScenarioBtn: document.getElementById('copy-scenario-btn'),
            reverseScenarioBtn: document.getElementById('reverse-scenario-btn'),
            clearTimesBtn: document.getElementById('clear-times-btn'),
            deleteScenarioBtn: document.getElementById('delete-scenario-btn'),
            saveScenarioBtn: document.getElementById('save-scenario-btn'),
            scenarioNameModal: document.getElementById('scenario-name-modal'),
            scenarioNameInput: document.getElementById('scenario-name-input'),
            confirmScenarioBtn: document.getElementById('confirm-scenario-btn'),
            cancelScenarioBtn: document.getElementById('cancel-scenario-btn'),
            stageNameModal: document.getElementById('stage-name-modal'),
            stageNameInput: document.getElementById('stage-name-input'),
            confirmStageBtn: document.getElementById('confirm-stage-btn'),
            cancelStageBtn: document.getElementById('cancel-stage-btn'),
            deleteConfirmModal: document.getElementById('delete-confirm-modal'),
            deleteConfirmText: document.getElementById('delete-confirm-text'),
            cancelDeleteBtn: document.getElementById('cancel-delete-btn'),
            confirmDeleteBtn: document.getElementById('confirm-delete-btn'),
            themeToggle: document.getElementById('theme-toggle'),
            viewToggle: document.getElementById('view-toggle')
        };
        
        // Инициализация приложения
        function init() {
            // Установка темы
            if (state.darkMode) {
                elements.body.classList.add('dark');
            }
            
            // Загрузка сценариев
            renderScenarioTabs();
            
            // Если есть сценарии, выбираем первый
            if (state.scenarios.length > 0) {
                selectScenario(state.scenarios[0].id);
            }
            
            // Настройка обработчиков событий
            setupEventListeners();
        }
        
        // Настройка обработчиков событий
        function setupEventListeners() {
            // Таймер
            elements.startBtn.addEventListener('click', startTimer);
            elements.selectStageBtn.addEventListener('click', toggleStageSelector);
            elements.resetBtn.addEventListener('click', resetTimer);
            
            // Сценарии
            elements.newScenarioBtn.addEventListener('click', showNewScenarioModal);
            elements.copyScenarioBtn.addEventListener('click', copyScenario);
            elements.reverseScenarioBtn.addEventListener('click', reverseScenario);
            elements.clearTimesBtn.addEventListener('click', clearScenarioTimes);
            elements.deleteScenarioBtn.addEventListener('click', confirmDeleteScenario);
            elements.saveScenarioBtn.addEventListener('click', saveScenario);
            
            // Модальные окна
            elements.confirmScenarioBtn.addEventListener('click', createScenario);
            elements.cancelScenarioBtn.addEventListener('click', hideScenarioNameModal);
            elements.confirmStageBtn.addEventListener('click', saveStageName);
            elements.cancelStageBtn.addEventListener('click', hideStageNameModal);
            elements.confirmDeleteBtn.addEventListener('click', deleteScenario);
            elements.cancelDeleteBtn.addEventListener('click', hideDeleteConfirmModal);
            
            // Настройки
            elements.themeToggle.addEventListener('click', toggleDarkMode);
            elements.viewToggle.addEventListener('click', toggleMobileView);
            
            // Клик вне селектора этапов
            document.addEventListener('click', (e) => {
                if (!elements.selectStageBtn.contains(e.target) && !elements.stageSelector.contains(e.target)) {
                    elements.stageSelector.classList.remove('show');
                }
            });
        }
        
        // Таймер
        function startTimer() {
            if (!state.timer.running) {
                state.timer.running = true;
                const now = Date.now();
                state.timer.startTime = now - state.timer.elapsed;
                
                // Если есть активный этап, обновляем его время начала
                if (state.timer.currentStageIndex >= 0) {
                    state.timer.stageStartTime = now - state.timer.stageElapsed;
                }
                
                state.timer.interval = setInterval(updateTimer, 10);
                
                elements.startBtn.innerHTML = '<i class="fas fa-pause mr-2"></i>Пауза';
                elements.startBtn.classList.remove('bg-green-500', 'hover:bg-green-600');
                elements.startBtn.classList.add('bg-yellow-500', 'hover:bg-yellow-600');
                elements.selectStageBtn.disabled = false;
            } else {
                pauseTimer();
            }
        }
        
        function pauseTimer() {
            if (state.timer.running) {
                clearInterval(state.timer.interval);
                state.timer.running = false;
                const now = Date.now();
                state.timer.elapsed = now - state.timer.startTime;
                
                // Если есть активный этап, сохраняем его время
                if (state.timer.currentStageIndex >= 0) {
                    state.timer.stageElapsed = now - state.timer.stageStartTime;
                }
                
                elements.startBtn.innerHTML = '<i class="fas fa-play mr-2"></i>Старт';
                elements.startBtn.classList.remove('bg-yellow-500', 'hover:bg-yellow-600');
                elements.startBtn.classList.add('bg-green-500', 'hover:bg-green-600');
            }
        }
        
        function resetTimer() {
            clearInterval(state.timer.interval);
            state.timer = {
                running: false,
                startTime: 0,
                elapsed: 0,
                laps: [],
                interval: null,
                currentStageIndex: -1,
                stageStartTime: 0,
                stageElapsed: 0
            };
            
            elements.timerDisplay.textContent = '00:00.00';
            elements.startBtn.innerHTML = '<i class="fas fa-play mr-2"></i>Старт';
            elements.startBtn.classList.remove('bg-yellow-500', 'hover:bg-yellow-600');
            elements.startBtn.classList.add('bg-green-500', 'hover:bg-green-600');
            elements.selectStageBtn.disabled = true;
            elements.currentStage.textContent = 'Не активно';
            
            // Очищаем отображение кругов
            document.querySelector('.progress-bar').style.transform = 'rotate(0deg)';
            
            // Очищаем список результатов
            const lapsContainer = elements.timerLaps.querySelector('.space-y-2');
            lapsContainer.innerHTML = '<div class="text-center text-gray-500 dark:text-gray-400 py-4">Нет данных</div>';
        }
        
        function updateTimer() {
            const elapsed = Date.now() - state.timer.startTime;
            state.timer.elapsed = elapsed;
            
            const minutes = Math.floor(elapsed / 60000);
            const seconds = Math.floor((elapsed % 60000) / 1000);
            const milliseconds = Math.floor((elapsed % 1000) / 10);
            
            elements.timerDisplay.textContent = 
                `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}.${milliseconds.toString().padStart(2, '0')}`;
            
            // Обновляем прогресс-бар (360 градусов за 60 секунд)
            const degrees = (elapsed % 60000) / 60000 * 360;
            document.querySelector('.progress-bar').style.transform = `rotate(${degrees}deg)`;
        }
        
        function toggleStageSelector() {
            const scenario = getCurrentScenario();
            if (!scenario || scenario.stages.length === 0) return;
            
            // Переключаем видимость селектора
            elements.stageSelector.classList.toggle('show');
            
            // Заполняем селектор этапами
            elements.stageSelector.innerHTML = '';
            
            scenario.stages.forEach((stage, index) => {
                const option = document.createElement('div');
                option.className = 'stage-option flex justify-between items-center';
                option.innerHTML = `
                    <span>${stage.name}</span>
                    ${stage.time ? `<span class="text-xs font-mono">${formatTime(stage.time)}</span>` : ''}
                `;
                
                option.addEventListener('click', () => {
                    selectStage(index);
                    elements.stageSelector.classList.remove('show');
                });
                
                elements.stageSelector.appendChild(option);
            });
        }
        
        function selectStage(index) {
            const scenario = getCurrentScenario();
            if (!scenario || index < 0 || index >= scenario.stages.length) return;
            
            // Если таймер не запущен, просто выбираем этап
            if (!state.timer.running) {
                state.timer.currentStageIndex = index;
                updateCurrentStageDisplay();
                return;
            }
            
            // Если уже замеряем этот этап, ничего не делаем
            if (state.timer.currentStageIndex === index) return;
            
            const now = Date.now();
            
            // Если был активный другой этап, сохраняем его время
            if (state.timer.currentStageIndex >= 0) {
                const prevStageElapsed = now - state.timer.stageStartTime;
                scenario.stages[state.timer.currentStageIndex].time = 
                    (scenario.stages[state.timer.currentStageIndex].time || 0) + prevStageElapsed;
                
                // Добавляем результат
                state.timer.laps.push({
                    name: scenario.stages[state.timer.currentStageIndex].name,
                    time: state.timer.elapsed,
                    lapTime: prevStageElapsed
                });
            }
            
            // Устанавливаем новый активный этап
            state.timer.currentStageIndex = index;
            state.timer.stageStartTime = now;
            state.timer.stageElapsed = 0;
            
            updateCurrentStageDisplay();
            renderLaps();
            renderScenarioDetails();
            updateScenarioSummary();
        }
        
        function formatTime(ms) {
            const minutes = Math.floor(ms / 60000);
            const seconds = Math.floor((ms % 60000) / 1000);
            const milliseconds = Math.floor((ms % 1000) / 10);
            
            return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}.${milliseconds.toString().padStart(2, '0')}`;
        }
        
        function updateCurrentStageDisplay() {
            const scenario = getCurrentScenario();
            
            if (scenario && state.timer.currentStageIndex >= 0 && state.timer.currentStageIndex < scenario.stages.length) {
                elements.currentStage.textContent = scenario.stages[state.timer.currentStageIndex].name;
                
                // Добавляем анимацию пульсации
                elements.currentStage.classList.add('pulse-animation');
            } else {
                elements.currentStage.textContent = 'Не активно';
                elements.currentStage.classList.remove('pulse-animation');
            }
        }
        
        function renderLaps() {
            const lapsContainer = elements.timerLaps.querySelector('.space-y-2');
            
            if (state.timer.laps.length === 0) {
                lapsContainer.innerHTML = '<div class="text-center text-gray-500 dark:text-gray-400 py-4">Нет данных</div>';
                return;
            }
            
            lapsContainer.innerHTML = '';
            
            state.timer.laps.forEach((lap, index) => {
                const lapElement = document.createElement('div');
                lapElement.className = 'flex justify-between items-center p-2 bg-gray-50 dark:bg-gray-700 rounded';
                
                const lapNumber = document.createElement('span');
                lapNumber.className = 'text-gray-600 dark:text-gray-300 font-medium';
                lapNumber.textContent = `${index + 1}. ${lap.name}`;
                
                const lapTime = document.createElement('span');
                lapTime.className = 'font-mono text-gray-900 dark:text-white';
                
                const totalMinutes = Math.floor(lap.time / 60000);
                const totalSeconds = Math.floor((lap.time % 60000) / 1000);
                const totalMilliseconds = Math.floor((lap.time % 1000) / 10);
                
                const lapMinutes = Math.floor(lap.lapTime / 60000);
                const lapSeconds = Math.floor((lap.lapTime % 60000) / 1000);
                const lapMilliseconds = Math.floor((lap.lapTime % 1000) / 10);
                
                lapTime.textContent = 
                    `${totalMinutes.toString().padStart(2, '0')}:${totalSeconds.toString().padStart(2, '0')}.${totalMilliseconds.toString().padStart(2, '0')} ` +
                    `(+${lapMinutes > 0 ? lapMinutes.toString().padStart(2, '0') + ':' : ''}${lapSeconds.toString().padStart(2, '0')}.${lapMilliseconds.toString().padStart(2, '0')})`;
                
                lapElement.appendChild(lapNumber);
                lapElement.appendChild(lapTime);
                lapsContainer.appendChild(lapElement);
            });
        }
        
        // Сценарии
        function getCurrentScenario() {
            return state.scenarios.find(scenario => scenario.id === state.currentScenarioId);
        }
        
        function renderScenarioTabs() {
            elements.scenarioTabs.innerHTML = '';
            
            state.scenarios.forEach(scenario => {
                const tab = document.createElement('div');
                tab.className = `scenario-tab px-4 py-2 font-medium text-sm border-b-2 mr-2 whitespace-nowrap flex items-center ${
                    scenario.id === state.currentScenarioId ? 
                    'border-blue-500 text-blue-600 dark:text-blue-400' : 
                    'border-transparent text-gray-500 hover:text-gray-700 dark:text-gray-400 dark:hover:text-gray-300'
                }`;
                tab.dataset.id = scenario.id;
                
                const tabText = document.createElement('span');
                tabText.textContent = scenario.name;
                tabText.className = 'mr-2';
                
                const closeBtn = document.createElement('button');
                closeBtn.className = 'tab-close-btn text-gray-400 hover:text-red-500 dark:hover:text-red-400';
                closeBtn.innerHTML = '<i class="fas fa-times"></i>';
                closeBtn.addEventListener('click', (e) => {
                    e.stopPropagation();
                    confirmDeleteScenario(scenario.id);
                });
                
                tab.appendChild(tabText);
                tab.appendChild(closeBtn);
                tab.addEventListener('click', () => selectScenario(scenario.id));
                
                elements.scenarioTabs.appendChild(tab);
            });
        }
        
        function selectScenario(scenarioId) {
            state.currentScenarioId = scenarioId;
            renderScenarioTabs();
            renderScenarioDetails();
            updateScenarioSummary();
            
            // Активируем кнопки управления сценарием
            elements.copyScenarioBtn.disabled = false;
            elements.reverseScenarioBtn.disabled = false;
            elements.clearTimesBtn.disabled = false;
            elements.deleteScenarioBtn.disabled = false;
            elements.saveScenarioBtn.disabled = false;
            
            // Если таймер запущен, обновляем выбор этапа
            if (state.timer.running && state.timer.currentStageIndex >= 0) {
                updateCurrentStageDisplay();
            }
        }
        
        function renderScenarioDetails() {
            const scenario = getCurrentScenario();
            
            if (!scenario) {
                elements.scenarioDetails.innerHTML = `
                    <div class="text-center text-gray-500 dark:text-gray-400 py-8">
                        <i class="fas fa-clock text-4xl mb-2"></i>
                        <p>Выберите или создайте сценарий</p>
                    </div>
                `;
                elements.scenarioSummary.classList.add('hidden');
                return;
            }
            
            let html = `
                <div class="mb-4 flex items-center justify-between">
                    <h3 class="text-lg font-semibold text-gray-900 dark:text-white">Этапы сценария</h3>
                    <button id="add-stage-btn" class="px-3 py-1 bg-blue-500 text-white rounded-full text-xs font-medium hover:bg-blue-600 transition">
                        <i class="fas fa-plus mr-1"></i>Добавить
                    </button>
                </div>
                <div id="scenario-stages" class="space-y-2 max-h-60 overflow-y-auto">
            `;
            
            if (scenario.stages.length === 0) {
                html += `
                    <div class="text-center text-gray-500 dark:text-gray-400 py-4">
                        Нет этапов. Нажмите "Добавить", чтобы создать первый этап.
                    </div>
                `;
            } else {
                scenario.stages.forEach((stage, index) => {
                    const timeDisplay = stage.time ? 
                        `${Math.floor(stage.time / 60000).toString().padStart(2, '0')}:${Math.floor((stage.time % 60000) / 1000).toString().padStart(2, '0')}.${Math.floor((stage.time % 1000) / 10).toString().padStart(2, '0')}` : 
                        '--:--.--';
                    
                    html += `
                        <div class="scenario-item flex items-center justify-between p-3 bg-gray-50 dark:bg-gray-700 rounded-lg ${
                            index === state.timer.currentStageIndex ? 'active' : ''
                        }" data-index="${index}" draggable="true">
                            <div class="flex items-center">
                                <button class="handle mr-2 text-gray-400 hover:text-gray-600 dark:hover:text-gray-300 cursor-move">
                                    <i class="fas fa-grip-vertical"></i>
                                </button>
                                <span class="font-medium text-gray-900 dark:text-white">${stage.name}</span>
                            </div>
                            <div class="flex items-center">
                                <span class="font-mono text-gray-700 dark:text-gray-300 mr-3">${timeDisplay}</span>
                                <button class="edit-stage-btn p-1 text-gray-400 hover:text-blue-500 dark:hover:text-blue-400">
                                    <i class="fas fa-pencil-alt"></i>
                                </button>
                                <button class="delete-btn p-1 text-gray-400 hover:text-red-500 dark:hover:text-red-400 ml-1">
                                    <i class="fas fa-trash"></i>
                                </button>
                                <button class="select-stage-btn p-1 text-gray-400 hover:text-green-500 dark:hover:text-green-400 ml-1">
                                    <i class="fas fa-play"></i>
                                </button>
                            </div>
                        </div>
                    `;
                });
            }
            
            html += `</div>`;
            elements.scenarioDetails.innerHTML = html;
            
            // Добавляем обработчики событий для новых элементов
            document.getElementById('add-stage-btn')?.addEventListener('click', showNewStageModal);
            
            document.querySelectorAll('.edit-stage-btn').forEach(btn => {
                btn.addEventListener('click', (e) => {
                    e.stopPropagation();
                    const stageIndex = btn.closest('.scenario-item').dataset.index;
                    showEditStageModal(parseInt(stageIndex));
                });
            });
            
            document.querySelectorAll('.delete-btn').forEach(btn => {
                btn.addEventListener('click', (e) => {
                    e.stopPropagation();
                    const stageIndex = btn.closest('.scenario-item').dataset.index;
                    confirmDeleteStage(parseInt(stageIndex));
                });
            });
            
            document.querySelectorAll('.select-stage-btn').forEach(btn => {
                btn.addEventListener('click', (e) => {
                    e.stopPropagation();
                    const stageIndex = btn.closest('.scenario-item').dataset.index;
                    selectStage(parseInt(stageIndex));
                });
            });
            
            // Настройка перетаскивания
            setupDragAndDrop();
            
            // Показываем сводку, если есть этапы с временем
            if (scenario.stages.some(stage => stage.time)) {
                elements.scenarioSummary.classList.remove('hidden');
            } else {
                elements.scenarioSummary.classList.add('hidden');
            }
        }
        
        function updateScenarioSummary() {
            const scenario = getCurrentScenario();
            
            if (!scenario || scenario.stages.length === 0) {
                elements.scenarioSummary.classList.add('hidden');
                return;
            }
            
            // Рассчитываем общее время
            const lastStageWithTime = [...scenario.stages].reverse().find(stage => stage.time);
            const totalTime = lastStageWithTime ? lastStageWithTime.time : 0;
            
            // Обновляем общее время
            const totalMinutes = Math.floor(totalTime / 60000);
            const totalSeconds = Math.floor((totalTime % 60000) / 1000);
            const totalMilliseconds = Math.floor((totalTime % 1000) / 10);
            
            elements.totalTime.textContent = 
                `${totalMinutes.toString().padStart(2, '0')}:${totalSeconds.toString().padStart(2, '0')}.${totalMilliseconds.toString().padStart(2, '0')}`;
            
            // Обновляем диаграмму
            updateTimeDistributionChart(scenario);
            
            // Обновляем прогресс-бары для каждого этапа
            elements.stageTimes.innerHTML = '';
            
            let prevTime = 0;
            scenario.stages.forEach((stage, index) => {
                if (!stage.time) return;
                
                const stageTime = stage.time - prevTime;
                prevTime = stage.time;
                const percentage = totalTime > 0 ? (stageTime / totalTime * 100).toFixed(1) : 0;
                
                const stageTimeElement = document.createElement('div');
                stageTimeElement.className = 'mb-2';
                
                stageTimeElement.innerHTML = `
                    <div class="flex justify-between mb-1">
                        <span class="text-sm font-medium text-gray-700 dark:text-gray-300">${stage.name}</span>
                        <span class="text-sm font-mono text-gray-600 dark:text-gray-400">
                            ${Math.floor(stageTime / 60000).toString().padStart(2, '0')}:
                            ${Math.floor((stageTime % 60000) / 1000).toString().padStart(2, '0')}.
                            ${Math.floor((stageTime % 1000) / 10).toString().padStart(2, '0')}
                            (${percentage}%)
                        </span>
                    </div>
                    <div class="stage-time-bar">
                        <div class="stage-time-progress" style="width: ${percentage}%"></div>
                    </div>
                `;
                
                elements.stageTimes.appendChild(stageTimeElement);
            });
            
            elements.scenarioSummary.classList.remove('hidden');
        }
        
        function updateTimeDistributionChart(scenario) {
            // Подготовка данных для диаграммы
            const labels = [];
            const data = [];
            const backgroundColors = [];
            const hoverBackgroundColors = [];
            
            let prevTime = 0;
            scenario.stages.forEach((stage, index) => {
                if (!stage.time) return;
                
                const stageTime = stage.time - prevTime;
                prevTime = stage.time;
                
                labels.push(stage.name);
                data.push(stageTime);
                
                // Генерируем цвет для каждого этапа
                const hue = (index * 30) % 360;
                backgroundColors.push(`hsla(${hue}, 80%, 60%, 0.7)`);
                hoverBackgroundColors.push(`hsla(${hue}, 80%, 60%, 0.9)`);
            });
            
            // Если нет данных для отображения, скрываем диаграмму
            if (data.length === 0) {
                if (state.timeChart) {
                    state.timeChart.destroy();
                    state.timeChart = null;
                }
                return;
            }
            
            // Создаем или обновляем диаграмму
            if (state.timeChart) {
                state.timeChart.data.labels = labels;
                state.timeChart.data.datasets[0].data = data;
                state.timeChart.data.datasets[0].backgroundColor = backgroundColors;
                state.timeChart.data.datasets[0].hoverBackgroundColor = hoverBackgroundColors;
                state.timeChart.update();
            } else {
                const ctx = elements.timeChartCanvas.getContext('2d');
                state.timeChart = new Chart(ctx, {
                    type: 'doughnut',
                    data: {
                        labels: labels,
                        datasets: [{
                            data: data,
                            backgroundColor: backgroundColors,
                            hoverBackgroundColor: hoverBackgroundColors,
                            borderWidth: 0
                        }]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                        plugins: {
                            legend: {
                                position: 'right',
                                labels: {
                                    color: state.darkMode ? '#fff' : '#000'
                                }
                            },
                            tooltip: {
                                callbacks: {
                                    label: function(context) {
                                        const value = context.raw;
                                        const total = context.dataset.data.reduce((a, b) => a + b, 0);
                                        const percentage = Math.round((value / total) * 100);
                                        
                                        const minutes = Math.floor(value / 60000);
                                        const seconds = Math.floor((value % 60000) / 1000);
                                        const milliseconds = Math.floor((value % 1000) / 10);
                                        
                                        return `${context.label}: ${minutes}m ${seconds}s ${milliseconds}ms (${percentage}%)`;
                                    }
                                }
                            }
                        },
                        cutout: '60%'
                    }
                });
            }
        }
        
        function showNewScenarioModal() {
            elements.scenarioNameInput.value = '';
            elements.scenarioNameModal.classList.remove('hidden');
            elements.scenarioNameInput.focus();
        }
        
        function hideScenarioNameModal() {
            elements.scenarioNameModal.classList.add('hidden');
        }
        
        function createScenario() {
            const name = elements.scenarioNameInput.value.trim();
            
            if (name) {
                const newScenario = {
                    id: Date.now().toString(),
                    name: name,
                    stages: []
                };
                
                state.scenarios.push(newScenario);
                selectScenario(newScenario.id);
                saveScenariosToStorage();
                hideScenarioNameModal();
            }
        }
        
        function copyScenario() {
            const scenario = getCurrentScenario();
            
            if (scenario) {
                const copy = {
                    id: Date.now().toString(),
                    name: `${scenario.name} (копия)`,
                    stages: JSON.parse(JSON.stringify(scenario.stages))
                };
                
                state.scenarios.push(copy);
                selectScenario(copy.id);
                saveScenariosToStorage();
            }
        }
        
        function reverseScenario() {
            const scenario = getCurrentScenario();
            
            if (scenario) {
                scenario.stages.reverse();
                renderScenarioDetails();
                updateScenarioSummary();
            }
        }
        
        function clearScenarioTimes() {
            const scenario = getCurrentScenario();
            
            if (scenario) {
                scenario.stages.forEach(stage => {
                    stage.time = null;
                });
                
                renderScenarioDetails();
                updateScenarioSummary();
            }
        }
        
        function confirmDeleteScenario(scenarioId = null) {
            const scenario = scenarioId ? 
                state.scenarios.find(s => s.id === scenarioId) : 
                getCurrentScenario();
            
            if (!scenario) return;
            
            elements.deleteConfirmText.textContent = `Вы уверены, что хотите удалить сценарий "${scenario.name}"?`;
            elements.deleteConfirmModal.classList.remove('hidden');
            
            // Сохраняем ID сценария для удаления
            state.deletingScenarioId = scenario.id;
        }
        
        function deleteScenario() {
            const index = state.scenarios.findIndex(s => s.id === state.deletingScenarioId);
            
            if (index !== -1) {
                // Если удаляем текущий сценарий, сбрасываем таймер
                if (state.currentScenarioId === state.deletingScenarioId) {
                    resetTimer();
                }
                
                state.scenarios.splice(index, 1);
                saveScenariosToStorage();
                
                // Выбираем другой сценарий, если есть
                if (state.scenarios.length > 0) {
                    selectScenario(state.scenarios[Math.min(index, state.scenarios.length - 1)].id);
                } else {
                    state.currentScenarioId = null;
                    renderScenarioTabs();
                    renderScenarioDetails();
                    updateScenarioSummary();
                    
                    // Деактивируем кнопки управления сценарием
                    elements.copyScenarioBtn.disabled = true;
                    elements.reverseScenarioBtn.disabled = true;
                    elements.clearTimesBtn.disabled = true;
                    elements.deleteScenarioBtn.disabled = true;
                    elements.saveScenarioBtn.disabled = true;
                }
            }
            
            hideDeleteConfirmModal();
        }
        
        function hideDeleteConfirmModal() {
            elements.deleteConfirmModal.classList.add('hidden');
            state.deletingScenarioId = null;
        }
        
        function saveScenario() {
            saveScenariosToStorage();
            elements.saveScenarioBtn.innerHTML = '<i class="fas fa-save mr-1"></i>Сохранено';
            setTimeout(() => {
                elements.saveScenarioBtn.innerHTML = '<i class="fas fa-save mr-1"></i>Сохранить';
            }, 2000);
        }
        
        function saveScenariosToStorage() {
            localStorage.setItem('scenarios', JSON.stringify(state.scenarios));
        }
        
        // Этапы сценария
        function showNewStageModal() {
            elements.stageNameInput.value = '';
            elements.stageNameModal.classList.remove('hidden');
            elements.stageNameInput.focus();
            state.editingStageIndex = null;
        }
        
        function showEditStageModal(index) {
            const scenario = getCurrentScenario();
            
            if (scenario && index >= 0 && index < scenario.stages.length) {
                elements.stageNameInput.value = scenario.stages[index].name;
                elements.stageNameModal.classList.remove('hidden');
                elements.stageNameInput.focus();
                state.editingStageIndex = index;
            }
        }
        
        function confirmDeleteStage(index) {
            const scenario = getCurrentScenario();
            
            if (!scenario || index < 0 || index >= scenario.stages.length) return;
            
            const stageName = scenario.stages[index].name;
            
            elements.deleteConfirmText.textContent = `Вы уверены, что хотите удалить этап "${stageName}"?`;
            elements.deleteConfirmModal.classList.remove('hidden');
            
            // Сохраняем индекс этапа для удаления
            state.deletingStageIndex = index;
        }
        
        function hideStageNameModal() {
            elements.stageNameModal.classList.add('hidden');
        }
        
        function saveStageName() {
            const name = elements.stageNameInput.value.trim();
            const scenario = getCurrentScenario();
            
            if (name && scenario) {
                if (state.editingStageIndex !== null) {
                    // Редактирование существующего этапа
                    scenario.stages[state.editingStageIndex].name = name;
                } else {
                    // Добавление нового этапа
                    scenario.stages.push({
                        name: name,
                        time: null
                    });
                }
                
                renderScenarioDetails();
                updateScenarioSummary();
                hideStageNameModal();
            }
        }
        
        function setupDragAndDrop() {
            const container = document.getElementById('scenario-stages');
            
            if (!container) return;
            
            let draggedItem = null;
            
            container.querySelectorAll('.scenario-item').forEach(item => {
                item.addEventListener('dragstart', function() {
                    draggedItem = this;
                    setTimeout(() => {
                        this.classList.add('dragging');
                    }, 0);
                });
                
                item.addEventListener('dragend', function() {
                    this.classList.remove('dragging');
                });
            });
            
            container.addEventListener('dragover', function(e) {
                e.preventDefault();
                const afterElement = getDragAfterElement(container, e.clientY);
                const draggingElement = document.querySelector('.dragging');
                
                if (afterElement == null) {
                    container.appendChild(draggingElement);
                } else {
                    container.insertBefore(draggingElement, afterElement);
                }
            });
            
            container.addEventListener('drop', function(e) {
                e.preventDefault();
                const scenario = getCurrentScenario();
                
                if (scenario && draggedItem) {
                    const oldIndex = parseInt(draggedItem.dataset.index);
                    const newIndex = Array.from(container.children).indexOf(draggedItem);
                    
                    if (oldIndex !== newIndex && newIndex >= 0) {
                        // Перемещаем этап в массиве
                        const [movedItem] = scenario.stages.splice(oldIndex, 1);
                        scenario.stages.splice(newIndex, 0, movedItem);
                        
                        // Обновляем индексы в DOM
                        container.querySelectorAll('.scenario-item').forEach((item, idx) => {
                            item.dataset.index = idx;
                        });
                        
                        // Если активный этап был перемещен, обновляем его индекс
                        if (state.timer.currentStageIndex === oldIndex) {
                            state.timer.currentStageIndex = newIndex;
                        } else if (state.timer.currentStageIndex === newIndex) {
                            state.timer.currentStageIndex = oldIndex;
                        }
                        
                        updateCurrentStageDisplay();
                        updateScenarioSummary();
                    }
                }
            });
        }
        
        function getDragAfterElement(container, y) {
            const draggableElements = [...container.querySelectorAll('.scenario-item:not(.dragging)')];
            
            return draggableElements.reduce((closest, child) => {
                const box = child.getBoundingClientRect();
                const offset = y - box.top - box.height / 2;
                
                if (offset < 0 && offset > closest.offset) {
                    return { offset: offset, element: child };
                } else {
                    return closest;
                }
            }, { offset: Number.NEGATIVE_INFINITY }).element;
        }
        
        // Настройки
        function toggleDarkMode() {
            state.darkMode = !state.darkMode;
            localStorage.setItem('darkMode', state.darkMode);
            
            if (state.darkMode) {
                elements.body.classList.add('dark');
            } else {
                elements.body.classList.remove('dark');
            }
            
            // Обновляем цветовую схему диаграммы
            if (state.timeChart) {
                state.timeChart.options.plugins.legend.labels.color = state.darkMode ? '#fff' : '#000';
                state.timeChart.update();
            }
        }
        
        function toggleMobileView() {
            state.mobileView = !state.mobileView;
            
            if (state.mobileView) {
                elements.body.classList.add('mobile-view');
                elements.viewToggle.innerHTML = '<i class="fas fa-desktop"></i>';
            } else {
                elements.body.classList.remove('mobile-view');
                elements.viewToggle.innerHTML = '<i class="fas fa-mobile"></i>';
            }
        }
        
        // Инициализация при загрузке
        document.addEventListener('DOMContentLoaded', init);
    </script>
</body>
</html>
