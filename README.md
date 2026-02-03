<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <title>注音發音精密教學網頁</title>
    <style>
        :root {
            --grid-size: 220px;
            --font-main: 120px;
            --font-zhuyin: 38px;
            --font-tone: 22px;
            --border-color: #57606f;
            --highlight-color: #d63031;
        }

        body {
            background-color: #f1f2f6;
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 40px;
            font-family: "標楷體", "DFKai-SB", serif;
        }

        /* 方格系統：確保象限固定 */
        .word-row {
            display: flex;
            gap: 35px;
            margin-bottom: 40px;
        }

        .char-unit {
            display: flex;
            width: var(--grid-size);
            height: var(--grid-size);
            background: white;
            border: 2px solid var(--border-color);
            position: relative;
            box-shadow: 5px 5px 0px rgba(0,0,0,0.05);
        }

        /* 繁體中文象限 (左) */
        .chinese-part {
            flex: 4;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: var(--font-main);
            border-right: 1px dashed #ccc;
        }

        /* 注音複合象限 (右) */
        .zhuyin-part {
            flex: 1.2;
            display: flex;
            position: relative;
        }

        /* 注音文字垂直排列 */
        .zhuyin-text {
            flex: 2;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-size: var(--font-zhuyin);
            line-height: 1.1;
        }

        /* 聲調絕對定位：標在右上方 */
        .tone-part {
            flex: 1;
            position: relative;
        }

        .tone-mark {
            position: absolute;
            left: -2px;
            top: 25%; /* 根據台灣排版習慣，聲調通常偏上方 */
            font-size: var(--font-tone);
            color: var(--highlight-color);
            font-weight: bold;
        }

        /* 教學提示區域 */
        .tutorial-panel {
            max-width: 850px;
            width: 100%;
            background: #ffffff;
            border-left: 8px solid #2f3542;
            padding: 25px;
            border-radius: 4px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
        }

        .anatomy-img {
            float: left;
            width: 220px;
            margin-right: 25px;
            border: 1px solid #eee;
        }

        .hint-item {
            margin-bottom: 15px;
            font-size: 1.2rem;
            line-height: 1.6;
        }

        .hint-category {
            color: #2980b9;
            font-weight: bold;
            display: block;
            margin-bottom: 5px;
        }

        button {
            padding: 15px 40px;
            font-size: 1.3rem;
            background: #2f3542;
            color: white;
            border: none;
            cursor: pointer;
            margin-bottom: 30px;
            border-radius: 4px;
        }
        button:hover { background: #57606f; }
    </style>
</head>
<body>

    <button onclick="generateWord()">更換單詞</button>

    <div id="word-display" class="word-row"></div>

    <div class="tutorial-panel">
        <img class="anatomy-img" src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/4e/VocalLines.png/300px-VocalLines.png" alt="人體構造圖">
        <div id="hint-content"></div>
        <div style="clear:both;"></div>
    </div>

<script>
    // 根據您提供的詳細發音提示整理的資料庫
    const pronunciationTips = {
        "ㄅ": "【雙唇音】發音時上下唇要先閉合，造成阻力後再讓氣流衝出。",
        "ㄆ": "【雙唇音】發音時上下唇要先閉合，造成阻力後再讓氣流衝出。",
        "ㄇ": "【雙唇音】發音時上下唇要先閉合，造成阻力後再讓氣流衝出。",
        "ㄈ": "【唇齒音】上齒接觸下唇發出的聲音。注意若上齒沒碰到下唇，聲音會模糊。",
        "ㄉ": "【舌尖中音】利用舌尖位置（牙齦或硬顎前部）造成阻礙發音。",
        "ㄊ": "【舌尖中音】利用舌尖位置（牙齦或硬顎前部）造成阻礙發音。",
        "ㄋ": "【鼻音】共鳴在鼻腔，念時鼻翼會有震動感。",
        "ㄌ": "【邊音】舌尖抵住上齒齦，氣流從舌頭兩邊流出，鼻腔不震動。",
        "ㄍ": "【舌根音】舌後根往上頂軟顎。嘴巴需打開約一個指節寬。",
        "ㄎ": "【舌根音】舌後根往上頂軟顎。嘴巴需打開約一個指節寬。",
        "ㄏ": "【舌根音】舌後根靠近軟顎。嘴巴需打開約一個指節寬。",
        "ㄐ": "【舌面音】舌面與硬軟顎中間接觸，氣流從舌面中間衝出。",
        "ㄒ": "【舌面音】必須是舌面貼住硬顎，與英文 C 不同。",
        "ㄓ": "【翹舌音】舌尖往後捲，指向硬顎位置。",
        "ㄖ": "【翹舌音】舌尖垂直放，不可碰到牙齒。",
        "ㄗ": "【平舌音】舌尖往前下靠近下齒背，舌面不可碰到上齒背。",
        "ㄜ": "【韻母】嘴巴呈現鴨蛋長橢圓形，舌面下壓，像含著湯匙。",
        "一": "【介符】嘴角往兩旁拉開，做出明顯「一」字形。",
        "ㄨ": "【結合規律】ㄅㄆㄇㄈ標準發音中不與ㄨ結合（如：ㄆㄨㄥˊ為誤音）。",
        "ㄣ": "【鼻韻母】氣流共鳴在鼻腔前部。",
        "ㄥ": "【鼻韻母】舌後根頂住軟顎，共鳴腔在脖子後方與鼻腔後部。"
    };

    const wordLib = [
        { chars: ["老", "虎"], zhuyin: ["ㄌㄠˇ", "ㄏㄨˇ"] },
        { chars: ["朋友", " "], zhuyin: ["ㄆㄥˊ", "ㄧㄡˇ"] }, // 示範ㄇㄈ不接ㄨ的規則
        { chars: ["咖", "啡"], zhuyin: ["ㄎㄚ", "ㄈㄟ"] },
        { chars: ["牛", "奶"], zhuyin: ["ㄋㄧㄡˊ", "ㄋㄞˇ"] },
        { chars: ["風", "扇"], zhuyin: ["ㄈㄥ", "ㄕㄢˋ"] }
    ];

    function generateWord() {
        const item = wordLib[Math.floor(Math.random() * wordLib.length)];
        const display = document.getElementById('word-display');
        const hintBox = document.getElementById('hint-content');

        display.innerHTML = "";
        let collectedTips = new Set();

        item.chars.forEach((c, i) => {
            if(c.trim() === "") return;
            
            // 拆解注音與聲調
            const fullZ = item.zhuyin[i];
            const toneMark = fullZ.match(/[ˊˇˋ˙]/) ? fullZ.match(/[ˊˇˋ˙]/)[0] : "";
            const pureZ = fullZ.replace(/[ˊˇˋ˙]/, "");

            // 建立方格
            const unit = document.createElement('div');
            unit.className = 'char-unit';
            unit.innerHTML = `
                <div class="chinese-part">${c}</div>
                <div class="zhuyin-part">
                    <div class="zhuyin-text">${pureZ.split('').join('<br>')}</div>
                    <div class="tone-part"><span class="tone-mark">${toneMark}</span></div>
                </div>
            `;
            display.appendChild(unit);

            // 蒐集發音提示
            pureZ.split('').forEach(char => {
                if(pronunciationTips[char]) collectedTips.add(pronunciationTips[char]);
            });
        });

        // 顯示提示
        hintBox.innerHTML = Array.from(collectedTips).map(t => `<div class="hint-item">${t}</div>`).join('');
    }

    generateWord();
</script>
</body>
</html>
