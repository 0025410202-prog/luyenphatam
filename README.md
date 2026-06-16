<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>App Luyện Phát Âm Mỹ</title>
    <style>
        * { box-sizing: border-box; }
        body {
            font-family: Arial, sans-serif;
            background-color: #eef2f3;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
        }
        .app-card {
            background: #ffffff;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.08);
            text-align: center;
            width: 100%;
            max-width: 400px;
        }
        h2 { color: #2c3e50; margin-top: 0; font-size: 22px; }
        .word { font-size: 34px; font-weight: bold; color: #2980b9; margin: 10px 0; }
        .ipa { font-size: 18px; color: #7f8c8d; font-style: italic; margin-bottom: 5px; }
        .meaning { font-size: 16px; color: #27ae60; font-weight: bold; margin-bottom: 25px; }
        
        .btn {
            border: none;
            padding: 12px 28px;
            font-size: 16px;
            border-radius: 30px;
            cursor: pointer;
            font-weight: bold;
            transition: 0.2s;
            width: 80%;
            margin: 5px 0;
        }
        #btn-speak { background-color: #e74c3c; color: white; }
        #btn-speak:hover { background-color: #c0392b; }
        #btn-speak:disabled { background-color: #bdc3c7; cursor: not-allowed; }
        
        #btn-next { background-color: #3498db; color: white; display: none; margin-top: 10px; }
        #btn-next:hover { background-color: #2980b9; }

        .result-box { margin-top: 20px; padding-top: 20px; border-top: 1px solid #eee; min-height: 120px; }
        .status { color: #7f8c8d; font-size: 14px; margin-bottom: 8px; }
        .score { font-size: 45px; font-weight: bold; margin: 5px 0; }
        .you-said { color: #34495e; font-style: italic; font-size: 15px; }
        
        /* Màu sắc điểm số */
        .good { color: #27ae60; }
        .normal { color: #f39c12; }
        .bad { color: #c0392b; }
    </style>
</head>
<body>

<div class="app-card">
    <h2>Luyện Phát Âm Chuẩn Mỹ</h2>
    
    <div class="word" id="lbl-word">Loading...</div>
    <div class="ipa" id="lbl-ipa">/.../</div>
    <div class="meaning" id="lbl-meaning">Nghĩa của từ</div>

    <button class="btn" id="btn-speak">🎤 Bấm và Đọc</button>
    <button class="btn" id="btn-next">Từ tiếp theo ➔</button>

    <div class="result-box">
        <div class="status" id="lbl-status">Đang tải danh sách từ...</div>
        <div class="score" id="lbl-score"></div>
        <div class="you-said" id="lbl-said"></div>
    </div>
</div>

<script>
    // Dữ liệu 10 từ của bạn đã chuẩn hóa phiên âm Mỹ
    const dataset = [
        { w: "Physical", i: "/ˈfɪzɪkl/", m: "Về mặt thể chất" },
        { w: "Admit", i: "/ədˈmɪt/", m: "Thú nhận, thừa nhận" },
        { w: "Campaign", i: "/kæmˈpeɪn/", m: "Chiến dịch" },
        { w: "Bully", i: "/ˈbʊli/", m: "Bắt nạt" },
        { w: "Crime", i: "/kraɪm/", m: "Tội phạm / Hành vi phạm tội" },
        { w: "Alcohol", i: "/ˈælkəhɔːl/", m: "Đồ uống có cồn (rượu, bia)" },
        { w: "Depression", i: "/dɪˈpreʃn/", m: "Sự/Tình trạng trầm cảm" },
        { w: "Self-confidence", i: "/self ˈkɑːnfɪdəns/", m: "Sự tự tin vào bản thân" },
        { w: "Poverty", i: "/ˈpɑːvərti/", m: "Sự nghèo đói" },
        { w: "Make fun of", i: "/meɪk fʌn əv/", m: "Trêu chọc, chế giễu" }
    ];

    let index = 0;

    // Kiểm tra tính năng nhận diện giọng nói của trình duyệt
    const SpeechAPI = window.SpeechRecognition || window.webkitSpeechRecognition;
    if (!SpeechAPI) {
        alert("LỖI: Trình duyệt này không hỗ trợ nhận diện giọng nói. Hãy dùng Google Chrome!");
    }

    const recognizer = new SpeechAPI();
    recognizer.lang = 'en-US'; // Ép cấu hình giọng Anh - Mỹ
    recognizer.interimResults = false;

    // Lấy các thẻ HTML
    const lblWord = document.getElementById('lbl-word');
    const lblIpa = document.getElementById('lbl-ipa');
    const lblMeaning = document.getElementById('lbl-meaning');
    const btnSpeak = document.getElementById('btn-speak');
    const btnNext = document.getElementById('btn-next');
    const lblStatus = document.getElementById('lbl-status');
    const lblScore = document.getElementById('lbl-score');
    const lblSaid = document.getElementById('lbl-said');

    // Hàm hiển thị từ vựng lên màn hình
    function showWord() {
        lblWord.textContent = dataset[index].w;
        lblIpa.textContent = dataset[index].i;
        lblMeaning.textContent = dataset[index].m;
        
        lblScore.textContent = "";
        lblSaid.textContent = "";
        lblStatus.textContent = "Nhấn nút màu đỏ, đợi hiện chữ 'Hãy nói đi' rồi đọc nhé.";
        btnNext.style.display = "none";
        btnSpeak.disabled = false;
    }

    // Sự kiện bấm nút Nói
    btnSpeak.addEventListener('click', () => {
        try {
            recognizer.start();
            lblStatus.textContent = "🎙️ Hãy nói đi... Máy đang lắng nghe bạn";
            btnSpeak.disabled = true;
        } catch (e) {
            lblStatus.textContent = "Có lỗi xảy ra, vui lòng thử lại.";
            btnSpeak.disabled = false;
        }
    });

    // Khi máy thu âm xong và trả về kết quả text
    recognizer.onresult = (event) => {
        // Lấy từ người dùng nói, viết thường, xóa dấu chấm câu thừa
        const userText = event.results[0][0].transcript.toLowerCase().replace(/[.,\/#!$%\^&\*;:{}=\-_`~()]/g,"").trim();
        const targetText = dataset[index].w.toLowerCase().replace(/[.,\/#!$%\^&\*;:{}=\-_`~()]/g,"").trim();

        lblSaid.textContent = `Máy nghe được bạn nói: "${event.results[0][0].transcript}"`;

        // Tính điểm phần trăm trùng khớp
        const score = matchScore(userText, targetText);
        lblScore.textContent = score + "%";

        // Đổi màu theo mức điểm
        if (score >= 80) {
            lblScore.className = "score good";
            lblStatus.textContent = "🌟 Tuyệt vời! Bạn phát âm rất chuẩn.";
        } else if (score >= 50) {
            lblScore.className = "score normal";
            lblStatus.textContent = "👍 Khá tốt! Cố gắng nghe lại phiên âm để chuẩn hơn.";
        } else {
            lblScore.className = "score bad";
            lblStatus.textContent = "❌ Sai rồi! Hãy bấm nói lại hoặc chuyển từ tiếp theo.";
        }

        btnSpeak.disabled = false;
        btnNext.style.display = "inline-block";
    };

    // Xử lý nếu thu âm bị lỗi (Ví dụ: bấm nút nhưng im lặng không nói gì)
    recognizer.onerror = () => {
        lblStatus.textContent = "⚠️ Máy không nghe thấy bạn nói gì. Vui lòng bấm làm lại.";
        btnSpeak.disabled = false;
    };

    recognizer.onspeechend = () => {
        recognizer.stop();
    };

    // Nút chuyển từ tiếp theo
    btnNext.addEventListener('click', () => {
        index = (index + 1) % dataset.length;
        showWord();
    });

    // Thuật toán so sánh độ chính xác giữa văn bản bạn nói và từ gốc
    function matchScore(s1, s2) {
        if (s1 === s2) return 100;
        let longer = s1.length > s2.length ? s1 : s2;
        let shorter = s1.length > s2.length ? s2 : s1;
        let longerLength = longer.length;
        if (longerLength === 0) return 100;
        
        // Tính khoảng cách Levenshtein
        let costs = new Array();
        for (let i = 0; i <= s1.length; i++) {
            let lastValue = i;
            for (let j = 0; j <= s2.length; j++) {
                if (i == 0) costs[j] = j;
                else {
                    if (j > 0) {
                        let newValue = costs[j - 1];
                        if (s1.charAt(i - 1) != s2.charAt(j - 1))
                            newValue = Math.min(Math.min(newValue, lastValue), costs[j]) + 1;
                        costs[j - 1] = lastValue;
                        lastValue = newValue;
                    }
                }
            }
            if (i > 0) costs[s2.length] = lastValue;
        }
        return Math.round(((longerLength - costs[s2.length]) / longerLength) * 100);
    }

    // Chạy ứng dụng khi vừa mở trang lên
    showWord();
</script>

</body>
</html>
