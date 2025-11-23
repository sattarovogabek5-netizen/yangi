<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mukammal QR Kod Skaner</title>
    <script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        
        body {
            background: linear-gradient(135deg, #1a2a6c, #b21f1f, #fdbb2d);
            color: white;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 20px;
        }
        
        .container {
            max-width: 1000px;
            width: 100%;
            margin: 0 auto;
        }
        
        header {
            text-align: center;
            margin-bottom: 30px;
            padding: 20px;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 20px;
            backdrop-filter: blur(10px);
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
            width: 100%;
        }
        
        h1 {
            font-size: 2.8rem;
            margin-bottom: 10px;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
            background: linear-gradient(to right, #ff7e5f, #feb47b);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }
        
        .subtitle {
            font-size: 1.3rem;
            opacity: 0.9;
            margin-bottom: 10px;
        }
        
        .app-container {
            display: flex;
            flex-wrap: wrap;
            gap: 30px;
            margin-bottom: 30px;
            justify-content: center;
        }
        
        .scanner-section, .result-section {
            flex: 1;
            min-width: 450px;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 20px;
            padding: 25px;
            backdrop-filter: blur(10px);
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
            display: flex;
            flex-direction: column;
        }
        
        .section-title {
            font-size: 1.8rem;
            margin-bottom: 20px;
            border-bottom: 2px solid rgba(255, 255, 255, 0.3);
            padding-bottom: 10px;
            text-align: center;
        }
        
        .scanner-container {
            position: relative;
            width: 100%;
            height: 400px;
            border-radius: 15px;
            overflow: hidden;
            margin-bottom: 20px;
            background: #000;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        
        #video {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }
        
        #canvas {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: none;
        }
        
        .scanner-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            pointer-events: none;
        }
        
        .scanner-frame {
            width: 70%;
            height: 70%;
            border: 3px solid #00ff00;
            border-radius: 15px;
            box-shadow: 0 0 0 500px rgba(0, 0, 0, 0.7);
            position: relative;
            animation: pulse 2s infinite;
        }
        
        .scanner-frame::before, .scanner-frame::after {
            content: '';
            position: absolute;
            width: 30px;
            height: 30px;
            border: 3px solid #00ff00;
        }
        
        .scanner-frame::before {
            top: -3px;
            left: -3px;
            border-right: none;
            border-bottom: none;
        }
        
        .scanner-frame::after {
            bottom: -3px;
            right: -3px;
            border-left: none;
            border-top: none;
        }
        
        @keyframes pulse {
            0% { border-color: #00ff00; }
            50% { border-color: #00cc00; }
            100% { border-color: #00ff00; }
        }
        
        .scanner-line {
            position: absolute;
            width: 100%;
            height: 3px;
            background: linear-gradient(to right, transparent, #00ff00, transparent);
            top: 50%;
            animation: scan 2s linear infinite;
        }
        
        @keyframes scan {
            0% { top: 10%; }
            50% { top: 90%; }
            100% { top: 10%; }
        }
        
        .controls {
            display: flex;
            justify-content: center;
            gap: 15px;
            margin-top: 15px;
            flex-wrap: wrap;
        }
        
        .btn {
            background: rgba(255, 255, 255, 0.2);
            border: none;
            color: white;
            padding: 14px 24px;
            border-radius: 10px;
            cursor: pointer;
            font-size: 1.1rem;
            transition: all 0.3s ease;
            display: inline-flex;
            align-items: center;
            gap: 8px;
            backdrop-filter: blur(5px);
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
        }
        
        .btn:hover {
            background: rgba(255, 255, 255, 0.3);
            transform: translateY(-3px);
            box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
        }
        
        .btn-primary {
            background: linear-gradient(135deg, #00b09b, #96c93d);
        }
        
        .btn-primary:hover {
            background: linear-gradient(135deg, #00a08b, #8ab82d);
        }
        
        .btn-danger {
            background: linear-gradient(135deg, #ff416c, #ff4b2b);
        }
        
        .btn-danger:hover {
            background: linear-gradient(135deg, #e53a5f, #e54527);
        }
        
        .btn-success {
            background: linear-gradient(135deg, #2193b0, #6dd5ed);
        }
        
        .btn-success:hover {
            background: linear-gradient(135deg, #1d84a0, #5dc5dd);
        }
        
        .result-box {
            background: rgba(0, 0, 0, 0.3);
            border-radius: 15px;
            padding: 20px;
            margin-top: 15px;
            min-height: 150px;
            word-break: break-all;
            flex-grow: 1;
            display: flex;
            flex-direction: column;
            overflow-y: auto;
            border: 1px solid rgba(255, 255, 255, 0.1);
        }
        
        .result-content {
            font-size: 1.1rem;
            line-height: 1.6;
            flex-grow: 1;
        }
        
        .result-type {
            margin-top: 15px;
            padding: 10px;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 10px;
            font-size: 0.9rem;
        }
        
        .status-indicator {
            display: flex;
            align-items: center;
            gap: 10px;
            margin-top: 15px;
            padding: 10px;
            border-radius: 10px;
            background: rgba(0, 0, 0, 0.2);
        }
        
        .status-dot {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            background: #ff4757;
        }
        
        .status-dot.active {
            background: #2ed573;
            animation: blink 1.5s infinite;
        }
        
        @keyframes blink {
            0% { opacity: 1; }
            50% { opacity: 0.5; }
            100% { opacity: 1; }
        }
        
        .stats {
            display: flex;
            justify-content: space-around;
            margin-top: 20px;
            padding: 15px;
            background: rgba(0, 0, 0, 0.2);
            border-radius: 10px;
        }
        
        .stat-item {
            text-align: center;
        }
        
        .stat-value {
            font-size: 1.5rem;
            font-weight: bold;
            color: #ffdd59;
        }
        
        .stat-label {
            font-size: 0.9rem;
            opacity: 0.8;
        }
        
        footer {
            text-align: center;
            margin-top: 30px;
            padding: 20px;
            font-size: 0.9rem;
            opacity: 0.8;
            width: 100%;
        }
        
        .instructions {
            margin-top: 20px;
            padding: 15px;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 10px;
            font-size: 0.95rem;
        }
        
        .instructions ol {
            padding-left: 20px;
            margin-top: 10px;
        }
        
        .instructions li {
            margin-bottom: 8px;
        }
        
        @media (max-width: 768px) {
            .app-container {
                flex-direction: column;
            }
            
            .scanner-section, .result-section {
                min-width: 100%;
            }
            
            h1 {
                font-size: 2.2rem;
            }
            
            .scanner-container {
                height: 300px;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>Mukammal QR Kod Skaner</h1>
            <p class="subtitle">QR kodni aniq va tez skaner qilish dasturi</p>
            <p>Kamerani yoqing va QR kodni ramkaga qarating</p>
        </header>
        
        <div class="app-container">
            <div class="scanner-section">
                <h2 class="section-title">QR Kod Skanerlash</h2>
                
                <div class="scanner-container">
                    <video id="video" autoplay playsinline></video>
                    <canvas id="canvas"></canvas>
                    
                    <div class="scanner-overlay">
                        <div class="scanner-frame">
                            <div class="scanner-line"></div>
                        </div>
                    </div>
                </div>
                
                <div class="status-indicator">
                    <div class="status-dot" id="statusDot"></div>
                    <span id="statusText">Kamera o'chirilgan</span>
                </div>
                
                <div class="controls">
                    <button class="btn btn-primary" id="startButton">
                        <i>▶</i> Kamerani Yoqish
                    </button>
                    <button class="btn btn-danger" id="stopButton" disabled>
                        <i>⏹</i> Kamerani O'chirish
                    </button>
                    <button class="btn btn-success" id="captureButton" disabled>
                        <i>📷</i> Rasmni Olish
                    </button>
                </div>
                
                <div class="instructions">
                    <p><strong>Ishlatish bo'yicha ko'rsatma:</strong></p>
                    <ol>
                        <li>Kamerani yoqish tugmasini bosing</li>
                        <li>QR kodni oq ramka ichiga qarating</li>
                        <li>QR kod avtomatik skanerlanadi va natija ko'rsatiladi</li>
                        <li>Agar skaner qilishda muammo bo'lsa, "Rasmni olish" tugmasini bosing</li>
                    </ol>
                </div>
            </div>
            
            <div class="result-section">
                <h2 class="section-title">Skanerlash Natijasi</h2>
                
                <div class="result-box">
                    <div class="result-content" id="resultContent">
                        QR kod ma'lumoti bu yerda paydo bo'ladi...
                    </div>
                    <div class="result-type" id="resultType">
                        QR kod turi: Skanerlanmagan
                    </div>
                </div>
                
                <div class="stats">
                    <div class="stat-item">
                        <div class="stat-value" id="scannedCount">0</div>
                        <div class="stat-label">Skanerlar</div>
                    </div>
                    <div class="stat-item">
                        <div class="stat-value" id="successRate">0%</div>
                        <div class="stat-label">Muvaffaqiyat</div>
                    </div>
                    <div class="stat-item">
                        <div class="stat-value" id="scanTime">0ms</div>
                        <div class="stat-label">Skaner Vaqti</div>
                    </div>
                </div>
                
                <div class="controls">
                    <button class="btn btn-primary" id="copyButton" disabled>
                        <i>📋</i> Nusxa Olish
                    </button>
                    <button class="btn" id="clearButton">
                        <i>🗑️</i> Tozalash
                    </button>
                    <button class="btn" id="shareButton" disabled>
                        <i>📤</i> Ulashish
                    </button>
                </div>
            </div>
        </div>
        
        <footer>
            <p>Mukammal QR Kod Skaner &copy; 2023 - Barcha huquqlar himoyalangan</p>
        </footer>
    </div>

    <script>
        // Elementlarni tanlab olish
        const video = document.getElementById('video');
        const canvas = document.getElementById('canvas');
        const startButton = document.getElementById('startButton');
        const stopButton = document.getElementById('stopButton');
        const captureButton = document.getElementById('captureButton');
        const resultContent = document.getElementById('resultContent');
        const resultType = document.getElementById('resultType');
        const copyButton = document.getElementById('copyButton');
        const clearButton = document.getElementById('clearButton');
        const shareButton = document.getElementById('shareButton');
        const statusDot = document.getElementById('statusDot');
        const statusText = document.getElementById('statusText');
        const scannedCount = document.getElementById('scannedCount');
        const successRate = document.getElementById('successRate');
        const scanTime = document.getElementById('scanTime');
        
        let stream = null;
        let scanning = false;
        let scanInterval = null;
        let totalScans = 0;
        let successfulScans = 0;
        let lastScanTime = 0;
        
        // Kamerani yoqish
        startButton.addEventListener('click', async () => {
            try {
                // Oldingi kamerani tozalash
                if (stream) {
                    stream.getTracks().forEach(track => track.stop());
                }
                
                // Kamerani sozlash
                const constraints = {
                    video: { 
                        facingMode: 'environment',
                        width: { ideal: 1280 },
                        height: { ideal: 720 }
                    } 
                };
                
                stream = await navigator.mediaDevices.getUserMedia(constraints);
                video.srcObject = stream;
                
                // Holatni yangilash
                statusDot.classList.add('active');
                statusText.textContent = "QR kod qidirilmoqda...";
                startButton.disabled = true;
                stopButton.disabled = false;
                captureButton.disabled = false;
                
                // Skanerlashni boshlash
                startScanning();
                
            } catch (err) {
                console.error("Kamerani ochishda xatolik: ", err);
                resultContent.textContent = "Kamerani ochishda xatolik yuz berdi. Iltimos, kameraga ruxsat bering.";
                statusText.textContent = "Xatolik: Kamera ochilmadi";
            }
        });
        
        // Kamerani o'chirish
        stopButton.addEventListener('click', () => {
            stopScanning();
            statusDot.classList.remove('active');
            statusText.textContent = "Kamera o'chirilgan";
            startButton.disabled = false;
            stopButton.disabled = true;
            captureButton.disabled = true;
        });
        
        // Rasmni olish
        captureButton.addEventListener('click', () => {
            captureAndDecode();
        });
        
        // Skanerlashni boshlash
        function startScanning() {
            scanning = true;
            scanInterval = setInterval(captureAndDecode, 500); // Har 500ms da skanerlash
        }
        
        // Skanerlashni to'xtatish
        function stopScanning() {
            scanning = false;
            if (scanInterval) {
                clearInterval(scanInterval);
                scanInterval = null;
            }
            
            if (stream) {
                stream.getTracks().forEach(track => track.stop());
                video.srcObject = null;
                stream = null;
            }
        }
        
        // Rasmni olish va QR kodni dekod qilish
        function captureAndDecode() {
            if (!stream || !scanning) return;
            
            const startTime = performance.now();
            
            // Canvas o'lchamlarini sozlash
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            
            // Videodan rasm olish
            const context = canvas.getContext('2d');
            context.drawImage(video, 0, 0, canvas.width, canvas.height);
            
            // Rasm ma'lumotlarini olish
            const imageData = context.getImageData(0, 0, canvas.width, canvas.height);
            
            // QR kodni dekod qilish
            const code = jsQR(imageData.data, imageData.width, imageData.height, {
                inversionAttempts: 'dontInvert',
            });
            
            const endTime = performance.now();
            lastScanTime = Math.round(endTime - startTime);
            totalScans++;
            
            // Natijani yangilash
            if (code) {
                successfulScans++;
                displayResult(code.data, code);
                statusText.textContent = "QR kod muvaffaqiyatli skanerlandi!";
            } else {
                statusText.textContent = "QR kod qidirilmoqda...";
            }
            
            // Statistikani yangilash
            updateStats();
        }
        
        // Natijani ko'rsatish
        function displayResult(data, code) {
            resultContent.textContent = data;
            
            // QR kod turini aniqlash
            let type = "Noma'lum";
            if (data.startsWith('http://') || data.startsWith('https://')) {
                type = "Veb-sahifa";
            } else if (data.startsWith('tel:')) {
                type = "Telefon raqami";
            } else if (data.startsWith('mailto:')) {
                type = "Elektron pochta";
            } else if (data.startsWith('WIFI:')) {
                type = "Wi-Fi tarmog'i";
            } else if (data.startsWith('BEGIN:VCARD')) {
                type = "Kontakt";
            } else if (/^\d+$/.test(data)) {
                type = "Raqamli ma'lumot";
            } else {
                type = "Matn";
            }
            
            resultType.textContent = `QR kod turi: ${type}`;
            
            // Tugmalarni faollashtirish
            copyButton.disabled = false;
            shareButton.disabled = false;
        }
        
        // Statistikani yangilash
        function updateStats() {
            scannedCount.textContent = totalScans;
            const rate = totalScans > 0 ? Math.round((successfulScans / totalScans) * 100) : 0;
            successRate.textContent = `${rate}%`;
            scanTime.textContent = `${lastScanTime}ms`;
        }
        
        // Nusxa olish
        copyButton.addEventListener('click', () => {
            navigator.clipboard.writeText(resultContent.textContent)
                .then(() => {
                    const originalText = copyButton.innerHTML;
                    copyButton.innerHTML = '<i>✓</i> Nusxa olindi!';
                    setTimeout(() => {
                        copyButton.innerHTML = originalText;
                    }, 2000);
                })
                .catch(err => {
                    console.error('Nusxa olishda xatolik: ', err);
                    resultContent.textContent = "Nusxa olishda xatolik yuz berdi.";
                });
        });
        
        // Tozalash
        clearButton.addEventListener('click', () => {
            resultContent.textContent = "QR kod ma'lumoti bu yerda paydo bo'ladi...";
            resultType.textContent = "QR kod turi: Skanerlanmagan";
            copyButton.disabled = true;
            shareButton.disabled = true;
            totalScans = 0;
            successfulScans = 0;
            updateStats();
        });
        
        // Ulashish
        shareButton.addEventListener('click', async () => {
            const shareData = {
                title: 'QR Kod Ma\'lumoti',
                text: resultContent.textContent
            };
            
            try {
                if (navigator.share) {
                    await navigator.share(shareData);
                } else {
                    // Agar ulashish API qo'llab-quvvatlanmasa, nusxa olish
                    await navigator.clipboard.writeText(resultContent.textContent);
                    const originalText = shareButton.innerHTML;
                    shareButton.innerHTML = '<i>✓</i> Nusxa olindi!';
                    setTimeout(() => {
                        shareButton.innerHTML = originalText;
                    }, 2000);
                }
            } catch (err) {
                console.error('Ulashishda xatolik: ', err);
            }
        });
        
        // Dastur yuklanganda kamera ruxsatini tekshirish
        window.addEventListener('load', () => {
            if (!navigator.mediaDevices || !navigator.mediaDevices.getUserMedia) {
                resultContent.textContent = "Uzr, brauzeringiz kamerani qo'llab-quvvatlamaydi. Iltimos, zamonaviy brauzerdan foydalaning.";
                startButton.disabled = true;
            }
        });
    </script>
</body>
</html>
