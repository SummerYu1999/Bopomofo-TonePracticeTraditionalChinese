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

        /* 這裡也建議順便貼上地圖與紅點的關鍵設定 */
        .learning-container {
            display: flex;
            gap: 25px;
            max-width: 1250px;
            margin: 0 auto;
            align-items: flex-start;
        }
        .image-map-wrapper {
            position: relative;
            width: var(--map-width);
            background: white;
            padding: 15px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            border-top: 6px solid var(--primary);
        }
        .pronounce-dot {
            position: absolute;
            width: 22px;
            height: 22px;
            background: rgba(52, 152, 219, 0.4);
            border: 2px solid var(--accent);
            border-radius: 50%;
            transform: translate(-50%, -50%);
            pointer-events: none;
            animation: pulse-blue 1.2s infinite;
            z-index: 100; 
        }
        @keyframes pulse-blue {
            0% { transform: translate(-50%, -50%) scale(1); opacity: 0.8; }
            50% { transform: translate(-50%, -50%) scale(1.5); opacity: 0.3; }
            100% { transform: translate(-50%, -50%) scale(1); opacity: 0.8; }
        }

        body {
            background-color: #f4f7f6;
            display: flex;
            flex-direction: column;
            align-items: center;
            font-family: "標楷體", "DFKai-SB", serif;
            padding: 30px;
        }

        .hotkey-hint { 
            margin-bottom: 20px; 
            background: #fff; 
            padding: 10px 20px; 
            border-radius: 30px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.05);
            font-size: 1.3rem;
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
            box-shadow: 0 0 20px rgba(214, 48, 49, 0.25);
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

        .warning-note {
            color: #e67e22;
            font-weight: bold;
            margin-top: 5px;
            display: block;
        }
        .tone-visual-box {
    margin-top: 20px;
    padding: 20px;
    background: #fff;
    border: 1px solid #eee;
    border-radius: 10px;
    text-align: center;
}

.pronounce-dot {
            position: absolute;
            width: 22px;
            height: 22px;
            background: rgba(52, 152, 219, 0.4);
            border: 2px solid var(--accent);
            border-radius: 50%;
            transform: translate(-50%, -50%);
            pointer-events: none;
            animation: pulse-blue 1.2s infinite;
            z-index: 100;
        }

        .learning-container {
            display: flex;
            flex-direction: row; 
            gap: 25px;
            align-items: flex-start;
        }

        .map-relative {
            position: relative;
            display: inline-block;
            width: 100%;
        }

    </style>
</head>
<body>

    <div class="hotkey-hint">
        <strong>操作指引</strong>： 點擊右方按鈕替換單字 <strong>||<strong> 點擊國字查看單字註釋  <button onclick="refresh()">更換教學單詞</button> <br> <strong>⌨️快捷鍵</strong>： [空白鍵] 更換單詞 <strong>||<strong> [1] [2] [3] 查看單字註釋

    <div id="display" class="word-row"></div>
<div class="learning-container">
    <div class="panel">
        <div id="title" class="active-title">點擊單字或按數字鍵開始學習</div>
        <div id="content"></div>
        <div id="toneSection" class="tone-visual-box" style="display:none;">
    <div style="display: flex; align-items: center; justify-content: center; gap: 15px;">
        <div style="display: flex; flex-direction: column-reverse; justify-content: space-between; height: 160px; font-size: 12px; color: #888; padding-bottom: 30px;">
            <span>1樓</span><span>2樓</span><span>3樓</span><span>4樓</span><span>5樓</span>
        </div>
        
        <div style="display: flex; flex-direction: column; align-items: center;">
            <svg id="toneCanvas" width="200" height="160" viewBox="0 0 100 100" style="background: #fff; border-left: 2px solid #555; border-bottom: 2px solid #555;">
                <g id="gridPoints" fill="#ccc">
                    <circle cx="0" cy="0" r="1" /><circle cx="25" cy="0" r="1" /><circle cx="50" cy="0" r="1" /><circle cx="75" cy="0" r="1" /><circle cx="100" cy="0" r="1" />
                    <circle cx="0" cy="25" r="1" /><circle cx="25" cy="25" r="1" /><circle cx="50" cy="25" r="1" /><circle cx="75" cy="25" r="1" /><circle cx="100" cy="25" r="1" />
                    <circle cx="0" cy="50" r="1" /><circle cx="25" cy="50" r="1" /><circle cx="50" cy="50" r="1" /><circle cx="75" cy="50" r="1" /><circle cx="100" cy="50" r="1" />
                    <circle cx="0" cy="75" r="1" /><circle cx="25" cy="75" r="1" /><circle cx="50" cy="75" r="1" /><circle cx="75" cy="75" r="1" /><circle cx="100" cy="75" r="1" />
                    <circle cx="0" cy="100" r="1" /><circle cx="25" cy="100" r="1" /><circle cx="50" cy="100" r="1" /><circle cx="75" cy="100" r="1" /><circle cx="100" cy="100" r="1" />
                </g>
                <path id="tonePath" d="" fill="none" stroke="#27ae60" stroke-width="6" stroke-linecap="round" stroke-linejoin="round" />
            </svg>
            <div style="display: flex; justify-content: space-between; width: 200px; font-size: 12px; color: #888; padding-top: 8px;">
                <span>1拍</span><span>2拍</span><span>3拍</span><span>4拍</span><span>5拍</span>
            </div>
        </div>
    </div>
    <div id="toneValueDisplay" style="color:#27ae60; font-weight:bold; margin-top:15px; font-size: 1.3rem;"></div>
