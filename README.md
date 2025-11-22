<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>QR Kod Skaneri</title>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html5-qrcode/2.3.8/html5-qrcode.min.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            -webkit-tap-highlight-color: transparent;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            background: #000;
            width: 100vw;
            height: 100vh;
            overflow: hidden;
            position: fixed;
        }

        #reader {
            width: 100vw;
            height: 100vh;
            position: fixed;
            top: 0;
            left: 0;
        }

        #reader video {
            width: 100vw !important;
            height: 100vh !important;
            object-fit: cover !important;
            position: fixed !important;
            top: 0 !important;
            left: 0 !important;
        }

        /* Qorong'u overlay */
        .overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            background: rgba(0, 0, 0, 0.6);
            z-index: 10;
            pointer-events: none;
        }

        /* Skaner ramkasi */
        .scan-area {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 260px;
            height: 260px;
            z-index: 15;
            pointer-events: none;
        }

        /* Shaffof o'rta qism */
        .scan-area::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: transparent;
            border-radius: 20px;
        }

        /* Burchak chiziqlari */
        .corner {
            position: absolute;
            width: 60px;
            height: 60px;
            border: 3px solid #fff;
        }

        .corner.tl {
            top: -3px;
            left: -3px;
            border-right: none;
            border-bottom: none;
            border-radius: 20px 0 0 0;
        }

        .corner.tr {
            top: -3px;
            right: -3px;
            border-left: none;
            border-bottom: none;
            border-radius: 0 20px 0 0;
        }

        .corner.bl {
            bottom: -3px;
            left: -3px;
            border-right: none;
            border-top: none;
            border-radius: 0 0 0 20px;
        }

        .corner.br {
            bottom: -3px;
            right: -3px;
            border-left: none;
            border-top: none;
            border-radius: 0 0 20px 0;
        }

        /* Animatsiyali chiziq */
        .scan-line {
            position: absolute;
            left: 10px;
            right: 10px;
            height: 2px;
            background: linear-gradient(90deg, transparent, #4ade80, transparent);
            animation: scan 2.5s ease-in-out infinite;
            box-shadow: 0 0 10px #4ade80;
        }

        @keyframes scan {
            0%, 100% { top: 10px; opacity: 0.5; }
            50% { top: calc(100% - 12px); opacity: 1; }
        }

        /* Matn */
        .info-text {
            position: fixed;
            bottom: 120px;
            left: 0;
            right: 0;
            text-align: center;
            color: #fff;
            font-size: 17px;
            font-weight: 500;
            z-index: 20;
            text-shadow: 0 2px 10px rgba(0,0,0,0.8);
        }

        /* Chiroq tugmasi */
        .torch-btn {
            position: fixed;
            bottom: 30px;
            right: 30px;
            width: 56px;
            height: 56px;
            border-radius: 50%;
            background: rgba(40, 40, 40, 0.9);
            border: 2px solid rgba(255, 255, 255, 0.3);
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 24px;
            cursor: pointer;
            z-index: 20;
            transition: all 0.3s;
            backdrop-filter: blur(10px);
        }

        .torch-btn:active {
            transform: scale(0.9);
        }

        .torch-btn.active {
            background: rgba(250, 204, 21, 0.9);
            border-color: #fbbf24;
        }

        /* Yopish tugmasi */
        .close-btn {
            position: fixed;
            top: 20px;
            right: 20px;
            width: 40px;
            height: 40px;
            border-radius: 50%;
            background: rgba(40, 40, 40, 0.8);
            border: none;
            color: #fff;
            font-size: 24px;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            z-index: 20;
            backdrop-filter: blur(10px);
        }

        .close-btn:active {
            transform: scale(0.9);
        }

        /* Natija modal */
        .result-modal {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            background: rgba(0, 0, 0, 0.95);
            display: none;
            align-items: center;
            justify-content: center;
            z-index: 100;
            padding: 20px;
        }

        .result-modal.show {
            display: flex;
        }

        .result-box {
            background: #1a1a1a;
            border-radius: 24px;
            padding: 30px;
            max-width: 380px;
            width: 100%;
        }

        .result-icon {
            font-size: 64px;
            text-align: center;
            margin-bottom: 20px;
        }

        .result-title {
            color: #fff;
            font-size: 20px;
            font-weight: 600;
            text-align: center;
            margin-bottom: 20px;
        }

        .result-content {
            background: #2a2a2a;
            padding: 16px;
            border-radius: 16px;
            color: #e0e0e0;
            word-break: break-all;
            margin-bottom: 24px;
            font-size: 14px;
            line-height: 1.5;
            max-height: 180px;
            overflow-y: auto;
        }

        .btn-group {
            display: flex;
            gap: 12px;
        }

        .btn {
            flex: 1;
            padding: 16px;
            border: none;
            border-radius: 14px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.2s;
        }

        .btn-primary {
            background: #4ade80;
            color: #000;
        }

        .btn-primary:active {
            transform: scale(0.95);
            background: #22c55e;
        }

        .btn-secondary {
            background: #2a2a2a;
            color: #fff;
        }

        .btn-secondary:active {
            transform: scale(0.95);
            background: #3a3a3a;
        }

        /* Loading */
        .loading {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            text-align: center;
            z-index: 30;
            color: #fff;
        }

        .spinner {
            width: 50px;
            height: 50px;
            border: 4px solid rgba(255,255,255,0.2);
            border-top-color: #fff;
            border-radius: 50%;
            animation: spin 1s linear infinite;
            margin: 0 auto 15px;
        }

        @keyframes spin {
            to { transform: rotate(360deg); }
        }

        /* Html5-qrcode yashirish */
        #reader__dashboard_section,
        #reader__dashboard_section_csr,
        #reader__header_message,
        #reader__scan_region {
            display: none !important;
        }
    </style>
