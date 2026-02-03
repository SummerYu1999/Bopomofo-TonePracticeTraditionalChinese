<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>注音精密發音教學系統 - 邏輯修正版</title>
    <style>
        :root {
            --box-size: 220px;
            --cn-width: 165px;
            --zy-width: 55px;
            --font-cn: 110px;
            --font-zy: 36px;
            --font-tone: 24px;
            --primary-color: #2c3e50;
            --accent-red: #d63031;
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

        .controls { margin-bottom: 30px; }

        button {
            padding: 12px 40px;
            font-size: 1.2rem;
            background: var(--primary-color);
            color: white;
            border: none;
            border-radius: 50px;
            cursor: pointer;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
        }

        .word-display-area {
            display: flex;
            flex-wrap: wrap;
            gap: 20px;
            margin-bottom: 40px;
            justify-content: center;
        }

        /* 象限固定佈局 */
        .char-box {
            display: grid;
            grid-template-columns: var(--cn-width) var(--zy-width);
            width: var(--box-size);
            height: var(--box-size);
            background: white;
            border: 2px solid var(--primary-color);
            cursor: pointer;
            box-sizing: border-box;
        }

        .cn-zone {
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: var(--font-cn);
            border-right: 1px dashed #ced4da;
        }

        .zy-compound-zone {
            display: grid;
            grid-template-columns: 2fr 1fr;
        }

        .zy-symbol-list {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-size: var(--font-zy);
            line-height: 1.15;
        }

        .tone-marker-zone { position: relative; }

        .tone-marker {
            position: absolute;
            top: 25%;
            left: 0;
            font-size: var(--font-tone);
            color: var(--accent-red);
            font-weight: bold;
        }

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

        .tip-content { font-size: 1.2rem; line-height: 1.8; color: #4a5568; }
    </style>
</head>
<body>

    <div class="controls">
        <button onclick="generateNewWord()">更換教學單詞</button>
    </div>

    <div id="word-display" class="word-display-area"></div>

    <div class="instruction-panel">
        <div id="active-title" class="active-title">點擊上方單字，啟動精密發音導引</div>
        <div id="details-container"></div>
    </div>

<script>
    const MasterTips = {
        "ㄎ": "【舌根音】必須將舌後根往上頂軟顎後方。細節：發音時嘴巴需打開約一個指節寬。",
        "ㄚ": "【開口音】重點在於嘴巴張開的幅度，必須像要塞進一口大獅子頭一樣，嘴巴要徹底打開。",
        "ㄈ": "【唇齒音】這是上齒接觸下唇發出的聲音。注意：若上齒沒碰到下唇，會變成「ㄏ」的模糊音。",
        "ㄟ": "【韻母】發音要「圓滿」，確保口腔內有足夠的共鳴空間。",
        "ㄏ": "【舌根音】舌後根靠近軟顎，讓氣流摩擦而出。",
        "ㄨ": "【合口呼】雙唇收圓，像吹口哨一樣的口型。",
        "ㄢ": "【鼻韻母】發音結束時，舌尖要抵住上齒齦。",
        "ㄧ": "【介符】嘴角需往兩旁拉開，做出明顯的「一」字形。",
        "ㄥ": "【鼻韻母】舌後根頂住軟顎，共鳴腔在鼻腔後部。",
        "ˊ": "【二聲】聲音由中點往上升。",
        "ˇ": "【三聲】音位先降後升。",
        "ˋ": "【四聲】由高音急降到低音。"
    };

    const wordLibrary = [
        { text: "咖啡", pinyin: ["ㄎㄚ", "ㄈㄟ"] },
        { text: "歡迎", pinyin: ["ㄏㄨㄢ", "ㄧㄥˊ"] },
        { text: "朋友", pinyin: ["ㄆㄥˊ", "ㄧㄡˇ"] }
    ];

    function showInstruction(char, fullPinyin) {
        document.getElementById('active-title').innerText = `正在學習：「${char}」 (${fullPinyin})`;
        const container = document.getElementById('details-container');
        container.innerHTML = "";

        const symbols = fullPinyin.split("");
        
        // 顯示基本注音提示
        symbols.forEach(s => {
            if (MasterTips[s]) {
                const card = document.createElement('div');
                card.className = 'detail-card';
                card.innerHTML = `<span class="symbol-tag">${s}</span><span class="tip-content">${MasterTips[s]}</span>`;
                container.appendChild(card);
            }
        });

        // --- 邏輯修正處 ---
        // 只有當開頭是 ㄅㄆㄇㄈ 且 注音中確實包含「ㄨ」時，才顯示結合規律
        const firstLetter = fullPinyin[0];
        const hasWu = fullPinyin.includes("ㄨ");
        
        if (["ㄅ","ㄆ","ㄇ","ㄈ"].includes(firstLetter) && hasWu) {
            const ruleCard = document.createElement('div');
            ruleCard.className = 'detail-card';
            ruleCard.style.borderLeftColor = "#e67e22";
            ruleCard.innerHTML = `<span class="symbol-tag">規律</span><span class="tip-content">「ㄅㄆㄇㄈ」在標準發音中不與「ㄨ」結合（如：朋友不念ㄆㄨㄥˊ）。</span>`;
            container.appendChild(ruleCard);
        }
    }

    function generateNewWord() {
        document.getElementById('active-title').innerText = "請點擊上方單字，啟動精密發音導引";
        document.getElementById('details-container').innerHTML = "";

        const currentItem = wordLibrary[Math.floor(Math.random() * wordLibrary.length)];
        const displayArea = document.getElementById('word-display');
        displayArea.innerHTML = "";

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
                </div>`;
            displayArea.appendChild(box);
        });
    }

    generateNewWord();
</script>
</body>
</html>
