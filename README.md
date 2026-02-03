<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
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
            --accent: #d63031;
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
    </style>
</head>
<body>

    <div class="hotkey-hint">
        ⌨️ <strong>快捷鍵操作</strong>： [空白鍵] 更換單詞 | [1] [2] [3] 查看單字註釋
    </div>

    <button onclick="refresh()">更換教學單詞</button>

    <div id="display" class="word-row"></div>

    <div class="panel">
        <div id="title" class="active-title">點擊單字或按數字鍵開始學習</div>
        <div id="content"></div>
    </div>

<script>
    /**
     * 【專業精密字典】 - 整合發音細節與編碼修正
     */
    const MasterDictionary = {
        // 1. 唇音類
        "ㄅ": "【雙唇音】上下唇先閉合，造成阻力後再讓氣流衝出。*注意不要造成爆音",
        "ㄆ": "【雙唇音】上下唇先閉合，造成阻力後再讓氣流衝出。*注意不要造成爆音",
        "ㄇ": "【雙唇音】上下唇先閉合，造成阻力後再讓氣流衝出。*注意不要造成爆音",
        "ㄈ": "【唇齒音】上齒接觸下唇。<span class='warning-note'>注意：若沒碰到下唇會變成「ㄏ」的模糊音。</span>",

        // 2. 舌尖與舌根
        "ㄉ": "【舌尖中音】利用舌尖位置（牙齦或硬顎前部）及舌面微凸造成阻礙發音。",
        "ㄊ": "【舌尖中音】利用舌尖位置（牙齦或硬顎前部）及舌面為凹造成阻礙發音。",
        "ㄋ": "【鼻音】鼻翼會有震動感，共鳴在鼻腔。",
        "ㄌ": "【邊音】鼻腔不震動，氣流從舌頭兩邊流出。",
        "ㄍ": "【舌根音】舌後根頂住軟顎。<span class='warning-note'>細節：嘴巴需打開約一個指節寬，否則韻母會跑掉。</span>",
        "ㄎ": "【舌根音】舌後根頂住軟顎。<span class='warning-note'>細節：嘴巴需打開約一個指節寬。</span>",
        "ㄏ": "【舌根音】舌後根頂住軟顎。<span class='warning-note'>細節：嘴巴需打開約一個指節寬。</span>",

        // 3. 舌面與平翹舌
        "ㄐ": "【舌面音】舌面與硬軟顎接觸，氣流從舌面中間衝出。",
        "ㄑ": "【舌面音】舌面與硬軟顎接觸，氣流從舌面中間衝出。",
        "ㄒ": "【舌面音】舌面貼住硬顎。<span class='warning-note'>注意：與英文「C」(舌尖往下)不同。</span>",
        "ㄓ": "【翹舌音】舌尖要往後捲，指向硬顎的位置。",
        "ㄔ": "【翹舌音】舌尖要往後捲，指向硬顎的位置。",
        "ㄕ": "【翹舌音】舌尖要往後捲，指向硬顎的位置。",
        "ㄖ": "【翹舌音】舌尖垂直放，不可碰到牙齒。",
        "ㄗ": "【平舌音】舌尖往前下靠近下齒背，舌面不可碰到上齒背。",
        "ㄘ": "【平舌音】舌尖往前下靠近下齒背，舌面不可碰到上齒背。",
        "ㄙ": "【平舌音】舌尖往前下靠近下齒背，舌面不可碰到上齒背。",

        // 4. 介符 (齊齒、合口、撮口)
        "ㄧ": "【介符/齊齒】嘴角往兩旁拉開做出「一」字形，有助於口型擺動。",
        "一": "【介符/齊齒】嘴角往兩旁拉開做出「一」字形，有助於口型擺動。",
        "ㄨ": "【介符/合口】雙唇收圓向前突。規律：ㄅㄆㄇㄈ不與ㄨ結合。",
        "ㄩ": "【介符/撮口】嘴唇需用力向中間收攏成圓孔狀。發音時嘴部空間與氣流流暢度並重。",

        // 5. 韻母 (單、複、鼻、特殊)
        "ㄚ": "【韻母】嘴巴要徹底打開，像要塞進大獅子頭一樣。",
        "ㄛ": "【單韻母】圓唇口型。注意：ㄅㄆㄇㄈ後應念ㄛ（早期教法），維持口型純淨。",
        "ㄜ": "【單韻母】鴨蛋形長橢圓口型，舌面往下拉壓（含湯匙感）。",
        "ㄝ": "【單韻母】口型要圓滿，確保足夠空間，避免因吃字導致音軌模糊。",
        "ㄞ": "【複韻母】運要圓滿。由「ㄚ」徹底打開起頭，滑向「ㄧ」音軌。",
        "ㄟ": "【複韻母】發音要圓滿，確保口腔內共鳴空間足夠，聲音才不悶。",
        "ㄠ": "【複韻母】由「ㄚ」轉向「ㄨ」，口型由大變小，過程需滑順。",
        "ㄡ": "【複韻母】由「ㄛ」起點收於「ㄨ」。維持氣流連貫，確保音軌完整。",
        "ㄢ": "【前鼻音】氣流在口腔前部與鼻腔產生共鳴。",
        "ㄣ": "【鼻韻母】氣流共鳴在鼻腔前部。",
        "ㄤ": "【後鼻音】舌後根頂住軟顎。共鳴腔在脖子後方與鼻腔後部，嘴巴開度要足。",
        "ㄥ": "【鼻韻母】舌後根頂住軟顎，共鳴腔在脖子後方與鼻腔後部。",
        "ㄦ": "【特殊韻母】舌尖擺到位後快速鬆開，避免舌頭過度勾起產生重捲舌雜音。"
    };

    const wordLib = [
        { c: ["老", "虎"], z: ["ㄌㄠˇ", "ㄏㄨˇ"] },
        { c: ["發", "風"], z: ["ㄈㄚ", "ㄈㄥ"] },
        { c: ["朋", "友"], z: ["ㄆㄥˊ", "ㄧㄡˇ"] },
        { c: ["咖", "啡"], z: ["ㄎㄚ", "ㄈㄟ"] },
        { c: ["歡", "迎"], z: ["ㄏㄨㄢ", "ㄧㄥˊ"] }
    ];

    let currentTask = null;

    /**
     * 核心顯示函式：負責顯示註釋並自動修正編碼錯誤
     */
    function showDetail(index) {
        if (!currentTask || !currentTask.c[index]) return;
        if (currentTask.c[index] === " ") return;

        const char = currentTask.c[index];
        const pinyin = currentTask.z[index];

        // 視覺回饋
        document.querySelectorAll('.char-unit').forEach(el => el.classList.remove('active-trigger'));
        const target = document.querySelectorAll('.char-unit')[index];
        if (target) target.classList.add('active-trigger');

        document.getElementById('title').innerText = `正在學習：「${char}」 (${pinyin})`;
        const container = document.getElementById('content');
        container.innerHTML = ""; 

        // 符號匹配迴圈 (處理 ㄧ 與 一 的編碼問題)
        [...pinyin].forEach(sym => {
            let cleanSym = (sym === "一") ? "ㄧ" : sym; 
            if (MasterDictionary[cleanSym]) {
                container.innerHTML += `
                    <div class="tip-card">
                        <span class="tag">${cleanSym}</span> ${MasterDictionary[cleanSym]}
                    </div>`;
            }
        });

        // 四大天王結合規律檢查
        const first = pinyin[0];
        if (["ㄅ","ㄆ","ㄇ","ㄈ"].includes(first) && pinyin.includes("ㄨ")) {
            container.innerHTML += `
                <div class="tip-card" style="border-left-color: #e67e22;">
                    <span class="tag">💡 規律</span> ㄅㄆㄇㄈ四大天王不與ㄨ結合（例如朋友不念ㄆㄨㄥˊ）。
                </div>`;
        }
    }

    /**
     * 刷新單詞與重置
     */
    function refresh() {
        document.getElementById('title').innerText = "點擊單字或按數字鍵開始學習";
        document.getElementById('content').innerHTML = "";
        
        currentTask = wordLib[Math.floor(Math.random() * wordLib.length)];
        const display = document.getElementById('display');
        display.innerHTML = "";

        currentTask.c.forEach((char, i) => {
            if (char === " ") return;
            const z = currentTask.z[i];
            const tone = z.match(/[ˊˇˋ˙]/) ? z.match(/[ˊˇˋ˙]/)[0] : "";
            const pure = z.replace(/[ˊˇˋ˙]/, "");

            const unit = document.createElement('div');
            unit.className = 'char-unit';
            unit.onclick = () => showDetail(i);
            unit.innerHTML = `
                <div class="cn-zone">${char}</div>
                <div class="zy-compound">
                    <div class="zy-symbols">${[...pure].join("<br>")}</div>
                    <div class="tone-zone"><span class="tone-mark">${tone}</span></div>
                </div>`;
            display.appendChild(unit);
        });
    }

    // 鍵盤監聽
    window.addEventListener('keydown', (e) => {
        if (e.code === 'Space') { e.preventDefault(); refresh(); }
        if (e.key === '1') showDetail(0);
        if (e.key === '2') showDetail(1);
        if (e.key === '3') showDetail(2);
    });

    refresh();
</script>
</body>
</html>
