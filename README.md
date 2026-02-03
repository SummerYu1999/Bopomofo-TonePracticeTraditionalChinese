<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <title>注音精密教學 - 座標鎖定版</title>
    <style>
        :root {
            /* 絕對鎖定尺寸 */
            --unit-w: 200px; /* 總寬 */
            --unit-h: 200px; /* 總高 */
            --cn-w: 150px;   /* 漢字區寬 */
            --zy-w: 50px;    /* 注音區寬 (含聲調) */
            --font-cn: 110px;
            --font-zy: 36px;
            --font-tn: 24px;
            --color-main: #2c3e50;
            --color-tone: #d63031;
        }

        body {
            background-color: #f4f7f6;
            display: flex;
            flex-direction: column;
            align-items: center;
            font-family: "標楷體", "DFKai-SB", serif;
            padding: 30px;
        }

        /* 單字容器：絕對尺寸鎖定 */
        .char-unit {
            display: block;
            width: var(--unit-w);
            height: var(--unit-h);
            background: white;
            border: 2px solid var(--color-main);
            position: relative;
            cursor: pointer;
            overflow: hidden;
            margin: 0 10px;
        }

        /* 漢字象限：固定定位 */
        .chinese-part {
            position: absolute;
            left: 0;
            top: 0;
            width: var(--cn-w);
            height: var(--unit-h);
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: var(--font-cn);
            border-right: 1px dashed #ccc;
        }

        /* 注音區容器：固定定位在右側 */
        .zhuyin-compound {
            position: absolute;
            left: var(--cn-w);
            top: 0;
            width: var(--zy-w);
            height: var(--unit-h);
            display: flex;
        }

        /* 注音文字：在注音區內垂直居中 */
        .zhuyin-text {
            width: 35px; /* 給注音符號固定寬度 */
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-size: var(--font-zy);
            line-height: 1.1;
        }

        /* 聲調：絕對定位在注音符號右側 */
        .tone-part {
            width: 15px; /* 給聲調符號固定寬度 */
            position: relative;
        }

        .tone-mark {
            position: absolute;
            left: -2px;
            top: 30%; /* 絕對定位在韻母右上方 */
            font-size: var(--font-tn);
            color: var(--color-tone);
            font-weight: bold;
        }

        /* 以下為教學面板與按鈕樣式... (保持精美) */
        .word-row { display: flex; margin-bottom: 30px; }
        .instruction-panel { max-width: 850px; width: 100%; background: white; padding: 25px; border-radius: 10px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
        .detail-card { border-left: 5px solid #3498db; padding: 15px; margin-top: 15px; background: #fafafa; }
        .symbol-tag { font-weight: bold; color: #2980b9; margin-right: 10px; font-size: 1.2rem; }
        button { padding: 12px 30px; font-size: 1.1rem; background: var(--color-main); color: white; border: none; border-radius: 5px; cursor: pointer; margin-bottom: 20px; }
    </style>
</head>
<body>

    <button onclick="generateWord()">換一組單詞</button>
    <div id="word-display" class="word-row"></div>

    <div class="instruction-panel">
        <h3 id="active-title">點擊單字查看專業發音提示</h3>
        <div id="details-container"></div>
    </div>

<script>
    // 您的精密教學資料庫 (節錄範例)
    const expertTips = {
        "ㄎ": "【舌根音】必須將舌後根往上頂軟顎後方。細節：發音時嘴巴需打開約一個指節寬。",
        "ㄚ": "【開口音】嘴巴要徹底打開，像要塞進大獅子頭一樣。",
        "ㄈ": "【唇齒音】上齒接觸下唇。注意：若沒碰到，會變成「ㄏ」的模糊音。",
        "ㄟ": "【韻母】發音要圓滿，確保口腔內有足夠共鳴空間。",
        "ㄅ": "【雙唇音】上下唇先閉合，氣流衝出。",
        "ㄨ": "【結合規律】ㄅㄆㄇㄈ不與ㄨ結合（例如朋友不念ㄆㄨㄥˊ）。"
    };

    const wordLib = [
        { chars: ["咖", "啡"], zhuyin: ["ㄎㄚ", "ㄈㄟ"] },
        { chars: ["朋", "友"], zhuyin: ["ㄆㄥˊ", "ㄧㄡˇ"] }
    ];

    function showDetails(char, zFull) {
        document.getElementById('active-title').innerText = `${char} (${zFull}) 發音導引`;
        const container = document.getElementById('details-container');
        container.innerHTML = "";
        const pure = zFull.replace(/[ˊˇˋ˙]/, "");
        pure.split('').forEach(s => {
            if(expertTips[s]) {
                container.innerHTML += `<div class="detail-card"><span class="symbol-tag">${s}</span>${expertTips[s]}</div>`;
            }
        });
    }

    function generateWord() {
        const item = wordLib[Math.floor(Math.random() * wordLib.length)];
        const display = document.getElementById('word-display');
        display.innerHTML = "";

        item.chars.forEach((c, i) => {
            if(c.trim()==="") return;
            const z = item.zhuyin[i];
            const tone = z.match(/[ˊˇˋ˙]/) ? z.match(/[ˊˇˋ˙]/)[0] : "";
            const pure = z.replace(/[ˊˇˋ˙]/, "");

            const div = document.createElement('div');
            div.className = 'char-unit';
            div.onclick = () => showDetails(c, z);
            div.innerHTML = `
                <div class="chinese-part">${c}</div>
                <div class="zhuyin-compound">
                    <div class="zhuyin-text">${pure.split('').join('<br>')}</div>
                    <div class="tone-part"><span class="tone-mark">${tone}</span></div>
                </div>`;
            display.appendChild(div);
        });
    }
    generateWord();
</script>
</body>
</html>
