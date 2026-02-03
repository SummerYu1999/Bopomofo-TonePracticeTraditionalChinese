<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <title>注音發音教學系統 - 專業優化版</title>
    <style>
        :root {
            --grid-size: 200px;
            --font-main: 110px;
            --font-zhuyin: 36px;
            --font-tone: 22px;
            --primary-dark: #2c3e50;
            --accent-red: #d63031;
            --bg-light: #f4f7f6;
        }

        body {
            background-color: var(--bg-light);
            display: flex;
            flex-direction: column;
            align-items: center;
            font-family: "標楷體", "DFKai-SB", serif;
            padding: 30px;
            color: var(--primary-dark);
        }

        .word-row {
            display: flex;
            gap: 20px;
            margin-bottom: 30px;
        }

        /* 象限固定系統 */
        .char-unit {
            display: flex;
            width: var(--grid-size);
            height: var(--grid-size);
            background: white;
            border: 2px solid var(--primary-dark);
            cursor: pointer;
            transition: all 0.2s;
        }

        .char-unit:hover {
            box-shadow: 0 8px 15px rgba(0,0,0,0.1);
            transform: translateY(-3px);
            border-color: var(--accent-red);
        }

        .chinese-part {
            flex: 3;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: var(--font-main);
            border-right: 1px dashed #ccc;
        }

        .zhuyin-compound {
            flex: 1.2;
            display: flex;
        }

        .zhuyin-text {
            flex: 2;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-size: var(--font-zhuyin);
            line-height: 1.1;
        }

        .tone-part {
            flex: 1;
            position: relative;
        }

        .tone-mark {
            position: absolute;
            left: -2px;
            top: 25%;
            font-size: var(--font-tone);
            color: var(--accent-red);
            font-weight: bold;
        }

        /* 優化後的教學面板 */
        .instruction-panel {
            max-width: 900px;
            width: 100%;
            background: white;
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.08);
            border-top: 5px solid var(--primary-dark);
        }

        .active-word-title {
            font-size: 1.8rem;
            margin-bottom: 20px;
            border-bottom: 2px solid #eee;
            padding-bottom: 10px;
            color: var(--accent-red);
        }

        .detail-card {
            background: #fdfdfd;
            border: 1px solid #edf2f7;
            border-left: 5px solid #3498db;
            padding: 15px;
            margin-bottom: 20px;
        }

        .symbol-tag {
            font-size: 1.5rem;
            font-weight: bold;
            color: #2980b9;
            margin-right: 10px;
            background: #ebf8ff;
            padding: 2px 10px;
            border-radius: 4px;
        }

        .tip-content {
            font-size: 1.15rem;
            line-height: 1.8;
            color: #4a5568;
        }

        .warning-note {
            color: #e67e22;
            font-weight: bold;
            margin-top: 8px;
            display: block;
        }

        button {
            padding: 12px 35px;
            font-size: 1.2rem;
            background: var(--primary-dark);
            color: white;
            border: none;
            border-radius: 30px;
            cursor: pointer;
            margin-bottom: 25px;
        }
    </style>
</head>
<body>

    <button onclick="generateWord()">更換教學單詞</button>

    <div id="word-display" class="word-row"></div>

    <div id="panel" class="instruction-panel">
        <div id="active-title" class="active-word-title">請點擊上方漢字，啟動精密發音導引</div>
        <div id="details-container"></div>
    </div>

