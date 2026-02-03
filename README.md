<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <title>精確象限注音教學系統</title>
    <style>
        :root {
            --grid-size: 200px; /* 方格大小 */
            --font-main: 120px; /* 漢字大小 */
            --font-zhuyin: 35px; /* 注音大小 */
            --border-color: #d1d1d1;
        }

        body {
            background-color: #f0f2f5;
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 40px;
            font-family: "標楷體", "DFKai-SB", serif;
        }

        /* 單字容器：模擬你附圖的格子 */
        .char-unit {
            display: grid;
            grid-template-columns: 1fr 45px; /* 左側漢字，右側固定寬度給注音 */
            width: var(--grid-size);
            height: var(--grid-size);
            border: 2px solid var(--border-color); /* 外框 */
            position: relative;
            background: white;
            margin: 0 10px; /* 字與字之間的適當間距 */
        }

        /* 模擬九宮格輔助線 (可選，若不需要可刪除) */
        .char-unit::before {
            content: "";
            position: absolute;
            top: 50%; left: 0; width: 100%; border-top: 1px dashed var(--border-color);
            pointer-events: none;
        }

        /* 繁體中文位置：左側中央 */
        .chinese-char {
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: var(--font-main);
            border-right: 1px dashed var(--border-color);
            z-index: 1;
        }

        /* 注音符號位置：右側垂直排列 */
        .zhuyin-box {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-size: var(--font-zhuyin);
            line-height: 1.1;
            color: #d63031;
            letter-spacing: -2px;
        }

        .word-row {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 20px;
            margin-bottom: 40px;
        }

        /* 構造圖與提示區 */
        .info-panel {
            max-width: 800px;
            display: flex;
            gap: 20px;
            background: white;
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.1);
            margin-top: 20px;
        }

        #anatomy-img { width: 200px; height: auto; }
        
        button {
            padding: 12px 30px;
            font-size: 1.2rem;
            cursor: pointer;
            background: #34495e;
            color: white;
            border: none;
            border-radius: 5px;
            margin-bottom: 30px;
        }
    </style>
</head>
<body>

    <button onclick="updateWord()">隨機生成單詞</button>

    <div id="display-row" class="word-row">
        </div>

    <div class="info-panel">
        <img id="anatomy-img" src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/4e/VocalLines.png/200px-VocalLines.png">
        <div id="hint-text">點擊按鈕查看發音與部位提示</div>
    </div>

<script>
    const wordLib = [
        { 
            chars: ["歡", "迎"], 
            zhuyin: ["ㄏㄨㄢ", "ㄧㄥˊ"], 
            hints: ["ㄏ：舌根音，舌根靠近軟顎", "ㄧ：齊齒呼，舌位高"] 
        },
        { 
            chars: ["草", "莓"], 
            zhuyin: ["ㄘㄠˇ", "ㄇㄟˊ"], 
            hints: ["ㄘ：平舌音，舌尖抵住齒背", "ㄇ：雙唇音，嘴唇閉合"] 
        }
    ];

    function updateWord() {
        const item = wordLib[Math.floor(Math.random() * wordLib.length)];
        const row = document.getElementById('display-row');
        const hint = document.getElementById('hint-text');

        row.innerHTML = "";
        item.chars.forEach((c, i) => {
            const unit = document.createElement('div');
            unit.className = 'char-unit';
            unit.innerHTML = `
                <div class="chinese-char">${c}</div>
                <div class="zhuyin-box">${item.zhuyin[i].split('').join('<br>')}</div>
            `;
            row.appendChild(unit);
        });

        hint.innerHTML = `<strong>發音提示：</strong><br>${item.hints.join('<br>')}`;
    }

    // 初始載入
    updateWord();
</script>
</body>
</html>
