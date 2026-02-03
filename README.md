<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>注音精密發音教學系統</title>
    <style>
        :root {
            /* 嚴格對照附圖比例：漢字 75%，注音 25% */
            --box-size: 220px;
            --cn-width: 165px;
            --zy-width: 55px;
            --font-cn: 110px;
            --font-zy: 36px;
            --font-tone: 24px;
            --primary-color: #2c3e50;
            --accent-red: #d63031;
            --bg-gray: #f8f9fa;
        }

        body {
            background-color: #e9ecef;
            display: flex;
            flex-direction: column;
            align-items: center;
            font-family: "標楷體", "DFKai-SB", serif;
            padding: 40px;
            margin: 0;
        }

        .controls {
            margin-bottom: 30px;
        }

        button {
            padding: 12px 40px;
            font-size: 1.2rem;
            background: var(--primary-color);
            color: white;
            border: none;
            border-radius: 50px;
            cursor: pointer;
            transition: 0.3s;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }

        button:hover {
            background: #34495e;
            transform: translateY(-2px);
        }

        /* 單字容器 */
        .word-display-area {
            display: flex;
            flex-wrap: wrap;
            gap: 20px;
            margin-bottom: 40px;
            justify-content: center;
        }

        /* 單個字框：象限對齊的核心 */
        .char-box {
            display: grid;
            grid-template-columns: var(--cn-width) var(--zy-width);
            width: var(--box-size);
            height: var(--box-size);
            background: white;
            border: 2px solid var(--primary-color);
            cursor: pointer;
            transition: all 0.2s;
            box-sizing: border-box;
        }

        .char-box:hover {
            border-color: var(--accent-red);
            box-shadow: 0 8px 25px rgba(0,0,0,0.15);
        }

        /* 左側：漢字象限 */
        .cn-zone {
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: var(--font-cn);
            border-right: 1px dashed #ced4da;
            color: #212529;
        }

        /* 右側：注音與聲調複合象限 */
        .zy-compound-zone {
            display: grid;
            grid-template-columns: 2fr 1fr; /* 分為注音符號區與聲調區 */
            padding: 5px 0;
        }

        /* 注音符號垂直列表 */
        .zy-symbol-list {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-size: var(--font-zy);
            line-height: 1.15;
            color: #495057;
        }

        /* 聲調絕對定位區 */
        .tone-marker-zone {
            position: relative;
        }

        .tone-marker {
            position: absolute;
            top: 25%; /* 固定在側邊象限的上方位置 */
            left: 0;
            font-size: var(--font-tone);
            color: var(--accent-red);
            font-weight: bold;
        }

        /* 教學註釋面板 */
        .instruction-panel {
            max-width: 850px;
            width: 100%;
            background: white;
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            border-top: 6px solid var(--primary-color);
        }

        .active-title {
            font-size: 1.8rem;
            color: var(--accent-red);
            margin-bottom: 20px;
            border-bottom: 2px solid #f1f3f5;
            padding-bottom: 12px;
            text-align: center;
        }

        .detail-card {
            background: #fdfdfd;
            border-left: 5px solid #3498db;
            padding: 18px;
            margin-bottom: 15px;
            border-radius: 4px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.02);
        }

        .symbol-tag {
            font-size: 1.5rem;
            font-weight: bold;
            color: #2980b9;
            margin-right: 12px;
            background: #e1f5fe;
            padding: 2px 10px;
            border-radius: 4px;
        }

        .tip-content {
            font-size: 1.2rem;
            line-height: 1.8;
            color: #4a5568;
        }

        .warning-text {
            color: #e67e22;
            font-weight: bold;
            display: block;
            margin-top: 8px;
        }
    </style>
</head>
<body>

    <div class="controls">
        <button onclick="generateNewWord()">更換教學單詞</button>
    </div>

    <div id="word-display" class="word-display-area"></div>

    <div class="instruction-panel">
        <div id="active-title" class="active-title">點擊上方單字，啟動精密發音導引</div>
        <div id="details-container">
            </div>
    </div>

