<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>生活單詞發音教學</title>
    <style>
        :root {
            --paper-bg: #fdfdfb;
            --ink-color: #2d3436;
            --zhuyin-color: #d63031;
        }

        body {
            background-color: #e5e5e5;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            font-family: "標楷體", "DFKai-SB", serif;
        }

        .main-card {
            background: var(--paper-bg);
            padding: 40px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            display: flex;
            gap: 40px;
            max-width: 900px;
            align-items: flex-start;
        }

        /* 漢字與注音排版關鍵 */
        .display-area {
            flex: 1;
            text-align: center;
            border-right: 1px dashed #ccc;
            padding-right: 40px;
        }

        ruby {
            font-size: 8rem;
            ruby-position: ruby-text; /* 讓注音在右側的關鍵 */
            display: inline-flex;
            margin: 0 10px;
        }

        rt {
            font-size: 2.2rem;
            color: var(--zhuyin-color);
            writing-mode: vertical-rl; /* 垂直排列注音 */
            text-orientation: upright;
            padding-left: 0.5rem;
            line-height: 1.1;
        }

        /* 構造圖與提示區 */
        .info-area {
            flex: 1;
            display: flex;
            flex-direction: column;
        }

        .anatomy-img {
            width: 100%;
            max-width: 250px;
            border: 1px solid #ddd;
            margin-bottom: 20px;
        }

        .hint-text {
            background: #fff9db;
            padding: 15px;
            border-radius: 8px;
            font-size: 1.1rem;
            line-height: 1.6;
            color: #5d4037;
        }

        button {
            margin-top: 20px;
            padding: 12px 25px;
            font-size: 1.2rem;
            cursor: pointer;
            background: var(--ink-color);
            color: white;
            border: none;
            border-radius: 5px;
        }
    </style>
</head>
<body>

<div class="main-card">
    <div class="display-area">
        <div id="word-box">
            <ruby>歡迎<rt>ㄏㄨㄢㄧㄥˊ</rt></ruby>
        </div>
        <button onclick="nextWord()">隨機更換單詞</button>
    </div>

    <div class="info-area">
        <h3>發音部位提示</h3>
        <img class="anatomy-img" src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/4e/VocalLines.png/300px-VocalLines.png" alt="發音部位圖">
        <div id="hint-box" class="hint-text">
            請點擊按鈕開始。
        </div>
    </div>
</div>

<script>
    // 詞庫資料：[漢字, 注音, 提示]
    const wordLib = [
        {w: "草莓", z: "ㄘㄠˇㄇㄟˊ", h: "<b>ㄘ (平舌音)：</b>舌尖抵住下齒背。<br><b>ㄇ (雙唇音)：</b>雙唇閉合發音。"},
        {w: "電腦", z: "ㄉㄧㄢˋㄋㄠˇ", h: "<b>ㄉ (舌尖音)：</b>舌尖抵住上齒齦。<br><b>ㄋ (鼻音)：</b>氣流由鼻腔洩出。"},
        {w: "葡萄", z: "ㄆㄨˊㄊㄠˊ", h: "<b>ㄆ (雙唇音)：</b>雙唇噴氣發音。<br><b>ㄊ (舌尖音)：</b>舌尖抵住上齒齦。"},
        {w: "咖啡", z: "ㄎㄚㄈㄟ", h: "<b>ㄎ (舌根音)：</b>舌根抬起靠近軟顎。<br><b>ㄈ (唇齒音)：</b>上齒輕咬下唇。"},
        {w: "獅子", z: "ㄕㄗ˙", h: "<b>ㄕ (翹舌音)：</b>舌尖翹起靠近硬顎。<br><b>ㄗ (平舌音)：</b>舌尖平伸靠近齒背。"}
    ];

    function nextWord() {
        const item = wordLib[Math.floor(Math.random() * wordLib.length)];
        const box = document.getElementById('word-box');
        const hint = document.getElementById('hint-box');

        // 為了達到你圖片中的精確對齊，如果是多個字，建議拆開標記
        // 這邊示範合併標記，大部分瀏覽器能自動對應
        box.innerHTML = `<ruby>${item.w}<rt>${item.z}</rt></ruby>`;
        hint.innerHTML = item.h;
    }
</script>

</body>
</html>