</div>
    <div class="image-map-wrapper">
        <h3 style="text-align:center; margin-top:0; color:var(--primary); font-size: 1.2rem;">發音部位</h3>
        <div class="map-relative">
            <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/7/75/Places_of_articulation.svg/500px-Places_of_articulation.svg.png" style="width:100%; height:auto; display:block;">
            <div id="dot-layer"></div>
        </div>
        <div id="location-label" style="text-align:center; margin-top:10px; color:var(--accent); font-weight:bold; min-height:1.2em;"></div>
    </div>
    </div>

<script>
    /**
     * 【專業精密字典】 - 整合發音細節與編碼修正
     * 座標說明：x, y 為地圖圖片上的百分比位置
     */
/* --- 1. 座標庫：存放你精準微調後的數據 --- */
const PronounceMap = {
    "1": { name: "上唇", x: 10.1, y: 40.8 },
    "1.1": { name: "下唇", x: 6, y: 71.7 },
    "2": { name: "上齒", x: 16, y: 44.8 },
    "2.1": { name: "下齒", x: 11.6, y: 65.5 },
    "3": { name: "上齒齦", x: 23.1, y: 44 },
    "4": { name: "齒齦後", x: 28.4, y: 40.8 },
    "5": { name: "齒齦後部", x: 34.2, y: 39.4 },
    "6": { name: "硬腭前部", x: 39.2, y: 36.8 },
    "7": { name: "硬腭", x: 52.2, y: 38 },
    "8": { name: "軟腭", x: 65, y: 40.2 },
    "9": { name: "小舌", x: 71.8, y: 49.4 },
    "10": { name: "咽腔壁", x: 83.7, y: 66.4 },
    "11": { name: "聲門", x: 85.5, y: 91.2 },
    "12": { name: "會厭", x: 73.5, y: 76.2 },
    "13": { name: "舌根", x: 67.7, y: 68.7 },
    "14": { name: "舌面後", x: 55.6, y: 56.6 },
    "15": { name: "舌面前", x: 35.2, y: 55.2 },
    "16": { name: "舌葉", x: 18.7, y: 56.6 },
    "17": { name: "舌尖", x: 14.2, y: 60.8 },
    "18": { name: "舌尖下部", x: 20.8, y: 64.2 },
};

