<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <title>注音發音構造教學系統</title>
    <style>
        :root {
            --paper-bg: #fdfdfb;
            --ink-color: #1a1a1a;
            --ruby-color: #cd6133;
        }

        body {
            background-color: #e0e0e0;
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
            font-family: "標楷體", "DFKai-SB", serif;
        }

        /* 主容器 */
        .main-container {
            background-color: var(--paper-bg);
            padding: 40px;
            border-radius: 8px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.15);
            display: flex;
            flex-direction: column;
            align-items: center;
            max-width: 900px;
            width: 100%;
        }

        /* 構造圖區域 */
        .anatomy-box {
            display: flex;
            align-items: center;
            gap: 20px;
            margin-bottom: 40px;
            border-bottom: 2px dashed #ccc;
            padding-bottom: 30px;
            width: 100%;
        }

        #anatomy-img {
            width: 250px;
            height: auto;
            border: 1px solid #ddd;
        }

        .hint-text {
            flex: 1;
            font-size: 1.4rem;
            line-height: 1.6;
            color: #444;
            background: #fff9e6;
            padding: 15px;
            border-radius: 10px;
        }

        /* 核心排版：如附圖般的字音對齊 */
        .word-display-area {
            display: flex;
            justify-content: center;
            gap: 40px; /* 控制字與字之間的適當間距 */
            margin: 40px 0;
        }

        ruby {
            display: inline-flex;
            align-items: center;
            font-size: 8rem; /* 漢字大小 */
            line-height: 1;
        }

        rt {
            display: block;
            font-size: 2.2rem; /* 注音大小 */
            writing-mode: vertical-rl; /* 直排 */
            text-orientation: upright;
            color: var(--ruby-color);
            margin-left: 15px; /* 字與注音的間距 */
            line-height: 1.1;
            letter-spacing: -2px; /* 讓注音緊湊一點，更像附圖 */
        }

        button {
            padding: 15px 50px;
            font-size: 1.5rem;
            background-color: #2f3542;
            color: white;
            border: none;
            border-radius: 50px;
            cursor: pointer;
            transition: 0.3s;
        }

        button:hover { background-color: #57606f; }
        .highlight { color: #d63031; font-weight: bold; border-bottom: 2px solid; }
    </style>
</head>
<body>

<div class="main-container">
    <div class="anatomy-box">
        <img id="anatomy-img" src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/4e/VocalLines.png/300px-VocalLines.png" alt="發音構造圖">
        <div id="hint-display" class="hint-text">點擊下方按鈕開始學習發音部位。</div>
    </div>

    <div id="word-area" class="word-display-area">
        <ruby>歡<rt>ㄏㄨㄢ</rt></ruby>
        <ruby>迎<rt>ㄧㄥˊ</rt></ruby>
    </div>

    <button onclick="generateWord()">下一個日常單詞</button>
</div>

<script>
    const wordLib = [
        {
            chars: ["草", "莓"],
            zhuyin: ["ㄘㄠˇ", "ㄇㄟˊ"],
            hint: "「ㄘ」是<span class='highlight'>平舌音</span>，舌尖靠近上齒背；「ㄇ」是<span class='highlight'>雙唇音</span>，雙唇閉合。"
        },
        {
            chars: ["長", "頸", "鹿"],
            zhuyin: ["ㄔㄤˊ", "ㄐㄧㄥˇ", "ㄌㄨˋ"],
            hint: "「ㄔ」是<span class='highlight'>翹舌音</span>，舌尖往上顎翹；「ㄐ」是<span class='highlight'>舌面音</span>，舌面抬起。"
        },
        {
            chars: ["咖", "啡"],
            zhuyin: ["ㄎㄚ", "ㄈㄟ"],
            hint: "「ㄎ」是<span class='highlight'>舌根音</span>，舌根隆起抵住軟顎；「ㄈ」是<span class='highlight'>唇齒音</span>，上齒輕觸下唇。"
        }
    ];

    function generateWord() {
        const item = wordLib[Math.floor(Math.random() * wordLib.length)];
        const wordArea = document.getElementById('word-area');
        const hintDisplay = document.getElementById('hint-display');

        let html = "";
        for (let i = 0; i < item.chars.length; i++) {
            html += `<ruby>${item.chars[i]}<rt>${item.zhuyin[i]}</rt></ruby>`;
        }

        wordArea.innerHTML = html;
        hintDisplay.innerHTML = item.hint;
    }
</script>

</body>
</html>
