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
            --map-width: 350px;
        }

        body {
            background-color: #f4f7f6;
            display: flex;
            flex-direction: column;
            align-items: center;
            font-family: "標楷體", "DFKai-SB", serif;
            padding: 30px;
        }

        /* 佈局容器 */
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

        button:hover { background: #34495e; transform: scale(1.05); }

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

        .zy-compound {
            display: grid;
            grid-template-columns: 2fr 1fr;
        }

        .zy-symbols {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-size: var(--font-zy);
            line-height: 1.1;
        }

        .tone-zone { position: relative; }

        .tone-mark {
            position: absolute;
            top: 25%;
            left: 0;
            font-size: var(--font-tone);
            color: var(--accent);
            font-weight: bold;
        }

        .panel {
            flex: 1;
            background: white;
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            border-top: 6px solid var(--primary);
        }

        .image-map-wrapper {
            width: var(--map-width);
            background: white;
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            border-top: 6px solid var(--accent);
        }

        .anatomy-dot {
            position: absolute;
            width: 20px;
            height: 20px;
            background: rgba(231, 76, 60, 0.7);
            border: 2px solid #e74c3c;
            border-radius: 50%;
            transform: translate(-50%, -50%);
            pointer-events: none;
            animation: pulse-red 1.2s infinite;
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

        .warning-note {
            color: #e67e22;
            font-weight: bold;
            margin-top: 5px;
            display: block;
        }

        .tone-visual-box {
            margin-top: 20px;
            padding: 20px;
            background: #f9f9f9;
            border-radius: 10px;
            text-align: center;
        }
    </style>
</head>
<body>

    <div class="hotkey-hint">
        <strong>操作指引</strong>： 點擊下方國字查看註釋 <button onclick="refresh()">更換教學單詞</button> <br>
        <strong>⌨️快捷鍵</strong>： [空白鍵] 更換單詞 || [1] [2] [3] 快速查看
    </div>

    <div id="display" class="word-row"></div>

    <div class="learning-container">
        <div class="panel">
            <div id="title" class="active-title">請點擊單字開始學習</div>
            <div id="content"></div>
            
            <div id="toneSection" class="tone-visual-box" style="display:none;">
                <div style="display: flex; align-items: center; justify-content: center; gap: 15px;">
                    <div style="flex-direction: column-reverse; display: flex; justify-content: space-between; height: 160px; font-size: 12px; color: #888; padding-bottom: 30px;">
                        <span>1樓</span><span>2樓</span><span>3樓</span><span>4樓</span><span>5樓</span>
                    </div>
                    <div>
                        <svg id="toneCanvas" width="200" height="160" viewBox="0 0 100 100" style="background: #fff; border-left: 2px solid #555; border-bottom: 2px solid #555;">
                            <g fill="#ccc">
                                <circle cx="0" cy="0" r="1" /><circle cx="50" cy="0" r="1" /><circle cx="100" cy="0" r="1" />
                                <circle cx="0" cy="50" r="1" /><circle cx="50" cy="50" r="1" /><circle cx="100" cy="50" r="1" />
                                <circle cx="0" cy="100" r="1" /><circle cx="50" cy="100" r="1" /><circle cx="100" cy="100" r="1" />
                            </g>
                            <path id="tonePath" d="" fill="none" stroke="#27ae60" stroke-width="6" stroke-linecap="round" />
                        </svg>
                    </div>
                </div>
                <div id="toneValueDisplay" style="color:#27ae60; font-weight:bold; margin-top:10px;"></div>
            </div>
        </div>

        <div class="image-map-wrapper">
            <h3 style="text-align:center; margin-top:0; color:var(--primary);">發音部位導航</h3>
            <div style="position: relative;">
                <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/4a/Human_Vocal_Tract.svg/350px-Human_Vocal_Tract.svg.png" 
                     style="width:100%; height:auto; border-radius: 8px;">
                <div id="marker-layer"></div>
            </div>
            <p style="font-size: 0.85rem; color: #7f8c8d; margin-top: 10px; text-align: center;">
                ※ 點擊符號將顯示共鳴點位置
            </p>
        </div>
    </div>

<script>
    const MasterDictionary = {
        "ㄅ": "【雙唇音】上下唇先閉合，造成阻力後再讓氣流衝出。",
        "ㄆ": "【雙唇音】同ㄅ，但需伴隨較強的噴氣感。",
        "ㄇ": "【雙唇音】鼻音。雙唇閉合，氣流從鼻腔流出。",
        "ㄈ": "【唇齒音】上齒接觸下唇。注意：若沒碰到下唇會變成ㄏ。",
        "ㄉ": "【舌尖中音】舌尖抵住上齒齦，氣流突發而出。",
        "ㄊ": "【舌尖中音】同ㄉ，但氣流較強。",
        "ㄋ": "【鼻音】舌尖抵住上齒齦，氣流從鼻腔出。",
        "ㄌ": "【邊音】舌尖抵住上齒齦，氣流從舌頭兩側流出。",
        "ㄍ": "【舌根音】舌後根頂住軟顎，氣流突發而出。",
        "ㄎ": "【舌根音】同ㄍ，但氣流較強。",
        "ㄏ": "【舌根音】舌後根靠近軟顎，氣流摩擦而出。",
        "ㄐ": "【舌面音】舌面前部抵住硬顎前部。",
        "ㄑ": "【舌面音】同ㄐ，但氣流較強。",
        "ㄒ": "【舌面音】舌面前部靠近硬顎，氣流摩擦而出。",
        "ㄓ": "【翹舌音】舌尖上翹，抵住硬顎前部。",
        "ㄔ": "【翹舌音】同ㄓ，但氣流較強。",
        "ㄕ": "【翹舌音】舌尖上翹，靠近硬顎，氣流摩擦。",
        "ㄖ": "【翹舌音】發音位置同ㄕ，但聲帶顫動。",
        "ㄗ": "【平舌音】舌尖抵住上齒背。",
        "ㄘ": "【平舌音】同ㄗ，但氣流較強。",
        "ㄙ": "【平舌音】舌尖靠近上齒背，氣流摩擦。",
        "ㄧ": "【齊齒介符】嘴角向兩邊展開。",
        "ㄨ": "【合口介符】雙唇收圓向前突出。",
        "ㄩ": "【撮口介符】雙唇收圓成小孔狀。",
        "ㄚ": "【韻母】口腔大開，舌位低。",
        "ㄛ": "【韻母】圓唇，舌位中後。",
        "ㄜ": "【韻母】不圓唇，舌位中後。",
        "ㄝ": "【韻母】不圓唇，舌位中前。",
        "ㄞ": "【複韻母】由ㄚ滑向ㄧ。",
        "ㄟ": "【複韻母】由ㄝ滑向ㄧ。",
        "ㄠ": "【複韻母】由ㄚ滑向ㄨ。",
        "ㄡ": "【複韻母】由ㄛ滑向ㄨ。",
        "ㄢ": "【鼻韻母】前鼻音，舌尖抵住齒齦結束。",
        "ㄣ": "【鼻韻母】前鼻音，發音短促。",
        "ㄤ": "【鼻韻母】後鼻音，舌根抵住軟顎結束。",
        "ㄥ": "【鼻韻母】後鼻音，共鳴在後方。",
        "ㄦ": "【捲舌韻母】舌位在中，舌尖捲起。"
    };

    const wordLib = [
        { c: ["老", "虎"], z: ["ㄌㄠˇ", "ㄏㄨˇ"] },
        { c: ["發", "風"], z: ["ㄈㄚ", "ㄈㄥ"] },
        { c: ["朋", "友"], z: ["ㄆㄥˊ", "ㄧㄡˇ"] },
        { c: ["咖", "啡"], z: ["ㄎㄚ", "ㄈㄟ"] },
        { c: ["歡", "迎"], z: ["ㄏㄨㄢ", "ㄧㄥˊ"] }
    ];

    let currentTask = null;
    const toneMap = {
        "":  { val: "陰平", path: "M 0 0 L 100 0" }, 
        "ˊ": { val: "陽平", path: "M 0 50 L 100 0" }, 
        "ˇ": { val: "上聲", path: "M 0 75 Q 50 125 100 25" }, 
        "ˋ": { val: "去聲", path: "M 0 0 L 100 100" }, 
        "˙": { val: "輕聲", path: "M 50 50 m -2 0 a 2 2 0 1 0 4 0 a 2 2 0 1 0 -4 0" } 
    };

    function showDetail(index) {
        if (!currentTask || !currentTask.c[index]) return;
        const char = currentTask.c[index];
        const pinyin = currentTask.z[index];

        document.querySelectorAll('.char-unit').forEach(el => el.classList.remove('active-trigger'));
        const target = document.querySelectorAll('.char-unit')[index];
        if (target) target.classList.add('active-trigger');

        document.getElementById('title').innerText = `正在學習：「${char}」 (${pinyin})`;
        const container = document.getElementById('content');
        container.innerHTML = "";
        
        [...pinyin].forEach(sym => {
            let cleanSym = (sym === "一") ? "ㄧ" : sym;
            if (MasterDictionary[cleanSym]) {
                container.innerHTML += `<div class="tip-card"><span class="tag">${cleanSym}</span> ${MasterDictionary[cleanSym]}</div>`;
            }
        });

        const toneMark = pinyin.match(/[ˊˇˋ˙]/) ? pinyin.match(/[ˊˇˋ˙]/)[0] : ""; 
        const toneInfo = toneMap[toneMark];
        if (toneInfo) {
            document.getElementById('toneSection').style.display = "block";
            document.getElementById('tonePath').setAttribute('d', toneInfo.path);
            document.getElementById('toneValueDisplay').innerText = toneInfo.val;
        }
    }

    function refresh() {
        document.getElementById('title').innerText = "請點擊單字開始學習";
        document.getElementById('content').innerHTML = "";
        document.getElementById('toneSection').style.display = "none";
        
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