</head>
<body>
    <div id="reader"></div>
    
    <div class="overlay"></div>
    
    <div class="scan-area">
        <div class="corner tl"></div>
        <div class="corner tr"></div>
        <div class="corner bl"></div>
        <div class="corner br"></div>
        <div class="scan-line"></div>
    </div>

    <div class="info-text">DataMatrix kodini skanerlang</div>

    <button class="close-btn" onclick="closeScanner()">✕</button>
    
    <button class="torch-btn" id="torchBtn" onclick="toggleTorch()">🔦</button>

    <div class="loading" id="loading">
        <div class="spinner"></div>
        <div>Kamera yuklanmoqda...</div>
    </div>

    <div class="result-modal" id="resultModal">
        <div class="result-box">
            <div class="result-icon">✅</div>
            <div class="result-title">Muvaffaqiyatli!</div>
            <div class="result-content" id="resultContent"></div>
            <div class="btn-group">
                <button class="btn btn-primary" onclick="sendToBot()">Botga Yuborish</button>
                <button class="btn btn-secondary" onclick="scanAgain()">Qayta</button>
            </div>
        </div>
    </div>

    <script>
        let tg = window.Telegram.WebApp;
        let html5Qrcode = null;
        let scannedData = null;
        let torchEnabled = false;
        let currentStream = null;

        tg.expand();
        tg.ready();

        // Kamerani ochish
        async function startCamera() {
            try {
                html5Qrcode = new Html5Qrcode("reader");
                
                const qrCodeSuccessCallback = (decodedText) => {
                    onScanSuccess(decodedText);
                };
                
                const config = {
                    fps: 10,
                    qrbox: 250,
                    aspectRatio: 1.0
                };

                await html5Qrcode.start(
                    { facingMode: "environment" },
                    config,
                    qrCodeSuccessCallback
                );

                // Stream olish
                setTimeout(() => {
                    const videoElement = document.querySelector('#reader video');
                    if (videoElement && videoElement.srcObject) {
                        currentStream = videoElement.srcObject;
                    }
                    document.getElementById('loading').style.display = 'none';
                }, 1000);

            } catch (err) {
                console.error("Kamera xatosi:", err);
                document.getElementById('loading').innerHTML = '<div style="color: #ff4444;">Kamera ochilmadi. Ruxsat bering.</div>';
            }
        }

        // Scan muvaffaqiyatli
        function onScanSuccess(decodedText) {
            scannedData = decodedText;
            
            if (tg.HapticFeedback) {
                tg.HapticFeedback.notificationOccurred('success');
            }

            document.getElementById('resultContent').textContent = decodedText;
            document.getElementById('resultModal').classList.add('show');
            
            if (html5Qrcode) {
                html5Qrcode.stop().catch(err => console.log(err));
            }
        }

        function onScanError(error) {
            // Xatoliklarni ignore qilish
        }

        // Chiroqni yoqish/o'chirish
        async function toggleTorch() {
            try {
                if (!currentStream) return;

                const track = currentStream.getVideoTracks()[0];
                const capabilities = track.getCapabilities();

                if (!capabilities.torch) {
                    alert("Chiroq qo'llab-quvvatlanmaydi");
                    return;
                }

                torchEnabled = !torchEnabled;
                await track.applyConstraints({
                    advanced: [{ torch: torchEnabled }]
                });

                const btn = document.getElementById('torchBtn');
                if (torchEnabled) {
                    btn.classList.add('active');
                    btn.textContent = '💡';
                } else {
                    btn.classList.remove('active');
                    btn.textContent = '🔦';
                }

                if (tg.HapticFeedback) {
                    tg.HapticFeedback.impactOccurred('light');
                }
            } catch (err) {
                console.error("Chiroq xatosi:", err);
            }
        }

        // Botga yuborish
        function sendToBot() {
            if (scannedData) {
                tg.sendData(scannedData);
                tg.close();
            }
        }

        // Qayta skanerlash
        function scanAgain() {
            scannedData = null;
            document.getElementById('resultModal').classList.remove('show');
            document.getElementById('loading').style.display = 'block';
            setTimeout(() => startCamera(), 300);
        }

        // Yopish
        function closeScanner() {
            if (html5Qrcode) {
                html5Qrcode.stop().catch(err => console.log(err));
            }
            tg.close();
        }

        // Boshlash
        window.addEventListener('load', () => {
            setTimeout(startCamera, 500);
        });
    </script>
</body>
</html>
