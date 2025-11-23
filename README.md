<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Markrof QR Skaner</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: Arial, sans-serif;
            background: #000;
            color: white;
            min-height: 100vh;
            padding: 10px;
        }

        .container {
            max-width: 100%;
            text-align: center;
        }

        .header {
            margin-bottom: 15px;
            padding: 10px;
        }

        .header h1 {
            font-size: 20px;
            margin-bottom: 5px;
            color: #00ff88;
        }

        .video-container {
            position: relative;
            width: 100%;
            height: 60vh;
            background: #000;
            margin-bottom: 10px;
            border: 2px solid #00ff88;
            border-radius: 10px;
            overflow: hidden;
        }

        video {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }

        .scan-box {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 70%;
            height: 70%;
            border: 2px solid #00ff88;
            border-radius: 10px;
        }

        .scan-line {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 3px;
            background: #00ff88;
            animation: scan 1s linear infinite;
        }

        @keyframes scan {
            0% { top: 0; }
            100% { top: 100%; }
        }

        .status {
            padding: 8px;
            border-radius: 8px;
            margin-bottom: 10px;
            font-weight: bold;
            background: #333;
            font-size: 14px;
        }

        .status-scanning {
            background: #0066cc;
        }

        .status-found {
            background: #00aa44;
        }

        .controls {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 8px;
            margin-bottom: 10px;
        }

        button {
            padding: 12px;
            border: none;
            border-radius: 8px;
            font-weight: bold;
            cursor: pointer;
            font-size: 14px;
        }

        .btn-start {
            background: #00ff88;
            color: #000;
        }

        .btn-stop {
            background: #ff4444;
            color: white;
        }

        .simple-controls {
            display: flex;
            gap: 8px;
            margin-bottom: 10px;
            justify-content: center;
            flex-wrap: wrap;
        }

        .zoom-btn {
            background: #333;
            color: white;
            padding: 8px 12px;
            border-radius: 20px;
            border: none;
            cursor: pointer;
            font-size: 12px;
        }

        .zoom-btn.active {
            background: #00ff88;
            color: #000;
        }

        .result {
            background: #333;
            padding: 15px;
            border-radius: 10px;
            margin-top: 10px;
            display: none;
            border: 2px solid #00ff88;
        }

        .result-text {
            background: #222;
            padding: 12px;
            border-radius: 8px;
            margin: 10px 0;
            word-break: break-all;
            font-size: 14px;
            line-height: 1.4;
            max-height: 200px;
            overflow-y: auto;
            text-align: left;
        }

        .result-buttons {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 8px;
        }

        .btn-copy {
            background: #0066cc;
            color: white;
        }

        .btn-new {
            background: #00ff88;
            color: #000;
        }

        .detection-info {
            font-size: 11px;
            color: #888;
            margin-top: 5px;
        }

        .fullscreen-btn {
            position: fixed;
            top: 15px;
            right: 15px;
            background: rgba(0, 255, 136, 0.9);
            color: #000;
            border: none;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            font-size: 18px;
            cursor: pointer;
            z-index: 1000;
        }

        /* Markrof optimizatsiyasi */
        @media (max-width: 768px) {
            .video-container {
                height: 50vh;
            }
            
            .scan-box {
                width: 80%;
                height: 60%;
            }
        }

        @media (max-width: 480px) {
            .video-container {
                height: 45vh;
            }
            
            .header h1 {
                font-size: 18px;
            }
            
            button {
                padding: 10px;
                font-size: 12px;
            }
        }
    </style>
