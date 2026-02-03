<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <title>注音精確定位系統</title>
    <style>
        :root {
            --grid-size: 200px;
            --font-main: 110px;
            --font-zhuyin: 35px;
            --font-tone: 20px;
            --ink-color: #2d3436;
            --tone-color: #d63031;
        }

        body {
            background-color: #f5f6fa;
            display: flex;
            flex-direction: column;
            align-items: center;
            font-family: "標楷體", "DFKai-SB", serif;
            padding: 50px;
        }

        .word-row {
            display: flex;
            gap: 30px;
            margin-bottom: 50px;
        }

        /* 漢字主容器 */
        .char-box {
            display: flex;
            width: var(--grid-size);
            height: var(--grid-size);
            background: white;
            border: 2px solid #ccc;
            position: relative;
        }

        /* 漢字象限 */
        .chinese {
            flex: 3;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: var(--font-main);
            border-right: 1px dashed #ddd;
        }

        /* 注音與聲調複合象限 */
        .zhuyin-compound {
            flex: 1;
            display: flex;
            position: relative;
            padding: 10px 0;
        }

        /* 注音符號垂直排列 */
        .zhuyin-text {
            flex: 2;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-size: var(--font-zhuyin);
            line-height: 1.1;
        }

        /* 聲調絕對定位：標在右側 */
        .tone {
            flex: 1;
            position: relative;
            font-size: var(--font-tone);
            color: var(--tone-color);
        }

        /* 聲調符號位置微調 */
        .tone-mark {
            position: absolute;
            right: 5px;
            /* 預設二三四聲在中間偏上，輕聲需額外邏輯 */
            top: 40%; 
        }

        .info-panel {
            background: #fff;
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 5px 15px rgba(0,0,0,0.05);
            max-width: 600px;
        }
        
        button {
            padding: 12px 30px;
            font-size: 1.2rem;
            cursor: pointer;
            background: #0984e3;
            color: white;
            border: none;
            border-radius: 5px;
            margin-bottom: 30px;
        }
    </style>
</head>
<body>

    <button onclick="nextWord()">生成隨機單詞</button>

    <div id="word-display" class="word-row"></div>

    <div class="info-panel">
        <div id="hint-content">點擊按鈕查看構造圖與發音提示</div>
    </div>

<script>
    const wordLib = [
        { 
            chars: ["歡", "迎"], 
            // 格式：[注音, 聲調]
            data: [
                { z: "ㄏㄨㄢ", t: "" },    // 一聲不標
                { z: "ㄧㄥ", t: "ˊ" }    // 二聲標在右側
            ],
            hint: "「ㄏ」是舌根音，舌根隆起；「ㄧㄥ」是齊齒呼。"
        },
        { 
            chars: ["草", "莓"], 
            data: [
                { z: "ㄘㄠ", t: "ˇ" },
                { z: "ㄇㄟ", t: "ˊ" }
            ],
            hint: "「ㄘ」是平舌音；「ㄇ」是雙唇音。"
        }
    ];

    function nextWord() {
        const item = wordLib[Math.floor(Math.random() * wordLib.length)];
        const display = document.getElementById('word-display');
        const hint = document.getElementById('hint-content');

        display.innerHTML = "";
        item.chars.forEach((c, i) => {
            const unit = item.data[i];
            const charDiv = document.createElement('div');
            charDiv.className = 'char-box';
            charDiv.innerHTML = `
                <div class="chinese">${c}</div>
                <div class="zhuyin-compound">
                    <div class="zhuyin-text">${unit.z.split('').join('<br>')}</div>
                    <div class="tone"><span class="tone-mark">${unit.t}</span></div>
                </div>
            `;
            display.appendChild(charDiv);
        });

        hint.innerHTML = `<strong>發音部位提示：</strong><br>${item.hint}`;
    }

    nextWord();
</script>
</body>
</html>
