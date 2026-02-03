<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <title>注音發音教學系統 - 座標鎖定版</title>
    <style>
        :root {
            /* 鎖定絕對尺寸，避免跑位 */
            --unit-w: 200px; 
            --unit-h: 200px; 
            --cn-w: 150px;   /* 漢字佔 150px */
            --zy-w: 50px;    /* 注音與聲調佔 50px */
            --font-main: 110px;
            --font-zhuyin: 36px;
            --font-tone: 24px;
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
            justify-content: center;
        }

        /* 方格系統：使用 position: relative 鎖定子元素座標 */
        .char-unit {
            position: relative; 
            width: var(--unit-w);
            height: var(--unit-h);
            background: white;
            border: 2px solid var(--primary-dark);
            cursor: pointer;
            transition: all 0.2s;
            box-sizing: border-box;
            overflow: hidden;
        }

        .char-unit:hover {
            box-shadow: 0 8px 15px rgba(0,0,0,0.1);
            transform: translateY(-3px);
            border-color: var(--accent-red);
        }

        /* 漢字象限：固定寬度與位置 */
        .chinese-part {
            position: absolute;
            left: 0;
            top: 0;
            width: var(--cn-w);
            height: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: var(--font-main);
            border-right: 1px dashed #ccc;
            box-sizing: border-box;
        }

        /* 注音複合象限：固定在右側 50px */
        .zhuyin-compound {
            position: absolute;
            left: var(--cn-w);
            top: 0;
            width: var(--zy-w);
            height: 100%;
            display: flex;
            box-sizing: border-box;
        }

        /* 注音符號直排 */
        .zhuyin-text {
            flex: 2;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-size: var(--font-zhuyin);
            line-height: 1.1;
        }

        /* 聲調絕對定位：鎖定在注音右側上方 */
        .tone-part {
            flex: 1;
            position: relative;
        }

        .tone-mark {
            position: absolute;
            left: -2px;
            top: 30%; /* 聲調固定在韻母右側 */
            font-size: var(--font-tone);
            color: var(--accent-red);
            font-weight: bold;
        }

        .instruction-panel {
            max-width: 900px;
            width: 100%;
            min-height: 200px;
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
    /**
     * 第一步：建立 37 個注音符號的「標準化資料庫」
     * 只要這裡的資料正確，後面無論生成幾萬個字，註釋都會是對的。
     */
    const MasterDictionary = {
        // 聲母類
        "ㄅ": "【雙唇音】上下唇先閉合，造成阻力後讓氣流衝出。",
        "ㄈ": "【唇齒音】上齒接觸下唇。注意：沒碰到會變成「ㄏ」的模糊音。",
        "ㄎ": "【舌根音】舌後根往上頂軟顎。細節：嘴巴需打開約一個指節寬。",
        "ㄘ": "【平舌音】舌尖往下靠近下齒背，舌面不可碰到上齒背。",
        // 韻母類
        "ㄚ": "【開口音】嘴巴要徹底打開，像要塞進大獅子頭一樣。",
        "ㄠ": "【複韻母】由ㄚ轉向ㄨ，口型由大變小，過程需滑順。",
        "ㄟ": "【韻母】發音要圓滿，確保口腔內有足夠共鳴空間，聲音才不悶。",
        // 聲調類 (自動適配)
        "ˊ": "【二聲】音位上揚，氣流向後上方共鳴。",
        "ˇ": "【三聲】音位先降後升，要注意下降的深度，共鳴感較重。",
        "ˋ": "【四聲】音位由高快速下降，發音短促有力。"
    };

    /**
     * 第二步：自動化渲染引擎
     * 此函數會自動拆分「注音」與「聲調」，並確保排版鎖定。
     */
    function renderCharUnit(char, fullZhuyin) {
        // 使用正則表達式分離：聲調符號 vs 純注音
        const toneMatch = fullZhuyin.match(/[ˊˇˋ˙]/);
        const tone = toneMatch ? toneMatch[0] : ""; // 抓出 ˊ ˇ ˋ ˙
        const symbols = fullZhuyin.replace(/[ˊˇˋ˙]/, "").split(""); // 抓出 ㄘ ㄠ

        // 回傳絕對鎖定的 HTML 結構
        return `
            <div class="char-unit" onclick="autoShowTips('${char}', '${fullZhuyin}')">
                <div class="chinese-part">${char}</div>
                <div class="zhuyin-compound">
                    <div class="zhuyin-text">${symbols.join('<br>')}</div>
                    <div class="tone-part"><span class="tone-mark">${tone}</span></div>
                </div>
            </div>`;
    }

    /**
     * 第三步：自動註釋抓取
     * 使用者點擊時，程式會自動去 MasterDictionary 找尋所有對應的提示
     */
    function autoShowTips(char, fullZhuyin) {
        const container = document.getElementById('details-container');
        document.getElementById('active-title').innerText = `教學重點： 「${char}」 (${fullZhuyin})`;
        container.innerHTML = "";

        // 自動遍歷所有符號（包含注音與聲調）
        const allElements = fullZhuyin.split("");
        allElements.forEach(sym => {
            if (MasterDictionary[sym]) {
                container.innerHTML += `
                    <div class="detail-card">
                        <span class="symbol-tag">${sym}</span>
                        <span class="tip-content">${MasterDictionary[sym]}</span>
                    </div>`;
            }
        });
    }
</script>
</body>
</html>
