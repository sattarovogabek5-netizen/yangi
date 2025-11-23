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

        .header p {
            color: #7f8c8d;
            font-size: 14px;
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
            width: 220px;
            height: 220px;
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
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 220px;
            height: 4px;
            background: linear-gradient(90deg, transparent, #00ff88, transparent);
            animation: scan 2s linear infinite;
        }

        @keyframes pulse {
            0%, 100% { border-color: #00ff88; }
            50% { border-color: #00ccff; }
        }

        @keyframes scan {
            0% { top: 50%; }
            50% { top: 30%; }
            100% { top: 70%; }
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

        .control-group:last-child {
            margin-bottom: 0;
        }

        .control-label {
            display: flex;
            justify-content: between;
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
            transform: translateY(-2px);
        }

        .enhance-option input {
            width: 18px;
            height: 18px;
        }

        .enhance-option label {
            font-size: 14px;
            color: #2c3e50;
            cursor: pointer;
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
            box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);
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

        .btn-secondary:hover {
            background: rgba(102, 126, 234, 0.1);
            transform: translateY(-2px);
        }

        .btn-success {
            background: linear-gradient(135deg, #00ff88, #00ccff);
            color: white;
            box-shadow: 0 4px 15px rgba(0, 255, 136, 0.4);
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
            box-shadow: 0 10px 25px rgba(0, 255, 136, 0.2);
        }

        .result h3 {
            color: #2c3e50;
            margin-bottom: 15px;
            text-align: center;
            font-size: 18px;
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

        .sensitivity-indicator {
            display: flex;
            justify-content: space-between;
            margin-top: 5px;
            font-size: 12px;
            color: #7f8c8d;
        }

        .level-indicator {
            display: flex;
            gap: 5px;
            margin-top: 5px;
        }

        .level-dot {
            width: 8px;
            height: 8px;
            border-radius: 50%;
            background: #ecf0f1;
            transition: all 0.3s ease;
        }

        .level-dot.active {
            background: #00ff88;
        }

        .level-dot.active.high {
            background: #ff6b6b;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Kuchli QR Skaner</h1>
            <p>Har qanday QR kodni tez va aniq o'qing</p>
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
                    Zoom kuchi
                    <span id="zoom-level">Normal</span>
                </div>
                <input type="range" id="zoom-range" min="1" max="5" step="0.1" value="1">
                <div class="zoom-display">
                    <small>1x</small>
                    <div class="zoom-value" id="zoom-value">1.0x</div>
                    <small>5x</small>
                </div>
            </div>

            <div class="control-group">
                <div class="control-label">
                    Skanerlash sezgirligi
                    <span id="sensitivity-level">Yuqori</span>
                </div>
                <input type="range" id="sensitivity-range" min="1" max="10" step="1" value="8">
                <div class="sensitivity-indicator">
                    <small>Past</small>
                    <div class="level-indicator" id="level-indicator">
                        <div class="level-dot"></div>
                        <div class="level-dot"></div>
                        <div class="level-dot"></div>
                        <div class="level-dot"></div>
                        <div class="level-dot"></div>
                        <div class="level-dot"></div>
                        <div class="level-dot"></div>
                        <div class="level-dot"></div>
                        <div class="level-dot"></div>
                        <div class="level-dot"></div>
                    </div>
                    <small>Yuqori</small>
                </div>
            </div>

            <div class="enhance-options">
                <div class="enhance-option">
                    <input type="checkbox" id="auto-enhance" checked>
                    <label for="auto-enhance">Avto yaxshilash</label>
                </div>
                <div class="enhance-option">
                    <input type="checkbox" id="brightness-enhance" checked>
                    <label for="brightness-enhance">Yorqinlik</label>
                </div>
                <div class="enhance-option">
                    <input type="checkbox" id="contrast-enhance" checked>
                    <label for="contrast-enhance">Kontrast</label>
                </div>
                <div class="enhance-option">
                    <input type="checkbox" id="sharpness-enhance">
                    <label for="sharpness-enhance">Tiniqlik</label>
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
        const levelIndicator = document.getElementById('level-indicator');
        const autoEnhance = document.getElementById('auto-enhance');
        const brightnessEnhance = document.getElementById('brightness-enhance');
        const contrastEnhance = document.getElementById('contrast-enhance');
        const sharpnessEnhance = document.getElementById('sharpness-enhance');

        let stream = null;
        let scanning = false;
        let scanInterval = null;
        let track = null;
        let currentZoom = 1;
        let sensitivity = 8;

        // Zoom sozlamalari
        zoomRange.addEventListener('input', function() {
            currentZoom = parseFloat(this.value);
            zoomValue.textContent = currentZoom.toFixed(1) + 'x';
            
            // Zoom darajasiga qarab matn
            if (currentZoom < 2) {
                zoomLevel.textContent = "Normal";
            } else if (currentZoom < 3) {
                zoomLevel.textContent = "Kuchli";
            } else if (currentZoom < 4) {
                zoomLevel.textContent = "Juda kuchli";
            } else {
                zoomLevel.textContent = "Maksimal";
            }
            
            applyZoom();
        });

        // Sezgirlik sozlamalari
        sensitivityRange.addEventListener('input', function() {
            sensitivity = parseInt(this.value);
            updateSensitivityDisplay();
        });

        function updateSensitivityDisplay() {
            // Sezgirlik darajasi
            const levels = ["Juda past", "Past", "O'rta", "Yuqori", "Juda yuqori"];
            sensitivityLevel.textContent = levels[Math.floor(sensitivity / 2)];
            
            // Indikatorni yangilash
            const dots = levelIndicator.querySelectorAll('.level-dot');
            dots.forEach((dot, index) => {
                dot.classList.remove('active', 'high');
                if (index < sensitivity) {
                    dot.classList.add('active');
                    if (index >= 7) {
                        dot.classList.add('high');
                    }
                }
            });
        }

        // Zoomni qo'llash
        function applyZoom() {
            if (track) {
                try {
                    track.applyConstraints({
                        advanced: [{ zoom: currentZoom }]
                    });
                } catch (e) {
                    // Digital zoom
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
                
                // Tez skanerlash
                scanInterval = setInterval(scanQRCode, 100);
                
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

        // Tasvirni kuchli yaxshilash
        function enhanceImage(imageData) {
            if (!autoEnhance.checked) return imageData;

            const data = imageData.data;
            const width = imageData.width;
            const height = imageData.height;
            
            // Sezgirlik asosida kuchlanish
            const enhanceFactor = 1 + (sensitivity * 0.08);
            
            for (let i = 0; i < data.length; i += 4) {
                let r = data[i];
                let g = data[i + 1];
                let b = data[i + 2];

                // Yorqinlik yaxshilash
                if (brightnessEnhance.checked) {
                    const brightness = 1.2 + (currentZoom * 0.1);
                    r = Math.min(255, r * brightness);
                    g = Math.min(255, g * brightness);
                    b = Math.min(255, b * brightness);
                }

                // Kontrast yaxshilash
                if (contrastEnhance.checked) {
                    const contrast = 1.3 + (sensitivity * 0.05);
                    r = Math.min(255, 128 + (r - 128) * contrast);
                    g = Math.min(255, 128 + (g - 128) * contrast);
                    b = Math.min(255, 128 + (b - 128) * contrast);
                }

                // Tiniqlik yaxshilash (sharpness)
                if (sharpnessEnhance.checked && i > width * 4 + 4) {
                    // Oddiy sharpness filter
                    const sharpness = 0.3;
                    r = Math.min(255, r + (r - data[i - 4]) * sharpness);
                    g = Math.min(255, g + (g - data[i - 3]) * sharpness);
                    b = Math.min(255, b + (b - data[i - 2]) * sharpness);
                }

                data[i] = r;
                data[i + 1] = g;
                data[i + 2] = b;
            }
            
            return imageData;
        }

        // QR kodni skanerlash
        function scanQRCode() {
            if (!scanning || video.readyState !== video.HAVE_ENOUGH_DATA) return;
            
            const canvas = document.createElement('canvas');
            const context = canvas.getContext('2d');
            
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            
            context.drawImage(video, 0, 0, canvas.width, canvas.height);
            
            let imageData = context.getImageData(0, 0, canvas.width, canvas.height);
            
            // Tasvirni yaxshilash
            imageData = enhanceImage(imageData);
            
            // Turli xil inversion sozlamalari bilan skanerlash
            let code = jsQR(imageData.data, imageData.width, imageData.height, {
                inversionAttempts: sensitivity > 7 ? 'attemptBoth' : 'dontInvert',
            });

            // Yuqori sezgirlikda qo'shimcha urinishlar
            if (!code && sensitivity > 8) {
                // Kichikroq maydonni tekshirish
                const smallerCanvas = document.createElement('canvas');
                const smallCtx = smallerCanvas.getContext('2d');
                smallerCanvas.width = canvas.width / 2;
                smallerCanvas.height = canvas.height / 2;
                
                smallCtx.drawImage(video, 0, 0, smallerCanvas.width, smallerCanvas.height);
                const smallImageData = smallCtx.getImageData(0, 0, smallerCanvas.width, smallerCanvas.height);
                
                code = jsQR(smallImageData.data, smallerCanvas.width, smallerCanvas.height, {
                    inversionAttempts: 'attemptBoth',
                });
            }
            
            if (code) {
                stopScanner();
                
                resultText.textContent = code.data;
                result.style.display = 'block';
                
                status.textContent = '✅ QR kod topildi!';
                status.className = 'status status-success';
                
                // Agar URL bo'lsa, avtomatik ochish taklifi
                try {
                    new URL(code.data);
                    setTimeout(() => {
                        if (confirm('Bu URL. Ochishni xohlaysizmi?')) {
                            window.open(code.data, '_blank');
                        }
                    }, 500);
                } catch (e) {
                    // URL emas
                }
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
            }, 3000);
        }

        // Dastur yuklanganda avtomatik boshlash
        window.addEventListener('load', startScanner);
        
        // Sezgirlikni boshlang'ich sozlash
        updateSensitivityDisplay();
    </script>
</body>
</html>
