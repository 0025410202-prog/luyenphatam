<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>App Chấm Điểm Phát Âm Anh-Mỹ</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f0f2f5;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
        }
        .container {
            background: white;
            padding: 30px;
            border-radius: 16px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            text-align: center;
            max-width: 450px;
            width: 100%;
        }
        h1 { color: #333; font-size: 24px; margin-bottom: 20px; }
        .word-box {
            font-size: 32px;
            font-weight: bold;
            color: #1a73e8;
            margin-bottom: 5px;
        }
        .ipa-box {
            font-size: 18px;
            color: #5f6368;
            margin-bottom: 5px;
            font-style: italic;
        }
        .meaning-box {
            font-size: 16px;
            color: #202124;
            margin-bottom: 25px;
            font-weight: 500;
        }
        .btn {
            background-color: #1a73e8;
            color: white;
            border: none;
            padding: 12px 24px;
            font-size: 16px;
            border-radius: 24px;
            cursor: pointer;
            transition: 0.3s;
            font-weight: bold;
        }
        .btn:hover { background-color: #1557b0; }
        .btn:disabled { background-color: #ccc; cursor: not-allowed; }
        .btn-next { background-color: #34a853; margin-top: 15px;}
        .btn-next:hover { background-color: #2b8a43; }
        .result-container { margin-top: 25px; min-height: 100px; }
        .status { font-weight: bold; color: #5f6368; margin-bottom: 10px; }
        .score { font-size: 40px; font-weight: bold; margin: 10px 0; }
        .user-text { color: #5f6368; font-style: italic; }
        .success { color: #2b8a43; }
        .warning { color: #e37400; }
        .danger { color: #c5221f; }
    </style>
</head>
<body>

<div class="container">
    <h1>Phát Âm Chuẩn Mỹ (AI)</h1>
    
    <div class="word-box" id="target-word">Loading...</div>
    <div class="ipa-box" id="ipa-word">/.../</div>
    <div class="meaning-box" id="meaning-word">Nghĩa của từ</div>

    <button class="btn" id="btn-speak">🎤 Bấm để Nói</button>
    <br>
    <button class="btn btn-next" id="btn-next" style="display:none;">Từ tiếp theo ➔</button>

    <div class="result-container">
        <div class="status" id="status">Sẵn sàng! Hãy bấm nút và đọc từ phía trên.</div>
        <div class="score" id="score"></div>
        <div class="user-text" id="user-text"></div>
    </div>
</div>

<script>
    // Danh sách 10 từ vựng của bạn
    const wordList = [
        { word: "Physical", ipa: "/ˈfɪzɪkl/", meaning: "Về mặt thể chất (adj)" },
        { word: "Admit", ipa: "/ədˈmɪt/", meaning: "Thú nhận, thừa nhận (verb)" },
        { word: "Campaign", ipa: "/kæmˈpeɪn/", meaning: "Chiến dịch (noun)" },
        { word: "Bully", ipa: "/ˈbʊli/", meaning: "Bắt nạt (verb)" },
        { word: "Crime", ipa: "/kraɪm/", meaning: "Tội phạm / Hành vi phạm tội (noun)" },
        { word: "Alcohol", ipa: "/ˈælkəhɔːl/", meaning: "Đồ uống có cồn - rượu, bia (noun)" },
        { word: "Depression", ipa: "/dɪˈpreʃn/", meaning: "Sự/Tình trạng trầm cảm (noun)" },
        { word: "Self-confidence", ipa: "/self ˈkɑːnfɪdəns/", meaning: "Sự tự tin vào bản thân (noun)" },
        { word: "Poverty", ipa: "/ˈpɑːvərti/", meaning: "Sự nghèo đói (noun)" },
        { word: "Make fun of", ipa: "/meɪk fʌn əv/", meaning: "Trêu chọc, chế giễu (phrase)" }
    ];

    let currentIndex = 0;

    // Khởi tạo Google Web Speech API
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    if (!SpeechRecognition) {
        alert("Trình duyệt của bạn không hỗ trợ Web Speech API. Hãy dùng Google Chrome hoặc Edge nhé!");
    }
    
    const recognition = new SpeechRecognition();
    recognition.lang = 'en-US'; 
    recognition.interimResults = false;
    recognition.maxAlternatives = 1;

    // DOM Elements
    const targetWordEl = document.getElementById('target-word');
    const ipaWordEl = document.getElementById('ipa-word');
    const meaningWordEl = document.getElementById('meaning-word');
    const btnSpeak = document.getElementById('btn-speak');
    const btnNext = document.getElementById('btn-next');
    const statusEl = document.getElementById('status');
    const scoreEl = document.getElementById('score');
    const userTextEl = document.getElementById('user-text');

    // Hiển thị từ hiện tại
    function loadWord() {
        targetWordEl.textContent = wordList[currentIndex].word;
        ipaWordEl.textContent = wordList[currentIndex].ipa;
        meaningWordEl.textContent = wordList[currentIndex].meaning;
        scoreEl.textContent = "";
        userTextEl.textContent = "";
        statusEl.textContent = "Sẵn sàng! Hãy bấm nút và đọc.";
        btnNext.style.display = "none";
        btnSpeak.disabled = false;
    }

    // Sự kiện khi bấm nút nói
    btnSpeak.addEventListener('click', () => {
        recognition.start();
        statusEl.textContent = "🔊 Đang lắng nghe... Hãy nói to, rõ ràng.";
        btnSpeak.disabled = true;
    });

    // Xử lý khi có kết quả trả về từ Google API
    recognition.onresult = (event) => {
        // Chuẩn hóa chuỗi (bỏ dấu chấm câu nếu có) để so sánh chính xác hơn
        const userSpeech = event.results[0][0].transcript.toLowerCase().replace(/[.,\/#!$%\^&\*;:{}=\-_`~()]/g,"").trim();
        const targetWord = wordList[currentIndex].word.toLowerCase().replace(/[.,\/#!$%\^&\*;:{}=\-_`~()]/g,"").trim();

        userTextEl.textContent = `Bạn đã nói: "${event.results[0][0].transcript}"`;
        
        // Tính điểm
        const matchPercent = calculateSimilarity(userSpeech, targetWord);
        
        scoreEl.textContent = `${matchPercent}%`;
        if (matchPercent >= 85) {
            scoreEl.className = "score success";
            statusEl.textContent = "Tuyệt vời! Phát âm chuẩn Mỹ rồi đấy!";
        } else if (matchPercent >= 60) {
            scoreEl.className = "score warning";
            statusEl.textContent = "Khá tốt, nhưng cần chính xác hơn.";
        } else {
            scoreEl.className = "score danger";
            statusEl.textContent = "Chưa chính xác. Hãy thử lại xem.";
        }

        btnSpeak.disabled = false;
        btnNext.style.display = "inline-block";
    };

    recognition.onerror = (event) => {
        statusEl.textContent = "Lỗi: Không nhận diện được giọng nói. Hãy thử lại.";
        btnSpeak.disabled = false;
    };

    recognition.onspeechend = () => {
        recognition.stop();
    };

    btnNext.addEventListener('click', () => {
        currentIndex = (currentIndex + 1) % wordList.length;
        loadWord();
    });

    // Thuật toán Levenshtein Distance
    function calculateSimilarity(str1, str2) {
        if(str1 === str2) return 100;
        const track = Array(str2.length + 1).fill(null).map(() => Array(str1.length + 1).fill(null));
        for (let i = 0; i <= str1.length; i += 1) track[0][i] = i;
        for (let j = 0; j <= str2.length; j += 1) track[j][0] = j;
        for (let j = 1; j <= str2.length; j += 1) {
            for (let i = 1; i <= str1.length; i += 1) {
                const indicator = str1[i - 1] === str2[j - 1] ? 0 : 1;
                track[j][i] = Math.min(
                    track[j][i - 1] + 1,
                    track[j - 1][i] + 1,
                    track[j - 1][i - 1] + indicator
                );
            }
        }
        const distance = track[str2.length][str1.length];
        const maxLength = Math.max(str1.length, str2.length);
        return Math.round(((maxLength - distance) / maxLength) * 100);
    }

    loadWord();
</script>

</body>
</html>
