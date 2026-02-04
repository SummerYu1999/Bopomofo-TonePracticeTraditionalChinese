<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>注音精密教學系統 - 專業全解析版</title>
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
            --map-width: 450px; /* 稍微調寬以利觀察地圖 */
            --red-dot: #e74c3c;
        }

        body {
            background-color: #f4f7f6;
            display: flex;
            flex-direction: column;
            align-items: center;
            font-family: "標楷體", "DFKai-SB", serif;
            padding: 30px;
        }

        .learning-container {
            display: flex;
            gap: 25px;
            max-width: 1250px;
            width: 95%;
            margin: 0 auto;
            align-items: flex-start;
            justify-content: center;
        }

        .hotkey-hint { 
            margin-bottom: 20px; 
            background: #fff; 
            padding: 10px 20px; 
            border-radius: 30px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.05);
            font-size: 1.1rem;
            color: #666;
            text-align: center;
        }

        button {
            padding: 8px 25px;
            font-size: 1rem;
            background: var(--primary);
            color: white;
            border: none;
            border-radius: 50px;
            cursor: pointer;
            margin-left: 10px;
            transition: 0.3s;
        }

        .word-row {
            display: flex;
            gap: 20px;
            margin-bottom: 30px;
            justify-content: center;
        }

        .char-unit {
            display: grid;
            grid-template-columns: var(--cn-w) var(--zy-w);
            width: var(--box-size);
            height: var(--box-size);
            background: white;
            border: 2px solid var(--primary);
            cursor: pointer;
            box-sizing: border-box;
            transition: all 0.2s;
        }

        .char-unit.active-trigger {
            border-color: var(--accent);
            box-shadow: 0 0 20px rgba(52, 152, 219, 0.25);
            transform: translateY(-5px);
        }

        .cn-zone {
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: var(--font-cn);
            border-right: 1px dashed #ccc;
        }

        .zy-compound { display: grid; grid-template-columns: 2fr 1fr; }
        .zy-symbols { display: flex; flex-direction: column; justify-content: center; align-items: center; font-size: var(--font-zy); line-height: 1.1; }
        .tone-zone { position: relative; }
        .tone-mark { position: absolute; top: 25%; left: 0; font-size: var(--font-tone); color: var(--accent); font-weight: bold; }

        .panel {
            flex: 1;
            background: white;
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            border-top: 6px solid var(--primary);
            min-height: 450px;
        }

        .image-map-wrapper {
            width: var(--map-width);
            background: white;
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            border-top: 6px solid var(--accent);
            position: relative;
        }

        /* 亮點容器 */
        .map-relative { position: relative; width: 100%; }
        
        .anatomy-dot {
            position: absolute;
            width: 28px;
            height: 28px;
            background: rgba(231, 76, 60, 0.4);
            border: 2px solid var(--red-dot);
            border-radius: 50%;
            transform: translate(-50%, -50%);
            pointer-events: none;
            animation: pulse-red 1s infinite;
            z-index: 10;
        }

        @keyframes pulse-red {
            0% { transform: translate(-50%, -50%) scale(1); opacity: 0.8; }
            50% { transform: translate(-50%, -50%) scale(1.5); opacity: 0.3; }
            100% { transform: translate(-50%, -50%) scale(1); opacity: 0.8; }
        }

        .active-title {
            font-size: 1.5rem;
            color: var(--accent);
            margin-bottom: 20px;
            border-bottom: 2px solid #eee;
            padding-bottom: 10px;
        }

        .tip-card {
            background: #fdfdfd;
            border-left: 5px solid #3498db;
            padding: 15px;
            margin-bottom: 15px;
            line-height: 1.8;
        }

        .tag {
            font-weight: bold;
            color: #2980b9;
            margin-right: 10px;
            background: #e1f5fe;
            padding: 2px 8px;
            border-radius: 4px;
        }
    </style>
