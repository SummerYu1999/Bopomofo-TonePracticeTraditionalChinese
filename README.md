<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>注音精密教學系統 - 鍵盤快捷版</title>
    <style>
        :root {
            --box-size: 220px;
            --cn-w: 165px;
            --zy-w: 55px;
            --font-cn: 110px;
            --font-zy: 36px;
            --font-tone: 24px;
            --primary: #2c3e50;
            --accent: #d63031;
        }

        body {
            background-color: #f1f3f5;
            display: flex;
            flex-direction: column;
            align-items: center;
            font-family: "標楷體", "DFKai-SB", serif;
            padding: 40px;
            outline: none;
        }

        .controls { margin-bottom: 20px; color: #7f8c8d; font-size: 0.9rem; }
        .hotkey-hint { margin-bottom: 20px; background: #eee; padding: 5px 15px; border-radius: 5px; }

        button {
            padding: 12px 40px;
            font-size: 1.2rem;
            background: var(--primary);
            color: white;
            border: none;
            border-radius: 50px;
            cursor: pointer;
            margin-bottom: 10px;
        }

        .word-row {
            display: flex;
            gap: 20px;
            margin-bottom: 40px;
            justify-content: center;
        }

        .char-unit {
            display: grid;
            grid-template-columns: var(--cn-w) var(--zy-w);
            width: var(--box-size);
            height: var(--box-size);
            background: white;
            border: 2px solid var(--primary);
            cursor: pointer;
            box-sizing: border-box;
            transition: all 0.2s;
        }

        /* 被快捷鍵選中時的視覺反饋 */
        .char-unit.active-trigger {
            border-color: var(--accent);
            box-shadow: 0 0 15px rgba(214, 48, 49, 0.3);
            transform: scale(1.02);
        }

        .cn-zone {
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: var(--font-cn);
            border-right: 1px dashed #ccc;
        }

        .zy-compound {
            display: grid;
            grid-template-columns: 2fr 1fr;
        }

        .zy-symbols {
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            font-size: var(--font-zy);
            line-height: 1.1;
        }

        .tone-zone { position: relative; }

        .tone-mark {
            position: absolute;
            top: 25%;
            left: 0;
            font-size: var(--font-tone);
            color: var(--accent);
            font-weight: bold;
        }

        .panel {
            max-width: 850px;
            width: 100%;
            background: white;
            padding: 30px;
            border-radius: 15px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            border-top: 6px solid var(--primary);
        }

        .active-title {
            font-size: 1.6rem;
            color: var(--accent);
            margin-bottom: 20px;
            border-bottom: 2px solid #eee;
            padding-bottom: 10px;
        }

        .tip-card {
            background: #fdfdfd;
            border-left: 5px solid #3498db;
            padding: 15px;
            margin-bottom: 10px;
        }

        .tag {
            font-weight: bold;
            color: #2980b9;
            margin-right: 10px;
            background: #e1f5fe;
            padding: 2px 8px;
            border-radius: 4px;
        }
    </style>
</head>
<body>

    <div class="hotkey-hint">
        快捷鍵：[空白鍵] 換詞 | [1][2][3] 查看對應單字註釋
    </div>

    <button onclick="refresh()">更換教學單詞</button>

    <div id="display" class="word-row"></div>

    <div class="panel">
        <div id="title" class="active-title">點擊單字或按數字鍵查看詳細發音</div>
        <div id="content"></div>
    </div>

<script>
    // 發音精密字典
    const Dictionary = {
        "ㄌ": "【邊音】舌尖抵住上齒齦，氣流從舌頭兩邊流出。",
        "ㄏ": "【舌根音】舌後根靠近軟顎，讓氣流摩擦而出。",
        "ㄠ": "【複韻母】由ㄚ轉向ㄨ，口型由大變小，過程需滑順。",
        "ㄨ": "【合口呼】雙唇收圓向前突，像吹笛子。",
        "ㄎ": "【舌根音】舌後根頂住軟顎，嘴巴需打開約一個指節寬。",
        "ㄚ": "【開口音】嘴巴要徹底打開，像要塞進大獅子頭一樣。",
        "ㄈ": "【唇齒音】上齒接觸下唇。若沒碰到會變成「ㄏ」。",
        "ㄟ": "【韻母】發音要圓滿，確保口腔內有足夠共鳴空間。",
        "ˇ": "【三聲】音位先降後升，注意下降深度。",
        "ˊ": "【二聲】聲音由中點往上升。"
    };

    const lib = [
        { c: ["老", "虎"], z: ["ㄌㄠˇ", "ㄏㄨˇ"] },
        { c: ["咖", "啡"], z: ["ㄎㄚ", "ㄈㄟ"] },
        { c: ["歡", "迎"], z: ["ㄏㄨㄢ", "ㄧㄥˊ"] }
    ];

    let currentItem = null; // 存儲當前單字狀態

    /**
     * 核心教學顯示邏輯
     */
    function show(index) {
        if (!currentItem || !currentItem.c[index]) return;

        const char = currentItem.c[index];
        const fullZ = currentItem.z[index];

        // 視覺反饋：移除所有高亮再加入新的
        document.querySelectorAll('.char-unit').forEach(el => el.classList.remove('active-trigger'));
        const targetEl = document.querySelectorAll('.char-unit')[index];
        if (targetEl) targetEl.classList.add('active-trigger');

        document.getElementById('title').innerText = `正在學習：「${char}」 (${fullZ})`;
        const container = document.getElementById('content');
        container.innerHTML = "";

        // 拆解符號與匹配字典
        const symbols = fullZ.split("");
        symbols.forEach(s => {
            if (Dictionary[s]) {
                container.innerHTML += `
                    <div class="tip-card">
                        <span class="tag">${s}</span> ${Dictionary[s]}
                    </div>`;
            }
        });

        // ㄅㄆㄇㄈ 規律校驗
        if (["ㄅ","ㄆ","ㄇ","ㄈ"].includes(fullZ[0]) && fullZ.includes("ㄨ")) {
            container.innerHTML += `
                <div class="tip-card" style="border-left-color: #e67e22;">
                    <span class="tag">規務</span> ㄅㄆㄇㄈ不與ㄨ結合。
                </div>`;
        }
    }

    /**
     * 刷新單字並重置面板
     */
    function refresh() {
        document.getElementById('title').innerText = "點擊單字或按數字鍵查看詳細發音";
        document.getElementById('content').innerHTML = "";

        currentItem = lib[Math.floor(Math.random() * lib.length)];
        const display = document.getElementById('display');
        display.innerHTML = "";

        currentItem.c.forEach((char, i) => {
            const z = currentItem.z[i];
            const tone = z.match(/[ˊˇˋ˙]/) ? z.match(/[ˊˇˋ˙]/)[0] : "";
            const pure = z.replace(/[ˊˇˋ˙]/, "");

            const unit = document.createElement('div');
            unit.className = 'char-unit';
            unit.onclick = () => show(i);
            unit.innerHTML = `
                <div class="cn-zone">${char}</div>
                <div class="zy-compound">
                    <div class="zy-symbols">${pure.split("").join("<br>")}</div>
                    <div class="tone-zone"><span class="tone-mark">${tone}</span></div>
                </div>`;
            display.appendChild(unit);
        });
    }

    /**
     * 鍵盤監聽邏輯
     */
    window.addEventListener('keydown', (e) => {
        // 空白鍵更換單字 (排除正在輸入的情況)
        if (e.code === 'Space') {
            e.preventDefault(); // 防止網頁捲動
            refresh();
        }
        
        // 數字鍵 1, 2, 3 對應單字註釋
        if (e.key === '1') show(0);
        if (e.key === '2') show(1);
        if (e.key === '3') show(2);
    });

    refresh();
</script>
</body>
</html>
