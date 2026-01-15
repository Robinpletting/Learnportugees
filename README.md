<!DOCTYPE html>
<html lang="nl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Taaltrainer Portugees</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body { 
            font-family: ui-sans-serif, system-ui, -apple-system, sans-serif;
            background-color: #F8FAFC;
            color: #000;
            user-select: none;
            -webkit-tap-highlight-color: transparent;
        }
        .card-contrast {
            background-color: #FFF;
            border: 3px solid #000;
            box-shadow: 4px 4px 0px #000;
            border-radius: 28px;
        }
        .locked {
            opacity: 0.4;
            filter: grayscale(1);
            cursor: not-allowed !important;
        }
        .option-btn {
            background-color: #FFF;
            border: 3px solid #000;
            transition: all 0.15s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            cursor: pointer;
            box-shadow: 4px 4px 0px #000;
            min-height: 64px;
            display: flex;
            align-items: center;
            padding: 0.75rem 1.5rem;
            font-weight: 800;
            text-align: left;
            border-radius: 24px;
            width: 100%;
            font-size: 1rem;
        }
        .option-btn:active:not(.locked) {
            transform: translate(2px, 2px);
            box-shadow: 1px 1px 0px #000;
        }
        .correct { 
            background-color: #22C55E !important; 
            color: white !important; 
            border-color: #000 !important; 
            box-shadow: none !important; 
            transform: scale(1.02) translate(2px, 2px);
        }
        .wrong { 
            background-color: #EF4444 !important; 
            color: white !important; 
            border-color: #000 !important; 
            box-shadow: none !important; 
            transform: translate(2px, 2px); 
        }
        
        .stat-pill {
            padding: 8px 16px;
            border: 2px solid #000;
            border-radius: 9999px;
            font-weight: 900;
            font-size: 12px;
            display: flex;
            align-items: center;
            gap: 6px;
        }
        
        .slide-up { animation: slideUp 0.3s cubic-bezier(0.4, 0, 0.2, 1); }
        @keyframes slideUp { from { transform: translateY(100%); } to { transform: translateY(0); } }
        .no-scrollbar::-webkit-scrollbar { display: none; }
    </style>
