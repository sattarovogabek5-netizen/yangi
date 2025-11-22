<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
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

        html, body {
            width: 100%;
            height: 100%;
            overflow: hidden;
            position: fixed;
            background: #000;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
        }

        /* Video container */
        #reader {
            position: fixed !important;
            top: 0 !important;
            left: 0 !important;
            width: 100vw !important;
            height: 100vh !important;
            z-index: 1 !important;
        }

        #reader video {
            position: fixed !important;
            top: 0 !important;
            left: 0 !important;
            width: 100vw !important;
            height: 100vh !important;
            object-fit: cover !important;
        }

        /* Qorong'u overlay */
        .dark-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.5);
            z-index: 10;
            pointer-events: none;
        }

        /* Header */
        .header {
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            height: 60px;
            background: linear-gradient(180deg, rgba(0,0,0,0.8) 0%, transparent 100%);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 30;
        }

        .header-text {
            color: #fff;
            font-size: 18px;
            font-weight: 600;
            text-shadow: 0 2px 8px rgba(0,0,0,0.8);
        }

        /* Close button */
        .close-btn {
            position: fixed;
            top: 12px;
            right: 12px;
            width: 44px;
            height: 44px;
            border-radius: 50%;
            background: rgba(60, 60, 60, 0.8);
            border: none;
            color: #fff;
            font-size: 24px;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            z-index: 40;
            backdrop-filter: blur(10px);
        }

        .close-btn:active {
            transform: scale(0.9);
            background: rgba(80, 80, 80, 0.9);
        }

        /* Scan area */
        .scan-frame {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 280px;
            height: 280px;
            z-index: 20;
            pointer-events: auto;
            touch-action: none;
        }

        /* Burchaklar */
        .corner {
            position: absolute;
            width: 60px;
            height: 60px;
            border: 3px solid #fff;
        }

        .corner.tl {
            top: 0;
            left: 0;
            border-right: none;
            border-bottom: none;
            border-radius: 20px 0 0 0;
        }

        .corner.tr {
            top: 0;
            right: 0;
            border-left: none;
            border-bottom: none;
            border-radius: 0 20px 0 0;
        }

        .corner.bl {
            bottom: 0;
            left: 0;
            border-right: none;
            border-top: none;
            border-radius: 0 0 0 20px;
        }

        .corner.br {
            bottom: 0;
            right: 0;
            border-left: none;
            border-top: none;
            border-radius: 0 0 20px 0;
        }

        /* Scan line */
        .scan-line {
            position: absolute;
            left: 10px;
            right: 10px;
            height: 2px;
            background: linear-gradient(90deg, transparent, #4ade80, transparent);
            animation: scan 1.5s ease-in-out infinite;
            box-shadow: 0 0 10px #4ade80;
        }

        @keyframes scan {
            0%, 100% { top: 10px; opacity: 0.6; }
            50% { top: calc(100% - 12px); opacity: 1; }
        }

        /* Bottom text */
        .bottom-text {
            position: fixed;
            bottom: 140px;
            left: 0;
            right: 0;
            text-align: center;
            z-index: 20;
        }

        .bottom-text-inner {
            display: inline-block;
            background: rgba(40, 40, 40, 0.85);
            color: #fff;
            padding: 12px 24px;
            border-radius: 24px;
            font-size: 16px;
            font-weight: 500;
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255, 255, 255, 0.1);
        }

        /* Torch button */
        .torch-btn {
            position: fixed;
            bottom: 40px;
            right: 30px;
            width: 60px;
            height: 60px;
            border-radius: 50%;
            background: rgba(60, 60, 60, 0.85);
            border: 2px solid rgba(255, 255, 255, 0.2);
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 26px;
            cursor: pointer;
            z-index: 20;
            backdrop-filter: blur(10px);
            transition: all 0.2s;
        }

        .torch-btn:active {
            transform: scale(0.9);
        }

        .torch-btn.active {
            background: rgba(250, 204, 21, 0.9);
            border-color: #fbbf24;
            box-shadow: 0 0 20px rgba(250, 204, 21, 0.5);
        }

        /* Zoom controls */
        .zoom-controls {
            position: fixed;
            bottom: 120px;
            left: 50%;
            transform: translateX(-50%);
            display: flex;
            flex-direction: column;
            align-items: center;
            z-index: 25;
            gap: 10px;
        }

        .zoom-text {
            background: rgba(40, 40, 40, 0.9);
            color: #4ade80;
            padding: 8px 18px;
            border-radius: 20px;
            font-size: 16px;
            font-weight: 700;
            backdrop-filter: blur(10px);
            border: 1px solid rgba(74, 222, 128, 0.3);
        }

        .zoom-buttons {
            display: flex;
            gap: 15px;
        }

        .zoom-btn {
            width: 50px;
            height: 50px;
            border-radius: 50%;
            background: rgba(60, 60, 60, 0.9);
            border: 2px solid rgba(255, 255, 255, 0.2);
            color: white;
            font-size: 20px;
            font-weight: bold;
            display: flex;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            backdrop-filter: blur(10px);
            transition: all 0.2s;
        }

        .zoom-btn:active {
            transform: scale(0.9);
            background: rgba(80, 80, 80, 0.9);
        }

        /* Result modal */
        .result-modal {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
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
            line-height: 1.6;
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
            z-index: 50;
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

        /* Error message */
        .error-message {
            background: rgba(220, 38, 38, 0.9);
            color: white;
            padding: 20px;
            border-radius: 16px;
            text-align: center;
            max-width: 300px;
            margin: 20px;
        }

        .retry-btn {
            background: #4ade80;
            color: black;
            border: none;
            padding: 12px 24px;
            border-radius: 12px;
            font-weight: bold;
            margin-top: 15px;
            cursor: pointer;
        }

        /* Hide html5-qrcode UI */
        #reader__dashboard_section,
        #reader__dashboard_section_csr,
        #reader__header_message,
        #reader__scan_region,
        #reader > div:not(video) {
            display: none !important;
        }
    </style>