<script>
    // 根據您提供的資料精煉出的專業教學庫
    const expertTips = {
        // 1. 唇音類
        "ㄅ": "【雙唇音】發音時上下唇要先閉合，造成阻力後再讓氣流衝出。",
        "ㄆ": "【雙唇音】發音時上下唇要先閉合，造成阻力後再讓氣流衝出。",
        "ㄇ": "【雙唇音】發音時上下唇要先閉合，造成阻力後再讓氣流衝出。",
        "ㄈ": "【唇齒音】這是上齒接觸下唇發出的聲音。<span class='warning-note'>注意：若上齒沒碰到下唇，會變成「ㄏ」的模糊音。</span>",

        // 2. 舌尖與舌根
        "ㄉ": "【舌尖中音】主要利用舌尖的位置（牙齦或硬顎前部）造成阻礙發音。",
        "ㄊ": "【舌尖中音】主要利用舌尖的位置（牙齦或硬顎前部）造成阻礙發音。",
        "ㄋ": "【舌尖中音/鼻音】念「ㄋ」時，鼻翼會有震動感，共鳴在鼻腔。",
        "ㄌ": "【舌尖中音/邊音】念「ㄌ」時鼻腔不震動，氣流從舌頭兩邊流出。",
        "ㄍ": "【舌根音】必須將舌後根往上頂軟顎後方。<span class='warning-note'>細節：發音時嘴巴需打開約一個指節寬，牙齒若不開會影響韻母準確度。</span>",
        "ㄎ": "【舌根音】必須將舌後根往上頂軟顎後方。<span class='warning-note'>細節：發音時嘴巴需打開約一個指節寬。</span>",
        "ㄏ": "【舌根音】必須將舌後根往上頂軟顎後方。<span class='warning-note'>細節：發音時嘴巴需打開約一個指節寬。</span>",

        // 3. 舌面與平翹舌
        "ㄐ": "【舌面音】舌面面積與硬、軟顎中間接觸，氣流從舌面中間衝出。",
        "ㄑ": "【舌面音】舌面面積與硬、軟顎中間接觸，氣流從舌面中間衝出。",
        "ㄒ": "【舌面音】必須是舌面貼住硬顎。<span class='warning-note'>注意：與英文「C」不同，「C」是舌尖往下。</span>",
        "ㄓ": "【翹舌音】舌尖要往後捲，指向硬顎的位置。",
        "ㄔ": "【翹舌音】舌尖要往後捲，指向硬顎的位置。",
        "ㄕ": "【翹舌音】舌尖要往後捲，指向硬顎的位置。",
        "ㄖ": "【翹舌音】舌尖往後捲。<span class='warning-note'>ㄖ的特殊發音：舌尖要垂直放，不可碰到牙齒。</span>",
        "ㄗ": "【平舌音】舌尖往前、往下，靠近下齒或下齒背，舌面絕對不可碰到上齒背。",
        "ㄘ": "【平舌音】舌尖往前、往下，靠近下齒或下齒背，舌面絕對不可碰到上齒背。",
        "ㄙ": "【平舌音】舌尖往前、往下，靠近下齒或下齒背，舌面絕對不可碰到上齒背。",

        // 4. 韻母細節
        "ㄜ": "【韻母】嘴巴呈現「鴨蛋般」的長橢圓形，舌面要往下拉壓，像含著湯匙的感覺。",
        "一": "【介符】嘴角需往兩旁拉開，做出明顯的「一」字形，有助於口型擺動。",
        "ㄨ": "【結合規律】「ㄅㄆㄇㄈ」在標準發音中不與「ㄨ」結合（例如：朋友不念ㄆㄨㄥˊ）。",
        "ㄣ": "【鼻韻母】氣流共鳴在鼻腔前部。",
        "ㄥ": "【鼻韻母】舌後根頂住軟顎，共鳴腔在脖子後方與鼻腔後部。"
    };

    const wordLib = [
        { chars: ["朋友", " "], zhuyin: ["ㄆㄥˊ", "ㄧㄡˇ"] },
        { chars: ["發", "風"], zhuyin: ["ㄈㄚ", "ㄈㄥ"] },
        { chars: ["牛奶", " "], zhuyin: ["ㄋㄧㄡˊ", "ㄋㄞˇ"] },
        { chars: ["老虎", " "], zhuyin: ["ㄌㄠˇ", "ㄏㄨˇ"] },
        { chars: ["草莓", " "], zhuyin: ["ㄘㄠˇ", "ㄇㄟˊ"] }
    ];

    function showDetailedTips(char, zhuyin) {
        document.getElementById('active-title').innerText = `教學重點： 「${char}」 (${zhuyin})`;
        const container = document.getElementById('details-container');
        container.innerHTML = "";

        const pureZhuyin = zhuyin.replace(/[ˊˇˋ˙]/, "");
        
        pureZhuyin.split('').forEach(sym => {
            if (expertTips[sym]) {
                const card = document.createElement('div');
                card.className = 'detail-card';
                card.innerHTML = `<span class="symbol-tag">${sym}</span><span class="tip-content">${expertTips[sym]}</span>`;
                container.appendChild(card);
            }
        });

        // 特別加入「ㄨ」的例外提示
        const firstConsonant = pureZhuyin[0];
        if (["ㄅ","ㄆ","ㄇ","ㄈ"].includes(firstConsonant)) {
            const warning = document.createElement('div');
            warning.className = 'detail-card';
            warning.style.borderLeftColor = "#e67e22";
            warning.innerHTML = `<span class="symbol-tag">注！</span><span class="tip-content">${expertTips["ㄨ"]}</span>`;
            container.appendChild(warning);
        }
    }

    function generateWord() {
        const item = wordLib[Math.floor(Math.random() * wordLib.length)];
        const display = document.getElementById('word-display');
        display.innerHTML = "";

        item.chars.forEach((c, i) => {
            if(c.trim() === "") return;
            const z = item.zhuyin[i];
            const tone = z.match(/[ˊˇˋ˙]/) ? z.match(/[ˊˇˋ˙]/)[0] : "";
            const pure = z.replace(/[ˊˇˋ˙]/, "");

            const unit = document.createElement('div');
            unit.className = 'char-unit';
            unit.onclick = () => showDetailedTips(c, z);
            unit.innerHTML = `
                <div class="chinese-part">${c}</div>
                <div class="zhuyin-compound">
                    <div class="zhuyin-text">${pure.split('').join('<br>')}</div>
                    <div class="tone-part"><span class="tone-mark">${tone}</span></div>
                </div>`;
            display.appendChild(unit);
        });
    }

    generateWord();
</script>
</body>
</html>