/* --- 2. 升級版 MasterDictionary：引用上面的座標庫 --- */
const MasterDictionary = {
    // 唇音類
    "ㄅ": { desc: "【雙唇音】上下唇先閉合，造成阻力後再讓氣流衝出。", pos: PronounceMap["1"], loc: "雙唇" },
    "ㄆ": { desc: "【雙唇音】發音部位與ㄅ同，但送出強烈氣流。", pos: PronounceMap["1"], loc: "雙唇" },
    "ㄇ": { desc: "【雙唇音】雙唇閉合，氣流由鼻腔流出。", pos: PronounceMap["1.1"], loc: "雙唇/鼻腔" },
    "ㄈ": { desc: "【唇齒音】上齒接觸下唇。<span class='warning-note'>注意：若沒碰到下唇會變成「ㄏ」的模糊音。</span>", pos: PronounceMap["2"], loc: "上齒與下唇" },

    // 舌尖與舌根
    "ㄉ": { desc: "【舌尖中音】舌尖抵住上齒齦，阻礙氣流後突然放開。", pos: PronounceMap["3"], loc: "舌尖與上齒齦" },
    "ㄊ": { desc: "【舌尖中音】與ㄉ同，但送出強烈氣流。", pos: PronounceMap["3"], loc: "舌尖與上齒齦" },
    "ㄋ": { desc: "【鼻音】舌尖抵住上齒齦，共鳴在鼻腔。", pos: PronounceMap["3"], loc: "鼻腔共鳴" },
    "ㄌ": { desc: "【邊音】舌尖抵住上齒齦，氣流從舌頭兩邊流出。", pos: PronounceMap["3"], loc: "舌尖齒齦" },
    "ㄍ": { desc: "【舌根音】舌後根頂住軟顎。<span class='warning-note'>細節：嘴巴需打開約一個指節寬。</span>", pos: PronounceMap["8"], loc: "舌根與軟顎" },
    "ㄎ": { desc: "【舌根音】與ㄍ同，但送出強烈氣流。", pos: PronounceMap["8"], loc: "舌根與軟顎" },
    "ㄏ": { desc: "【舌根音】舌根靠近軟顎，氣流由縫隙摩擦而出。", pos: PronounceMap["8"], loc: "舌根與軟顎" },

    // 舌面與平翹舌
    "ㄐ": { desc: "【舌面前音】舌面前部抬起抵住硬腭，氣流爆發摩擦而成。", pos: PronounceMap["7"], loc: "舌面與硬顎" },
    "ㄑ": { desc: "【舌面前音】與ㄐ同，但送出強烈氣流。", pos: PronounceMap["7"], loc: "舌面與硬顎" },
    "ㄒ": { desc: "【舌面前音】舌面前部靠近硬腭，氣流摩擦而出。", pos: PronounceMap["7"], loc: "舌面與硬顎" },
    "ㄓ": { desc: "【翹舌音】舌尖上翹抵住硬腭前部，氣流爆發摩擦而成。", pos: PronounceMap["6"], loc: "舌尖與硬顎" },
    "ㄔ": { desc: "【翹舌音】與ㄓ同，但送出強烈氣流。", pos: PronounceMap["6"], loc: "舌尖與硬顎" },
    "ㄕ": { desc: "【翹舌音】舌尖上翹靠近硬腭前部，氣流摩擦而出。", pos: PronounceMap["6"], loc: "舌尖與硬顎" },
    "ㄖ": { desc: "【翹舌音】舌尖垂直放，不可碰到牙齒。", pos: PronounceMap["6"], loc: "舌尖與硬顎" },
    "ㄗ": { desc: "【平舌音】舌尖往前下靠近下齒背，舌面不可碰到上齒背。", pos: PronounceMap["2"], loc: "舌尖與齒背" },
    "ㄘ": { desc: "【平舌音】與ㄗ同，但送出強烈氣流。", pos: PronounceMap["2"], loc: "舌尖與齒背" },
    "ㄙ": { desc: "【平舌音】舌尖靠近上齒背，氣流摩擦而出。", pos: PronounceMap["2"], loc: "舌尖與齒背" },

    // 介符
    "ㄧ": { desc: "【介符/齊齒】嘴角往兩旁拉開做出「一」字形，有助於口型擺動。", pos: PronounceMap["15"], loc: "舌面前部" },
    "ㄨ": { desc: "【介符/合口】雙唇收圓向前突。規律：ㄅㄆㄇㄈ不與ㄨ結合。", pos: PronounceMap["13"], loc: "舌根後縮" },
    "ㄩ": { desc: "【介符/撮口】嘴唇需用力向中間收攏成圓孔狀。", pos: PronounceMap["1"], loc: "雙唇撮攏" },

    // 韻母
    "ㄚ": { desc: "【韻母】嘴巴要徹底打開，舌頭下沉。", pos: PronounceMap["14"], loc: "口腔後部下沉" },
    "ㄛ": { desc: "【單韻母】圓唇口型。注意維持口型純淨。", pos: PronounceMap["14"], loc: "口腔後部" },
    "ㄜ": { desc: "【單韻母】鴨蛋形長橢圓口型，舌面往下拉壓。", pos: PronounceMap["14"], loc: "舌面中後" },
    "ㄝ": { desc: "【單韻母】口型要圓滿，確保足夠空間。", pos: PronounceMap["15"], loc: "口腔中心" },
    "ㄢ": { desc: "【前鼻音】氣流在口腔前部與鼻腔產生共鳴。", pos: PronounceMap["4"], loc: "前鼻腔" },
    "ㄤ": { desc: "【後鼻音】舌後根頂住軟顎。共鳴腔在鼻腔後部。", pos: PronounceMap["8"], loc: "後鼻腔" },
    "ㄦ": { desc: "【特殊韻母】發音時舌尖上捲，不接觸任何部位。", pos: PronounceMap["17"], loc: "捲舌位置" }
};

    const wordLib = [
        { c: ["老", "虎"], z: ["ㄌㄠˇ", "ㄏㄨˇ"] },
        { c: ["發", "風"], z: ["ㄈㄚ", "ㄈㄥ"] },
        { c: ["朋", "友"], z: ["ㄆㄥˊ", "ㄧㄡˇ"] },
        { c: ["咖", "啡"], z: ["ㄎㄚ", "ㄈㄟ"] },
        { c: ["歡", "迎"], z: ["ㄏㄨㄢ", "ㄧㄥˊ"] }
    ];

    let currentTask = null;
