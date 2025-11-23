<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Zoomli QR Skaner</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 20px;
            background: #f5f5f5;
            text-align: center;
        }
        .container {
            max-width: 500px;
            margin: 0 auto;
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
        }
        h1 {
            color: #333;
            margin-bottom: 20px;
        }
        #video-container {
            position: relative;
            width: 100%;
            height: 400px;
            margin: 20px 0;
            border-radius: 10px;
            overflow: hidden;
            background: #000;
        }
        video {
            width: 100%;
            height: 100%;
            object-fit: cover;
            transition: transform 0.3s ease;
        }
        #scan-box {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 250px;
            height: 250px;
            border: 2px solid #00ff00;
            border-radius: 10px;
            box-shadow: 0 0 0 1000px rgba(0, 0, 0, 0.5);
            pointer-events: none;
        }
        #result {
            margin-top: 20px;
            padding: 15px;
            background: #f0f8ff;
            border-radius: 8px;
            border: 1px solid #d0e8ff;
            display: none;
        }
        #result-text {
            word-break: break-all;
            margin: 10px 0;
            padding: 10px;
            background: white;
            border-radius: 5px;
            border: 1px solid #ddd;
        }
        button {
            background: #007bff;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 5px;
            cursor: pointer;
            margin: 5px;
            font-size: 16px;
        }
        button:hover {
            background: #0056b3;
        }
        button.secondary {
            background: #6c757d;
        }
        button.secondary:hover {
            background: #545b62;
        }
        #status {
            margin: 10px 0;
            padding: 10px;
            border-radius: 5px;
        }
        .scanning {
            background: #fff8e1;
            border: 1px solid #ffecb3;
            color: #856404;
        }
        .success {
            background: #e8f5e8;
            border: 1px solid #c8e6c9;
            color: #155724;
        }
        .error {
            background: #ffebee;
            border: 1px solid #ffcdd2;
            color: #721c24;
        }
        .zoom-controls {
            margin: 15px 0;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 10px;
        }
        .zoom-controls label {
            font-weight: bold;
            color: #333;
        }
        input[type="range"] {
            width: 150px;
        }
        .zoom-value {
            font-weight: bold;
            color: #007bff;
            min-width: 40px;
        }
        .auto-enhance {
            margin: 10px 0;
            padding: 10px;
            background: #e8f4ff;
            border-radius: 8px;
            border: 1px solid #cce5ff;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Zoomli QR Kod Skaneri</h1>
        
        <div class="zoom-controls">
            <label>Zoom:</label>
            <input type="range" id="zoom-range" min="1" max="5" step="0.1" value="1">
            <span id="zoom-value" class="zoom-value">1x</span>
            <button id="reset-zoom" class="secondary">Normal</button>
        </div>

        <div class="auto-enhance">
            <label>
                <input type="checkbox" id="auto-enhance" checked>
                Yaqinlashganda tasvirni tiniqlashtirish
            </label>
        </div>
        
        <div id="video-container">
            <video id="video" playsinline autoplay muted></video>
            <div id="scan-box"></div>
        </div>
        
        <div id="status" class="scanning">Kamera ochilmoqda...</div>
        
        <div>
            <button id="start-btn">Skanerlashni boshlash</button>
            <button id="stop-btn" class="secondary">To'xtatish</button>
        </div>
        
        <div id="result">
            <h3>QR kod ma'lumoti:</h3>
            <div id="result-text"></div>
            <button id="copy-btn">Nusxa olish</button>
            <button id="open-btn" class="secondary">Ochish</button>
            <button id="new-scan-btn">Yana skanerlash</button>
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
        const openBtn = document.getElementById('open-btn');
        const newScanBtn = document.getElementById('new-scan-btn');
        const zoomRange = document.getElementById('zoom-range');
        const zoomValue = document.getElementById('zoom-value');
        const resetZoomBtn = document.getElementById('reset-zoom');
        const autoEnhance = document.getElementById('auto-enhance');

        let stream = null;
        let scanning = false;
        let scanInterval = null;
        let track = null;
        let currentZoom = 1;
        let useHardwareZoom = false;

        // Zoom sozlamalari
        zoomRange.addEventListener('input', function() {
            currentZoom = parseFloat(this.value);
            zoomValue.textContent = currentZoom.toFixed(1) + 'x';
            applyZoom();
        });

        resetZoomBtn.addEventListener('click', function() {
            zoomRange.value = 1;
            currentZoom = 1;
            zoomValue.textContent = '1x';
            applyZoom();
        });

        // Zoomni qo'llash
        function applyZoom() {
            if (track && useHardwareZoom) {
                // Hardware zoom
                try {
                    track.applyConstraints({
                        advanced: [{ zoom: currentZoom }]
                    });
                } catch (e) {
                    console.warn('Hardware zoom ishlamadi, digital zoom ishlatiladi');
                    useHardwareZoom = false;
                    applyDigitalZoom();
                }
            } else {
                // Digital zoom
                applyDigitalZoom();
            }
        }

        // Digital zoom
        function applyDigitalZoom() {
            video.style.transform = `scale(${currentZoom})`;
            video.style.transformOrigin = 'center center';
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
                alert('Matn nusxalandi!');
            } catch (err) {
                alert('Nusxalashda xatolik: ' + err.message);
            }
        });

        // URLni ochish
        openBtn.addEventListener('click', () => {
            const text = resultText.textContent;
            try {
                const url = new URL(text);
                window.open(url.toString(), '_blank');
            } catch (e) {
                alert('Bu URL emas: ' + text);
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
                status.textContent = 'Kamera ochilmoqda...';
                status.className = 'status scanning';
                
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
                
                // Hardware zoom mavjudligini tekshirish
                const capabilities = track.getCapabilities ? track.getCapabilities() : {};
                if (capabilities.zoom) {
                    useHardwareZoom = true;
                    zoomRange.min = capabilities.zoom.min || 1;
                    zoomRange.max = Math.min(capabilities.zoom.max || 5, 5);
                    zoomRange.step = capabilities.zoom.step || 0.1;
                    
                    const settings = track.getSettings();
                    currentZoom = settings.zoom || 1;
                    zoomRange.value = currentZoom;
                    zoomValue.textContent = currentZoom.toFixed(1) + 'x';
                }
                
                status.textContent = 'QR kod qidirilmoqda...';
                scanning = true;
                
                // Har 200ms da QR kodni tekshirish
                scanInterval = setInterval(scanQRCode, 200);
                
            } catch (err) {
                status.textContent = 'Kamera ochilmadi: ' + err.message;
                status.className = 'status error';
                console.error('Camera error:', err);
            }
        }

        // Skanerlashni to'xtatish
        function stopScanner() {
            scanning = false;
            if (scanInterval) {
                clearInterval(scanInterval);
                scanInterval = null;
            }
            if (stream) {
                stream.getTracks().forEach(track => track.stop());
                stream = null;
                track = null;
            }
            status.textContent = 'Skanerlash to\'xtatildi';
            status.className = 'status error';
        }

        // Tasvirni tiniqlashtirish
        function enhanceImage(imageData) {
            if (!autoEnhance.checked || currentZoom < 1.5) {
                return imageData;
            }

            const data = imageData.data;
            const width = imageData.width;
            const height = imageData.height;
            
            // Kontrastni oshirish
            const contrast = 1 + (currentZoom * 0.1);
            const brightness = 1 + (currentZoom * 0.05);
            
            for (let i = 0; i < data.length; i += 4) {
                // Brightness
                data[i] = Math.min(255, Math.max(0, (data[i] - 128) * brightness + 128));
                data[i+1] = Math.min(255, Math.max(0, (data[i+1] - 128) * brightness + 128));
                data[i+2] = Math.min(255, Math.max(0, (data[i+2] - 128) * brightness + 128));
                
                // Contrast
                data[i] = Math.min(255, Math.max(0, (data[i] - 128) * contrast + 128));
                data[i+1] = Math.min(255, Math.max(0, (data[i+1] - 128) * contrast + 128));
                data[i+2] = Math.min(255, Math.max(0, (data[i+2] - 128) * contrast + 128));
            }
            
            return imageData;
        }

        // QR kodni skanerlash
        function scanQRCode() {
            if (!scanning || video.readyState !== video.HAVE_ENOUGH_DATA) return;
            
            const canvas = document.createElement('canvas');
            const context = canvas.getContext('2d');
            
            // Canvas o'lchamlarini sozlash
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            
            // Videodan rasm olish
            context.drawImage(video, 0, 0, canvas.width, canvas.height);
            
            // Rasm ma'lumotlarini olish
            let imageData = context.getImageData(0, 0, canvas.width, canvas.height);
            
            // Agar zoom katta bo'lsa va auto-enhance yoqilgan bo'lsa, tasvirni yaxshilash
            if (currentZoom > 1.5 && autoEnhance.checked) {
                imageData = enhanceImage(imageData);
            }
            
            // QR kodni dekod qilish
            const code = jsQR(imageData.data, imageData.width, imageData.height, {
                inversionAttempts: 'attemptBoth',
            });
            
            // Agar QR kod topilsa
            if (code) {
                stopScanner();
                
                // Natijani ko'rsatish
                resultText.textContent = code.data;
                result.style.display = 'block';
                
                status.textContent = 'QR kod topildi!';
                status.className = 'status success';
            }
        }

        // Dastur yuklanganda avtomatik boshlash
        window.addEventListener('load', startScanner);
        
        // Sahifa yopilganda kamerani to'xtatish
        window.addEventListener('beforeunload', stopScanner);
    </script>
</body>
</html>
