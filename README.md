<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>QR Kod O'qish va Tiniqlashtirish Dasturi</title>
    <script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        
        body {
            background: linear-gradient(135deg, #6a11cb 0%, #2575fc 100%);
            color: white;
            min-height: 100vh;
            padding: 20px;
        }
        
        .container {
            max-width: 1200px;
            margin: 0 auto;
        }
        
        header {
            text-align: center;
            margin-bottom: 30px;
            padding: 20px;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 15px;
            backdrop-filter: blur(10px);
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
        }
        
        h1 {
            font-size: 2.5rem;
            margin-bottom: 10px;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
        }
        
        .subtitle {
            font-size: 1.2rem;
            opacity: 0.9;
        }
        
        .app-container {
            display: flex;
            flex-wrap: wrap;
            gap: 20px;
            margin-bottom: 30px;
        }
        
        .upload-section, .camera-section {
            flex: 1;
            min-width: 300px;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 15px;
            padding: 20px;
            backdrop-filter: blur(10px);
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
        }
        
        .section-title {
            font-size: 1.5rem;
            margin-bottom: 15px;
            border-bottom: 2px solid rgba(255, 255, 255, 0.3);
            padding-bottom: 10px;
        }
        
        .upload-area {
            border: 2px dashed rgba(255, 255, 255, 0.5);
            border-radius: 10px;
            padding: 30px;
            text-align: center;
            cursor: pointer;
            transition: all 0.3s ease;
            margin-bottom: 15px;
        }
        
        .upload-area:hover {
            background: rgba(255, 255, 255, 0.1);
            border-color: rgba(255, 255, 255, 0.8);
        }
        
        .upload-icon {
            font-size: 48px;
            margin-bottom: 10px;
        }
        
        .btn {
            background: rgba(255, 255, 255, 0.2);
            border: none;
            color: white;
            padding: 12px 20px;
            border-radius: 8px;
            cursor: pointer;
            font-size: 1rem;
            transition: all 0.3s ease;
            display: inline-block;
            margin: 5px;
            backdrop-filter: blur(5px);
        }
        
        .btn:hover {
            background: rgba(255, 255, 255, 0.3);
            transform: translateY(-2px);
        }
        
        .btn-primary {
            background: rgba(41, 128, 185, 0.7);
        }
        
        .btn-primary:hover {
            background: rgba(41, 128, 185, 0.9);
        }
        
        .preview-container {
            margin-top: 20px;
            text-align: center;
        }
        
        .preview-title {
            margin-bottom: 10px;
            font-size: 1.2rem;
        }
        
        #imagePreview, #cameraPreview {
            max-width: 100%;
            max-height: 300px;
            border-radius: 10px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
        }
        
        .controls {
            margin-top: 15px;
            display: flex;
            justify-content: center;
            flex-wrap: wrap;
            gap: 10px;
        }
        
        .result-section {
            background: rgba(255, 255, 255, 0.1);
            border-radius: 15px;
            padding: 20px;
            margin-top: 20px;
            backdrop-filter: blur(10px);
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
        }
        
        .result-box {
            background: rgba(0, 0, 0, 0.2);
            border-radius: 10px;
            padding: 15px;
            margin-top: 15px;
            min-height: 100px;
            word-break: break-all;
        }
        
        .hidden {
            display: none;
        }
        
        .zoom-controls {
            display: flex;
            align-items: center;
            justify-content: center;
            margin-top: 15px;
            gap: 10px;
        }
        
        .slider-container {
            flex-grow: 1;
            max-width: 300px;
        }
        
        .slider {
            width: 100%;
            height: 8px;
            border-radius: 5px;
            background: rgba(255, 255, 255, 0.2);
            outline: none;
            -webkit-appearance: none;
        }
        
        .slider::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 20px;
            height: 20px;
            border-radius: 50%;
            background: white;
            cursor: pointer;
        }
        
        .enhance-controls {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            justify-content: center;
            margin-top: 15px;
        }
        
        footer {
            text-align: center;
            margin-top: 30px;
            padding: 20px;
            font-size: 0.9rem;
            opacity: 0.8;
        }
        
        @media (max-width: 768px) {
            .app-container {
                flex-direction: column;
            }
            
            h1 {
                font-size: 2rem;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>QR Kod O'qish va Tiniqlashtirish Dasturi</h1>
            <p class="subtitle">QR kodni yuklang yoki kameradan oling, yaqinlashtiring va tiniqlashtiring</p>
        </header>
        
        <div class="app-container">
            <div class="upload-section">
                <h2 class="section-title">Fayl Yuklash</h2>
                <div class="upload-area" id="uploadArea">
                    <div class="upload-icon">📁</div>
                    <p>QR kod rasmini bu yerga tortib keling yoki bosing</p>
                    <input type="file" id="fileInput" accept="image/*" class="hidden">
                </div>
                <button class="btn btn-primary" id="uploadBtn">Fayl Tanlash</button>
                
                <div class="preview-container">
                    <p class="preview-title">Yuklangan Rasm:</p>
                    <img id="imagePreview" class="hidden">
                </div>
                
                <div class="zoom-controls">
                    <button class="btn" id="zoomOutBtn">-</button>
                    <div class="slider-container">
                        <input type="range" min="1" max="300" value="100" class="slider" id="zoomSlider">
                    </div>
                    <button class="btn" id="zoomInBtn">+</button>
                </div>
                
                <div class="enhance-controls">
                    <button class="btn" id="brightnessBtn">Yorqinlik</button>
                    <button class="btn" id="contrastBtn">Kontrast</button>
                    <button class="btn" id="sharpenBtn">Tiniqlashtirish</button>
                    <button class="btn" id="resetBtn">Asl holati</button>
                </div>
            </div>
            
            <div class="camera-section">
                <h2 class="section-title">Kamera Orqali</h2>
                <div class="preview-container">
                    <video id="cameraPreview" class="hidden" autoplay playsinline></video>
                    <canvas id="qrCanvas" class="hidden"></canvas>
                </div>
                
                <div class="controls">
                    <button class="btn btn-primary" id="startCameraBtn">Kamerani Yoqish</button>
                    <button class="btn" id="captureBtn" disabled>Rasmni Olish</button>
                    <button class="btn" id="stopCameraBtn" disabled>Kamerani O'chirish</button>
                </div>
                
                <div class="zoom-controls">
                    <button class="btn" id="camZoomOutBtn">-</button>
                    <div class="slider-container">
                        <input type="range" min="1" max="300" value="100" class="slider" id="camZoomSlider">
                    </div>
                    <button class="btn" id="camZoomInBtn">+</button>
                </div>
            </div>
        </div>
        
        <div class="result-section">
            <h2 class="section-title">QR Kod Ma'lumoti</h2>
            <div class="result-box" id="resultBox">
                QR kod ma'lumoti bu yerda paydo bo'ladi...
            </div>
            <div class="controls">
                <button class="btn btn-primary" id="decodeBtn" disabled>QR Kodni O'qish</button>
                <button class="btn" id="copyBtn" disabled>Nusxa Olish</button>
                <button class="btn" id="clearBtn">Tozalash</button>
            </div>
        </div>
        
        <footer>
            <p>QR Kod O'qish va Tiniqlashtirish Dasturi &copy; 2023</p>
        </footer>
    </div>

    <script>
        // Elementlarni tanlab olish
        const fileInput = document.getElementById('fileInput');
        const uploadArea = document.getElementById('uploadArea');
        const uploadBtn = document.getElementById('uploadBtn');
        const imagePreview = document.getElementById('imagePreview');
        const cameraPreview = document.getElementById('cameraPreview');
        const qrCanvas = document.getElementById('qrCanvas');
        const startCameraBtn = document.getElementById('startCameraBtn');
        const captureBtn = document.getElementById('captureBtn');
        const stopCameraBtn = document.getElementById('stopCameraBtn');
        const decodeBtn = document.getElementById('decodeBtn');
        const copyBtn = document.getElementById('copyBtn');
        const clearBtn = document.getElementById('clearBtn');
        const resultBox = document.getElementById('resultBox');
        const zoomSlider = document.getElementById('zoomSlider');
        const zoomInBtn = document.getElementById('zoomInBtn');
        const zoomOutBtn = document.getElementById('zoomOutBtn');
        const camZoomSlider = document.getElementById('camZoomSlider');
        const camZoomInBtn = document.getElementById('camZoomInBtn');
        const camZoomOutBtn = document.getElementById('camZoomOutBtn');
        const brightnessBtn = document.getElementById('brightnessBtn');
        const contrastBtn = document.getElementById('contrastBtn');
        const sharpenBtn = document.getElementById('sharpenBtn');
        const resetBtn = document.getElementById('resetBtn');
        
        let stream = null;
        let originalImage = null;
        let currentZoom = 100;
        let camCurrentZoom = 100;
        
        // Fayl yuklash funksiyalari
        uploadArea.addEventListener('click', () => {
            fileInput.click();
        });
        
        uploadBtn.addEventListener('click', () => {
            fileInput.click();
        });
        
        fileInput.addEventListener('change', function(e) {
            if (this.files && this.files[0]) {
                const reader = new FileReader();
                
                reader.onload = function(e) {
                    imagePreview.src = e.target.result;
                    imagePreview.classList.remove('hidden');
                    originalImage = new Image();
                    originalImage.src = e.target.result;
                    decodeBtn.disabled = false;
                    resetZoom();
                }
                
                reader.readAsDataURL(this.files[0]);
            }
        });
        
        // Zoom funksiyalari
        function updateZoom() {
            imagePreview.style.transform = `scale(${currentZoom / 100})`;
            zoomSlider.value = currentZoom;
        }
        
        function resetZoom() {
            currentZoom = 100;
            updateZoom();
        }
        
        zoomInBtn.addEventListener('click', () => {
            if (currentZoom < 300) {
                currentZoom += 10;
                updateZoom();
            }
        });
        
        zoomOutBtn.addEventListener('click', () => {
            if (currentZoom > 10) {
                currentZoom -= 10;
                updateZoom();
            }
        });
        
        zoomSlider.addEventListener('input', () => {
            currentZoom = parseInt(zoomSlider.value);
            updateZoom();
        });
        
        // Kamera zoom funksiyalari
        function updateCamZoom() {
            cameraPreview.style.transform = `scale(${camCurrentZoom / 100})`;
            camZoomSlider.value = camCurrentZoom;
        }
        
        camZoomInBtn.addEventListener('click', () => {
            if (camCurrentZoom < 300) {
                camCurrentZoom += 10;
                updateCamZoom();
            }
        });
        
        camZoomOutBtn.addEventListener('click', () => {
            if (camCurrentZoom > 10) {
                camCurrentZoom -= 10;
                updateCamZoom();
            }
        });
        
        camZoomSlider.addEventListener('input', () => {
            camCurrentZoom = parseInt(camZoomSlider.value);
            updateCamZoom();
        });
        
        // Kamera funksiyalari
        startCameraBtn.addEventListener('click', async () => {
            try {
                stream = await navigator.mediaDevices.getUserMedia({ 
                    video: { 
                        facingMode: 'environment',
                        width: { ideal: 1280 },
                        height: { ideal: 720 }
                    } 
                });
                
                cameraPreview.srcObject = stream;
                cameraPreview.classList.remove('hidden');
                startCameraBtn.disabled = true;
                captureBtn.disabled = false;
                stopCameraBtn.disabled = false;
                
                // Kamerani ishga tushirgach, avtomatik ravishda QR kodni skaner qilish
                scanQRFromCamera();
            } catch (err) {
                console.error("Kamerani ochishda xatolik: ", err);
                resultBox.textContent = "Kamerani ochishda xatolik yuz berdi. Iltimos, kameraga ruxsat bering.";
            }
        });
        
        stopCameraBtn.addEventListener('click', () => {
            if (stream) {
                stream.getTracks().forEach(track => track.stop());
                cameraPreview.classList.add('hidden');
                startCameraBtn.disabled = false;
                captureBtn.disabled = true;
                stopCameraBtn.disabled = true;
            }
        });
        
        // Rasmni olish funksiyasi
        captureBtn.addEventListener('click', () => {
            const context = qrCanvas.getContext('2d');
            qrCanvas.width = cameraPreview.videoWidth;
            qrCanvas.height = cameraPreview.videoHeight;
            context.drawImage(cameraPreview, 0, 0, qrCanvas.width, qrCanvas.height);
            
            // Olingan rasmni ko'rsatish
            imagePreview.src = qrCanvas.toDataURL();
            imagePreview.classList.remove('hidden');
            originalImage = new Image();
            originalImage.src = qrCanvas.toDataURL();
            decodeBtn.disabled = false;
            resetZoom();
        });
        
        // QR kodni o'qish funksiyasi
        decodeBtn.addEventListener('click', () => {
            decodeQRCode(imagePreview);
        });
        
        function decodeQRCode(imageElement) {
            const canvas = document.createElement('canvas');
            const context = canvas.getContext('2d');
            
            canvas.width = imageElement.naturalWidth || imageElement.width;
            canvas.height = imageElement.naturalHeight || imageElement.height;
            
            context.drawImage(imageElement, 0, 0, canvas.width, canvas.height);
            
            const imageData = context.getImageData(0, 0, canvas.width, canvas.height);
            const code = jsQR(imageData.data, imageData.width, imageData.height);
            
            if (code) {
                resultBox.textContent = code.data;
                copyBtn.disabled = false;
            } else {
                resultBox.textContent = "QR kod topilmadi. Iltimos, rasmni yaqinlashtiring yoki tiniqlashtiring.";
                copyBtn.disabled = true;
            }
        }
        
        // Kamera orqali QR kodni avtomatik skaner qilish
        function scanQRFromCamera() {
            if (!cameraPreview.srcObject) return;
            
            const canvas = document.createElement('canvas');
            const context = canvas.getContext('2d');
            
            canvas.width = cameraPreview.videoWidth;
            canvas.height = cameraPreview.videoHeight;
            
            context.drawImage(cameraPreview, 0, 0, canvas.width, canvas.height);
            
            const imageData = context.getImageData(0, 0, canvas.width, canvas.height);
            const code = jsQR(imageData.data, imageData.width, imageData.height);
            
            if (code) {
                resultBox.textContent = code.data;
                copyBtn.disabled = false;
                
                // QR kod topilganda, rasmni avtomatik olish va ko'rsatish
                imagePreview.src = canvas.toDataURL();
                imagePreview.classList.remove('hidden');
                originalImage = new Image();
                originalImage.src = canvas.toDataURL();
                decodeBtn.disabled = false;
                resetZoom();
            }
            
            // Agar kamera yoqilgan bo'lsa, skanerlashni davom ettirish
            if (cameraPreview.srcObject) {
                requestAnimationFrame(scanQRFromCamera);
            }
        }
        
        // Rasmni tiniqlashtirish funksiyalari
        brightnessBtn.addEventListener('click', () => {
            applyFilter('brightness', 1.5);
        });
        
        contrastBtn.addEventListener('click', () => {
            applyFilter('contrast', 1.5);
        });
        
        sharpenBtn.addEventListener('click', () => {
            applySharpenFilter();
        });
        
        resetBtn.addEventListener('click', () => {
            if (originalImage) {
                imagePreview.src = originalImage.src;
            }
        });
        
        function applyFilter(filter, value) {
            const canvas = document.createElement('canvas');
            const context = canvas.getContext('2d');
            
            canvas.width = imagePreview.naturalWidth || imagePreview.width;
            canvas.height = imagePreview.naturalHeight || imagePreview.height;
            
            context.filter = `${filter}(${value})`;
            context.drawImage(imagePreview, 0, 0, canvas.width, canvas.height);
            
            imagePreview.src = canvas.toDataURL();
        }
        
        function applySharpenFilter() {
            const canvas = document.createElement('canvas');
            const context = canvas.getContext('2d');
            
            canvas.width = imagePreview.naturalWidth || imagePreview.width;
            canvas.height = imagePreview.naturalHeight || imagePreview.height;
            
            context.drawImage(imagePreview, 0, 0, canvas.width, canvas.height);
            
            const imageData = context.getImageData(0, 0, canvas.width, canvas.height);
            const data = imageData.data;
            const width = imageData.width;
            const height = imageData.height;
            
            // Oddiy sharpen filtr
            for (let y = 1; y < height - 1; y += 1) {
                for (let x = 1; x < width - 1; x += 1) {
                    const idx = (y * width + x) * 4;
                    
                    // Sharpen matritsasi
                    const sharpen = [
                        [0, -1, 0],
                        [-1, 5, -1],
                        [0, -1, 0]
                    ];
                    
                    let r = 0, g = 0, b = 0;
                    
                    for (let ky = -1; ky <= 1; ky++) {
                        for (let kx = -1; kx <= 1; kx++) {
                            const kidx = ((y + ky) * width + (x + kx)) * 4;
                            const weight = sharpen[ky + 1][kx + 1];
                            
                            r += data[kidx] * weight;
                            g += data[kidx + 1] * weight;
                            b += data[kidx + 2] * weight;
                        }
                    }
                    
                    data[idx] = Math.max(0, Math.min(255, r));
                    data[idx + 1] = Math.max(0, Math.min(255, g));
                    data[idx + 2] = Math.max(0, Math.min(255, b));
                }
            }
            
            context.putImageData(imageData, 0, 0);
            imagePreview.src = canvas.toDataURL();
        }
        
        // Qolgan funksiyalar
        copyBtn.addEventListener('click', () => {
            navigator.clipboard.writeText(resultBox.textContent)
                .then(() => {
                    const originalText = copyBtn.textContent;
                    copyBtn.textContent = 'Nusxa olindi!';
                    setTimeout(() => {
                        copyBtn.textContent = originalText;
                    }, 2000);
                })
                .catch(err => {
                    console.error('Nusxa olishda xatolik: ', err);
                });
        });
        
        clearBtn.addEventListener('click', () => {
            resultBox.textContent = "QR kod ma'lumoti bu yerda paydo bo'ladi...";
            imagePreview.classList.add('hidden');
            fileInput.value = '';
            decodeBtn.disabled = true;
            copyBtn.disabled = true;
            
            if (stream) {
                stream.getTracks().forEach(track => track.stop());
                cameraPreview.classList.add('hidden');
                startCameraBtn.disabled = false;
                captureBtn.disabled = true;
                stopCameraBtn.disabled = true;
            }
        });
        
        // Drayv va tush bilan fayl tortib olish
        uploadArea.addEventListener('dragover', (e) => {
            e.preventDefault();
            uploadArea.style.background = 'rgba(255, 255, 255, 0.2)';
        });
        
        uploadArea.addEventListener('dragleave', () => {
            uploadArea.style.background = '';
        });
        
        uploadArea.addEventListener('drop', (e) => {
            e.preventDefault();
            uploadArea.style.background = '';
            
            if (e.dataTransfer.files && e.dataTransfer.files[0]) {
                fileInput.files = e.dataTransfer.files;
                
                const reader = new FileReader();
                reader.onload = function(e) {
                    imagePreview.src = e.target.result;
                    imagePreview.classList.remove('hidden');
                    originalImage = new Image();
                    originalImage.src = e.target.result;
                    decodeBtn.disabled = false;
                    resetZoom();
                }
                
                reader.readAsDataURL(e.dataTransfer.files[0]);
            }
        });
    </script>
</body>
</html>