</head>
<body class="p-0 m-0">

    <div id="app">
        <!-- Dashboard -->
        <div id="dashboard" class="max-w-md mx-auto p-6 pb-24">
            <header class="flex justify-between items-center mb-8 pt-4">
                <div class="card-contrast p-3 px-6 bg-white border-blue-500 rounded-full flex flex-col">
                    <span class="text-[9px] font-black uppercase tracking-widest opacity-50">Rang</span>
                    <span id="rank-text" class="text-xs font-bold italic text-blue-600">Beginner</span>
                </div>
                <div class="card-contrast p-3 px-6 bg-white rounded-full flex flex-col items-center">
                    <span class="text-[9px] font-black uppercase tracking-widest opacity-50">Totaal XP</span>
                    <span id="display-xp" class="text-2xl font-black leading-none mt-1">0</span>
                </div>
            </header>

            <!-- Niveau Selector -->
            <div class="flex gap-2 overflow-x-auto mb-8 pb-2 no-scrollbar">
                <button onclick="setCategory('A1')" id="nav-A1" class="px-6 py-2 border-2 border-black rounded-full font-black whitespace-nowrap">A1</button>
                <button onclick="setCategory('A2')" id="nav-A2" class="px-6 py-2 border-2 border-black rounded-full font-black whitespace-nowrap">A2</button>
                <button onclick="setCategory('B1')" id="nav-B1" class="px-6 py-2 border-2 border-black rounded-full font-black whitespace-nowrap">B1</button>
                <button onclick="setCategory('B2')" id="nav-B2" class="px-6 py-2 border-2 border-black rounded-full font-black whitespace-nowrap">B2</button>
            </div>
            
            <div id="levels-list" class="space-y-4"></div>
        </div>

        <!-- Quiz Interface -->
        <div id="quiz-overlay" class="hidden fixed inset-0 bg-white z-50 flex flex-col">
            <div class="p-4 bg-white border-b-4 border-black">
                <div class="flex justify-between items-center mb-4">
                    <button onclick="showExitModal()" class="w-10 h-10 flex items-center justify-center bg-black text-white rounded-full font-black hover:bg-red-500 transition-colors">✕</button>
                    <div class="flex gap-2">
                        <div class="stat-pill bg-green-50 text-green-700 border-green-700">✓ <span id="stat-correct">0</span></div>
                        <div class="stat-pill bg-red-50 text-red-700 border-red-700">✗ <span id="stat-wrong">0</span></div>
                        <div class="stat-pill bg-slate-50 border-slate-400 font-mono"><span id="stat-count">0</span>/30</div>
                    </div>
                </div>
                <div class="w-full bg-slate-100 border-2 border-black h-4 rounded-full overflow-hidden">
                    <div id="progress-bar" class="bg-blue-500 h-full transition-all duration-500" style="width: 0%"></div>
                </div>
            </div>
            
            <div class="flex-1 flex flex-col p-6 items-center justify-center relative overflow-hidden text-black">
                <div class="w-full max-w-lg">
                    <p id="level-indicator" class="text-center font-black text-[10px] opacity-40 mb-2 uppercase tracking-widest text-blue-600"></p>
                    <div class="flex justify-center mb-6">
                        <span id="mode-badge" class="px-3 py-1 bg-black text-white text-[10px] font-black rounded-md uppercase tracking-tighter"></span>
                    </div>
                    <h2 id="q-text" class="text-2xl md:text-3xl font-black mb-10 text-center leading-tight px-4 min-h-[80px] flex items-center justify-center"></h2>
                    <div id="options-container" class="grid grid-cols-1 gap-3 px-2"></div>
                </div>

                <!-- Feedback Paneel -->
                <div id="feedback-panel" class="hidden absolute bottom-0 left-0 right-0 bg-white border-t-8 border-black p-8 rounded-t-[40px] shadow-[0_-20px_50px_rgba(0,0,0,0.2)] slide-up z-20">
                    <div class="max-w-md mx-auto">
                        <h3 class="text-red-500 font-black text-2xl mb-1 uppercase italic">Fout!</h3>
                        <p class="text-[10px] font-bold opacity-50 uppercase mb-2">Het juiste antwoord was:</p>
                        <p id="fb-correct" class="text-xl font-extrabold text-black mb-8 p-6 bg-slate-50 border-4 border-black rounded-2xl text-center"></p>
                        <button onclick="closeFeedback()" class="w-full bg-black text-white p-5 rounded-full font-black uppercase shadow-[4px_4px_0px_#444] active:translate-y-1">Doorgaan</button>
                    </div>
                </div>
            </div>
        </div>

        <!-- Resume Session Modal -->
        <div id="resume-modal" class="hidden fixed inset-0 z-[110] flex items-center justify-center p-6 bg-black/60 backdrop-blur-md">
            <div class="bg-white border-4 border-black p-8 rounded-[32px] max-w-sm w-full shadow-[10px_10px_0px_#000]">
                <h3 class="text-2xl font-black mb-4 uppercase italic">Sessie gevonden!</h3>
                <p id="resume-text" class="font-bold text-slate-600 mb-8 leading-relaxed">Je was halverwege dit level. Wil je doorgaan waar je gebleven was?</p>
                <div class="flex flex-col gap-3">
                    <button id="resume-btn" class="w-full bg-blue-500 text-white py-4 rounded-full font-black uppercase border-2 border-black shadow-[4px_4px_0px_#000] active:translate-y-0.5">Hervatten</button>
                    <button id="restart-btn" class="w-full bg-slate-200 text-black py-4 rounded-full font-black uppercase border-2 border-black active:translate-y-0.5">Opnieuw beginnen</button>
                </div>
            </div>
        </div>

        <!-- Custom Exit Confirmation Modal -->
        <div id="exit-modal" class="hidden fixed inset-0 z-[100] flex items-center justify-center p-6 bg-black/50 backdrop-blur-sm">
            <div class="bg-white border-4 border-black p-8 rounded-[32px] max-w-sm w-full shadow-[8px_8px_0px_#000]">
                <h3 class="text-2xl font-black mb-4 uppercase italic">Stoppen?</h3>
                <p class="font-bold text-slate-600 mb-8 leading-relaxed">Je voortgang wordt opgeslagen. Je kunt later doorgaan vanaf dit punt.</p>
                <div class="flex gap-4">
                    <button onclick="hideExitModal()" class="flex-1 bg-slate-200 text-black py-4 rounded-full font-black uppercase border-2 border-black active:translate-y-0.5">Doorgaan</button>
                    <button onclick="confirmExit()" class="flex-1 bg-red-500 text-white py-4 rounded-full font-black uppercase border-2 border-black shadow-[3px_3px_0px_#000] active:translate-y-0.5">Ja, stop</button>
                </div>
            </div>
        </div>

        <!-- Resultaat Scherm -->
        <div id="result-screen" class="hidden fixed inset-0 z-[60] flex flex-col items-center justify-center p-8 text-center bg-white border-[12px] border-black">
            <h2 id="result-title" class="text-4xl font-black mb-6 uppercase italic"></h2>
            <div class="bg-white border-4 border-black p-12 rounded-[48px] mb-10 shadow-[12px_12px_0px_#000]">
                <p id="result-score" class="text-8xl font-black leading-none">0/30</p>
                <p class="mt-4 font-black opacity-40 uppercase tracking-widest text-[12px]">Totaal Score</p>
            </div>
            <p id="result-msg" class="mb-10 font-bold text-lg max-w-xs leading-relaxed"></p>
            <button onclick="returnToDashboard()" class="w-full max-w-xs py-5 rounded-full bg-black text-white font-black uppercase text-xl shadow-[6px_6px_0px_#444] active:scale-95 transition-transform">Klaar</button>
        </div>
    </div>

    <script>
        const MASTER_DATA = {
            "A1": [
                {p:"Olá, como estás?", n:"Hallo, hoe gaat het?"}, {p:"Eu bebo água", n:"Ik drink water"}, {p:"Sim, por favor", n:"Ja, alstublieft"},
                {p:"Obrigado", n:"Bedankt"}, {p:"A maçã é vermelha", n:"De appel is rood"}, {p:"Bom dia, amigo", n:"Goedemorgen, vriend"},
                {p:"O gato dorme", n:"De kat slaapt"}, {p:"Um café preto", n:"Een zwarte koffie"}, {p:"A casa é grande", n:"Het huis is groot"},
                {p:"Onde estás?", n:"Waar ben je?"}, {p:"Eu gosto de pão", n:"Ik hou van brood"}, {p:"Até amanhã", n:"Tot morgen"},
                {p:"Eu sou estudante", n:"Ik ben student"}, {p:"Ela é bonita", n:"Zij is mooi"}, {p:"Nós falamos português", n:"Wij spreken Portugees"},
                {p:"O carro é azul", n:"De auto is blauw"}, {p:"Eu tenho um cão", n:"Ik heb een hond"}, {p:"Boa noite", n:"Goedenavond"},
                {p:"A caneta escreve", n:"De pen schrijft"}, {p:"O livro é novo", n:"Het boek is nieuw"}
            ],
            "A2": [
                {p:"Ontem fui ao cinema", n:"Gisteren ben ik naar de bioscoop gegaan"}, {p:"Amanhã vou trabalhar", n:"Morgen ga ik werken"},
                {p:"Eu gosto de comer peixe", n:"Ik hou van vis eten"}, {p:"Estou muito cansado", n:"Ik ben erg moe"},
                {p:"Queres beber algo?", n:"Wil je iets drinken?"}, {p:"O tempo está bom", n:"Het weer is goed"},
                {p:"Onde fica a estação?", n:"Waar is het station?"}, {p:"Quanto custa isto?", n:"Hoeveel kost dit?"},
                {p:"Eu perdi as chaves", n:"Ik ben de sleutels kwijt"}, {p:"Eles vivem aqui", n:"Zij wonen hier"}
            ],
            "B1": [
                {p:"Eu acredito na tua palavra", n:"Ik geloof in jouw woord"}, {p:"Espero que tenhas sorte", n:"Ik hoop dat je geluk hebt"},
                {p:"O problema foi resolvido", n:"Het probleem is opgelost"}, {p:"Não me lembro de ti", n:"Ik herinner me jou niet"},
                {p:"Temos que decidir agora", n:"We moeten nu beslissen"}
            ],
            "B2": [
                {p:"A sustentabilidade é vital", n:"Duurzaamheid is vitaal"}, {p:"As consequências são graves", n:"De gevolgen zijn ernstig"},
                {p:"É necessário investir mais", n:"Het is nodig om meer te investeren"}
            ]
        };

        const CATS = ["A1", "A2", "B1", "B2"];
        let state = JSON.parse(localStorage.getItem('port_trainer_v6_save')) || {
            xp: 0,
            bestScores: {}, 
            unlockedCats: ["A1"],
            activeSession: null // Bevat { id, questions, index, correct, wrong }
        };

        let currentCat = "A1";
        let activeQuiz = null;

        const shuffle = (arr) => [...arr].sort(() => Math.random() - 0.5);

        function renderDashboard() {
            document.getElementById('display-xp').innerText = state.xp;
            const list = document.getElementById('levels-list');
            list.innerHTML = '';
            
            CATS.forEach(cat => {
                const btn = document.getElementById(`nav-${cat}`);
                const isUnlocked = state.unlockedCats.includes(cat);
                btn.disabled = !isUnlocked;
                btn.className = `px-6 py-2 border-2 border-black rounded-full font-black transition-all ${!isUnlocked ? 'locked' : (cat === currentCat ? 'bg-yellow-400 shadow-[3px_3px_0px_#000] -translate-y-0.5' : 'bg-white hover:bg-slate-50')}`;
            });

            const rank = state.xp > 5000 ? "Linguista" : (state.xp > 2000 ? "Gevorderd" : (state.xp > 500 ? "Student" : "Beginner"));
            document.getElementById('rank-text').innerText = rank;

            for(let i=1; i<=10; i++) {
                const levelId = `${currentCat}-${i}`;
                const score = state.bestScores[levelId] || 0;
                const isUnlocked = i === 1 || (state.bestScores[`${currentCat}-${i-1}`] >= 80);
                const isInterrupted = state.activeSession && state.activeSession.id === levelId;
                
                const card = document.createElement('div');
                card.className = `p-5 rounded-[32px] border-4 border-black flex items-center gap-4 bg-white shadow-[4px_4px_0px_#000] transition-transform ${!isUnlocked ? 'locked' : 'cursor-pointer active:scale-95'}`;
                
                if(isUnlocked) card.onclick = () => initQuizCheck(levelId);
                
                card.innerHTML = `
                    <div class="w-12 h-12 shrink-0 rounded-full flex items-center justify-center text-lg border-2 border-black font-black ${score >= 80 ? 'bg-green-400' : (isUnlocked ? 'bg-blue-400 text-white' : 'bg-slate-100')}">${i}</div>
                    <div class="flex-1">
                        <div class="flex justify-between font-black text-[10px] uppercase text-black mb-1.5">
                            <span>Niveau ${i} ${!isUnlocked ? '🔒' : ''} ${isInterrupted ? '🕒' : ''}</span>
                            <span>${score}%</span>
                        </div>
                        <div class="w-full bg-slate-100 border-2 border-black h-3 rounded-full overflow-hidden">
                            <div class="bg-blue-600 h-full transition-all duration-700" style="width: ${score}%"></div>
                        </div>
                    </div>
                `;
                list.appendChild(card);
            }
        }

        function initQuizCheck(id) {
            if (state.activeSession && state.activeSession.id === id) {
                document.getElementById('resume-modal').classList.remove('hidden');
                document.getElementById('resume-text').innerText = `Je bent bij vraag ${state.activeSession.index + 1}. Wil je doorgaan?`;
                
                document.getElementById('resume-btn').onclick = () => {
                    activeQuiz = state.activeSession;
                    document.getElementById('resume-modal').classList.add('hidden');
                    launchQuiz();
                };
                
                document.getElementById('restart-btn').onclick = () => {
                    state.activeSession = null;
                    document.getElementById('resume-modal').classList.add('hidden');
                    startQuiz(id);
                };
            } else {
                startQuiz(id);
            }
        }

        function startQuiz(id) {
            const cat = id.split('-')[0];
            const pool = MASTER_DATA[cat] || MASTER_DATA["A1"];
            
            let sessionPool = [];
            pool.forEach(item => sessionPool.push({ q: item.p, a: item.n, mode: 'Vertaal naar NL', type: 'PT_NL' }));
            pool.forEach(item => sessionPool.push({ q: item.n, a: item.p, mode: 'Vertaal naar PT', type: 'NL_PT' }));
            pool.forEach(item => sessionPool.push({ q: `Juiste spelling van: "${item.p}"`, a: item.p, mode: 'Spelling Focus', type: 'SPELL' }));

            activeQuiz = {
                id,
                questions: shuffle(sessionPool).slice(0, 30),
                index: 0,
                correct: 0,
                wrong: 0
            };
            
            launchQuiz();
        }

        function launchQuiz() {
            document.getElementById('quiz-overlay').classList.remove('hidden');
            document.getElementById('level-indicator').innerText = `Training ${activeQuiz.id.split('-')[0]} • Niveau ${activeQuiz.id.split('-')[1]}`;
            renderQuestion();
            updateStats();
        }

        function renderQuestion() {
            const currentQ = activeQuiz.questions[activeQuiz.index];
            document.getElementById('q-text').innerText = currentQ.q;
            document.getElementById('mode-badge').innerText = currentQ.mode;
            document.getElementById('feedback-panel').classList.add('hidden');
            
            const container = document.getElementById('options-container');
            container.innerHTML = '';

            // Zoek de cat pool voor afleiders
            const cat = activeQuiz.id.split('-')[0];
            const pool = MASTER_DATA[cat] || MASTER_DATA["A1"];

            let distractors = [];
            if(currentQ.type === 'PT_NL') {
                distractors = pool.map(x => x.n).filter(x => x !== currentQ.a);
            } else {
                distractors = pool.map(x => x.p).filter(x => x !== currentQ.a);
            }
            
            const options = shuffle([currentQ.a, ...shuffle(distractors).slice(0, 3)]);

            options.forEach(opt => {
                const btn = document.createElement('button');
                btn.className = "option-btn";
                btn.innerText = opt;
                btn.onclick = () => handleAnswer(opt === currentQ.a, btn);
                container.appendChild(btn);
            });
            
            // Sla sessie tussentijds op
            state.activeSession = activeQuiz;
            save();
        }

        function handleAnswer(isCorrect, btn) {
            document.querySelectorAll('.option-btn').forEach(b => b.onclick = null);
            
            if(isCorrect) {
                btn.classList.add('correct');
                activeQuiz.correct++;
                setTimeout(next, 800);
            } else {
                btn.classList.add('wrong');
                activeQuiz.wrong++;
                showFeedback(activeQuiz.questions[activeQuiz.index].a);
            }
        }

        function showFeedback(correctTxt) {
            document.getElementById('fb-correct').innerText = correctTxt;
            document.getElementById('feedback-panel').classList.remove('hidden');
        }

        function closeFeedback() {
            document.getElementById('feedback-panel').classList.add('hidden');
            next();
        }

        function next() {
            activeQuiz.index++;
            updateStats();
            if(activeQuiz.index < 30) {
                renderQuestion();
            } else {
                finishQuiz();
            }
        }

        function updateStats() {
            document.getElementById('stat-correct').innerText = activeQuiz.correct;
            document.getElementById('stat-wrong').innerText = activeQuiz.wrong;
            document.getElementById('stat-count').innerText = activeQuiz.index;
            document.getElementById('progress-bar').style.width = `${(activeQuiz.index / 30) * 100}%`;
        }

        function finishQuiz() {
            const score = Math.round((activeQuiz.correct / 30) * 100);
            const success = score >= 80;
            const cat = activeQuiz.id.split('-')[0];
            const levelNum = parseInt(activeQuiz.id.split('-')[1]);

            state.bestScores[activeQuiz.id] = Math.max(state.bestScores[activeQuiz.id] || 0, score);
            state.xp += activeQuiz.correct * 10;
            
            // Verwijder actieve sessie bij voltooiing
            state.activeSession = null;

            if(success && levelNum === 10) {
                const nextIdx = CATS.indexOf(cat) + 1;
                if(nextIdx < CATS.length && !state.unlockedCats.includes(CATS[nextIdx])) {
                    state.unlockedCats.push(CATS[nextIdx]);
                }
            }

            const screen = document.getElementById('result-screen');
            screen.classList.remove('hidden');
            screen.style.backgroundColor = success ? '#DCFCE7' : '#FEE2E2';
            document.getElementById('result-title').innerText = success ? "GEWELDIG!" : "BIJNA...";
            document.getElementById('result-score').innerText = `${activeQuiz.correct}/30`;
            document.getElementById('result-msg').innerText = success 
                ? "Je hebt dit niveau gehaald met een score van " + score + "%!" 
                : "Je hebt 80% nodig om door te gaan. Je score was " + score + "%. Probeer het nog eens!";
            
            save();
        }

        function returnToDashboard() {
            document.getElementById('result-screen').classList.add('hidden');
            document.getElementById('quiz-overlay').classList.add('hidden');
            renderDashboard();
        }

        function showExitModal() {
            document.getElementById('exit-modal').classList.remove('hidden');
        }

        function hideExitModal() {
            document.getElementById('exit-modal').classList.add('hidden');
        }

        function confirmExit() {
            hideExitModal();
            document.getElementById('quiz-overlay').classList.add('hidden');
            // We behouden activeSession in state zodat we later kunnen hervatten
            renderDashboard();
        }

        function save() { 
            localStorage.setItem('port_trainer_v6_save', JSON.stringify(state)); 
        }

        function setCategory(c) { 
            currentCat = c; 
            renderDashboard(); 
        }

        renderDashboard();
    </script>
</body>
</html>

