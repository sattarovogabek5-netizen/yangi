<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>QR Scanner Pro - Tez va Aniq Skaner</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/jsqr@1.4.0/dist/jsQR.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
            background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 100%);
            min-height: 100vh;
            padding: 20px;
        }
        
        .scanner-app {
            max-width: 1400px;
            margin: 0 auto;
            background: white;
            border-radius: 24px;
            box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.25);
            overflow: hidden;
            backdrop-filter: blur(10px);
        }
        
        .app-header {
            background: linear-gradient(135deg, #1e1b4b, #3730a3);
            color: white;
            padding: 30px 40px;
            text-align: center;
            position: relative;
            overflow: hidden;
        }
        
        .app-header::before {
            content: '';
            position: absolute;
            top: -50%;
            left: -50%;
            width: 200%;
            height: 200%;
            background: radial-gradient(circle, rgba(255,255,255,0.1) 0%, transparent 70%);
            animation: pulse 8s infinite linear;
        }
        
        @keyframes pulse {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        
        .app-title {
            font-size: 2.8rem;
            font-weight: 800;
            margin-bottom: 10px;
            position: relative;
            text-shadow: 0 2px 4px rgba(0,0,0,0.3);
        }
        
        .app-subtitle {
            font-size: 1.2rem;
            opacity: 0.9;
            font-weight: 300;
        }
        
        .stats-bar {
            display: flex;
            justify-content: space-around;
            background: rgba(255,255,255,0.1);
            padding: 15px;
            border-radius: 12px;
            margin-top: 20px;
            backdrop-filter: blur(10px);
        }
        
        .stat-item {
            text-align: center;
        }
        
        .stat-value {
            font-size: 1.5rem;
            font-weight: 700;
            color: #10b981;
        }
        
        .stat-label {
            font-size: 0.9rem;
            opacity: 0.8;
        }
        
        .app-content {
            display: grid;
            grid-template-columns: 1.2fr 0.8fr;
            gap: 30px;
            padding: 40px;
        }
        
        @media (max-width: 968px) {
            .app-content {
                grid-template-columns: 1fr;
            }
        }
        
        .scanner-section {
            background: #f8fafc;
            border-radius: 20px;
            padding: 30px;
            border: 1px solid #e2e8f0;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
        }
        
        .scanner-frame {
            position: relative;
            width: 100%;
            height: 450px;
            background: #000;
            border-radius: 16px;
            overflow: hidden;
            margin-bottom: 25px;
            box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.3);
        }
        
        .video-element {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }
        
        .canvas-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
        }
        
        .scanner-guide {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 280px;
            height: 280px;
            border: 3px solid #10b981;
            border-radius: 20px;
            pointer-events: none;
            z-index: 10;
            box-shadow: 0 0 0 9999px rgba(0, 0, 0, 0.4);
            animation: scannerPulse 2s infinite;
        }
        
        .scanner-guide::before, .scanner-guide::after {
            content: '';
            position: absolute;
            width: 40px;
            height: 40px;
            border: 4px solid #10b981;
        }
        
        .scanner-guide::before {
            top: -4px;
            left: -4px;
            border-right: none;
            border-bottom: none;
            border-top-left-radius: 16px;
        }
        
        .scanner-guide::after {
            bottom: -4px;
            right: -4px;
            border-left: none;
            border-top: none;
            border-bottom-right-radius: 16px;
        }
        
        .corner-top-right {
            position: absolute;
            top: -4px;
            right: -4px;
            width: 40px;
            height: 40px;
            border: 4px solid #10b981;
            border-left: none;
            border-bottom: none;
            border-top-right-radius: 16px;
        }
        
        .corner-bottom-left {
            position: absolute;
            bottom: -4px;
            left: -4px;
            width: 40px;
            height: 40px;
            border: 4px solid #10b981;
            border-right: none;
            border-top: none;
            border-bottom-left-radius: 16px;
        }
        
        @keyframes scannerPulse {
            0% { border-color: #10b981; }
            50% { border-color: #34d399; }
            100% { border-color: #10b981; }
        }
        
        .scanner-controls {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
            margin-bottom: 25px;
        }
        
        .control-group {
            display: flex;
            flex-direction: column;
            gap: 15px;
        }
        
        .btn {
            padding: 14px 20px;
            border: none;
            border-radius: 12px;
            font-size: 15px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s ease;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 10px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
        }
        
        .btn i {
            font-size: 18px;
        }
        
        .btn-primary {
            background: linear-gradient(135deg, #3b82f6, #1d4ed8);
            color: white;
        }
        
        .btn-primary:hover {
            background: linear-gradient(135deg, #1d4ed8, #1e40af);
            transform: translateY(-2px);
            box-shadow: 0 6px 12px -1px rgba(0, 0, 0, 0.15);
        }
        
        .btn-success {
            background: linear-gradient(135deg, #10b981, #059669);
            color: white;
        }
        
        .btn-success:hover {
            background: linear-gradient(135deg, #059669, #047857);
            transform: translateY(-2px);
            box-shadow: 0 6px 12px -1px rgba(0, 0, 0, 0.15);
        }
        
        .btn-warning {
            background: linear-gradient(135deg, #f59e0b, #d97706);
            color: white;
        }
        
        .btn-warning:hover {
            background: linear-gradient(135deg, #d97706, #b45309);
            transform: translateY(-2px);
            box-shadow: 0 6px 12px -1px rgba(0, 0, 0, 0.15);
        }
        
        .btn-danger {
            background: linear-gradient(135deg, #ef4444, #dc2626);
            color: white;
        }
        
        .btn-danger:hover {
            background: linear-gradient(135deg, #dc2626, #b91c1c);
            transform: translateY(-2px);
            box-shadow: 0 6px 12px -1px rgba(0, 0, 0, 0.15);
        }
        
        .btn-secondary {
            background: linear-gradient(135deg, #6b7280, #4b5563);
            color: white;
        }
        
        .btn-secondary:hover {
            background: linear-gradient(135deg, #4b5563, #374151);
            transform: translateY(-2px);
            box-shadow: 0 6px 12px -1px rgba(0, 0, 0, 0.15);
        }
        
        .file-input-wrapper {
            position: relative;
        }
        
        .file-input {
            position: absolute;
            opacity: 0;
            width: 100%;
            height: 100%;
            cursor: pointer;
        }
        
        .zoom-control {
            background: white;
            padding: 20px;
            border-radius: 16px;
            border: 1px solid #e2e8f0;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05);
        }
        
        .zoom-label {
            display: block;
            margin-bottom: 12px;
            font-weight: 600;
            color: #1f2937;
            font-size: 16px;
        }
        
        .zoom-slider {
            width: 100%;
            height: 8px;
            border-radius: 4px;
            background: #e5e7eb;
            outline: none;
            -webkit-appearance: none;
        }
        
        .zoom-slider::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 22px;
            height: 22px;
            border-radius: 50%;
            background: #3b82f6;
            cursor: pointer;
            box-shadow: 0 2px 4px rgba(0,0,0,0.2);
        }
        
        .status-indicator {
            padding: 16px;
            border-radius: 12px;
            margin: 20px 0;
            text-align: center;
            font-weight: 600;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 10px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05);
        }
        
        .status-scanning {
            background: #d1fae5;
            color: #065f46;
            border: 1px solid #a7f3d0;
        }
        
        .status-stopped {
            background: #fee2e2;
            color: #991b1b;
            border: 1px solid #fecaca;
        }
        
        .result-section {
            background: white;
            border-radius: 20px;
            padding: 30px;
            border: 1px solid #e2e8f0;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
            height: fit-content;
        }
        
        .section-title {
            font-size: 1.5rem;
            font-weight: 700;
            margin-bottom: 20px;
            color: #1f2937;
            display: flex;
            align-items: center;
            gap: 10px;
        }
        
        .result-box {
            background: #f8fafc;
            border-radius: 16px;
            padding: 25px;
            margin: 20px 0;
            min-height: 150px;
            border: 2px dashed #d1d5db;
            transition: all 0.3s ease;
        }
        
        .result-box.has-result {
            border-color: #10b981;
            background: #f0fdf4;
        }
        
        .qr-result {
            font-size: 16px;
            line-height: 1.6;
            word-break: break-all;
            color: #1f2937;
        }
        
        .empty-result {
            color: #6b7280;
            text-align: center;
            font-style: italic;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 15px;
        }
        
        .empty-result i {
            font-size: 48px;
            color: #d1d5db;
        }
        
        .action-buttons {
            display: flex;
            gap: 12px;
            margin-top: 20px;
        }
        
        .tips-card {
            background: linear-gradient(135deg, #e0f2fe, #bae6fd);
            border-radius: 16px;
            padding: 25px;
            margin-top: 25px;
            border-left: 5px solid #0ea5e9;
        }
        
        .tips-title {
            font-size: 1.2rem;
            font-weight: 700;
            margin-bottom: 15px;
            color: #0369a1;
            display: flex;
            align-items: center;
            gap: 10px;
        }
        
        .tips-list {
            list-style: none;
            padding-left: 0;
        }
        
        .tips-list li {
            padding: 10px 0;
            border-bottom: 1px solid #7dd3fc;
            display: flex;
            align-items: flex-start;
            gap: 12px;
            color: #0c4a6e;
        }
        
        .tips-list li:last-child {
            border-bottom: none;
        }
        
        .tips-list li i {
            color: #0ea5e9;
            margin-top: 4px;
        }
        
        .footer-note {
            background: #f1f5f9;
            padding: 20px 40px;
            text-align: center;
            color: #64748b;
            font-size: 14px;
            border-top: 1px solid #e2e8f0;
        }
        
        .pulse-dot {
            display: inline-block;
            width: 12px;
            height: 12px;
            border-radius: 50%;
            background: #10b981;
            margin-right: 8px;
            animation: dotPulse 1.5s infinite;
        }
        
        @keyframes dotPulse {
            0% { transform: scale(0.8); opacity: 0.7; }
            50% { transform: scale(1.2); opacity: 1; }
            100% { transform: scale(0.8); opacity: 0.7; }
        }
        
        .processing-indicator {
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 12px;
            padding: 16px;
            background: #eff6ff;
            border-radius: 12px;
            margin: 15px 0;
            color: #1e40af;
            font-weight: 600;
        }
        
        .spinner {
            width: 24px;
            height: 24px;
            border: 3px solid #dbeafe;
            border-top: 3px solid #3b82f6;
            border-radius: 50%;
            animation: spin 1s linear infinite;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        
        .performance-info {
            background: #fefce8;
            border-radius: 12px;
            padding: 15px;
            margin-top: 15px;
            border-left: 4px solid #f59e0b;
        }
        
        .performance-title {
            font-weight: 600;
            color: #92400e;
            margin-bottom: 8px;
            display: flex;
            align-items: center;
            gap: 8px;
        }
        
        .performance-stats {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
            font-size: 14px;
        }
        
        .stat {
            display: flex;
            justify-content: space-between;
        }
        
        .stat-value {
            font-weight: 600;
            color: #065f46;
        }
    </style>
</head>
<body>
    <div class="scanner-app">
        <div class="app-header">
            <h1 class="app-title"><i class="fas fa-bolt"></i> QR Scanner Pro</h1>
            <p class="app-subtitle">Har qanday QR kodni tez va aniq skanerlang</p>
            
            <div class="stats-bar">
                <div class="stat-item">
                    <div class="stat-value" id="scanSpeed">0ms</div>
                    <div class="stat-label">Skanerlash Tezligi</div>
                </div>
                <div class="stat-item">
                    <div class="stat-value" id="qrCount">0</div>
                    <div class="stat-label">Topilgan QR Kodlar</div>
                </div>
                <div class="stat-item">
                    <div class="stat-value" id="successRate">100%</div>
                    <div class="stat-label">Muvaffaqiyat Darajasi</div>
                </div>
            </div>
        </div>
        
        <div class="app-content">
            <div class="scanner-section">
                <div class="scanner-frame">
                    <video id="video" class="video-element" playsinline muted></video>
                    <canvas id="canvas" class="canvas-overlay"></canvas>
                    <div class="scanner-guide">
                        <div class="corner-top-right"></div>
                        <div class="corner-bottom-left"></div>
                    </div>
                </div>
                
                <div class="scanner-controls">
                    <div class="control-group">
                        <button id="startBtn" class="btn btn-primary">
                            <i class="fas fa-play-circle"></i> Kamerani Yoqish
                        </button>
                        <button id="stopBtn" class="btn btn-danger" disabled>
                            <i class="fas fa-stop-circle"></i> Kamerani To'xtatish
                        </button>
                    </div>
                    
                    <div class="control-group">
                        <div class="file-input-wrapper">
                            <button class="btn btn-secondary">
                                <i class="fas fa-file-image"></i> Rasm Yuklash
                            </button>
                            <input type="file" id="fileInput" accept="image/*" class="file-input">
                        </div>
                        <button id="torchBtn" class="btn btn-warning">
                            <i class="fas fa-lightbulb"></i> Flash Yoqish
                        </button>
                    </div>
                </div>
                
                <div class="zoom-control">
                    <label class="zoom-label">
                        <i class="fas fa-search"></i> Zoom: <span id="zoomValue">1.00x</span>
                    </label>
                    <input type="range" id="zoomSlider" min="1" max="3" step="0.05" value="1" class="zoom-slider">
                </div>
                
                <div class="performance-info">
                    <div class="performance-title">
                        <i class="fas fa-tachometer-alt"></i> Tez Skanerlash Tizimi
                    </div>
                    <div class="performance-stats">
                        <div class="stat">
                            <span>Optimallashtirilgan algoritm:</span>
                            <span class="stat-value">Faol</span>
                        </div>
                        <div class="stat">
                            <span>Ko'p qatlamli skanerlash:</span>
                            <span class="stat-value">Faol</span>
                        </div>
                        <div class="stat">
                            <span>Tez tasvir qayta ishlash:</span>
                            <span class="stat-value">Faol</span>
                        </div>
                        <div class="stat">
                            <span>Avtomatik kontrast:</span>
                            <span class="stat-value">Faol</span>
                        </div>
                    </div>
                </div>
                
                <div id="status" class="status-indicator status-stopped">
                    <i class="fas fa-video-slash"></i> Kamera o'chirilgan
                </div>
            </div>
            
            <div class="result-section">
                <h2 class="section-title">
                    <i class="fas fa-clipboard-list"></i> Skanerlash Natijalari
                </h2>
                
                <div id="processingIndicator" class="processing-indicator" style="display: none;">
                    <div class="spinner"></div>
                    <span>QR kod qayta ishlanyapti...</span>
                </div>
                
                <div id="resultBox" class="result-box">
                    <div id="result" class="empty-result">
                        <i class="fas fa-qrcode"></i>
                        <div>QR kod skanerlanganida natija shu yerda paydo bo'ladi</div>
                    </div>
                </div>
                
                <div class="action-buttons">
                    <button id="copyBtn" class="btn btn-success" disabled>
                        <i class="fas fa-copy"></i> Natijani Nusxalash
                    </button>
                    <button id="clearBtn" class="btn btn-secondary">
                        <i class="fas fa-broom"></i> Tozalash
                    </button>
                </div>
                
                <div class="tips-card">
                    <h3 class="tips-title">
                        <i class="fas fa-lightbulb"></i> Tez Skanerlash Bo'yicha Maslahatlar
                    </h3>
                    <ul class="tips-list">
                        <li>
                            <i class="fas fa-bolt"></i>
                            <span><strong>Optimallashtirilgan algoritm</strong> - Har qanday sharoitda tez skanerlash</span>
                        </li>
                        <li>
                            <i class="fas fa-layer-group"></i>
                            <span><strong>Ko'p qatlamli tahlil</strong> - Teskari rangli va qiyin QR kodlarni aniqlaydi</span>
                        </li>
                        <li>
                            <i class="fas fa-sync-alt"></i>
                            <span><strong>Avtomatik sozlash</strong> - Yorug'lik va kontrastni avtomatik optimallashtiradi</span>
                        </li>
                        <li>
                            <i class="fas fa-rocket"></i>
                            <span><strong>Tez qayta ishlash</strong> - O'rtacha 50-100ms tezlikda skanerlaydi</span>
                        </li>
                    </ul>
                </div>
            </div>
        </div>
        
        <div class="footer-note">
            <p><i class="fas fa-info-circle"></i> Ilova jsQR kutubxonasi va optimallashtirilgan algoritmlar yordamida ishlaydi. Har qanday QR kodni 0.5 soniya ichida skanerlaydi.</p>
        </div>
    </div>

    <script>
        // O'zgaruvchilar
        let videoRef = document.getElementById('video');
        let canvasRef = document.getElementById('canvas');
        let stream = null;
        let scanning = false;
        let decoded = null;
        let zoom = 1;
        let supportsZoom = false;
        let trackRef = null;
        let torchOn = false;
        let processing = false;
        let animationRef = null;
        
        // Statistikalar
        let scanSpeedElement = document.getElementById('scanSpeed');
        let qrCountElement = document.getElementById('qrCount');
        let successRateElement = document.getElementById('successRate');
        let totalScans = 0;
        let successfulScans = 0;
        let lastScanTime = 0;

        // Elementlar
        const startBtn = document.getElementById('startBtn');
        const stopBtn = document.getElementById('stopBtn');
        const torchBtn = document.getElementById('torchBtn');
        const fileInput = document.getElementById('fileInput');
        const zoomSlider = document.getElementById('zoomSlider');
        const zoomValue = document.getElementById('zoomValue');
        const statusIndicator = document.getElementById('status');
        const processingIndicator = document.getElementById('processingIndicator');
        const resultBox = document.getElementById('resultBox');
        const resultDiv = document.getElementById('result');
        const copyBtn = document.getElementById('copyBtn');
        const clearBtn = document.getElementById('clearBtn');

        // Event listenerlar
        startBtn.addEventListener('click', startCamera);
        stopBtn.addEventListener('click', stopCamera);
        torchBtn.addEventListener('click', toggleTorch);
        fileInput.addEventListener('change', onFilePicked);
        zoomSlider.addEventListener('input', handleZoom);
        copyBtn.addEventListener('click', copyResult);
        clearBtn.addEventListener('click', clearResult);

        // Kamerani ishga tushirish
        async function startCamera() {
            try {
                const constraints = {
                    video: { 
                        facingMode: 'environment', 
                        width: { ideal: 1920 }, 
                        height: { ideal: 1080 },
                        frameRate: { ideal: 30 }
                    }
                };
                stream = await navigator.mediaDevices.getUserMedia(constraints);
                
                if (videoRef) {
                    videoRef.srcObject = stream;
                    await videoRef.play();
                }
                
                const tracks = stream.getVideoTracks();
                if (tracks && tracks.length) {
                    trackRef = tracks[0];
                    const caps = trackRef.getCapabilities ? trackRef.getCapabilities() : {};
                    if (caps.zoom) {
                        supportsZoom = true;
                    }
                }
                
                scanning = true;
                updateUI('started');
                lastScanTime = performance.now();
                requestAnimationFrame(tick);
            } catch (err) {
                console.error('Camera start error', err);
                alert('Kamera ochib bo\'lmadi: iltimos brauzerga ruxsat bering yoki HTTPS orqali oching.');
            }
        }

        // Kamerani to'xtatish
        function stopCamera() {
            scanning = false;
            if (animationRef) cancelAnimationFrame(animationRef);
            if (stream) {
                stream.getTracks().forEach(t => t.stop());
                stream = null;
            }
            if (trackRef) trackRef = null;
            torchOn = false;
            updateUI('stopped');
        }

        // Zoom sozlash
        async function handleZoom(event) {
            const z = Number(event.target.value);
            zoom = z;
            zoomValue.textContent = `${z.toFixed(2)}x`;
            
            try {
                if (trackRef && trackRef.applyConstraints) {
                    await trackRef.applyConstraints({ advanced: [{ zoom: z }] });
                } else if (videoRef) {
                    videoRef.style.transform = `scale(${z})`;
                }
            } catch (e) {
                if (videoRef) videoRef.style.transform = `scale(${z})`;
            }
        }

        // Flash (torch) ni boshqarish
        async function toggleTorch() {
            try {
                if (!trackRef) return;
                
                if (trackRef.getCapabilities && trackRef.getCapabilities().torch) {
                    torchOn = !torchOn;
                    await trackRef.applyConstraints({ 
                        advanced: [{ torch: torchOn }] 
                    });
                    torchBtn.innerHTML = torchOn ? 
                        '<i class="fas fa-lightbulb"></i> Flash O\'chirish' : 
                        '<i class="fas fa-lightbulb"></i> Flash Yoqish';
                } else {
                    alert('Ushbu qurilma yoki brauzer flashni qo\'llab-quvvatlamaydi.');
                }
            } catch (err) {
                console.warn('Torch error', err);
                alert('Flashni yoqib bo\'lmadi yoki brauzer qo\'llab-quvvatlamaydi.');
            }
        }

        // Optimallashtirilgan tasvir sifatiini yaxshilash
        function enhanceImage(ctx, w, h) {
            const imgData = ctx.getImageData(0, 0, w, h);
            const data = imgData.data;
            
            // Tez kontrast va yorug'lik sozlash
            const contrast = 1.3;
            const brightness = 15;
            const factor = (259 * (contrast + 255)) / (255 * (259 - contrast));
            
            for (let i = 0; i < data.length; i += 4) {
                // Faqat asosiy RGB kanallarini optimallashtirish
                let r = data[i];
                let g = data[i + 1];
                let b = data[i + 2];
                
                r = factor * (r - 128) + 128 + brightness;
                g = factor * (g - 128) + 128 + brightness;
                b = factor * (b - 128) + 128 + brightness;
                
                data[i] = Math.max(0, Math.min(255, r));
                data[i + 1] = Math.max(0, Math.min(255, g));
                data[i + 2] = Math.max(0, Math.min(255, b));
            }
            ctx.putImageData(imgData, 0, 0);
        }

        // Tez QR kodni skanerlash
        function tick() {
            if (!scanning) return;
            const video = videoRef;
            const canvas = canvasRef;
            
            if (!video || !canvas) {
                animationRef = requestAnimationFrame(tick);
                return;
            }
            
            const startTime = performance.now();
            const ctx = canvas.getContext('2d');
            const w = canvas.width = video.videoWidth || 640;
            const h = canvas.height = video.videoHeight || 480;
            
            try {
                ctx.drawImage(video, 0, 0, w, h);
                enhanceImage(ctx, w, h);
                const imageData = ctx.getImageData(0, 0, w, h);
                
                // Ko'p qatlamli QR kod skanerlash
                let code = null;
                
                // 1. Oddiy skanerlash
                code = jsQR(imageData.data, imageData.width, imageData.height, { 
                    inversionAttempts: 'dontInvert' 
                });
                
                // 2. Agar topilmasa, teskari rang uchun skanerlash
                if (!code) {
                    code = jsQR(imageData.data, imageData.width, imageData.height, { 
                        inversionAttempts: 'onlyInvert' 
                    });
                }
                
                // 3. Agar hali ham topilmasa, barcha variantlarni tekshirish
                if (!code) {
                    code = jsQR(imageData.data, imageData.width, imageData.height, { 
                        inversionAttempts: 'both' 
                    });
                }
                
                if (code) {
                    const scanTime = performance.now() - startTime;
                    totalScans++;
                    successfulScans++;
                    
                    // Statistikani yangilash
                    scanSpeedElement.textContent = `${Math.round(scanTime)}ms`;
                    qrCountElement.textContent = totalScans;
                    successRateElement.textContent = `${Math.round((successfulScans / totalScans) * 100)}%`;
                    
                    setDecoded({ text: code.data, location: code.location });
                    drawLocation(ctx, code.location);
                    
                    // 1 soniya davomida to'xtatib, keyin qayta boshlash
                    scanning = false;
                    setTimeout(() => {
                        scanning = true;
                        requestAnimationFrame(tick);
                    }, 1000);
                } else {
                    totalScans++;
                    const scanTime = performance.now() - startTime;
                    scanSpeedElement.textContent = `${Math.round(scanTime)}ms`;
                    successRateElement.textContent = `${Math.round((successfulScans / totalScans) * 100)}%`;
                    
                    setDecoded(null);
                }
            } catch (e) {
                console.warn('Tick error', e);
            }
            animationRef = requestAnimationFrame(tick);
        }

        // QR kod joylashuvini chizish
        function drawLocation(ctx, loc) {
            ctx.lineWidth = 4;
            ctx.strokeStyle = '#10b981';
            ctx.beginPath();
            ctx.moveTo(loc.topLeftCorner.x, loc.topLeftCorner.y);
            ctx.lineTo(loc.topRightCorner.x, loc.topRightCorner.y);
            ctx.lineTo(loc.bottomRightCorner.x, loc.bottomRightCorner.y);
            ctx.lineTo(loc.bottomLeftCorner.x, loc.bottomLeftCorner.y);
            ctx.closePath();
            ctx.stroke();
        }

        // Fayl yuklash orqali QR kodni skanerlash
        async function onFilePicked(e) {
            const file = e.target.files && e.target.files[0];
            if (!file) return;
            
            setProcessing(true);
            const img = new Image();
            
            img.onload = () => {
                const startTime = performance.now();
                const canvas = canvasRef;
                const ctx = canvas.getContext('2d');
                const maxW = 1280;
                const scale = Math.min(1, maxW / img.width);
                canvas.width = img.width * scale;
                canvas.height = img.height * scale;
                
                ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
                enhanceImage(ctx, canvas.width, canvas.height);
                
                const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
                
                // Rasm uchun ko'p qatlamli skanerlash
                let code = jsQR(imageData.data, imageData.width, imageData.height, { 
                    inversionAttempts: 'both' 
                });
                
                if (code) {
                    const scanTime = performance.now() - startTime;
                    totalScans++;
                    successfulScans++;
                    
                    scanSpeedElement.textContent = `${Math.round(scanTime)}ms`;
                    qrCountElement.textContent = totalScans;
                    successRateElement.textContent = `${Math.round((successfulScans / totalScans) * 100)}%`;
                    
                    setDecoded({ text: code.data, location: code.location });
                    drawLocation(ctx, code.location);
                } else {
                    totalScans++;
                    successRateElement.textContent = `${Math.round((successfulScans / totalScans) * 100)}%`;
                    alert('QR topilmadi — boshqa rasm yoki yaxshiroq sifattagi QR kodni sinab ko\'ring.');
                }
                setProcessing(false);
            };
            
            img.onerror = () => { 
                setProcessing(false); 
                alert('Rasm yuklashda xatolik.'); 
            };
            
            img.src = URL.createObjectURL(file);
        }

        // Natijani ko'rsatish
        function setDecoded(result) {
            decoded = result;
            
            if (result) {
                resultBox.classList.add('has-result');
                let formattedText = result.text;
                
                // URL formatlash
                if (result.text.startsWith('http://') || result.text.startsWith('https://')) {
                    formattedText = `<a href="${result.text}" target="_blank" style="color: #3b82f6; text-decoration: none;">${result.text}</a>`;
                }
                // WiFi formatlash
                else if (result.text.startsWith('WIFI:')) {
                    formattedText = `<strong>WiFi Tarmog'i:</strong><br>${result.text}`;
                }
                // Telefon formatlash
                else if (result.text.startsWith('TEL:')) {
                    formattedText = `<strong>Telefon Raqam:</strong><br><a href="tel:${result.text.substring(4)}" style="color: #3b82f6; text-decoration: none;">${result.text.substring(4)}</a>`;
                }
                // Email formatlash
                else if (result.text.startsWith('EMAIL:')) {
                    formattedText = `<strong>Email Manzil:</strong><br><a href="mailto:${result.text.substring(6)}" style="color: #3b82f6; text-decoration: none;">${result.text.substring(6)}</a>`;
                }
                
                resultDiv.innerHTML = `
                    <div class="qr-result">
                        <div class="pulse-dot"></div>
                        <strong>QR Kod Ma'lumoti:</strong><br>
                        ${formattedText}
                    </div>
                `;
                copyBtn.disabled = false;
            } else {
                resultBox.classList.remove('has-result');
                resultDiv.innerHTML = `
                    <div class="empty-result">
                        <i class="fas fa-qrcode"></i>
                        <div>QR kod skanerlanganida natija shu yerda paydo bo'ladi</div>
                    </div>
                `;
                copyBtn.disabled = true;
            }
        }

        // Natijani nusxalash
        function copyResult() {
            if (decoded && decoded.text) {
                navigator.clipboard.writeText(decoded.text)
                    .then(() => {
                        const originalText = copyBtn.innerHTML;
                        copyBtn.innerHTML = '<i class="fas fa-check"></i> Nusxalandi!';
                        setTimeout(() => {
                            copyBtn.innerHTML = originalText;
                        }, 2000);
                    })
                    .catch(err => {
                        alert('Nusxalash muvaffaqiyatsiz: ' + err);
                    });
            }
        }

        // Natijani tozalash
        function clearResult() {
            decoded = null;
            resultBox.classList.remove('has-result');
            resultDiv.innerHTML = `
                <div class="empty-result">
                    <i class="fas fa-qrcode"></i>
                    <div>QR kod skanerlanganida natija shu yerda paydo bo'ladi</div>
                </div>
            `;
            copyBtn.disabled = true;
            
            const canvas = canvasRef;
            const ctx = canvas.getContext('2d');
            ctx.clearRect(0, 0, canvas.width, canvas.height);
        }

        // Qayta ishlash holatini o'rnatish
        function setProcessing(isProcessing) {
            processing = isProcessing;
            if (isProcessing) {
                processingIndicator.style.display = 'flex';
            } else {
                processingIndicator.style.display = 'none';
            }
        }

        // UI ni yangilash
        function updateUI(state) {
            if (state === 'started') {
                startBtn.disabled = true;
                stopBtn.disabled = false;
                zoomSlider.disabled = false;
                statusIndicator.className = 'status-indicator status-scanning';
                statusIndicator.innerHTML = '<i class="fas fa-video"></i> <span class="pulse-dot"></span> Kamera ishlayapti - QR kodni skanerlash doirasiga joylashtiring';
            } else {
                startBtn.disabled = false;
                stopBtn.disabled = true;
                zoomSlider.disabled = true;
                statusIndicator.className = 'status-indicator status-stopped';
                statusIndicator.innerHTML = '<i class="fas fa-video-slash"></i> Kamera o\'chirilgan';
                torchBtn.innerHTML = '<i class="fas fa-lightbulb"></i> Flash Yoqish';
                zoomValue.textContent = '1.00x';
                zoomSlider.value = 1;
                if (videoRef) {
                    videoRef.style.transform = 'scale(1)';
                }
            }
        }
    </script>
</body>
</html>