</head>
<body>
    <div id="reader"></div>
    
    <div class="dark-overlay"></div>

    <div class="header">
        <div class="header-text">Diruni skanerlang</div>
    </div>

    <button class="close-btn" onclick="closeScanner()">✕</button>
    
    <div class="scan-frame">
        <div class="corner tl"></div>
        <div class="corner tr"></div>
        <div class="corner bl"></div>
        <div class="corner br"></div>
        <div class="scan-line"></div>
    </div>

    <div class="bottom-text">
        <div class="bottom-text-inner">DataMatrix kodini skanerlang</div>
    </div>

    <button class="torch-btn" id="torchBtn" onclick="toggleTorch()">🔦</button>

    <div class="zoom-controls" id="zoomControls">
        <div class="zoom-text" id="zoomText">1.0x</div>
        <div class="zoom-buttons">
            <button class="zoom-btn" onclick="zoomOut()">-</button>
            <button class="zoom-btn" onclick="zoomIn()">+</button>
        </div>
    </div>

    <div class="loading" id="loading">
        <div class="spinner"></div>
        <div>Kamera yuklanmoqda...</div>
    </div>

    <div class="result-modal" id="resultModal">
        <div class="result-box">
            <div class="result-icon">✅</div>
            <div class="result-title">Muvaffaqiyatli skanerlandi!</div>
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
        let currentZoom = 1.0;
        let maxZoom = 3.0;
        let minZoom = 1.0;
        let videoTrack = null;
        let isScanning = false;

        tg.expand();
        tg.ready();

        // Kamerani ochish - soddalashtirilgan versiya
        async function startCamera() {
            try {
                console.log("Kamera ochish boshlandi...");
                
                // Soddaroq kamera sozlamalari
                const constraints = {
                    video: {
                        facingMode: "environment",
                        width: { ideal: 1280 },
                        height: { ideal: 720 },
                        frameRate: { ideal: 30 }
                    }
                };

                // Kamerani ochish
                currentStream = await navigator.mediaDevices.getUserMedia(constraints);
                console.log("Kamera muvaffaqiyatli ochildi");

                videoTrack = currentStream.getVideoTracks()[0];
                console.log("Video track:", videoTrack);

                // HTML5 QRCode ni ishga tushirish
                html5Qrcode = new Html5Qrcode("reader");
                
                // Soddalashtirilgan konfiguratsiya
                const config = {
                    fps: 10,
                    qrbox: 250,
                    aspectRatio: 1.0
                };

                await html5Qrcode.start(
                    { facingMode: "environment" },
                    config,
                    onScanSuccess,
                    onScanFailure
                );

                console.log("QR skaner muvaffaqiyatli ishga tushdi");
                
                // Yuklashni yashirish
                document.getElementById('loading').style.display = 'none';
                isScanning = true;

            } catch (err) {
                console.error("Kamera xatosi:", err);
                showCameraError(err);
            }
        }

        // Kamera xatosini ko'rsatish
        function showCameraError(error) {
            const loading = document.getElementById('loading');
            loading.innerHTML = `
                <div class="error-message">
                    <div style="font-size: 48px; margin-bottom: 15px;">📷</div>
                    <div style="font-weight: bold; margin-bottom: 10px;">Kamera ochilmadi</div>
                    <div style="font-size: 14px; margin-bottom: 15px; opacity: 0.8;">
                        Iltimos, kameraga ruxsat bering yoki qayta urinib ko'ring
                    </div>
                    <button class="retry-btn" onclick="retryCamera()">Qayta Urinish</button>
                </div>
            `;
        }

        // Qayta urinish
        function retryCamera() {
            document.getElementById('loading').innerHTML = `
                <div class="spinner"></div>
                <div>Kamera yuklanmoqda...</div>
            `;
            setTimeout(startCamera, 500);
        }

        // Zoom in
        function zoomIn() {
            if (currentZoom < maxZoom) {
                currentZoom = Math.min(maxZoom, currentZoom + 0.3);
                applyZoom();
                updateZoomUI();
            }
        }

        // Zoom out
        function zoomOut() {
            if (currentZoom > minZoom) {
                currentZoom = Math.max(minZoom, currentZoom - 0.3);
                applyZoom();
                updateZoomUI();
            }
        }

        // Zoom qo'llash
        async function applyZoom() {
            if (!videoTrack) return;
            
            try {
                const capabilities = videoTrack.getCapabilities();
                
                if (capabilities.zoom) {
                    await videoTrack.applyConstraints({
                        advanced: [{ zoom: currentZoom }]
                    });
                }
            } catch (err) {
                console.log("Zoom xatosi:", err);
            }
        }

        // Zoom UI yangilash
        function updateZoomUI() {
            document.getElementById('zoomText').textContent = currentZoom.toFixed(1) + 'x';
        }

        // Scan success
        function onScanSuccess(decodedText) {
            if (scannedData || !isScanning) return;
            
            scannedData = decodedText;
            isScanning = false;
            
            console.log("QR kod topildi:", decodedText);
            
            // Haptic feedback
            if (tg.HapticFeedback) {
                tg.HapticFeedback.notificationOccurred('success');
            }

            // Natijani ko'rsatish
            document.getElementById('resultContent').textContent = decodedText;
            document.getElementById('resultModal').classList.add('show');
            
            // Skanerlashni to'xtatish
            if (html5Qrcode && html5Qrcode.isScanning) {
                html5Qrcode.stop().catch(err => console.log("Stop error:", err));
            }
        }

        // Scan failure
        function onScanFailure(error) {
            // Oddiy xatolarni ignore qilish
            if (!error || error === "NotFoundException" || error.includes("No QR")) {
                return;
            }
            console.log("Scan error:", error);
        }

        // Chiroq
        async function toggleTorch() {
            if (!videoTrack) return;

            try {
                const capabilities = videoTrack.getCapabilities();
                
                if (!capabilities.torch) {
                    alert("Ushbu qurilmada chiroq qo'llab-quvvatlanmaydi");
                    return;
                }

                torchEnabled = !torchEnabled;
                await videoTrack.applyConstraints({
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
            
            setTimeout(() => {
                startCamera();
            }, 500);
        }

        // Yopish
        function closeScanner() {
            isScanning = false;
            if (html5Qrcode && html5Qrcode.isScanning) {
                html5Qrcode.stop().catch(err => console.log(err));
            }
            if (currentStream) {
                currentStream.getTracks().forEach(track => track.stop());
            }
            tg.close();
        }

        // Touch zoom
        let initialDistance = null;

        document.addEventListener('touchstart', (e) => {
            if (e.touches.length === 2) {
                initialDistance = Math.hypot(
                    e.touches[0].clientX - e.touches[1].clientX,
                    e.touches[0].clientY - e.touches[1].clientY
                );
            }
        });

        document.addEventListener('touchmove', (e) => {
            if (e.touches.length === 2 && initialDistance) {
                const currentDistance = Math.hypot(
                    e.touches[0].clientX - e.touches[1].clientX,
                    e.touches[0].clientY - e.touches[1].clientY
                );
                
                const zoomFactor = currentDistance / initialDistance;
                const newZoom = Math.max(minZoom, Math.min(maxZoom, currentZoom * zoomFactor));
                
                if (Math.abs(newZoom - currentZoom) > 0.1) {
                    currentZoom = newZoom;
                    applyZoom();
                    updateZoomUI();
                }
            }
        });

        document.addEventListener('touchend', () => {
            initialDistance = null;
        });

        // Boshlash
        window.addEventListener('load', () => {
            console.log("Sayt yuklandi, kamera ochilmoqda...");
            setTimeout(startCamera, 1000);
        });

        // Ruxsatlarni tekshirish
        async function checkPermissions() {
            try {
                const permissionStatus = await navigator.permissions.query({ name: 'camera' });
                console.log("Camera permission:", permissionStatus.state);
                
                permissionStatus.onchange = () => {
                    console.log("Camera permission changed:", permissionStatus.state);
                };
            } catch (err) {
                console.log("Permission API not supported");
            }
        }

        // Dastlabki tekshiruv
        checkPermissions();
    </script>
</body>
</html>