</head>
<body>

    <div class="hotkey-hint">
        <strong>操作指引</strong>： 點擊下方國字查看註釋與發音部位 <button onclick="refresh()">更換教學單詞</button> <br>
        <strong>⌨️快捷鍵</strong>： [空白鍵] 更換單詞 || [1] [2] [3] 快速查看
    </div>

    <div id="display" class="word-row"></div>

    <div class="learning-container">
        <div class="panel">
            <div id="title" class="active-title">請點擊單字開始學習</div>
            <div id="content"></div>
        </div>

        <div class="image-map-wrapper">
            <h3 style="text-align:center; margin-top:0; color:var(--primary);">調音部位導航 (連動顯示)</h3>
            <div class="map-relative">
                <img id="main-map" src="https://upload.wikimedia.org/wikipedia/commons/thumb/7/75/Places_of_articulation.svg/500px-Places_of_articulation.svg.png" style="width:100%; height:auto; border-radius: 8px;">
                <div id="dot-layer"></div>
            </div>
            <p id="part-name-display" style="font-size: 0.9rem; color: #e67e22; margin-top: 10px; text-align: center; font-weight: bold;">
                </p>
        </div>
    </div>

<script>
    // 座標映射表 (依據 500px 比例估算百分比)
    const PartCoords = {
        1: {x: 8, y: 39, n: "外唇"}, 2: {x: 14, y: 44, n: "內唇"}, 3: {x: 22, y: 40, n: "牙齒"},
        4: {x: 27, y: 35, n: "齒齦"}, 5: {x: 34, y: 34, n: "齒齦後部"}, 6: {x: 41, y: 32, n: "硬腭前部"},
        7: {x: 52, y: 31, n: "硬腭"}, 8: {x: 71, y: 35, n: "軟腭"}, 9: {x: 78, y: 48, n: "小舌"},
        10: {x: 84, y: 64, n: "咽腔壁"}, 11: {x: 85, y: 92, n: "聲門"}, 12: {x: 76, y: 83, n: "會厭"},
        13: {x: 71, y: 67, n: "舌根"}, 14: {x: 62, y: 57, n: "舌面後"}, 15: {x: 42, y: 55, n: "舌面前"},
        16: {x: 22, y: 55, n: "舌葉"}, 17: {x: 17, y: 58, n: "舌尖"}, 18: {x: 21, y: 65, n: "舌尖下部"}
    };

    const MasterDictionary = {
        "ㄅ": { d: "【雙唇音】上下唇先閉合，造成阻力後再讓氣流衝出。", p: [1, 2] },
        "ㄆ": { d: "【雙唇音】同ㄅ，但需伴隨較強的噴氣感。", p: [1, 2] },
        "ㄇ": { d: "【雙唇音】鼻音。雙唇閉合，氣流從鼻腔流出。", p: [1, 2, 8] },
        "ㄈ": { d: "【唇齒音】上齒接觸下唇。注意：若沒碰到下唇會變成ㄏ。", p: [1, 3] },
        "ㄉ": { d: "【舌尖中音】舌尖抵住上齒齦，氣流突發而出。", p: [4, 17] },
        "ㄊ": { d: "【舌尖中音】同ㄉ，但氣流較強。", p: [4, 17] },
        "ㄋ": { d: "【鼻音】舌尖抵住上齒齦，氣流從鼻腔出。", p: [4, 17, 8] },
        "ㄌ": { d: "【邊音】舌尖抵住上齒齦，氣流從舌頭兩側流出。", p: [4, 17] },
        "ㄍ": { d: "【舌根音】舌後根頂住軟顎，氣流突發而出。", p: [8, 14] },
        "ㄎ": { d: "【舌根音】同ㄍ，但氣流較強。", p: [8, 14] },
        "ㄏ": { d: "【舌根音】舌後根靠近軟顎，氣流摩擦而出。", p: [8, 14] },
        "ㄐ": { d: "【舌面音】舌面前部抵住硬顎前部。", p: [6, 15] },
        "ㄑ": { d: "【舌面音】同ㄐ，但氣流較強。", p: [6, 15] },
        "ㄒ": { d: "【舌面音】舌面前部靠近硬顎，氣流摩擦。", p: [6, 15] },
        "ㄓ": { d: "【翹舌音】舌尖上翹，抵住硬顎前部。", p: [5, 17] },
        "ㄔ": { d: "【翹舌音】同ㄓ，但氣流較強。", p: [5, 17] },
        "ㄕ": { d: "【翹舌音】舌尖上翹，靠近硬顎。", p: [5, 17] },
        "ㄖ": { d: "【翹舌音】位置同ㄕ，但帶聲帶顫動。", p: [5, 17, 11] },
        "ㄗ": { d: "【平舌音】舌尖抵住上齒背。", p: [3, 17] },
        "ㄘ": { d: "【平舌音】同ㄗ，但氣流較強。", p: [3, 17] },
        "ㄙ": { d: "【平舌音】舌尖靠近上齒背。", p: [3, 17] },
        "ㄧ": { d: "【齊齒介符】嘴角向兩邊展開。", p: [15] },
        "ㄨ": { d: "【合口介符】雙唇收圓向前突出。", p: [1, 2] },
        "ㄩ": { d: "【撮口介符】雙唇收圓成小孔狀。", p: [1, 2, 15] },
        "ㄚ": { d: "【韻母】口腔大開，舌位低。", p: [10, 14] },
        "ㄛ": { d: "【韻母】圓唇，舌位中後。", p: [1, 2, 8] },
        "ㄜ": { d: "【韻母】不圓唇，舌位中後。", p: [8, 14] },
        "ㄝ": { d: "【韻母】不圓唇，舌位中前。", p: [7, 15] },
        "ㄢ": { d: "【前鼻音】舌尖抵住齒齦結束。", p: [4, 17, 8] },
        "ㄤ": { d: "【後鼻音】舌根抵住軟顎結束。", p: [8, 14] },
        "ㄦ": { d: "【捲舌韻母】舌位在中，舌尖捲起。", p: [5, 17] }
    };

    const wordLib = [
        { c: ["老", "虎"], z: ["ㄌㄠˇ", "ㄏㄨˇ"] },
        { c: ["發", "風"], z: ["ㄈㄚ", "ㄈㄥ"] },
        { c: ["朋", "友"], z: ["ㄆㄥˊ", "ㄧㄡˇ"] },
        { c: ["注", "音"], z: ["ㄓㄨˋ", "ㄧㄣ"] },
        { c: ["教", "學"], z: ["ㄐㄧㄠˋ", "ㄒㄩㄝˊ"] }
    ];

    let currentTask = null;

    function showDetail(index) {
        if (!currentTask || !currentTask.c[index]) return;
        const char = currentTask.c[index];
        const pinyin = currentTask.z[index];

        document.querySelectorAll('.char-unit').forEach(el => el.classList.remove('active-trigger'));
        const target = document.querySelectorAll('.char-unit')[index];
        if (target) target.classList.add('active-trigger');

        document.getElementById('title').innerText = `正在學習：「${char}」 (${pinyin})`;
        const container = document.getElementById('content');
        const dotLayer = document.getElementById('dot-layer');
        const nameDisplay = document.getElementById('part-name-display');
        
        container.innerHTML = "";
        dotLayer.innerHTML = "";
        let activePartNames = new Set();

        [...pinyin].forEach(sym => {
            let cleanSym = (sym === "一") ? "ㄧ" : sym;
            const entry = MasterDictionary[cleanSym];
            if (entry) {
                container.innerHTML += `<div class="tip-card"><span class="tag">${cleanSym}</span> ${entry.d}</div>`;
                // 產生亮點
                entry.p.forEach(pid => {
                    const coord = PartCoords[pid];
                    activePartNames.add(`${pid}.${coord.n}`);
                    const dot = document.createElement('div');
                    dot.className = 'anatomy-dot';
                    dot.style.left = coord.x + '%';
                    dot.style.top = coord.y + '%';
                    dotLayer.appendChild(dot);
                });
            }
        });
        nameDisplay.innerText = "啟用部位：" + Array.from(activePartNames).join("、");
    }

    function refresh() {
        document.getElementById('title').innerText = "請點擊單字開始學習";
        document.getElementById('content').innerHTML = "";
        document.getElementById('dot-layer').innerHTML = "";
        document.getElementById('part-name-display').innerText = "";
        
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
    }

    window.addEventListener('keydown', (e) => {
        if (e.code === 'Space') { e.preventDefault(); refresh(); }
        if (['1','2','3'].includes(e.key)) showDetail(parseInt(e.key)-1);
    });

    refresh();
</script>
</body>
</html>
