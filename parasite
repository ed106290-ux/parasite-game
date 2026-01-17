<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>寄生蟲榮譽挑戰 2.0</title>
    <style>
        :root { --primary: #2e7d32; --accent: #d32f2f; --bg: #f5f5f5; }
        body { font-family: "Microsoft JhengHei", sans-serif; background: var(--bg); display: flex; flex-direction: column; align-items: center; padding: 20px; margin: 0; }
        
        h1 { color: #1b5e20; margin-bottom: 10px; }
        #status-panel { background: white; padding: 10px 30px; border-radius: 50px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); font-size: 24px; font-weight: bold; color: var(--accent); margin-bottom: 20px; }

        /* 遊戲區域佈局 - 放大格子 */
        #game-board { display: grid; grid-template-columns: repeat(4, 160px); grid-template-rows: repeat(4, 160px); gap: 15px; }
        
        .card { width: 160px; height: 160px; cursor: pointer; perspective: 1000px; transition: transform 0.3s, opacity 0.5s; }
        .card.matched { opacity: 0; cursor: default; pointer-events: none; transform: scale(0.8); }
        .card-inner { position: relative; width: 100%; height: 100%; transition: transform 0.6s; transform-style: preserve-3d; }
        .card.flipped .card-inner { transform: rotateY(180deg); }
        
        .card-face { position: absolute; width: 100%; height: 100%; backface-visibility: hidden; border-radius: 12px; border: 3px solid white; display: flex; align-items: center; justify-content: center; box-shadow: 0 4px 8px rgba(0,0,0,0.1); box-sizing: border-box; overflow: hidden; }
        .card-back { background: var(--primary); color: white; font-size: 50px; font-weight: bold; }
        .card-front { background: white; transform: rotateY(180deg); padding: 10px; text-align: center; }
        
        /* 圖片放大 2 倍效果 */
        .card-front img { max-width: 100%; max-height: 100%; object-fit: contain; transition: transform 0.3s; }
        .card.zoomed .card-front img { transform: scale(2.0); z-index: 100; position: relative; }
        .card-text { font-size: 16px; font-weight: bold; color: #2c3e50; word-break: break-all; }

        /* 排行榜與按鈕 */
        .side-panel { margin-top: 30px; width: 100%; max-width: 685px; display: flex; justify-content: space-between; gap: 20px; }
        #leaderboard { background: white; padding: 20px; border-radius: 15px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); flex-grow: 1; }
        #leaderboard h2 { margin-top: 0; font-size: 20px; color: var(--primary); border-bottom: 2px solid var(--primary); }
        #score-list { list-style: none; padding: 0; margin: 0; }
        #score-list li { padding: 8px 0; border-bottom: 1px solid #eee; display: flex; justify-content: space-between; }

        button { padding: 12px 40px; font-size: 20px; cursor: pointer; border-radius: 50px; background: #1976d2; color: white; border: none; transition: 0.3s; box-shadow: 0 4px 6px rgba(0,0,0,0.2); }
        button:hover { background: #1565c0; transform: translateY(-2px); }

        #fireworks-canvas { position: fixed; top: 0; left: 0; width: 100%; height: 100%; pointer-events: none; z-index: 1000; }
    </style>
</head>
<body>

    <h1>寄生蟲榮譽挑戰 2.0</h1>
    <div id="status-panel">倒數計時：<span id="timer">180</span>s</div>

    <canvas id="fireworks-canvas"></canvas>

    <div id="game-board"></div>

    <div class="side-panel">
        <button onclick="initGame()">開始新挑戰</button>
        <div id="leaderboard">
            <h2>🏆 本機最速通關排行榜</h2>
            <ul id="score-list"><li>尚無紀錄</li></ul>
        </div>
    </div>

    <audio id="bgm" src="馬力歐闖關音訊.mp3" loop></audio>
    <audio id="sfx-win" src="過關.mp3"></audio>
    <audio id="sfx-fail" src="Game Over.mp3"></audio>

    <script>
        // 資料集 (包含 22 組全集)
        const allParasites = [
            {id:"1dlZyjiKiqPDpf9dXv8BoD7br1MMp7yhP", name:"梨形鞭毛蟲-活動體"},
            {id:"11f0PJvgTj1UX3hVDOdPhfxJYf088xqKz", name:"梨形鞭毛蟲-囊體"},
            {id:"15Pz9ZJkquyevqXLzeQ5_ZVxDeDVEcTOi", name:"唇形鞭毛蟲活動體"},
            {id:"1zMQUWZWWap4Ph3mz1Uuzn0MmJXBEFl6w", name:"唇形鞭毛蟲囊體"},
            {id:"1SOaFSIjtkxjBW0XUEIZjPUIoyzucPWif", name:"人芽蘘原蟲"},
            {id:"1ntCwp9fjYXMypSzTGWd97kOVssZd3Pm7", name:"鉤蟲"},
            {id:"1qUlYkkJiQzNwhgoS-pd4hewEFwLoSmbS", name:"蛔蟲"},
            {id:"1YjiuiemW05uK_TrzQtDwupapQz5OTfmN", name:"糞小桿線蟲"},
            {id:"13EamFAiPuAk-GXjw0UTKTd_e-BA4Lk0l", name:"鞭蟲"},
            {id:"10UMOCi5eqKIGm7WqH5LZKBVoabr5aPlQ", name:"蟯蟲"},
            {id:"19LOrh_vSHQNfsroRJy-NM4qjkybU2GP6", name:"衛氏肺吸蟲"},
            {id:"1jK_94Qi_qDO-Tiew_zE7iAiFPNs0vixk", name:"中華肝吸蟲"},
            {id:"1_jAO_Wh0zHnTjQQIs0JJTWG5KzLdg4Yk", name:"埃及血吸蟲"},
            {id:"1MDQnhoSlwXaxsQSKLROTbgXs6bYhazoU", name:"曼森血吸蟲"},
            {id:"1GSgF5UpjjkmSZehplgbwNI5JHBAa1Faa", name:"日本血吸蟲"},
            {id:"1lUIvRXm4NULjITKWVBQ2-X--t3ceHtef", name:"短小包膜絛蟲"},
            {id:"1PSS0CI57ztTgLNSIu_xIBoH4Sgn7niwZ", name:"縮小包膜絛蟲"},
            {id:"1q3wtgAkR7r9tMlcYbTulatzbzF9kY0IM", name:"廣節裂頭絛蟲"},
            {id:"1gLJ-QnjHmTGECND808V0ceKw8yNYD6xp", name:"肉絛蟲"},
            {id:"1FIQDAR-u2swxqIL-UOvnlJXRW072Rr4E", name:"疑似痢疾阿米巴"},
            {id:"1SNLJoJ3A8NrUC_JmV0EUClHkGIni7VhR", name:"微小阿米巴"},
            {id:"1MUeb42ID7MPhShEnsy8qX8aAsqrBILIs", name:"大腸阿米巴"}
        ];

        let flippedCards = [], matchedPairs = 0, timeLeft = 180, timerId = null;
        const bgm = document.getElementById('bgm');

        window.onload = updateLeaderboard;

        function initGame() {
            clearInterval(timerId);
            timeLeft = 180; matchedPairs = 0; flippedCards = [];
            document.getElementById('timer').innerText = timeLeft;
            document.getElementById('game-board').innerHTML = '';
            
            bgm.playbackRate = 1.0; 
            bgm.currentTime = 0;
            bgm.play().catch(()=>{});

            // 抽樣 8 組
            const selected = [...allParasites].sort(() => 0.5 - Math.random()).slice(0, 8);
            let deck = [];
            selected.forEach(p => {
                deck.push({ type: 'img', id: p.id, val: p.id });
                deck.push({ type: 'txt', id: p.id, val: p.name });
            });
            deck.sort(() => 0.5 - Math.random());

            deck.forEach(data => {
                const card = document.createElement('div');
                card.className = 'card';
                card.dataset.id = data.id;
                let content = data.type === 'img' 
                    ? `<img src="https://drive.google.com/thumbnail?id=${data.val}&sz=w400">`
                    : `<div class="card-text">${data.val}</div>`;
                card.innerHTML = `<div class="card-inner"><div class="card-face card-back">?</div><div class="card-face card-front">${content}</div></div>`;
                card.onclick = () => flipCard(card, data.type);
                document.getElementById('game-board').appendChild(card);
            });
            timerId = setInterval(tick, 1000);
        }

        function flipCard(card, type) {
            if (flippedCards.length < 2 && !card.classList.contains('flipped') && !card.classList.contains('matched')) {
                card.classList.add('flipped');
                if (type === 'img') card.classList.add('zoomed');
                flippedCards.push(card);
                if (flippedCards.length === 2) setTimeout(checkMatch, 800);
            }
        }

        function checkMatch() {
            const [c1, c2] = flippedCards;
            if (c1.dataset.id === c2.dataset.id) {
                c1.classList.add('matched'); c2.classList.add('matched');
                matchedPairs++;
                if (matchedPairs === 8) end(true);
            } else {
                c1.classList.remove('flipped', 'zoomed'); c2.classList.remove('flipped', 'zoomed');
            }
            flippedCards = [];
        }

        function tick() {
            timeLeft--;
            document.getElementById('timer').innerText = timeLeft;
            if (timeLeft === 120) bgm.playbackRate = 1.25;
            if (timeLeft === 60) bgm.playbackRate = 1.5;
            if (timeLeft === 30) bgm.playbackRate = 1.9;
            if (timeLeft <= 0) end(false);
        }

        function end(isWin) {
            clearInterval(timerId); bgm.pause();
            if (isWin) {
                document.getElementById('sfx-win').play();
                startFireworks();
                const score = 180 - timeLeft;
                saveScore(score);
                alert(`恭喜過關！用時 ${score} 秒。`);
            } else {
                document.getElementById('sfx-fail').play();
                alert("超時了，再挑戰一次吧！");
            }
        }

        // --- 排行榜系統 ---
        function saveScore(score) {
            let name = prompt("破紀錄了！請輸入大名：", "挑戰者");
            if (!name) return;
            let scores = JSON.parse(localStorage.getItem('p_ranks')) || [];
            scores.push({ name, score });
            scores.sort((a, b) => a.score - b.score);
            localStorage.setItem('p_ranks', JSON.stringify(scores.slice(0, 5)));
            updateLeaderboard();
        }

        function updateLeaderboard() {
            let scores = JSON.parse(localStorage.getItem('p_ranks')) || [];
            const list = document.getElementById('score-list');
            if (!scores.length) { list.innerHTML = "<li>尚無紀錄</li>"; return; }
            list.innerHTML = scores.map((s, idx) => `<li><span>${idx+1}. ${s.name}</span><span>${s.score}s</span></li>`).join('');
        }

        // --- 煙火特效系統 ---
        function startFireworks() {
            const canvas = document.getElementById("fireworks-canvas");
            const ctx = canvas.getContext("2d");
            canvas.width = window.innerWidth; canvas.height = window.innerHeight;
            let particles = [];
            function createFirework() {
                const x = Math.random() * canvas.width, y = Math.random() * canvas.height;
                for (let i = 0; i < 50; i++) particles.push({ x, y, vx: (Math.random()-0.5)*10, vy: (Math.random()-0.5)*10, life: 100, color: `hsl(${Math.random()*360}, 100%, 50%)` });
            }
            let fireworkInterval = setInterval(createFirework, 300);
            function animate() {
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                particles.forEach((p, i) => { p.x += p.vx; p.y += p.vy; p.life--; if (p.life <= 0) particles.splice(i, 1); ctx.fillStyle = p.color; ctx.fillRect(p.x, p.y, 4, 4); });
                if (matchedPairs === 8) requestAnimationFrame(animate);
            }
            animate();
            setTimeout(() => { clearInterval(fireworkInterval); ctx.clearRect(0,0,canvas.width,canvas.height); }, 5000);
        }
    </script>
</body>
</html>
