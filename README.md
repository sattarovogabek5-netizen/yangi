<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kuchli QR Skaner</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            padding: 20px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .container {
            max-width: 450px;
            width: 100%;
            background: rgba(255, 255, 255, 0.95);
            backdrop-filter: blur(10px);
            border-radius: 20px;
            padding: 25px;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.3);
            border: 1px solid rgba(255, 255, 255, 0.2);
        }

        .header {
            text-align: center;
            margin-bottom: 25px;
        }

        .header h1 {
            color: #2c3e50;
            font-size: 28px;
            font-weight: 700;
            margin-bottom: 8px;
            background: linear-gradient(135deg, #667eea, #764ba2);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }

        .video-container {
            position: relative;
            width: 100%;
            height: 350px;
            border-radius: 15px;
            overflow: hidden;
            background: #000;
            margin-bottom: 20px;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.3);
        }

        video {
            width: 100%;
            height: 100%;
            object-fit: cover;
            transition: transform 0.3s ease;
        }

        .scan-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
        }

        .scan-box {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 200px;
            height: 200px;
            border: 3px solid #00ff88;
            border-radius: 15px;
            box-shadow: 
                0 0 0 1000px rgba(0, 0, 0, 0.7),
                0 0 20px rgba(0, 255, 136, 0.5),
                inset 0 0 20px rgba(0, 255, 136, 0.3);
            animation: pulse 2s infinite;
        }

        .scan-lines {
            position: absolute;
            top: 0;
            left: 50%;
            transform: translateX(-50%);
            width: 200px;
            height: 4px;
            background: linear-gradient(90deg, transparent, #00ff88, transparent);
            animation: scan 2s linear infinite;
        }

        @keyframes pulse {
            0%, 100% { border-color: #00ff88; }
            50% { border-color: #00ccff; }
        }

        @keyframes scan {
            0% { top: 0; }
            100% { top: 200px; }
        }

        .controls {
            background: rgba(255, 255, 255, 0.8);
            border-radius: 15px;
            padding: 20px;
            margin-bottom: 20px;
            border: 1px solid rgba(255, 255, 255, 0.3);
        }

        .control-group {
            margin-bottom: 15px;
        }

        .control-label {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 8px;
            font-weight: 600;
            color: #2c3e50;
        }

        .control-label span {
            font-size: 12px;
            color: #00ff88;
            font-weight: bold;
        }

        input[type="range"] {
            width: 100%;
            height: 6px;
            background: linear-gradient(90deg, #667eea, #764ba2);
            border-radius: 10px;
            outline: none;
            -webkit-appearance: none;
        }

        input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 20px;
            height: 20px;
            background: #00ff88;
            border-radius: 50%;
            cursor: pointer;
            box-shadow: 0 2px 10px rgba(0, 255, 136, 0.5);
        }

        .zoom-display {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-top: 5px;
        }

        .zoom-value {
            font-weight: bold;
            color: #667eea;
            background: rgba(102, 126, 234, 0.1);
            padding: 4px 12px;
            border-radius: 20px;
            font-size: 14px;
        }

        .enhance-options {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            margin-top: 10px;
        }

        .enhance-option {
            display: flex;
            align-items: center;
            gap: 8px;
            padding: 10px;
            background: rgba(102, 126, 234, 0.1);
            border-radius: 10px;
            cursor: pointer;
            transition: all 0.3s ease;
        }

        .enhance-option:hover {
            background: rgba(102, 126, 234, 0.2);
        }

        .enhance-option input {
            width: 18px;
            height: 18px;
        }

        .buttons {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 12px;
            margin-bottom: 20px;
        }

        button {
            padding: 14px;
            border: none;
            border-radius: 12px;
            font-weight: 600;
            font-size: 15px;
            cursor: pointer;
            transition: all 0.3s ease;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 8px;
        }

        .btn-primary {
            background: linear-gradient(135deg, #667eea, #764ba2);
            color: white;
        }

        .btn-primary:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba(102, 126, 234, 0.6);
        }

        .btn-secondary {
            background: rgba(255, 255, 255, 0.8);
            color: #2c3e50;
            border: 2px solid rgba(102, 126, 234, 0.3);
        }

        .btn-success {
            background: linear-gradient(135deg, #00ff88, #00ccff);
            color: white;
        }

        .status {
            padding: 15px;
            border-radius: 12px;
            text-align: center;
            font-weight: 600;
            margin-bottom: 20px;
            transition: all 0.3s ease;
        }

        .status-scanning {
            background: linear-gradient(135deg, #fff8e1, #ffecb3);
            color: #856404;
            border: 2px solid #ffd54f;
        }

        .status-success {
            background: linear-gradient(135deg, #e8f5e8, #c8e6c9);
            color: #155724;
            border: 2px solid #00ff88;
        }

        .status-error {
            background: linear-gradient(135deg, #ffebee, #ffcdd2);
            color: #721c24;
            border: 2px solid #ff1744;
        }

        .result {
            background: rgba(255, 255, 255, 0.9);
            border-radius: 15px;
            padding: 20px;
            margin-top: 20px;
            display: none;
            border: 2px solid #00ff88;
        }

        .result h3 {
            color: #2c3e50;
            margin-bottom: 15px;
            text-align: center;
        }

        .result-text {
            background: white;
            padding: 15px;
            border-radius: 10px;
            border: 1px solid #e0e0e0;
            margin-bottom: 15px;
            font-size: 14px;
            line-height: 1.5;
            word-break: break-all;
            max-height: 120px;
            overflow-y: auto;
        }

        .result-buttons {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
        }

        .detection-info {
            font-size: 12px;
            color: #666;
            text-align: center;
            margin-top: 10px;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Kuchli QR Skaner</h1>
        </div>

        <div class="video-container">
            <video id="video" playsinline autoplay muted></video>
            <div class="scan-overlay">
                <div class="scan-box"></div>
                <div class="scan-lines"></div>
            </div>
        </div>

        <div class="status status-scanning" id="status">
            📱 Kamera ochilmoqda...
        </div>

        <div class="controls">
            <div class="control-group">
                <div class="control-label">
                    Zoom
                    <span id="zoom-level">Normal</span>
                </div>
                <input type="range" id="zoom-range" min="1" max="10" step="0.5" value="1">
                <div class="zoom-display">
                    <small>1x</small>
                    <div class="zoom-value" id="zoom-value">1x</div>
                    <small>10x</small>
                </div>
            </div>

            <div class="control-group">
                <div class="control-label">
                    Sezgirlik
                    <span id="sensitivity-level">Yuqori</span>
                </div>
                <input type="range" id="sensitivity-range" min="1" max="10" step="1" value="8">
                <div class="zoom-display">
                    <small>Past</small>
                    <small>Yuqori</small>
                </div>
            </div>

            <div class="enhance-options">
                <div class="enhance-option">
                    <input type="checkbox" id="auto-enhance" checked>
                    <label for="auto-enhance">Avto yaxshilash</label>
                </div>
                <div class="enhance-option">
                    <input type="checkbox" id="high-quality" checked>
                    <label for="high-quality">Yuqori sifat</label>
                </div>
            </div>
        </div>

        <div class="buttons">
            <button class="btn-primary" id="start-btn">
                <span>▶️</span>Skanerlash
            </button>
            <button class="btn-secondary" id="stop-btn">
                <span>⏹️</span>To'xtatish
            </button>
        </div>

        <div class="result" id="result">
            <h3>✅ QR Kod Topildi!</h3>
            <div class="result-text" id="result-text"></div>
            <div class="result-buttons">
                <button class="btn-success" id="copy-btn">
                    <span>📋</span>Nusxa olish
                </button>
                <button class="btn-primary" id="new-scan-btn">
                    <span>🔄</span>Qayta skaner
                </button>
            </div>
        </div>

        <div class="detection-info" id="detection-info">
            Kichik QR kodlarni o'qish uchun zoom qiling
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
        const zoomRange = document.getElementById('zoom-range');
        const zoomValue = document.getElementById('zoom-value');
        const zoomLevel = document.getElementById('zoom-level');
        const sensitivityRange = document.getElementById('sensitivity-range');
        const sensitivityLevel = document.getElementById('sensitivity-level');
        const autoEnhance = document.getElementById('auto-enhance');
        const highQuality = document.getElementById('high-quality');
        const detectionInfo = document.getElementById('detection-info');

        let stream = null;
        let scanning = false;
        let scanInterval = null;
        let track = null;
        let currentZoom = 1;
        let sensitivity = 8;
        let detectionAttempts = 0;

        // Zoom sozlamalari - ISHLASHI ANIQLANGAN
        zoomRange.addEventListener('input', function() {
            currentZoom = parseFloat(this.value);
            zoomValue.textContent = currentZoom + 'x';
            
            // Zoom darajasiga qarab matn
            if (currentZoom < 2) {
                zoomLevel.textContent = "Normal";
                detectionInfo.textContent = "Kichik QR kodlarni o'qish uchun zoom qiling";
            } else if (currentZoom < 4) {
                zoomLevel.textContent = "Kuchli";
                detectionInfo.textContent = "Yaxshi! Kichik QR kodlarni o'qish mumkin";
            } else if (currentZoom < 7) {
                zoomLevel.textContent = "Juda kuchli";
                detectionInfo.textContent = "Ajoyib! Juda kichik QR kodlarni ham o'qiy oladi";
            } else {
                zoomLevel.textContent = "Maksimal";
                detectionInfo.textContent = "Maksimal zoom - eng kichik QR kodlarni o'qing";
            }
            
            applyZoom();
        });

        // Sezgirlik sozlamalari
        sensitivityRange.addEventListener('input', function() {
            sensitivity = parseInt(this.value);
            const levels = ["Juda past", "Past", "O'rta", "Yuqori", "Juda yuqori"];
            sensitivityLevel.textContent = levels[Math.floor(sensitivity / 2)];
        });

        // ZOOM ISHLASHI - Digital zoom bilan
        function applyZoom() {
            if (track) {
                try {
                    // Hardware zoom
                    const capabilities = track.getCapabilities();
                    if (capabilities.zoom) {
                        track.applyConstraints({
                            advanced: [{ zoom: currentZoom }]
                        });
                    } else {
                        // Digital zoom
                        video.style.transform = `scale(${currentZoom})`;
                        video.style.transformOrigin = 'center center';
                    }
                } catch (e) {
                    // Digital zoom fallback
                    video.style.transform = `scale(${currentZoom})`;
                    video.style.transformOrigin = 'center center';
                }
            } else {
                // Digital zoom
                video.style.transform = `scale(${currentZoom})`;
                video.style.transformOrigin = 'center center';
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
            detectionAttempts = 0;
            startScanner();
        });

        // Skanerlashni boshlash funksiyasi
        async function startScanner() {
            try {
                status.textContent = '📱 Kamera ochilmoqda...';
                status.className = 'status status-scanning';
                
                // Kamerani ochish
                stream = await navigator.mediaDevices.getUserMedia({ 
                    video: { 
                        facingMode: 'environment',
                        width: { ideal: 1920 },
                        height: { ideal: 1080 }
                    } 
                });
                
                video.srcObject = stream;
                await video.play();
                
                // Kamera trackini olish
                track = stream.getVideoTracks()[0];
                
                status.textContent = '🔍 QR kod qidirilmoqda...';
                scanning = true;
                detectionAttempts = 0;
                
                // Tez skanerlash
                scanInterval = setInterval(scanQRCode, 150);
                
            } catch (err) {
                status.textContent = '❌ Kamera ochilmadi';
                status.className = 'status status-error';
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
                track = null;
            }
            status.textContent = '⏹️ Skanerlash to\'xtatildi';
            status.className = 'status status-error';
        }

        // Kuchli tasvir yaxshilash
        function enhanceImage(imageData) {
            if (!autoEnhance.checked) return imageData;

            const data = imageData.data;
            
            // Zoom va sezgirlik asosida kuchlanish
            const brightness = 1.0 + (currentZoom * 0.15);
            const contrast = 1.2 + (sensitivity * 0.08);
            
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

        // Kichik QR kodlarni o'qish uchun maxsus funksiya
        function scanSmallQR(imageData, scale = 2) {
            const originalWidth = imageData.width;
            const originalHeight = imageData.height;
            
            // Rasmni kattalashtirish
            const scaledCanvas = document.createElement('canvas');
            scaledCanvas.width = originalWidth * scale;
            scaledCanvas.height = originalHeight * scale;
            const scaledCtx = scaledCanvas.getContext('2d');
            
            // Original rasmni chizish
            scaledCtx.putImageData(imageData, 0, 0);
            
            // Rasmni kattalashtirish
            scaledCtx.imageSmoothingEnabled = true;
            scaledCtx.imageSmoothingQuality = 'high';
            scaledCtx.drawImage(scaledCanvas, 0, 0, originalWidth, originalHeight, 0, 0, scaledCanvas.width, scaledCanvas.height);
            
            const scaledImageData = scaledCtx.getImageData(0, 0, scaledCanvas.width, scaledCanvas.height);
            
            return jsQR(scaledImageData.data, scaledCanvas.width, scaledCanvas.height, {
                inversionAttempts: 'attemptBoth',
            });
        }

        // Asosiy skanerlash funksiyasi
        function scanQRCode() {
            if (!scanning || video.readyState !== video.HAVE_ENOUGH_DATA) return;
            
            detectionAttempts++;
            
            const canvas = document.createElement('canvas');
            const context = canvas.getContext('2d');
            
            // Yuqori sifatli skanerlash
            if (highQuality.checked) {
                canvas.width = video.videoWidth;
                canvas.height = video.videoHeight;
            } else {
                canvas.width = 800;
                canvas.height = 600;
            }
            
            context.drawImage(video, 0, 0, canvas.width, canvas.height);
            
            let imageData = context.getImageData(0, 0, canvas.width, canvas.height);
            
            // Tasvirni yaxshilash
            if (autoEnhance.checked) {
                imageData = enhanceImage(imageData);
            }
            
            // QR kodni qidirish
            let code = jsQR(imageData.data, imageData.width, imageData.height, {
                inversionAttempts: sensitivity > 7 ? 'attemptBoth' : 'dontInvert',
            });

            // Agar kichik QR kod bo'lsa, kattalashtirib qayta urinish
            if (!code && currentZoom > 3) {
                code = scanSmallQR(imageData, 2);
            }

            // Yuqori sezgirlikda qo'shimcha urinishlar
            if (!code && sensitivity > 8) {
                code = scanSmallQR(imageData, 3);
            }

            // Agar kod topilsa - ISHONCHLI RAVISHDA KO'RSATISH
            if (code && code.data && code.data.length > 0) {
                stopScanner();
                
                resultText.textContent = code.data;
                result.style.display = 'block';
                
                status.textContent = '✅ QR kod topildi!';
                status.className = 'status status-success';
                
                // URLni tekshirish
                try {
                    new URL(code.data);
                    setTimeout(() => {
                        if (confirm('Bu URL. Ochishni xohlaysizmi?')) {
                            window.open(code.data, '_blank');
                        }
                    }, 300);
                } catch (e) {
                    // URL emas
                }
            }
            
            // Statusni yangilash
            if (detectionAttempts % 10 === 0) {
                status.textContent = `🔍 QR kod qidirilmoqda... (${detectionAttempts})`;
            }
        }

        // Notification ko'rsatish
        function showNotification(message) {
            const notification = document.createElement('div');
            notification.style.cssText = `
                position: fixed;
                top: 20px;
                right: 20px;
                background: #00ff88;
                color: white;
                padding: 12px 20px;
                border-radius: 10px;
                box-shadow: 0 5px 15px rgba(0,0,0,0.3);
                z-index: 1000;
                font-weight: 600;
            `;
            notification.textContent = message;
            document.body.appendChild(notification);
            
            setTimeout(() => {
                notification.remove();
            }, 2000);
        }

        // Dastur yuklanganda avtomatik boshlash
        window.addEventListener('load', startScanner);
    </script>
</body>
</html>