</head>
<body>
    <button class="fullscreen-btn" id="fullscreen-btn">⛶</button>

    <div class="container">
        <div class="header">
            <h1>MARKROF QR SKANER</h1>
            <p>Kodni to'liq o'qiydi</p>
        </div>

        <div class="video-container">
            <video id="video" playsinline autoplay muted></video>
            <div class="scan-box">
                <div class="scan-line"></div>
            </div>
        </div>

        <div class="status status-scanning" id="status">
            🔍 QR kod qidirilmoqda...
        </div>

        <div class="simple-controls">
            <button class="zoom-btn active" data-zoom="1">1x</button>
            <button class="zoom-btn" data-zoom="1.5">1.5x</button>
            <button class="zoom-btn" data-zoom="2">2x</button>
            <button class="zoom-btn" data-zoom="3">3x</button>
        </div>

        <div class="controls">
            <button class="btn-start" id="start-btn">▶️ SKANERLASH</button>
            <button class="btn-stop" id="stop-btn">⏹️ TO'XTATISH</button>
        </div>

        <div class="detection-info" id="detection-info">
            Kamerani QR kodga qarating
        </div>

        <div class="result" id="result">
            <h3>✅ QR KOD TOPILDI!</h3>
            <div class="result-text" id="result-text"></div>
            <div class="result-buttons">
                <button class="btn-copy" id="copy-btn">📋 NUSXA OLISH</button>
                <button class="btn-new" id="new-scan-btn">🔄 YANGI SKANER</button>
            </div>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.js"></script>
    <script>
        // Elementlarni olish
        const video = document.getElementById('video');
        const startBtn = document.getElementById('start-btn');
        const stopBtn = document.getElementById('stop-btn');
        const status = document.getElementById('status');
        const result = document.getElementById('result');
        const resultText = document.getElementById('result-text');
        const copyBtn = document.getElementById('copy-btn');
        const newScanBtn = document.getElementById('new-scan-btn');
        const zoomBtns = document.querySelectorAll('.zoom-btn');
        const detectionInfo = document.getElementById('detection-info');
        const fullscreenBtn = document.getElementById('fullscreen-btn');

        let stream = null;
        let scanning = false;
        let scanInterval = null;
        let currentZoom = 1;
        let scanCounter = 0;
        let lastDetectionTime = 0;

        // Ekranni to'liq ochish
        fullscreenBtn.addEventListener('click', function() {
            if (!document.fullscreenElement) {
                document.documentElement.requestFullscreen().catch(err => {
                    console.log('Fullscreen error:', err);
                });
            } else {
                document.exitFullscreen();
            }
        });

        // Zoom tugmalari
        zoomBtns.forEach(btn => {
            btn.addEventListener('click', function() {
                zoomBtns.forEach(b => b.classList.remove('active'));
                this.classList.add('active');
                currentZoom = parseFloat(this.dataset.zoom);
                applyZoom();
            });
        });

        // Zoomni qo'llash
        function applyZoom() {
            if (stream) {
                try {
                    const track = stream.getVideoTracks()[0];
                    const capabilities = track.getCapabilities();
                    if (capabilities.zoom) {
                        track.applyConstraints({
                            advanced: [{ zoom: currentZoom }]
                        });
                    } else {
                        video.style.transform = `scale(${currentZoom})`;
                    }
                } catch (e) {
                    video.style.transform = `scale(${currentZoom})`;
                }
            } else {
                video.style.transform = `scale(${currentZoom})`;
            }
        }

        // Skanerlashni boshlash
        startBtn.addEventListener('click', startScanner);
        
        // Skanerlashni to'xtatish
        stopBtn.addEventListener('click', stopScanner);
        
        // Nusxa olish
        copyBtn.addEventListener('click', async () => {
            const text = resultText.textContent;
            try {
                await navigator.clipboard.writeText(text);
                showNotification('✅ Matn nusxalandi!');
            } catch (err) {
                showNotification('❌ Nusxalashda xatolik');
            }
        });
        
        // Yangi skanerlash
        newScanBtn.addEventListener('click', () => {
            result.style.display = 'none';
            scanCounter = 0;
            startScanner();
        });

        // Skanerlashni boshlash funksiyasi
        async function startScanner() {
            try {
                status.textContent = '📱 Kamera ochilmoqda...';
                status.className = 'status status-scanning';
                
                // Kamerani ochish - markrof uchun optimallashtirilgan
                stream = await navigator.mediaDevices.getUserMedia({ 
                    video: { 
                        facingMode: 'environment',
                        width: { ideal: 1920 },
                        height: { ideal: 1080 }
                    } 
                });
                
                video.srcObject = stream;
                await video.play();
                
                status.textContent = '🔍 QR kod qidirilmoqda...';
                scanning = true;
                scanCounter = 0;
                
                // JUDA TEZ SKANERLASH - har 30ms
                scanInterval = setInterval(scanQRCode, 30);
                
            } catch (err) {
                status.textContent = '❌ Kamera ochilmadi';
                status.className = 'status';
            }
        }

        // Skanerlashni to'xtatish
        function stopScanner() {
            scanning = false;
            if (scanInterval) {
                clearInterval(scanInterval);
            }
            if (stream) {
                stream.getTracks().forEach(track => track.stop());
                stream = null;
            }
            status.textContent = '⏹️ Skanerlash to\'xtatildi';
            status.className = 'status';
        }

        // Kuchli tasvir yaxshilash
        function enhanceImage(imageData) {
            const data = imageData.data;
            const contrast = 1.3;
            const brightness = 1.1;
            
            for (let i = 0; i < data.length; i += 4) {
                // Brightness
                data[i] = Math.min(255, data[i] * brightness);
                data[i+1] = Math.min(255, data[i+1] * brightness);
                data[i+2] = Math.min(255, data[i+2] * brightness);
                
                // Contrast
                data[i] = Math.min(255, 128 + (data[i] - 128) * contrast);
                data[i+1] = Math.min(255, 128 + (data[i+1] - 128) * contrast);
                data[i+2] = Math.min(255, 128 + (data[i+2] - 128) * contrast);
            }
            
            return imageData;
        }

        // ASOSIY SKANERLASH FUNKSIYASI - MARKROF UCHUN
        function scanQRCode() {
            if (!scanning || video.readyState !== video.HAVE_ENOUGH_DATA) return;
            
            scanCounter++;
            const now = Date.now();
            
            // Har 100ms dan tez-tez skanerlamaslik
            if (now - lastDetectionTime < 100) return;
            
            const canvas = document.createElement('canvas');
            const context = canvas.getContext('2d');
            
            // Markrof uchun optimal o'lcham
            canvas.width = 800;
            canvas.height = 600;
            
            context.drawImage(video, 0, 0, canvas.width, canvas.height);
            
            let imageData = context.getImageData(0, 0, canvas.width, canvas.height);
            
            // Tasvirni yaxshilash
            imageData = enhanceImage(imageData);
            
            // 3 xil usulda QR kodni qidirish
            let code = null;
            
            // 1. Oddiy qidirish
            code = jsQR(imageData.data, canvas.width, canvas.height, {
                inversionAttempts: 'attemptBoth',
            });

            // 2. Agar topilmasa, kichik qismlarni tekshirish
            if (!code && currentZoom > 1.5) {
                // Markaziy qismni ajratib olish
                const centerX = canvas.width / 2;
                const centerY = canvas.height / 2;
                const size = 300;
                
                const centerImageData = context.getImageData(
                    centerX - size/2, 
                    centerY - size/2, 
                    size, 
                    size
                );
                
                code = jsQR(centerImageData.data, size, size, {
                    inversionAttempts: 'attemptBoth',
                });
            }

            // 3. Kattalashtirib qidirish
            if (!code) {
                const scaledCanvas = document.createElement('canvas');
                scaledCanvas.width = canvas.width * 1.5;
                scaledCanvas.height = canvas.height * 1.5;
                const scaledCtx = scaledCanvas.getContext('2d');
                
                scaledCtx.drawImage(canvas, 0, 0, scaledCanvas.width, scaledCanvas.height);
                const scaledImageData = scaledCtx.getImageData(0, 0, scaledCanvas.width, scaledCanvas.height);
                
                code = jsQR(scaledImageData.data, scaledCanvas.width, scaledCanvas.height, {
                    inversionAttempts: 'attemptBoth',
                });
            }

            // AGAR TOPILSA - DARHOL TO'XTATISH
            if (code && code.data && code.data.length > 0) {
                lastDetectionTime = now;
                stopScanner();
                
                // To'liq matnni ko'rsatish
                resultText.textContent = code.data;
                result.style.display = 'block';
                
                status.textContent = '✅ QR kod topildi!';
                status.className = 'status status-found';
                
                detectionInfo.textContent = `Kod ${scanCounter} urinishda topildi`;
                
                // Agar URL bo'lsa, darhol ochish
                try {
                    new URL(code.data);
                    setTimeout(() => {
                        if (confirm('Bu URL. Ochishni xohlaysizmi?')) {
                            window.open(code.data, '_blank');
                        }
                    }, 200);
                } catch (e) {
                    // URL emas
                }
            }
            
            // Status yangilash
            if (scanCounter % 20 === 0) {
                detectionInfo.textContent = `${scanCounter} marta skanerlandi`;
            }
        }

        // Notification ko'rsatish
        function showNotification(message) {
            const notification = document.createElement('div');
            notification.style.cssText = `
                position: fixed;
                top: 50%;
                left: 50%;
                transform: translate(-50%, -50%);
                background: #00ff88;
                color: #000;
                padding: 15px 25px;
                border-radius: 10px;
                box-shadow: 0 5px 15px rgba(0,0,0,0.5);
                z-index: 1000;
                font-weight: bold;
                font-size: 16px;
            `;
            notification.textContent = message;
            document.body.appendChild(notification);
            
            setTimeout(() => {
                notification.remove();
            }, 2000);
        }

        // Avtomatik boshlash
        window.addEventListener('load', startScanner);

        // Markrof sensori uchun optimallashtirish
        window.addEventListener('deviceorientation', function() {
            // Telefon harakatlanganda skanerlash tezligini oshirish
            if (scanning && scanInterval) {
                clearInterval(scanInterval);
                scanInterval = setInterval(scanQRCode, 20);
            }
        });
    </script>
</body>
</html>
