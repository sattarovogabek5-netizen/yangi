<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>QR Scanner - To'liq Ishlaydigan</title>
    <script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.js"></script>
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
        }
        
        .container {
            max-width: 1000px;
            margin: 0 auto;
            background: white;
            border-radius: 20px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.1);
            overflow: hidden;
        }
        
        .header {
            background: linear-gradient(135deg, #2c3e50, #34495e);
            color: white;
            padding: 30px;
            text-align: center;
        }
        
        .header h1 {
            font-size: 2.5rem;
            margin-bottom: 10px;
        }
        
        .header p {
            opacity: 0.9;
            font-size: 1.1rem;
        }
        
        .content {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 30px;
            padding: 30px;
        }
        
        @media (max-width: 768px) {
            .content {
                grid-template-columns: 1fr;
            }
        }
        
        .scanner-section {
            background: #f8f9fa;
            border-radius: 15px;
            padding: 25px;
        }
        
        .video-container {
            position: relative;
            width: 100%;
            height: 300px;
            background: #000;
            border-radius: 10px;
            overflow: hidden;
            margin-bottom: 20px;
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
        }
        
        .controls {
            display: flex;
            gap: 10px;
            flex-wrap: wrap;
            margin-bottom: 20px;
        }
        
        .btn {
            padding: 12px 20px;
            border: none;
            border-radius: 8px;
            font-size: 14px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s ease;
            flex: 1;
            min-width: 120px;
        }
        
        .btn-primary {
            background: #3498db;
            color: white;
        }
        
        .btn-primary:hover {
            background: #2980b9;
        }
        
        .btn-success {
            background: #27ae60;
            color: white;
        }
        
        .btn-success:hover {
            background: #219a52;
        }
        
        .btn-warning {
            background: #f39c12;
            color: white;
        }
        
        .btn-warning:hover {
            background: #d68910;
        }
        
        .btn-danger {
            background: #e74c3c;
            color: white;
        }
        
        .btn-danger:hover {
            background: #c0392b;
        }
        
        .file-input-wrapper {
            position: relative;
            flex: 1;
        }
        
        .file-input {
            position: absolute;
            opacity: 0;
            width: 100%;
            height: 100%;
            cursor: pointer;
        }
        
        .file-label {
            display: block;
            padding: 12px 20px;
            background: #95a5a6;
            color: white;
            border-radius: 8px;
            text-align: center;
            cursor: pointer;
            transition: background 0.3s ease;
        }
        
        .file-label:hover {
            background: #7f8c8d;
        }
        
        .zoom-control {
            background: white;
            padding: 15px;
            border-radius: 10px;
            border: 2px solid #e9ecef;
        }
        
        .zoom-label {
            display: block;
            margin-bottom: 10px;
            font-weight: 600;
            color: #2c3e50;
        }
        
        .zoom-slider {
            width: 100%;
            height: 6px;
            border-radius: 3px;
            background: #ddd;
            outline: none;
        }
        
        .result-section {
            background: white;
            border-radius: 15px;
            padding: 25px;
            border: 2px solid #e9ecef;
        }
        
        .result-box {
            background: #f8f9fa;
            border-radius: 10px;
            padding: 20px;
            margin: 20px 0;
            min-height: 100px;
            border: 2px dashed #dee2e6;
        }
        
        .result-text {
            font-size: 16px;
            line-height: 1.5;
            word-break: break-all;
        }
        
        .empty-result {
            color: #6c757d;
            text-align: center;
            font-style: italic;
        }
        
        .status {
            padding: 15px;
            border-radius: 10px;
            margin: 15px 0;
            text-align: center;
            font-weight: 600;
        }
        
        .status-scanning {
            background: #d4edda;
            color: #155724;
            border: 1px solid #c3e6cb;
        }
        
        .status-stopped {
            background: #f8d7da;
            color: #721c24;
            border: 1px solid #f5c6cb;
        }
        
        .tips {
            background: #e8f4fd;
            border-radius: 10px;
            padding: 20px;
            margin-top: 20px;
            border-left: 4px solid #3498db;
        }
        
        .tips h3 {
            color: #2c3e50;
            margin-bottom: 10px;
        }
        
        .tips ul {
            list-style: none;
            padding-left: 0;
        }
        
        .tips li {
            padding: 8px 0;
            border-bottom: 1px solid #d6eaf8;
        }
        
        .tips li:last-child {
            border-bottom: none;
        }
        
        .qr-frame {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            width: 200px;
            height: 200px;
            border: 3px solid #00ff00;
            border-radius: 10px;
            pointer-events: none;
            animation: framePulse 2s infinite;
        }
        
        @keyframes framePulse {
            0% { border-color: #00ff00; }
            50% { border-color: #00cc00; }
            100% { border-color: #00ff00; }
        }
        
        .qr-detected {
            border-color: #ff0000 !important;
            animation: detectedPulse 0.5s infinite;
        }
        
        @keyframes detectedPulse {
            0% { border-color: #ff0000; }
            50% { border-color: #ff6666; }
            100% { border-color: #ff0000; }
        }
        
        .processing {
            display: inline-block;
            width: 20px;
            height: 20px;
            border: 3px solid #f3f3f3;
            border-top: 3px solid #3498db;
            border-radius: 50%;
            animation: spin 1s linear infinite;
            margin-right: 10px;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>QR Scanner</h1>
            <p>Kamera yoki rasm orqali QR kodlarni skanerlang</p>
        </div>
        
        <div class="content">
            <div class="scanner-section">
                <div class="video-container">
                    <video id="video" playsinline></video>
                    <canvas id="canvas"></canvas>
                    <div class="qr-frame" id="qrFrame"></div>
                </div>
                
                <div class="controls">
                    <button class="btn btn-primary" id="startBtn">Kamerani Yoqish</button>
                    <button class="btn btn-danger" id="stopBtn" disabled>Kamerani O'chirish</button>
                    <div class="file-input-wrapper">
                        <input type="file" id="fileInput" class="file-input" accept="image/*">
                        <label for="fileInput" class="file-label">Rasm Yuklash</label>
                    </div>
                </div>
                
                <div class="zoom-control">
                    <label class="zoom-label">Zoom: <span id="zoomValue">1.00x</span></label>
                    <input type="range" id="zoomSlider" class="zoom-slider" min="1" max="3" step="0.1" value="1" disabled>
                </div>
                
                <div id="status" class="status status-stopped">
                    🔴 Kamera o'chirilgan
                </div>
            </div>
            
            <div class="result-section">
                <h2>Natijalar</h2>
                <div class="result-box">
                    <div id="result" class="empty-result">
                        QR kod skanerlanganida natija shu yerda paydo bo'ladi...
                    </div>
                </div>
                
                <button class="btn btn-success" id="copyBtn" disabled>Natijani Nusxalash</button>
                <button class="btn btn-warning" id="clearBtn">Tozalash</button>
                
                <div class="tips">
                    <h3>📝 Qo'llanma</h3>
                    <ul>
                        <li>✅ "Kamerani Yoqish" tugmasini bosing va ruxsat bering</li>
                        <li>✅ QR kodni kameraga ko'rsating</li>
                        <li>✅ Yoki "Rasm Yuklash" orqali QR kod rasmini yuklang</li>
                        <li>✅ Skanerlangan matnni "Natijani Nusxalash" tugmasi bilan nusxalashingiz mumkin</li>
                    </ul>
                </div>
            </div>
        </div>
    </div>

    <script>
        class QRScanner {
            constructor() {
                this.video = document.getElementById('video');
                this.canvas = document.getElementById('canvas');
                this.ctx = this.canvas.getContext('2d');
                this.startBtn = document.getElementById('startBtn');
                this.stopBtn = document.getElementById('stopBtn');
                this.fileInput = document.getElementById('fileInput');
                this.zoomSlider = document.getElementById('zoomSlider');
                this.zoomValue = document.getElementById('zoomValue');
                this.status = document.getElementById('status');
                this.result = document.getElementById('result');
                this.copyBtn = document.getElementById('copyBtn');
                this.clearBtn = document.getElementById('clearBtn');
                this.qrFrame = document.getElementById('qrFrame');
                
                this.stream = null;
                this.scanning = false;
                this.animationId = null;
                this.lastResult = null;
                
                this.initEventListeners();
            }
            
            initEventListeners() {
                this.startBtn.addEventListener('click', () => this.startCamera());
                this.stopBtn.addEventListener('click', () => this.stopCamera());
                this.fileInput.addEventListener('change', (e) => this.handleFileUpload(e));
                this.zoomSlider.addEventListener('input', (e) => this.handleZoom(e));
                this.copyBtn.addEventListener('click', () => this.copyResult());
                this.clearBtn.addEventListener('click', () => this.clearResult());
            }
            
            async startCamera() {
                try {
                    this.stream = await navigator.mediaDevices.getUserMedia({
                        video: {
                            facingMode: 'environment',
                            width: { ideal: 1280 },
                            height: { ideal: 720 }
                        }
                    });
                    
                    this.video.srcObject = this.stream;
                    await this.video.play();
                    
                    // Canvas o'lchamlarini sozlash
                    this.canvas.width = this.video.videoWidth;
                    this.canvas.height = this.video.videoHeight;
                    
                    this.scanning = true;
                    this.updateUI('started');
                    this.scanFrame();
                    
                } catch (error) {
                    console.error('Kamera ochishda xatolik:', error);
                    this.showError('Kamera ochib boʻlmadi. Iltimos, ruxsatnomalarni tekshiring.');
                }
            }
            
            stopCamera() {
                this.scanning = false;
                if (this.animationId) {
                    cancelAnimationFrame(this.animationId);
                }
                if (this.stream) {
                    this.stream.getTracks().forEach(track => track.stop());
                    this.stream = null;
                }
                this.updateUI('stopped');
                this.qrFrame.classList.remove('qr-detected');
            }
            
            scanFrame() {
                if (!this.scanning) return;
                
                try {
                    // Videoni canvasga chizish
                    this.ctx.drawImage(this.video, 0, 0, this.canvas.width, this.canvas.height);
                    
                    // Tasvir ma'lumotlarini olish
                    const imageData = this.ctx.getImageData(0, 0, this.canvas.width, this.canvas.height);
                    
                    // jsQR orqali QR kodni skanerlash
                    const code = jsQR(imageData.data, imageData.width, imageData.height, {
                        inversionAttempts: "dontInvert",
                    });
                    
                    if (code) {
                        this.handleQRDetected(code);
                    } else {
                        this.qrFrame.classList.remove('qr-detected');
                    }
                    
                } catch (error) {
                    console.error('Skanerlashda xatolik:', error);
                }
                
                this.animationId = requestAnimationFrame(() => this.scanFrame());
            }
            
            handleQRDetected(code) {
                if (code.data !== this.lastResult) {
                    this.lastResult = code.data;
                    this.showResult(code.data);
                    this.qrFrame.classList.add('qr-detected');
                    
                    // QR kod joylashuvini chizish
                    this.drawQRCodeLocation(code.location);
                    
                    // Avtomatik to'xtatish va qayta boshlash
                    setTimeout(() => {
                        this.qrFrame.classList.remove('qr-detected');
                        this.lastResult = null;
                    }, 2000);
                }
            }
            
            drawQRCodeLocation(location) {
                this.ctx.strokeStyle = '#00ff00';
                this.ctx.lineWidth = 4;
                this.ctx.beginPath();
                this.ctx.moveTo(location.topLeftCorner.x, location.topLeftCorner.y);
                this.ctx.lineTo(location.topRightCorner.x, location.topRightCorner.y);
                this.ctx.lineTo(location.bottomRightCorner.x, location.bottomRightCorner.y);
                this.ctx.lineTo(location.bottomLeftCorner.x, location.bottomLeftCorner.y);
                this.ctx.closePath();
                this.ctx.stroke();
            }
            
            async handleFileUpload(event) {
                const file = event.target.files[0];
                if (!file) return;
                
                if (!file.type.startsWith('image/')) {
                    this.showError('Iltimos, faqat rasm fayllarini yuklang.');
                    return;
                }
                
                try {
                    this.status.innerHTML = '<div class="processing"></div> Rasm qayta ishlanyapti...';
                    
                    const img = new Image();
                    img.onload = () => {
                        this.canvas.width = img.width;
                        this.canvas.height = img.height;
                        this.ctx.drawImage(img, 0, 0);
                        
                        // Rasm uchun QR skanerlash
                        const imageData = this.ctx.getImageData(0, 0, this.canvas.width, this.canvas.height);
                        const code = jsQR(imageData.data, imageData.width, imageData.height, {
                            inversionAttempts: "both",
                        });
                        
                        if (code) {
                            this.showResult(code.data);
                            this.drawQRCodeLocation(code.location);
                            this.status.textContent = '✅ QR kod muvaffaqiyatli topildi!';
                        } else {
                            this.showError('QR kod topilmadi. Boshqa rasm yoki yaxshiroq sifattagi QR kodni sinab ko\'ring.');
                        }
                    };
                    
                    img.onerror = () => {
                        this.showError('Rasm yuklashda xatolik yuz berdi.');
                    };
                    
                    img.src = URL.createObjectURL(file);
                    
                } catch (error) {
                    this.showError('Rasm yuklashda xatolik: ' + error.message);
                }
            }
            
            handleZoom(event) {
                const zoom = event.target.value;
                this.zoomValue.textContent = `${zoom}x`;
                
                if (this.video) {
                    this.video.style.transform = `scale(${zoom})`;
                }
            }
            
            showResult(text) {
                this.result.textContent = text;
                this.result.className = 'result-text';
                this.copyBtn.disabled = false;
                
                // Natija turiga qarab formatlash
                this.formatResult(text);
            }
            
            formatResult(text) {
                if (text.startsWith('http://') || text.startsWith('https://')) {
                    this.result.innerHTML = `<a href="${text}" target="_blank" style="color: #3498db; text-decoration: none;">${text}</a>`;
                } else if (text.startsWith('WIFI:')) {
                    this.result.innerHTML = `<strong>WiFi Ma'lumotlari:</strong><br>${text}`;
                } else if (text.startsWith('MATN:')) {
                    this.result.innerHTML = `<strong>Matn:</strong><br>${text.substring(5)}`;
                } else if (text.startsWith('TEL:')) {
                    this.result.innerHTML = `<strong>Telefon:</strong><br><a href="tel:${text.substring(4)}" style="color: #3498db; text-decoration: none;">${text.substring(4)}</a>`;
                } else if (text.startsWith('EMAIL:')) {
                    this.result.innerHTML = `<strong>Email:</strong><br><a href="mailto:${text.substring(6)}" style="color: #3498db; text-decoration: none;">${text.substring(6)}</a>`;
                }
            }
            
            clearResult() {
                this.result.textContent = 'QR kod skanerlanganida natija shu yerda paydo bo\'ladi...';
                this.result.className = 'empty-result';
                this.copyBtn.disabled = true;
                this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
            }
            
            copyResult() {
                const text = this.result.textContent;
                navigator.clipboard.writeText(text).then(() => {
                    const originalText = this.copyBtn.textContent;
                    this.copyBtn.textContent = '✅ Nusxalandi!';
                    setTimeout(() => {
                        this.copyBtn.textContent = originalText;
                    }, 2000);
                }).catch(err => {
                    this.showError('Nusxalash muvaffaqiyatsiz: ' + err);
                });
            }
            
            showError(message) {
                this.status.innerHTML = `❌ ${message}`;
                this.status.style.background = '#f8d7da';
                this.status.style.color = '#721c24';
                this.status.style.border = '1px solid #f5c6cb';
            }
            
            updateUI(state) {
                if (state === 'started') {
                    this.startBtn.disabled = true;
                    this.stopBtn.disabled = false;
                    this.zoomSlider.disabled = false;
                    this.status.innerHTML = '🟢 Kamera ishlayapti - QR kodni ko\'rsating';
                    this.status.className = 'status status-scanning';
                } else {
                    this.startBtn.disabled = false;
                    this.stopBtn.disabled = true;
                    this.zoomSlider.disabled = true;
                    this.status.innerHTML = '🔴 Kamera o\'chirilgan';
                    this.status.className = 'status status-stopped';
                    this.zoomValue.textContent = '1.00x';
                    this.zoomSlider.value = 1;
                    if (this.video) {
                        this.video.style.transform = 'scale(1)';
                    }
                }
            }
        }
        
        // Ilovani ishga tushirish
        document.addEventListener('DOMContentLoaded', () => {
            new QRScanner();
        });
    </script>
</body>
</html>
