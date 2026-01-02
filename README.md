<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>School & Social Lab</title>
    <style>
        :root {
            --primary-color: #2c3e50;
            --secondary-color: #27ae60;
            --accent-color: #e67e22;
            --light-bg: #fdfefe;
            --success-color: #27ae60;
            --warning-color: #f1c40f; 
            --error-color: #e74c3c;
        }

        body { font-family: 'Segoe UI', Tahoma, sans-serif; background: var(--light-bg); margin: 0; padding-bottom: 50px; }
        header { background: var(--primary-color); color: white; padding: 25px; text-align: center; position: relative; }
        .score-display { position: absolute; top: 25px; right: 25px; background: rgba(255,255,255,0.1); padding: 5px 15px; border-radius: 20px; font-weight: bold; border: 1px solid white; }
        .container { max-width: 900px; margin: 30px auto; padding: 0 20px; }
        
        .nav-buttons { display: flex; gap: 10px; margin-bottom: 25px; }
        .nav-button { flex: 1; padding: 12px; border: none; border-radius: 8px; background: #dfe6e9; cursor: pointer; font-weight: bold; }
        .nav-button.active { background: var(--secondary-color); color: white; }

        .exercise-section { background: white; padding: 30px; border-radius: 12px; box-shadow: 0 10px 30px rgba(0,0,0,0.05); }
        .word-bank { display: flex; flex-wrap: wrap; gap: 12px; background: #f1f4f8; padding: 20px; border-radius: 10px; margin-bottom: 20px; border: 2px dashed #bdc3c7; }
        .word-bank-item { background: white; padding: 8px 15px; border-radius: 6px; font-weight: bold; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }

        .gap-sentence { margin: 20px 0; font-size: 1.1em; line-height: 1.6; padding: 15px; background: #fff; border-radius: 8px; border: 1px solid #f0f0f0; }
        .gap-input { border: none; border-bottom: 2px solid var(--secondary-color); width: 170px; text-align: center; font-size: inherit; outline: none; background: transparent; color: var(--accent-color); font-weight: bold; }
        
        .matching-columns { display: grid; grid-template-columns: 1fr 1fr; gap: 40px; }
        .match-box { padding: 15px; border: 2px solid #ecf0f1; border-radius: 8px; cursor: pointer; transition: 0.2s; background: white; text-align: center; display: flex; align-items: center; justify-content: center; min-height: 60px; }
        .match-box.selected { border-color: var(--secondary-color); background: #e8f8f5; }
        .match-box.correct { background: var(--success-color); color: white; border-color: var(--success-color); cursor: default; }

        .status-green { color: var(--success-color); font-weight: bold; }
        .status-yellow { color: #b7950b; font-weight: bold; }
        .status-red { color: var(--error-color); font-weight: bold; }
        .correction-note { font-size: 0.85em; display: block; margin-top: 5px; color: #666; font-style: italic; }

        .speaker-btn, .mic-btn { background: #f1f4f8; border: none; border-radius: 50%; width: 42px; height: 42px; cursor: pointer; display: inline-flex; align-items: center; justify-content: center; font-size: 1.2em; }
        .action-btn { background: var(--primary-color); color: white; border: none; padding: 12px 30px; border-radius: 8px; cursor: pointer; font-weight: bold; margin-top: 20px; }
        
        .vocab-item { display: flex; align-items: center; gap: 20px; padding: 15px 0; border-bottom: 1px solid #eee; }
        .word-cell { flex: 0 0 160px; font-weight: bold; color: var(--secondary-color); font-size: 1.1em; }
    </style>
</head>
<body>

<header>
    <h1>School & Social Lab</h1>
    <div class="score-display">Score: <span id="score-val">0</span></div>
</header>

<div class="container">
    <div class="nav-buttons">
        <button class="nav-button active" onclick="switchTab(0)">1. Study (US Voice)</button>
        <button class="nav-button" onclick="switchTab(1)">2. Writing</button>
        <button class="nav-button" onclick="switchTab(2)">3. Matching</button>
        <button class="nav-button" onclick="switchTab(3)">4. Speaking</button>
    </div>

    <div id="tab-0" class="exercise-section">
        <h2>📚 American Accent Study List</h2>
        <div id="vocab-list"></div>
    </div>

    <div id="tab-1" class="exercise-section" style="display:none">
        <h2>✍️ Mixed Context Writing</h2>
        <div id="writing-bank" class="word-bank"></div>
        <div id="writing-content"></div>
        <button class="action-btn" onclick="checkWriting()">Verify All</button>
    </div>

    <div id="tab-2" class="exercise-section" style="display:none">
        <h2>🔗 Match Word to Meaning</h2>
        <div class="matching-columns">
            <div id="match-left" class="match-column"></div>
            <div id="match-right" class="match-column"></div>
        </div>
    </div>

    <div id="tab-3" class="exercise-section" style="display:none">
        <h2>🗣️ Guess & Pronounce (Hidden)</h2>
        <div id="pron-bank" class="word-bank"></div>
        <div id="pron-content"></div>
    </div>
</div>

<script>
    const data = [
        { word: "PE", def: "Physical Education (Gym class).", sents: ["I have ___ on Tuesday mornings.", "Don't forget your sneakers for ___ class."] },
        { word: "French", def: "The language spoken in France.", sents: ["I am learning ___ at school.", "My ___ teacher is very nice."] },
        { word: "desk", def: "A piece of furniture used for reading or writing.", sents: ["I keep my books inside my ___.", "Please sit at your ___ and start the test."] },
        { word: "get along", def: "To have a friendly relationship with someone.", sents: ["I really ___ with my new roommate.", "Do you ___ with your siblings?"] },
        { word: "classmate", def: "A member of the same class at school.", sents: ["She is my favorite ___.", "I studied with a ___ for the exam."] },
        { word: "busy", def: "Having a lot of things to do.", sents: ["I am very ___ with work this week.", "Is the teacher ___ right now?"] },
        { word: "well", def: "In a good or satisfactory way (adverb).", sents: ["He speaks English very ___.", "I hope you are doing ___."] },
        { word: "earphones", def: "Small speakers worn over or in the ears.", sents: ["I lost my ___ on the bus.", "Please use ___ to listen to music."] },
        { word: "both", def: "Used for two people or things together.", sents: ["___ of my parents are doctors.", "I like ___ pizza and pasta."] }
    ];

    let totalScore = 0;
    function shuffle(array) { return array.sort(() => Math.random() - 0.5); }

    function getDist(a, b) {
        a = a.toLowerCase().replace(/[-\s]/g, ''); b = b.toLowerCase().replace(/[-\s]/g, '');
        const matrix = Array.from({ length: b.length + 1 }, (_, i) => [i]);
        for (let j = 0; j <= a.length; j++) matrix[0][j] = j;
        for (let i = 1; i <= b.length; i++) {
            for (let j = 1; j <= a.length; j++) {
                matrix[i][j] = b[i - 1] === a[j - 1] ? matrix[i - 1][j - 1] : Math.min(matrix[i - 1][j - 1] + 1, matrix[i][j - 1] + 1, matrix[i - 1][j] + 1);
            }
        }
        return matrix[b.length][a.length];
    }

    function speakUS(text) {
        const synth = window.speechSynthesis;
        const utter = new SpeechSynthesisUtterance(text);
        const voices = synth.getVoices();
        utter.voice = voices.find(v => v.lang === 'en-US' && (v.name.includes('Google') || v.name.includes('Natural'))) || voices.find(v => v.lang === 'en-US') || voices[0];
        utter.rate = 0.85;
        synth.speak(utter);
    }

    function switchTab(idx) {
        document.querySelectorAll('.exercise-section').forEach((s, i) => s.style.display = i === idx ? 'block' : 'none');
        document.querySelectorAll('.nav-button').forEach((b, i) => b.classList.toggle('active', i === idx));
    }

    function init() {
        const words = data.map(d => d.word);
        document.getElementById('vocab-list').innerHTML = data.map(i => `<div class="vocab-item"><div class="word-cell">${i.word}</div><button class="speaker-btn" onclick="speakUS('${i.word}')">🔊</button><div>${i.def}</div></div>`).join('');

        document.getElementById('writing-bank').innerHTML = shuffle([...words]).map(w => `<div class="word-bank-item">${w}</div>`).join('');
        let wPool = [];
        data.forEach(d => {
            wPool.push({t: d.sents[0], a: d.word});
            wPool.push({t: d.sents[1], a: d.word});
        });
        shuffle(wPool);
        document.getElementById('writing-content').innerHTML = wPool.slice(0, 10).map((s, i) => `<div class="gap-sentence">${s.t.replace(s.a, '___').replace('___', `<input type="text" class="gap-input" data-ans="${s.a}" id="w-${i}">`)}<div id="fb-w-${i}"></div></div>`).join('');

        const mDef = shuffle(data.map((d, i) => ({ t: d.def, id: i, type: 'd' })));
        const mWord = shuffle(data.map((d, i) => ({ t: d.word, id: i, type: 'w' })));
        document.getElementById('match-left').innerHTML = mDef.map(o => `<div class="match-box" data-id="${o.id}" data-type="${o.type}" onclick="handleMatch(this)">${o.t}</div>`).join('');
        document.getElementById('match-right').innerHTML = mWord.map(o => `<div class="match-box" data-id="${o.id}" data-type="${o.type}" onclick="handleMatch(this)">${o.t}</div>`).join('');

        document.getElementById('pron-bank').innerHTML = shuffle([...words]).map(w => `<div class="word-bank-item">${w}</div>`).join('');
        let pPool = [];
        data.forEach(d => pPool.push({t: d.sents[1], a: d.word}));
        shuffle(pPool);
        document.getElementById('pron-content').innerHTML = pPool.map((s, i) => `<div class="gap-sentence">${s.t.replace(s.a, '_________')} <button class="mic-btn" onclick="testSpeech('${s.a}', 'p-fb-${i}')">🎤</button><div id="p-fb-${i}" style="font-size: 0.85em; color:#888; margin-top:5px;">Guess the word from the bank!</div></div>`).join('');
    }

    function checkWriting() {
        document.querySelectorAll('.gap-input').forEach(input => {
            const fb = document.getElementById(`fb-${input.id}`);
            const val = input.value.trim().toLowerCase();
            const target = input.dataset.ans.toLowerCase();
            const dist = getDist(val, target);
            if (val === target) { fb.innerHTML = `<span class="status-green">✅ Correct!</span>`; totalScore += 1; }
            else if (dist <= 2) { fb.innerHTML = `<span class="status-yellow">⚠️ Typo!</span> <span class="correction-note">Correct spelling: "${input.dataset.ans}"</span>`; totalScore += 0.5; }
            else { fb.innerHTML = `<span class="status-red">❌ Correct: "${input.dataset.ans}"</span>`; }
        });
        document.getElementById('score-val').innerText = Math.floor(totalScore);
    }

    let firstM = null;
    function handleMatch(el) {
        if (el.classList.contains('correct')) return;
        if (!firstM) { firstM = el; el.classList.add('selected'); }
        else {
            if (firstM.dataset.id === el.dataset.id && firstM.dataset.type !== el.dataset.type) {
                firstM.classList.replace('selected', 'correct'); el.classList.add('correct');
                totalScore += 2; document.getElementById('score-val').innerText = Math.floor(totalScore);
            } else { firstM.classList.remove('selected'); }
            firstM = null;
        }
    }

    function testSpeech(word, fbId) {
        const rec = new (window.SpeechRecognition || window.webkitSpeechRecognition)();
        rec.lang = 'en-US'; rec.start();
        document.getElementById(fbId).innerText = "Listening...";
        rec.onresult = (e) => {
            const heard = e.results[0][0].transcript.toLowerCase();
            const target = word.toLowerCase();
            const dist = getDist(heard, target);
            const fb = document.getElementById(fbId);
            if (heard.includes(target)) { fb.innerHTML = `<span class="status-green">✅ Perfect!</span>`; totalScore += 1; }
            else if (dist <= 3) { fb.innerHTML = `<span class="status-yellow">⚠️ Close (heard "${heard}")</span>`; totalScore += 0.5; }
            else { fb.innerHTML = `<span class="status-red">❌ Heard "${heard}". Word: "${word}"</span>`; }
            document.getElementById('score-val').innerText = Math.floor(totalScore);
        };
    }

    window.onload = () => {
        if ('speechSynthesis' in window) {
            window.speechSynthesis.onvoiceschanged = init;
            if (window.speechSynthesis.getVoices().length > 0) init();
        } else { init(); }
    };
</script>
</body>
</html>
