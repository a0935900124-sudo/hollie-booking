<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HOLLIE | 預約系統</title>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@300;400;500&family=Playfair+Display:italic,wght@600&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg-cream: #F9F6F2;      /* 奶白色背景 */
            --main-brown: #A69080;    /* 奶茶色線條與文字 */
            --light-line: #E3DED8;    /* 淺色裝飾線 */
            --booked-bg: #EAEAEA;     /* 被預約走的灰色 */
            --white: #FFFFFF;
        }

        * { margin: 0; padding: 0; box-sizing: border-box; }

        body {
            background-color: var(--bg-cream);
            font-family: 'Noto Sans TC', sans-serif;
            color: var(--main-brown);
            display: flex;
            justify-content: center;
            padding: 40px 20px;
        }

        /* 主容器 */
        .card {
            background: var(--white);
            width: 100%;
            max-width: 400px;
            padding: 45px 25px;
            border-radius: 40px;
            position: relative;
            border: 1px solid var(--light-line);
            box-shadow: 0 15px 45px rgba(166, 144, 128, 0.1);
        }

        /* 裝飾框線 */
        .card::after {
            content: "";
            position: absolute;
            top: 12px; left: 12px; right: 12px; bottom: 12px;
            border: 0.5px solid var(--light-line);
            border-radius: 32px;
            pointer-events: none;
        }

        header {
            text-align: center;
            margin-bottom: 40px;
        }

        .logo {
            font-family: 'Playfair Display', serif;
            font-size: 2.8rem;
            font-style: italic;
            letter-spacing: 2px;
            margin-bottom: 5px;
        }

        .tagline {
            font-size: 0.75rem;
            letter-spacing: 3px;
            text-transform: uppercase;
            opacity: 0.8;
        }

        /* 表單區域 */
        .form-section { margin-bottom: 25px; }
        .label-text {
            font-size: 0.85rem;
            font-weight: 500;
            margin-bottom: 12px;
            display: block;
            border-left: 2px solid var(--main-brown);
            padding-left: 10px;
        }

        input[type="date"], input[type="text"] {
            width: 100%;
            padding: 14px;
            background: var(--bg-cream);
            border: none;
            border-radius: 12px;
            color: var(--main-brown);
            font-size: 0.95rem;
            outline: none;
        }

        /* 時段網格 */
        .time-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 10px;
        }

        .time-item {
            padding: 12px 0;
            text-align: center;
            border: 1px solid var(--light-line);
            border-radius: 10px;
            font-size: 0.85rem;
            cursor: pointer;
            transition: all 0.3s ease;
            background: var(--white);
        }

        .time-item:hover:not(.booked) {
            border-color: var(--main-brown);
            background: #FAF9F7;
        }

        .time-item.selected {
            background: var(--main-brown);
            color: var(--white);
            border-color: var(--main-brown);
        }

        .time-item.booked {
            background-color: var(--booked-bg);
            color: #B0B0B0;
            border: 1px dashed #D1D1D1;
            cursor: not-allowed;
            position: relative;
        }

        .time-item.booked::after {
            content: "已滿";
            font-size: 0.6rem;
            display: block;
            margin-top: 2px;
        }

        /* 確認按鈕 */
        .btn-submit {
            width: 100%;
            padding: 18px;
            background: var(--main-brown);
            color: var(--white);
            border: none;
            border-radius: 50px;
            font-size: 1rem;
            letter-spacing: 2px;
            cursor: pointer;
            margin-top: 20px;
            transition: 0.3s ease;
            box-shadow: 0 5px 15px rgba(166, 144, 128, 0.3);
        }

        .btn-submit:hover {
            transform: translateY(-2px);
            box-shadow: 0 8px 20px rgba(166, 144, 128, 0.4);
        }

        footer {
            text-align: center;
            margin-top: 30px;
            font-size: 0.8rem;
        }

        footer a {
            color: var(--main-brown);
            text-decoration: none;
            border-bottom: 1px solid var(--main-brown);
            padding-bottom: 2px;
        }
    </style>
</head>
<body>

<div class="card">
    <header>
        <div class="logo">Hollie</div>
        <div class="tagline">Online Booking</div>
    </header>

    <div class="form-section">
        <label class="label-text">Select Date</label>
        <input type="date" id="dateInput">
    </div>

    <div class="form-section">
        <label class="label-text">Select Time (11:30 - 18:30)</label>
        <div class="time-grid" id="timeGrid">
            </div>
    </div>

    <div class="form-section">
        <label class="label-text">Your Name</label>
        <input type="text" id="nameInput" placeholder="請輸入預約姓名">
    </div>

    <button class="btn-submit" onclick="handleBooking()">RESERVE NOW</button>

    <footer>
        <p>如有問題請洽 <a href="https://line.me/R/ti/p/@530bowcg" target="_blank">LINE @530bowcg</a></p>
    </footer>
</div>

<script>
    // 預約設定
    const bookingTimes = [
        "11:30", "12:30", "13:30", "14:30", 
        "15:30", "16:30", "17:30", "18:30"
    ];

    // 模擬後端數據：這裡填入「已經被預約走」的時間
    const alreadyBooked = ["13:30", "15:30"]; 

    const timeGrid = document.getElementById('timeGrid');
    let selectedTime = null;

    // 初始化時間格
    function initTimes() {
        timeGrid.innerHTML = '';
        bookingTimes.forEach(time => {
            const div = document.createElement('div');
            div.className = 'time-item';
            
            if (alreadyBooked.includes(time)) {
                div.classList.add('booked');
                div.innerText = time;
            } else {
                div.innerText = time;
                div.onclick = () => selectTime(div, time);
            }
            timeGrid.appendChild(div);
        });
    }

    function selectTime(element, time) {
        document.querySelectorAll('.time-item').forEach(el => el.classList.remove('selected'));
        element.classList.add('selected');
        selectedTime = time;
    }

    function handleBooking() {
        const date = document.getElementById('dateInput').value;
        const name = document.getElementById('nameInput').value;

        if (!date || !selectedTime || !name) {
            alert('請完整填寫日期、時間與姓名喔！');
            return;
        }

        // 整理預約訊息
        const msg = `您好，我想預約 HOLLIE！\n\n【預約資料】\n日期：${date}\n時間：${selectedTime}\n姓名：${name}`;
        
        // 編碼訊息並跳轉至 LINE
        const lineUrl = `https://line.me/R/ti/p/@530bowcg`;
        
        alert('預約資訊已準備好，將為您導向 LINE 進行確認');
        window.location.href = lineUrl;
    }

    // 預設今天日期
    document.getElementById('dateInput').valueAsDate = new Date();
    initTimes();
</script>

</body>
</html>
