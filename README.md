<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>注音精密教學系統</title>
    <style>
        :root {
            --box-size: 220px;
            --cn-w: 165px;
            --zy-w: 55px;
            --font-cn: 110px;
            --font-zy: 36px;
            --font-tone: 24px;
            --primary: #2c3e50;
            --accent: #27ae60;
        }
        body {
            background-color: #f4f7f6;
            display: flex;
            flex-direction: column;
            align-items: center;
            font-family: "標楷體", "DFKai-SB", serif;
            padding: 30px;
            margin: 0;
        }
        .hotkey-hint { 
            margin-bottom: 20px; 
            background: #fff; 
            padding: 10px 20px; 
            border-radius: 30px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.05);
            font-size: 0.95rem;
            color: #666;
        }
        button {
            padding: 12px 45px;
            font-size: 1.2rem;
            background: var(--primary);
            color: white;
            border: none;
            border-radius: 50px;
            cursor: pointer;
            margin-bottom: 25px;
            transition: 0.3s;
        }
        button:hover { background: #34495e; transform: scale(1.05); }
        .word-row {
            display: flex;
            gap: 20px;
            margin-bottom: 40px;
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
            box-shadow: 0 0 20px rgba(39, 174, 96, 0.2);
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
            max-width: 850px;
            width: 100%;
            background: white;
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            border-top: 6px solid var(--primary);
        }
        .active-title {
            font-size: 1.6rem;
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
        .tone-visual-box {
            margin-top: 20px;
            padding: 20px;
            background: #fff;
            border: 1px solid #eee;
            border-radius: 10px;
            text-align: center;
        }
    </style>
</head>
<body>
    <div class="hotkey-hint">
        ⌨️ <strong>快捷鍵操作</strong>： [空白鍵] 更換單詞 | [1] [2] [3] 查看單字註釋
    </div>

    <button onclick="refresh()">更換教學單詞</button>

    <div id="display" class="word-row"></div>

    <div class="panel">
        <div id="title" class="active-title">點擊單字開始學習</div>
        <div id="content"></div>
        <div id="toneSection" class="tone-visual-box" style="display:none;">
            <div style="display: flex; align-items: center; justify-content: center; gap: 15px;">
                <div style="display: flex; flex-direction: column-reverse; justify-content: space-between; height: 160px; font-size: 12px; color: #888; padding-bottom: 30px;">
                    <span>1樓</span><span>2樓</span><span>3樓</span><span>4樓</span><span>5樓</span>
                </div>
                <div style="display: flex; flex-direction: column; align-items: center;">
                    <svg id="toneCanvas" width="200" height="160" viewBox="0 0 100 100" style="background: #fff; border-left: 2px solid #555; border-bottom: 2px solid #555;">
                        <g fill="#ccc">
                            <circle cx="0" cy="0" r="1"/><circle cx="50" cy="0" r="1"/><circle cx="100" cy="0" r="1"/>
                            <circle cx="0" cy="25" r="1"/><circle cx="50" cy="25" r="1"/><circle cx="100" cy="25" r="1"/>
                            <circle cx="0" cy="50" r="1"/><circle cx="50" cy="50" r="1"/><circle cx="100" cy="50" r="1"/>
                            <circle cx="0" cy="75" r="1"/><circle cx="50" cy="75" r="1"/><circle cx="100" cy="75" r="1"/>
                            <circle cx="0" cy="100" r="1"/><circle cx="50" cy="100" r="1"/><circle cx="100" cy="100" r="1"/>
                        </g>
                        <path id="tonePath" d="" fill="none" stroke="#27ae60" stroke-width="6" stroke-linecap="round" stroke-linejoin="round"/>
                    </svg>
                    <div style="display: flex; justify-content: space-between; width: 200px; font-size: 12px; color: #888; padding-top: 8px;">
                        <span>1拍</span><span>3拍</span><span>5拍</span>
                    </div>
                </div>
            </div>
            <div id="toneValueDisplay" style="color:#27ae60; font-weight:bold; margin-top:15px; font-size: 1.3rem;"></div>
        </div>
    </div>

    <script>
        const MasterDictionary = {
            "ㄅ": "【雙唇音】上下唇先閉合，造成阻力後再讓氣流衝出。",
            "ㄆ": "【雙唇音】送氣音。氣流強力衝出。",
            "ㄇ": "【雙唇音】鼻音。氣流從鼻腔流出。",
            "ㄈ": "【唇齒音】上齒接觸下唇。",
            "ㄉ": "【舌尖中音】舌尖抵住上齒齦。",
            "ㄊ": "【舌尖中音】送氣音。",
            "ㄋ": "【鼻音】共鳴在鼻腔。",
            "ㄌ": "【邊音】氣流從舌頭兩側流出。",
            "ㄍ": "【舌根音】舌後根頂住軟顎。",
            "ㄎ": "【舌根音】送氣音。",
            "ㄏ": "【舌根音】舌後根靠近軟顎。",
            "ㄐ": "【舌面音】舌面前部貼住硬顎前部。",
            "ㄑ": "【舌面音】送氣音。",
            "ㄒ": "【舌面音】摩擦發音。",
            "ㄓ": "【翹舌音】舌尖上捲。",
            "ㄔ": "【翹舌音】送氣音。",
            "ㄕ": "【翹舌音】摩擦發音。",
            "ㄖ": "【翹舌音】帶有震動感。",
            "ㄗ": "【平舌音】舌尖抵住下齒背。",
            "ㄘ": "【平舌音】送氣音。",
            "ㄙ": "【平舌音】摩擦發音。",
            "ㄧ": "【介符】嘴角往兩旁拉開。",
            "ㄨ": "【介符】雙唇收圓向前突。",
            "ㄩ": "【介符】嘴唇用力向中間收攏。",
            "ㄚ": "【韻母】嘴巴徹底打開。",
            "ㄛ": "【單韻母】圓唇口型。",
            "ㄜ": "【單韻母】鴨蛋形長橢圓口型。",
            "ㄝ": "【單韻母】口型圓滿。",
            "ㄞ": "【複韻母】由ㄚ滑向ㄧ。",
            "ㄟ": "【複韻母】由ㄝ滑向ㄧ。",
            "ㄠ": "【複韻母】由ㄚ滑向ㄨ。",
            "ㄡ": "【複韻母】由ㄛ滑向ㄨ。",
            "ㄢ": "【前鼻音】氣流在鼻腔前部。",
            "ㄣ": "【鼻韻母】共鳴在鼻腔前部。",
            "ㄤ": "【後鼻音】舌後根頂住軟顎。",
            "ㄥ": "【鼻韻母】共鳴在鼻腔後部。",
            "ㄦ": "【特殊韻母】捲舌音。"
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
                container.innerHTML += `<div class="tip-card" style="border-left-color: #27ae60;">這是「${toneInfo.val}」，請觀察下方的聲音走勢。</div>`;
                document.getElementById('toneSection').style.display = "block";
                const pathElement = document.getElementById('tonePath');
                pathElement.setAttribute('d', toneInfo.path);
                pathElement.setAttribute('fill', toneMark === "˙" ? "#27ae60" : "none");
                document.getElementById('toneValueDisplay').innerText = toneInfo.val;
            }
        }

        function refresh() {
            document.getElementById('title').innerText = "點擊單字開始學習";
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
