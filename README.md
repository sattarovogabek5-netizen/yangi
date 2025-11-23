<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tez QR Scanner</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 20px;
            color: white;
        }
        
        .container {
            width: 100%;
            max-width: 400px;
            text-align: center;
        }
        
        .header {
            margin-bottom: 20px;
        }
        
        .header h1 {
            font-size: 24px;
            margin-bottom: 8px;
            text-shadow: 0 2px 4px rgba(0,0,0,0.3);
        }
        
        .header p {
            opacity: 0.9;
            font-size: 14px;
        }
        
        .scanner-container {
            position: relative;
            width: 100%;
            height: 400px;
            background: black;
            border-radius: 20px;
            overflow: hidden;
            box-shadow: 0 10px 30px rgba(0,0,0,0.3);
            margin-bottom: 20px;
        }
        
        #video {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }
        
        .scan-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
        }
        
        .scan-frame {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 250px;
            height: 250px;
            border: 2px solid rgba(255,255,255,0.8);
            border-radius: 15px;
            box-shadow: 0 0 0 500px rgba(0,0,0,0.5);
        }
        
        .scan-line {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 3px;
            background: linear-gradient(90deg, transparent, #00ff88, transparent);
            animation: scan 2s linear infinite;
            border-radius: 3px;
        }
        
        @keyframes scan {
            0% { top: 0; }
            100% { top: 100%; }
        }
        
        .corner {
            position: absolute;
            width: 20px;
            height: 20px;
            border: 3px solid #00ff88;
        }
        
        .corner-tl {
            top: -3px;
            left: -3px;
            border-right: none;
            border-bottom: none;
            border-top-left-radius: 10px;
        }
        
        .corner-tr {
            top: -3px;
            right: -3px;
            border-left: none;
            border-bottom: none;
            border-top-right-radius: 10px;
        }
        
        .corner-bl {
            bottom: -3px;
            left: -3px;
            border-right: none;
            border-top: none;
            border-bottom-left-radius: 10px;
        }
        
        .corner-br {
            bottom: -3px;
            right: -3px;
            border-left: none;
            border-top: none;
            border-bottom-right-radius: 10px;
        }
        
        .controls {
            display: flex;
            gap: 10px;
            margin-bottom: 15px;
            flex-wrap: wrap;
            justify-content: center;
        }
        
        .btn {
            padding: 12px 20px;
            border: none;
            border-radius: 25px;
            background: rgba(255,255,255,0.2);
            backdrop-filter: blur(10px);
            color: white;
            font-size: 14px;
            cursor: pointer;
            transition: all 0.3s ease;
            display: flex;
            align-items: center;
            gap: 8px;
            min-width: 120px;
            justify-content: center;
        }
        
        .btn:hover {
            background: rgba(255,255,255,0.3);
            transform: translateY(-2px);
        }
        
        .btn:active {
            transform: translateY(0);
        }
        
        .btn-primary {
            background: #00ff88;
            color: #333;
            font-weight: 600;
        }
        
        .btn-primary:hover {
            background: #00e676;
        }
        
        .status {
            background: rgba(255,255,255,0.1);
            padding: 15px;
            border-radius: 15px;
            margin-bottom: 15px;
            backdrop-filter: blur(10px);
        }
        
        .status-text {
            font-size: 14px;
            margin-bottom: 8px;
        }
        
        .stats {
            display: flex;
            justify-content: space-around;
            font-size: 12px;
            opacity: 0.8;
        }
        
        .stat {
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        
        .stat-value {
            font-weight: bold;
            font-size: 16px;
            margin-bottom: 2px;
        }
        
        .result-panel {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: white;
            color: #333;
            padding: 25px;
            border-radius: 20px;
            box-shadow: 0 20px 50px rgba(0,0,0,0.5);
            z-index: 1000;
            max-width: 90%;
            width: 350px;
            text-align: center;
            display: none;
        }
        
        .result-panel h3 {
            margin-bottom: 15px;
            color: #333;
        }
        
        .result-content {
            background: #f8f9fa;
            padding: 15px;
            border-radius: 10px;
            margin-bottom: 20px;
            word-break: break-all;
            font-size: 14px;
            text-align: left;
            max-height: 150px;
            overflow-y: auto;
        }
        
        .result-actions {
            display: flex;
            gap: 10px;
            justify-content: center;
            flex-wrap: wrap;
        }
        
        .result-btn {
            padding: 10px 20px;
            border: none;
            border-radius: 20px;
            background: #667eea;
            color: white;
            cursor: pointer;
            font-size: 14px;
            transition: all 0.3s ease;
            min-width: 100px;
        }
        
        .result-btn:hover {
            background: #5a6fd8;
            transform: translateY(-2px);
        }
        
        .result-btn-copy {
            background: #00b894;
        }
        
        .result-btn-copy:hover {
            background: #00a885;
        }
        
        .backdrop {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.7);
            z-index: 999;
            display: none;
        }
        
        .loading {
            display: none;
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: white;
            font-size: 16px;
        }
        
        .detection-box {
            position: absolute;
            border: 2px solid #00ff88;
            background: rgba(0, 255, 136, 0.1);
            pointer-events: none;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>⚡ Tez QR Scanner</h1>
            <p>Har qanday QR kodni tezda va aniq o'qing</p>
        </div>
        
        <div class="scanner-container">
            <video id="video" playsinline autoplay muted></video>
            <div class="scan-overlay">
                <div class="scan-frame">
                    <div class="scan-line"></div>
                    <div class="corner corner-tl"></div>
                    <div class="corner corner-tr"></div>
                    <div class="corner corner-bl"></div>
                    <div class="corner corner-br"></div>
                </div>
            </div>
            <div id="loading" class="loading">Kamera yuklanmoqda...</div>
        </div>
        
        <div class="status">
            <div class="status-text" id="statusText">Kamera ishga tushmoqda...</div>
            <div class="stats">
                <div class="stat">
                    <div class="stat-value" id="fpsCounter">0</div>
                    <div class="stat-label">FPS</div>
                </div>
                <div class="stat">
                    <div class="stat-value" id="scanTime">0ms</div>
                    <div class="stat-label">Tezlik</div>
                </div>
                <div class="stat">
                    <div class="stat-value" id="qrCount">0</div>
                    <div class="stat-label">Topildi</div>
                </div>
            </div>
        </div>
        
        <div class="controls">
            <button class="btn" id="toggleTorch">
                <span>🔦</span> Flesh
            </button>
            <button class="btn" id="zoomIn">
                <span>🔍</span> Zoom +
            </button>
            <button class="btn" id="zoomOut">
                <span>🔍</span> Zoom -
            </button>
            <button class="btn btn-primary" id="restartScanner">
                <span>🔄</span> Qayta ishga tushurish
            </button>
        </div>
    </div>
    
    <div class="backdrop" id="backdrop"></div>
    
    <div class="result-panel" id="resultPanel">
        <h3>✅ QR Kod Topildi!</h3>
        <div class="result-content" id="resultContent"></div>
        <div class="result-actions">
            <button class="result-btn result-btn-copy" id="copyResult">Nusxa olish</button>
            <button class="result-btn" id="openResult">Ochish</button>
            <button class="result-btn" id="closeResult">Yopish</button>
        </div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.js"></script>
    <script>
        // DOM elementlari
        const video = document.getElementById('video');
        const statusText = document.getElementById('statusText');
        const fpsCounter = document.getElementById('fpsCounter');
        const scanTime = document.getElementById('scanTime');
        const qrCount = document.getElementById('qrCount');
        const toggleTorchBtn = document.getElementById('toggleTorch');
        const zoomInBtn = document.getElementById('zoomIn');
        const zoomOutBtn = document.getElementById('zoomOut');
        const restartScannerBtn = document.getElementById('restartScanner');
        const resultPanel = document.getElementById('resultPanel');
        const resultContent = document.getElementById('resultContent');
        const copyResultBtn = document.getElementById('copyResult');
        const openResultBtn = document.getElementById('openResult');
        const closeResultBtn = document.getElementById('closeResult');
        const backdrop = document.getElementById('backdrop');
        const loading = document.getElementById('loading');

        // O'zgaruvchilar
        let stream = null;
        let track = null;
        let scanCanvas = null;
        let scanCtx = null;
        let scanLoopId = null;
        let torchOn = false;
        let currentZoom = 1;
        let fps = 0;
        let frameCount = 0;
        let lastFpsUpdate = 0;
        let totalScanTime = 0;
        let scanCount = 0;
        let qrFoundCount = 0;

        // Dasturni ishga tushirish
        window.addEventListener('load', initializeScanner);

        async function initializeScanner() {
            loading.style.display = 'block';
            statusText.textContent = 'Kamera ochilmoqda...';
            
            try {
                // Yuqori sifatli kamera sozlamalari
                stream = await navigator.mediaDevices.getUserMedia({
                    video: {
                        facingMode: 'environment',
                        width: { ideal: 1920 },
                        height: { ideal: 1080 },
                        frameRate: { ideal: 60 }
                    },
                    audio: false
                });

                video.srcObject = stream;
                
                video.onloadedmetadata = () => {
                    video.play();
                    track = stream.getVideoTracks()[0];
                    setupCanvas();
                    startScanning();
                    loading.style.display = 'none';
                    statusText.textContent = 'QR kodni skanerlang...';
                    updateStatus('success', 'Kamera tayyor!');
                };

            } catch (error) {
                loading.style.display = 'none';
                updateStatus('error', `Kamera xatosi: ${error.message}`);
                console.error('Camera error:', error);
            }
        }

        function setupCanvas() {
            scanCanvas = document.createElement('canvas');
            scanCtx = scanCanvas.getContext('2d', { willReadFrequently: true });
            scanCanvas.width = video.videoWidth;
            scanCanvas.height = video.videoHeight;
        }

        function startScanning() {
            if (scanLoopId) {
                cancelAnimationFrame(scanLoopId);
            }
            scanLoop();
        }

        function scanLoop() {
            const startTime = performance.now();
            
            if (video.readyState === video.HAVE_ENOUGH_DATA) {
                // Canvas o'lchamlarini yangilash
                if (scanCanvas.width !== video.videoWidth || scanCanvas.height !== video.videoHeight) {
                    scanCanvas.width = video.videoWidth;
                    scanCanvas.height = video.videoHeight;
                }

                // Tasvirni canvasga chizish
                scanCtx.drawImage(video, 0, 0, scanCanvas.width, scanCanvas.height);
                
                // ImageData olish
                const imageData = scanCtx.getImageData(0, 0, scanCanvas.width, scanCanvas.height);
                
                // QR kodni qidirish
                try {
                    const code = jsQR(imageData.data, imageData.width, imageData.height, {
                        inversionAttempts: 'attemptBoth',
                        // Qo'shimcha optimizatsiyalar
                    });

                    if (code) {
                        handleQRDetection(code, imageData);
                    }
                } catch (error) {
                    console.error('QR scanning error:', error);
                }
            }

            // FPS hisoblash
            updateFPS();
            scanCount++;
            
            const endTime = performance.now();
            totalScanTime += (endTime - startTime);
            
            // O'rtacha skanerlash vaqtini yangilash
            if (scanCount % 10 === 0) {
                const avgScanTime = (totalScanTime / 10).toFixed(1);
                scanTime.textContent = `${avgScanTime}ms`;
                totalScanTime = 0;
            }

            scanLoopId = requestAnimationFrame(scanLoop);
        }

        function handleQRDetection(code, imageData) {
            qrFoundCount++;
            qrCount.textContent = qrFoundCount;
            
            // Natijani ko'rsatish
            showResult(code.data);
            
            // Vaqinchalik to'xtatish (1 soniya)
            setTimeout(() => {
                startScanning();
            }, 1000);
        }

        function showResult(data) {
            resultContent.textContent = data;
            resultPanel.style.display = 'block';
            backdrop.style.display = 'block';
            updateStatus('success', 'QR kod topildi!');
        }

        // Flesh (torch) boshqaruvi
        toggleTorchBtn.addEventListener('click', async () => {
            if (!track) return;
            
            try {
                const capabilities = track.getCapabilities();
                if (capabilities.torch) {
                    torchOn = !torchOn;
                    await track.applyConstraints({
                        advanced: [{ torch: torchOn }]
                    });
                    toggleTorchBtn.innerHTML = torchOn ? 
                        '<span>💡</span> Flesh Och' : 
                        '<span>🔦</span> Flesh Yoq';
                    updateStatus('info', torchOn ? 'Flesh yoqildi' : 'Flesh o\'chirildi');
                } else {
                    updateStatus('warning', 'Flesh qo\'llab-quvvatlanmaydi');
                }
            } catch (error) {
                console.error('Torch error:', error);
                updateStatus('error', 'Flesh boshqaruvi xatosi');
            }
        });

        // Zoom boshqaruvi
        zoomInBtn.addEventListener('click', () => {
            if (currentZoom < 3) {
                currentZoom += 0.5;
                applyZoom();
            }
        });

        zoomOutBtn.addEventListener('click', () => {
            if (currentZoom > 1) {
                currentZoom -= 0.5;
                applyZoom();
            }
        });

        function applyZoom() {
            if (track && track.getCapabilities().zoom) {
                try {
                    track.applyConstraints({
                        advanced: [{ zoom: currentZoom }]
                    });
                } catch (error) {
                    console.error('Zoom error:', error);
                }
            }
            updateStatus('info', `Zoom: ${currentZoom}x`);
        }

        // Qayta ishga tushurish
        restartScannerBtn.addEventListener('click', () => {
            if (stream) {
                stream.getTracks().forEach(track => track.stop());
            }
            initializeScanner();
            updateStatus('info', 'Skanner qayta ishga tushirildi');
        });

        // Natija paneli boshqaruvi
        copyResultBtn.addEventListener('click', async () => {
            try {
                await navigator.clipboard.writeText(resultContent.textContent);
                updateStatus('success', 'Nusxa olindi!');
            } catch (error) {
                updateStatus('error', 'Nusxa olish xatosi');
            }
        });

        openResultBtn.addEventListener('click', () => {
            const text = resultContent.textContent;
            if (text.startsWith('http://') || text.startsWith('https://')) {
                window.open(text, '_blank');
            } else {
                updateStatus('warning', 'Bu URL emas');
            }
        });

        closeResultBtn.addEventListener('click', () => {
            resultPanel.style.display = 'none';
            backdrop.style.display = 'none';
            updateStatus('info', 'QR qidirilmoqda...');
        });

        // FPS hisoblash
        function updateFPS() {
            frameCount++;
            const now = performance.now();
            
            if (now >= lastFpsUpdate + 1000) {
                fps = Math.round((frameCount * 1000) / (now - lastFpsUpdate));
                fpsCounter.textContent = fps;
                frameCount = 0;
                lastFpsUpdate = now;
            }
        }

        // Status yangilash
        function updateStatus(type, message) {
            statusText.textContent = message;
            
            // Status rangini yangilash
            const statusElement = statusText.parentElement;
            statusElement.style.background = 
                type === 'success' ? 'rgba(0, 255, 136, 0.2)' :
                type === 'error' ? 'rgba(255, 0, 68, 0.2)' :
                type === 'warning' ? 'rgba(255, 193, 7, 0.2)' :
                'rgba(255, 255, 255, 0.1)';
        }

        // Tozalash
        window.addEventListener('beforeunload', () => {
            if (stream) {
                stream.getTracks().forEach(track => track.stop());
            }
            if (scanLoopId) {
                cancelAnimationFrame(scanLoopId);
            }
        });
    </script>
</body>
</html>