// 聲調與調值坐標對應表 (y坐標 20=5, 40=4, 60=3, 80=2, 100=1)
    const toneMap = {
    // 陰平 55：從1拍5樓到5拍5樓
    "":  { val: "陰平", path: "M 0 0 L 100 0" }, 
    // 陽平 35：從1拍3樓到5拍5樓
    "ˊ": { val: "陽平", path: "M 0 50 L 100 0" }, 
    // 上聲 214：1拍2樓 -> 3拍1樓 -> 5拍4樓 (紮實轉折)
    "ˇ": { val: "上聲", path: "M 0 75 Q 50 125 100 25" }, 
    // 去聲 51：從1拍5樓到5拍1樓
    "ˋ": { val: "去聲", path: "M 0 0 L 100 100" }, 
    // 輕聲：半拍，定位在第3拍的3樓中心
    "˙": { val: "輕聲", path: "M 50 50 m -2 0 a 2 2 0 1 0 4 0 a 2 2 0 1 0 -4 0" } 
};

function showDetail(index) {
    if (!currentTask || !currentTask.c[index]) return;
    const char = currentTask.c[index];
    const pinyin = currentTask.z[index];

    // 更新選中狀態樣式
    document.querySelectorAll('.char-unit').forEach(el => el.classList.remove('active-trigger'));
    const target = document.querySelectorAll('.char-unit')[index];
    if (target) target.classList.add('active-trigger');

    document.getElementById('title').innerText = `正在學習：「${char}」 (${pinyin})`;
    const container = document.getElementById('content');
    container.innerHTML = "";

    //----【新增：點擊新字時，先清空舊的藍點與部位標籤】
    document.getElementById('dot-layer').innerHTML = "";
    document.getElementById('location-label').innerText = "";
        
        // 1. 發音解析 (處理 ㄅㄆㄇ ㄧㄨㄩ ㄚㄛㄝ等)
    [...pinyin].forEach(sym => {
        let cleanSym = (sym === "一") ? "ㄧ" : sym;
        const data = MasterDictionary[cleanSym]; 

        if (data) {
            //----【修正：改為讀取 data.desc】
            container.innerHTML += `<div class="tip-card"><span class="tag">${cleanSym}</span> ${data.desc}</div>`;

            //----【新增：繪製藍色閃爍點】
            if (data.pos) {
                const dot = document.createElement('div');
                dot.className = 'pronounce-dot'; // CSS 中必須設定為藍色
                dot.style.left = data.pos.x + '%';
                dot.style.top = data.pos.y + '%';
                document.getElementById('dot-layer').appendChild(dot);
                
                // 更新部位文字標籤
                document.getElementById('location-label').innerText = `發音部位：${data.loc}`;
            }
        }
    });
        
       // 2. 聲調卡片化 (修正 undefined 並換成綠色)
        const toneMark = pinyin.match(/[ˊˇˋ˙]/) ? pinyin.match(/[ˊˇˋ˙]/)[0] : ""; 
        const toneKey = toneMark === "" ? "" : toneMark; 
        const toneInfo = toneMap[toneKey];

        if (toneInfo) {
            const displayMark = (toneMark === "") ? "—" : toneMark; 
            
            container.innerHTML += `
                <div class="tip-card" style="border-left-color: #27ae60;">
                    <span class="tag" style="background:#f0fff4; color:#27ae60; border:1px solid #27ae60;">${displayMark}</span> 
                    這是「${toneInfo.val}」，請觀察下方的聲音走勢。
                </div>`;

            // 3. 更新象限圖 (顏色改為綠色)
            document.getElementById('toneSection').style.display = "block";
            const pathElement = document.getElementById('tonePath');
            pathElement.setAttribute('d', toneInfo.path);
            pathElement.setAttribute('stroke', '#27ae60'); // 線條改綠色
            
            if (toneMark === "˙") {
                pathElement.setAttribute('fill', '#27ae60'); // 輕聲實心點改綠色
            } else {
                pathElement.setAttribute('fill', 'none');
            }
            
            document.getElementById('toneValueDisplay').style.color = "#27ae60"; // 文字改綠色
            document.getElementById('toneValueDisplay').innerText = toneInfo.val;
        }
    }
    function refresh() {
        document.getElementById('title').innerText = "點擊單字開始學習";
        document.getElementById('content').innerHTML = "";
        document.getElementById('toneSection').style.display = "none";
        if(document.getElementById('dot-layer')) document.getElementById('dot-layer').innerHTML = "";
        if(document.getElementById('location-label')) document.getElementById('location-label').innerText = "";
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