<script>
    /**
     * 【精密字典資料庫】
     * 只需要在此處維護每個符號的發音細節。
     * 系統會自動拆解單字注音並在此匹配說明。
     */
    const MasterTips = {
        // 聲母
        "ㄎ": "【舌根音】必須將舌後根往上頂軟顎後方。<span class='warning-text'>細節：發音時嘴巴需打開約一個指節寬。</span>",
        "ㄈ": "【唇齒音】這是上齒接觸下唇發出的聲音。<span class='warning-text'>注意：若上齒沒碰到下唇，會變成「ㄏ」的模糊音。</span>",
        "ㄏ": "【舌根音】必須將舌後根往上頂軟顎後方。",
        "ㄌ": "【邊音】念「ㄌ」時鼻腔不震動，氣流是從舌頭兩邊流出。",
        "ㄘ": "【平舌音】舌尖往前、往下靠近下齒背，舌面絕對不可碰到上齒背。",
        "ㄆ": "【雙唇音】發音時上下唇要先閉合，造成阻力後再讓氣流衝出。",
        
        // 韻母與介符
        "ㄚ": "【開口音】重點在於嘴巴張開的幅度，必須像要塞進一口大獅子頭一樣，嘴巴要徹底打開。",
        "ㄜ": "【韻母】嘴巴呈現鴨蛋般的長橢圓形，且舌面要往下拉壓（像含著湯匙的感覺）。",
        "ㄟ": "【韻母】發音要「圓滿」，嘴巴空間要夠，聲音才不會悶。確保口腔內有足夠的共鳴空間。",
        "ㄨ": "【合口呼】雙唇收圓，像吹口哨一樣的口型。",
        "ㄢ": "【鼻韻母】發音結束時，舌尖要抵住上齒齦。",
        "ㄧ": "【介符】嘴角需往兩旁拉開，做出明顯的「一」字形。",
        "ㄥ": "【鼻韻母】舌後根頂住軟顎，共鳴腔在脖子後方與鼻腔後部。",

        // 聲調
        "ˊ": "【二聲】聲音由中點往上升。想像聲音向頭頂斜後方拉去。",
        "ˇ": "【三聲】音位先降後升，注意下降的深度，要感覺到喉嚨深處的共鳴。",
        "ˋ": "【四聲】由高音急降到低音，發音短促、堅定、有力。",
        "˙": "【輕聲】發音極短、極輕，像羽毛落下。"
    };

    // 教學字詞庫 (可自由擴充)
    const wordLibrary = [
        { text: "咖啡", pinyin: ["ㄎㄚ", "ㄈㄟ"] },
        { text: "歡迎", pinyin: ["ㄏㄨㄢ", "ㄧㄥˊ"] },
        { text: "老虎", pinyin: ["ㄌㄠˇ", "ㄏㄨˇ"] },
        { text: "草莓", pinyin: ["ㄘㄠˇ", "ㄇㄟˊ"] },
        { text: "朋友", pinyin: ["ㄆㄥˊ", "ㄧㄡˇ"] }
    ];

    /**
     * 點擊單字後的自動顯示註釋邏輯
     */
    function showInstruction(char, fullPinyin) {
        // 更新標題
        document.getElementById('active-title').innerText = `正在學習：「${char}」 (${fullPinyin})`;
        
        const container = document.getElementById('details-container');
        container.innerHTML = ""; // 點擊時清空舊註釋

        // 將注音字串拆成單個符號（如：ㄏ, ㄨ, ㄢ）
        const symbols = fullPinyin.split("");
        
        symbols.forEach(s => {
            if (MasterTips[s]) {
                const card = document.createElement('div');
                card.className = 'detail-card';
                card.innerHTML = `<span class="symbol-tag">${s}</span><span class="tip-content">${MasterTips[s]}</span>`;
                container.appendChild(card);
            }
        });

        // 特別規則檢測：ㄅㄆㄇㄈ 遇到 ㄨ 的提示
        const firstLetter = fullPinyin[0];
        if (["ㄅ","ㄆ","ㄇ","ㄈ"].includes(firstLetter)) {
            const ruleCard = document.createElement('div');
            ruleCard.className = 'detail-card';
            ruleCard.style.borderLeftColor = "#e67e22";
            ruleCard.innerHTML = `<span class="symbol-tag">規律</span><span class="tip-content">「ㄅㄆㄇㄈ」在標準發音中不與「ㄨ」結合（如：朋友不念ㄆㄨㄥˊ）。</span>`;
            container.appendChild(ruleCard);
        }
    }

    /**
     * 更換單詞邏輯
     */
    function generateNewWord() {
        // 核心要求：更換單詞時重置下方面板
        document.getElementById('active-title').innerText = "請點擊上方漢字，啟動精密發音導引";
        document.getElementById('details-container').innerHTML = "";

        // 隨機抽選
        const currentItem = wordLibrary[Math.floor(Math.random() * wordLibrary.length)];
        const displayArea = document.getElementById('word-display');
        displayArea.innerHTML = "";

        // 渲染象限格子
        currentItem.text.split("").forEach((char, index) => {
            const py = currentItem.pinyin[index];
            const tone = py.match(/[ˊˇˋ˙]/) ? py.match(/[ˊˇˋ˙]/)[0] : "";
            const pure = py.replace(/[ˊˇˋ˙]/, "");

            const box = document.createElement('div');
            box.className = 'char-box';
            box.onclick = () => showInstruction(char, py);
            
            box.innerHTML = `
                <div class="cn-zone">${char}</div>
                <div class="zy-compound-zone">
                    <div class="zy-symbol-list">${pure.split("").join("<br>")}</div>
                    <div class="tone-marker-zone"><span class="tone-marker">${tone}</span></div>
                </div>
            `;
            displayArea.appendChild(box);
        });
    }

    // 初始載入
    generateNewWord();
</script>
</body>
</html>
