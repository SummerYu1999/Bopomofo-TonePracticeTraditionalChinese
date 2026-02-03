<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <title>注音精密教學系統</title>
    <style>
        :root {
            /* 絕對鎖定尺寸：確保排版永不跑位 */
            --unit-w: 200px; 
            --unit-h: 200px; 
            --cn-w: 150px;   
            --zy-w: 50px;    
            --font-cn: 110px;
            --font-zy: 36px;
            --font-tn: 24px;
            --color-main: #2c3e50;
            --color-tone: #d63031;
        }

        body {
            background-color: #f1f2f6;
            display: flex;
            flex-direction: column;
            align-items: center;
            font-family: "標楷體", "DFKai-SB", serif;
            padding: 40px;
            color: var(--color-main);
        }

        /* 單字行 */
        .word-row {
            display: flex;
            gap: 20px;
            margin-bottom: 40px;
            justify-content: center;
        }

        /* 方格：絕對定位鎖定 */
        .char-unit {
            position: relative;
            width: var(--unit-w);
            height: var(--unit-h);
            background: white;
            border: 2px solid var(--color-main);
            cursor: pointer;
            transition: all 0.2s;
            box-sizing: border-box;
        }

        .char-unit:hover {
            border-color: var(--color-tone);
            transform: translateY(-5px);
            box-shadow: 0 5px 15px rgba(0,0,0,0.1);
        }

        /* 漢字象限 */
        .chinese-part {
            position: absolute;
            left: 0;
            top: 0;
            width: var(--cn-w);
            height: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: var(--font-cn);
            border-right: 1px dashed #ccc;
        }

        /* 注音區容器 */
        .zhuyin-compound {
            position: absolute;
            left: var(--cn-w);
            top: 0;
            width: var(--zy-w);
            height: 100%;
            display: flex;
        }

        /* 注音符號 */
        .zhuyin-text {
            flex: 2;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-size: var(--font-zy);
            line-height: 1.1;
        }

        /* 聲調 */
        .tone-part {
            flex: 1;
            position: relative;
        }

        .tone-mark {
            position: absolute;
            left: -2px;
            top: 30%; /* 聲調固定在右上方 */
            font-size: var(--font-tn);
            color: var(--color-tone);
            font-weight: bold;
        }

        /* 教學註釋面板 */
        .instruction-panel {
            max-width: 800px;
            width: 100%;
            min-height: 200px;
            background: white;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.08);
            border-top: 6px solid var(--color-main);
        }

        .active-title {
            font-size: 1.6rem;
            margin-bottom: 15px;
            color: var(--color-tone);
            border-bottom: 1px solid #eee;
            padding-bottom: 10px;
        }

        .detail-card {
            background: #fafafa;
            border-left: 5px solid #3498db;
            padding: 15px;
            margin-top: 15px;
            line-height: 1.8;
            font-size: 1.2rem;
        }

        .symbol-tag {
            font-weight: bold;
            color: #2980b9;
            margin-right: 10px;
            background: #e1f5fe;
            padding: 2px 8px;
            border-radius: 4px;
        }

        button {
            padding: 15px 40px;
            font-size: 1.2rem;
            background: var(--color-main);
            color: white;
            border: none;
            border-radius: 50px;
            cursor: pointer;
            margin-bottom: 30px;
            transition: 0.3s;
        }

        button:hover { background: #34495e; }
    </style>
</head>
<body>

    <button onclick="generateNewWord()">更換教學單詞</button>

    <div id="word-display" class="word-row"></div>

    <div class="instruction-panel">
        <div id="active-title" class="active-title">點擊上方單字，查看詳細發音導引</div>
        <div id="details-container">
            </div>
    </div>

<script>
    // 您的詳細教學資料庫
    const expertTips = {
        "ㄎ": "【舌根音】必須將舌後根往上頂軟顎後方。細節：發音時嘴巴需打開約一個指節寬。",
        "ㄚ": "【開口音】重點在於嘴巴張開的幅度，必須像要塞進一大口的食物一樣（如大獅子頭），嘴巴要徹底打開。",
        "ㄈ": "【唇齒音】這是上齒接觸下唇發出的聲音。注意：若上齒沒碰到下唇，會變成「ㄏ」的模糊音。",
        "ㄟ": "【韻母】發音要「圓滿」，確保口腔內有足夠的共鳴空間，聲音才不會悶在裡面。",
        "ㄘ": "【平舌音】舌尖往前、往下靠近下齒背，舌面絕對不可碰到上齒背。",
        "ㄠ": "【複韻母】發音時由ㄚ轉向ㄨ的位置，口型由大變小，過程需滑順。",
        "ㄇ": "【雙唇音】發音時上下唇要先閉合，造成阻力後再讓氣流衝出。",
        "ˇ": "【聲調：三聲】音位先降後升，要注意下降的深度，共鳴感較重。"
    };

    const wordLib = [
        { chars: ["咖", "啡"], zhuyin: ["ㄎㄚ", "ㄈㄟ"] },
        { chars: ["草", "莓"], zhuyin: ["ㄘㄠˇ", "ㄇㄟˊ"] },
        { chars: ["朋友", " "], zhuyin: ["ㄆㄥˊ", "ㄧㄡˇ"] }
    ];

    // 更新顯示細節
    function showDetails(char, zFull) {
        document.getElementById('active-title').innerText = `正在學習： 「${char}」 (${zFull})`;
        const container = document.getElementById('details-container');
        container.innerHTML = "";

        const pure = zFull.replace(/[ˊˇˋ˙]/, "");
        const tone = zFull.match(/[ˊˇˋ˙]/) ? zFull.match(/[ˊˇˋ˙]/)[0] : "";

        // 顯示注音符號提示
        pure.split('').forEach(s => {
            if(expertTips[s]) {
                container.innerHTML += `<div class="detail-card"><span class="symbol-tag">${s}</span>${expertTips[s]}</div>`;
            }
        });

        // 如果有聲調，也顯示聲調提示 (可選)
        if(tone && expertTips[tone]) {
            container.innerHTML += `<div class="detail-card"><span class="symbol-tag">${tone}</span>${expertTips[tone]}</div>`;
        }
    }

    // 更換單詞並重置面板
    function generateNewWord() {
        // 1. 重置提示面板 (核心需求)
        document.getElementById('active-title').innerText = "請點擊上方單字，查看詳細發音導引";
        document.getElementById('details-container').innerHTML = "";

        // 2. 隨機生成新字
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
            unit.onclick = () => showDetails(c, z); // 再次點擊顯示新單詞的註釋
            unit.innerHTML = `
                <div class="chinese-part">${c}</div>
                <div class="zhuyin-compound">
                    <div class="zhuyin-text">${pure.split('').join('<br>')}</div>
                    <div class="tone-part"><span class="tone-mark">${tone}</span></div>
                </div>`;
            display.appendChild(unit);
        });
    }

    // 初始載入
    generateNewWord();
</script>
</body>
</html>
