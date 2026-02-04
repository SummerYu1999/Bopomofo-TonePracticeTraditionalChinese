<html lang="zh-Hant">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>注音精密教學系統 - 聲調與解剖全整合版</title>
    <style>
        :root {
            --box-size: 220px;
            --cn-w: 165px;
            --zy-w: 55px;
            --font-cn: 110px;
            --font-zy: 36px;
            --font-tone: 24px;
            --primary: #2c3e50;
            --accent: #3498db;
            --map-width: 420px;
            --red-dot: #e74c3c;
        }

        body {
            background-color: #f4f7f6;
            display: flex;
            flex-direction: column;
            align-items: center;
            font-family: "標楷體", "DFKai-SB", serif;
            padding: 20px;
        }

        .learning-container {
            display: flex;
            gap: 20px;
            max-width: 1300px;
            width: 98%;
            margin: 0 auto;
            align-items: stretch;
        }

        .hotkey-hint { 
            margin-bottom: 20px; background: #fff; padding: 10px 20px; 
            border-radius: 30px; box-shadow: 0 2px 5px rgba(0,0,0,0.05);
            text-align: center; color: #666;
        }

        button {
            padding: 8px 25px; background: var(--primary); color: white;
            border: none; border-radius: 50px; cursor: pointer; transition: 0.3s;
        }

        .word-row { display: flex; gap: 20px; margin-bottom: 25px; }

        .char-unit {
            display: grid; grid-template-columns: var(--cn-w) var(--zy-w);
            width: var(--box-size); height: var(--box-size);
            background: white; border: 2px solid var(--primary); cursor: pointer;
        }

        .char-unit.active-trigger { border-color: var(--accent); box-shadow: 0 0 15px rgba(52, 152, 219, 0.3); }

        .cn-zone { display: flex; justify-content: center; align-items: center; font-size: var(--font-cn); border-right: 1px dashed #ccc; }
        .zy-compound { display: grid; grid-template-columns: 2fr 1fr; }
        .zy-symbols { display: flex; flex-direction: column; justify-content: center; align-items: center; font-size: var(--font-zy); line-height: 1.1; }
        .tone-zone { position: relative; }
        .tone-mark { position: absolute; top: 25%; left: 0; font-size: var(--font-tone); color: var(--accent); font-weight: bold; }

        .panel {
            flex: 1.2; background: white; padding: 25px; border-radius: 15px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.05); border-top: 6px solid var(--primary);
        }

        .image-map-wrapper {
            flex: 1; background: white; padding: 20px; border-radius: 15px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.05); border-top: 6px solid var(--accent);
        }

        .map-relative { position: relative; width: 100%; }
        
        .anatomy-dot {
            position: absolute; width: 24px; height: 24px;
            background: rgba(231, 76, 60, 0.4); border: 2px solid var(--red-dot);
            border-radius: 50%; transform: translate(-50%, -50%);
            animation: pulse-red 1s infinite; z-index: 10;
        }

        @keyframes pulse-red {
            0% { transform: translate(-50%, -50%) scale(1); opacity: 0.8; }
            50% { transform: translate(-50%, -50%) scale(1.4); opacity: 0.3; }
            100% { transform: translate(-50%, -50%) scale(1); opacity: 0.8; }
        }

        .tip-card { background: #fdfdfd; border-left: 5px solid #3498db; padding: 12px; margin-bottom: 10px; line-height: 1.6; }
        .tag { font-weight: bold; color: #2980b9; margin-right: 8px; background: #e1f5fe; padding: 2px 6px; border-radius: 4px; }

        .tone-visual-box {
            margin-top: 15px; padding: 15px; background: #f9f9f9; border-radius: 10px; text-align: center;
        }
    </style>
</head>
<body>

    <div class="hotkey-hint">
        <strong>教學系統</strong>：點擊國字同步顯示「描述、五度調值、發音部位」 <button onclick="refresh()">隨機更換單詞</button>
    </div>

    <div id="display" class="word-row"></div>

    <div class="learning-container">
        <div class="panel">
            <div id="title" style="font-size: 1.4rem; color: var(--accent); margin-bottom: 15px; border-bottom: 2px solid #eee;">請選擇單字</div>
            <div id="content"></div>
            
            <div id="toneSection" class="tone-visual-box" style="display:none;">
                <h4 style="margin:0 0 10px 0; color:#27ae60;">五度聲調標記</h4>
                <div style="display: flex; align-items: center; justify-content: center; gap: 15px;">
                    <div style="flex-direction: column-reverse; display: flex; justify-content: space-between; height: 120px; font-size: 11px; color: #888;">
                        <span>1</span><span>2</span><span>3</span><span>4</span><span>5</span>
                    </div>
                    <svg id="toneCanvas" width="160" height="120" viewBox="0 0 100 100" style="background: #fff; border-left: 2px solid #555; border-bottom: 2px solid #555;">
                        <path id="tonePath" d="" fill="none" stroke="#27ae60" stroke-width="6" stroke-linecap="round" />
                    </svg>
                </div>
                <div id="toneValueDisplay" style="color:#27ae60; font-weight:bold; margin-top:5px;"></div>
            </div>
        </div>

        <div class="image-map-wrapper">
            <h3 style="text-align:center; margin:0 0 10px 0; font-size:1.1rem;">調音部位 (連動顯示)</h3>
            <div class="map-relative">
                <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/7/75/Places_of_articulation.svg/500px-Places_of_articulation.svg.png" style="width:100%; height:auto;">
                <div id="dot-layer"></div>
            </div>
            <p id="part-name-display" style="font-size: 0.85rem; color: #e67e22; margin-top: 10px; text-align: center; height: 1.2em;"></p>
        </div>
    </div>

<script>
    const PartCoords = {
        1: {x: 8, y: 39, n: "外唇"}, 2: {x: 14, y: 44, n: "內唇"}, 3: {x: 22, y: 40, n: "牙齒"},
        4: {x: 27, y: 35, n: "齒齦"}, 5: {x: 34, y: 34, n: "齒齦後部"}, 6: {x: 41, y: 32, n: "硬腭前部"},
        7: {x: 52, y: 31, n: "硬腭"}, 8: {x: 71, y: 35, n: "軟腭"}, 9: {x: 78, y: 48, n: "小舌"},
        11: {x: 85, y: 92, n: "聲門"}, 14: {x: 62, y: 57, n: "舌面後"}, 15: {x: 42, y: 55, n: "舌面前"},
        17: {x: 17, y: 58, n: "舌尖"}
    };

    const MasterDictionary = {
        "ㄅ": { d: "【雙唇音】上下唇閉合阻礙氣流。", p: [1, 2] },
        "ㄆ": { d: "【雙唇音】伴隨強噴氣感。", p: [1, 2] },
        "ㄇ": { d: "【雙唇音】氣流由鼻腔出。", p: [1, 2, 8] },
        "ㄈ": { d: "【唇齒音】上齒觸下唇。", p: [1, 3] },
        "ㄉ": { d: "【舌尖中音】舌尖抵齒齦。", p: [4, 17] },
        "ㄋ": { d: "【鼻音】舌尖抵齒齦，鼻腔共鳴。", p: [4, 17, 8] },
        "ㄌ": { d: "【邊音】氣流由舌側流出。", p: [4, 17] },
        "ㄍ": { d: "【舌根音】舌後根抵軟顎。", p: [8, 14] },
        "ㄎ": { d: "【舌根音】強氣流舌根音。", p: [8, 14] },
        "ㄏ": { d: "【舌根音】摩擦音。", p: [8, 14] },
        "ㄒ": { d: "【舌面音】舌面靠近硬顎。", p: [6, 15] },
        "ㄓ": { d: "【翹舌音】舌尖上翹抵硬顎前。", p: [5, 17] },
        "ㄕ": { d: "【翹舌音】舌尖上翹摩擦。", p: [5, 17] },
        "ㄖ": { d: "【翹舌音】帶聲帶顫動。", p: [5, 17, 11] },
        "ㄧ": { d: "【齊齒】嘴角兩張。", p: [15] },
        "ㄨ": { d: "【合口】雙唇收圓。", p: [1, 2] },
        "ㄡ": { d: "【複韻母】由ㄛ滑向ㄨ。", p: [1, 2, 8] },
        "ㄥ": { d: "【鼻韻母】後鼻音。", p: [8, 14] },
        "ˇ": { d: "【上聲】先降後升。", p: [] },
        "ˊ": { d: "【陽平】中升音。", p: [] }
    };

    const toneMap = {
        "":  { val: "陰平 (55)", path: "M 0 0 L 100 0" },
        "ˊ": { val: "陽平 (35)", path: "M 0 50 L 100 0" },
        "ˇ": { val: "上聲 (214)", path: "M 0 75 Q 50 125 100 25" },
        "ˋ": { val: "去聲 (51)", path: "M 0 0 L 100 100" },
        "˙": { val: "輕聲", path: "M 50 50 m -2 0 a 2 2 0 1 0 4 0 a 2 2 0 1 0 -4 0" }
    };

    const wordLib = [
        { c: ["朋", "友"], z: ["ㄆㄥˊ", "ㄧㄡˇ"] },
        { c: ["老", "虎"], z: ["ㄌㄠˇ", "ㄏㄨˇ"] },
        { c: ["教", "學"], z: ["ㄐㄧㄠˋ", "ㄒㄩㄝˊ"] }
    ];

    let currentTask = null;

    function showDetail(index) {
        if (!currentTask) return;
        const char = currentTask.c[index];
        const pinyin = currentTask.z[index];

        document.querySelectorAll('.char-unit').forEach(el => el.classList.remove('active-trigger'));
        document.querySelectorAll('.char-unit')[index].classList.add('active-trigger');

        document.getElementById('title').innerText = `正在學習：「${char}」 (${pinyin})`;
        
        // 處理註釋與亮點
        const container = document.getElementById('content');
        const dotLayer = document.getElementById('dot-layer');
        container.innerHTML = "";
        dotLayer.innerHTML = "";
        let activeNames = new Set();

        [...pinyin].forEach(sym => {
            let s = (sym === "一") ? "ㄧ" : sym;
            if (MasterDictionary[s]) {
                container.innerHTML += `<div class="tip-card"><span class="tag">${s}</span> ${MasterDictionary[s].d}</div>`;
                MasterDictionary[s].p.forEach(pid => {
                    const c = PartCoords[pid];
                    activeNames.add(c.n);
                    const dot = document.createElement('div');
                    dot.className = 'anatomy-dot';
                    dot.style.left = c.x + '%'; dot.style.top = c.y + '%';
                    dotLayer.appendChild(dot);
                });
            }
        });
        document.getElementById('part-name-display').innerText = activeNames.size ? "啟動部位：" + [...activeNames].join("、") : "";

        // 處理聲調表 (恢復功能)
        const toneMark = pinyin.match(/[ˊˇˋ˙]/) ? pinyin.match(/[ˊˇˋ˙]/)[0] : "";
        const toneInfo = toneMap[toneMark];
        if (toneInfo) {
            document.getElementById('toneSection').style.display = "block";
            document.getElementById('tonePath').setAttribute('d', toneInfo.path);
            document.getElementById('toneValueDisplay').innerText = toneInfo.val;
        }
    }

    function refresh() {
        currentTask = wordLib[Math.floor(Math.random() * wordLib.length)];
        const display = document.getElementById('display');
        display.innerHTML = "";
        currentTask.c.forEach((char, i) => {
            const z = currentTask.z[i];
            const tone = z.match(/[ˊˇˋ˙]/) ? z.match(/[ˊˇˋ˙]/)[0] : "";
            const pure = z.replace(/[ˊˇˋ˙]/, "");
            const unit = document.createElement('div');
            unit.className = 'char-unit';
            unit.onclick = () => showDetail(i);
            unit.innerHTML = `<div class="cn-zone">${char}</div><div class="zy-compound"><div class="zy-symbols">${[...pure].join("<br>")}</div><div class="tone-zone"><span class="tone-mark">${tone}</span></div></div>`;
            display.appendChild(unit);
        });
        document.getElementById('toneSection').style.display = "none";
        document.getElementById('content').innerHTML = "";
        document.getElementById('dot-layer').innerHTML = "";
        document.getElementById('title').innerText = "請點擊單字...";
    }

    window.addEventListener('keydown', (e) => {
        if (e.code === 'Space') { e.preventDefault(); refresh(); }
        if (['1','2','3'].includes(e.key)) showDetail(parseInt(e.key)-1);
    });

    refresh();
</script>
</body>
</html>
