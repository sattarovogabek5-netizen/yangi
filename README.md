<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>QR Scanner Mini App</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/jsqr@1.4.0/dist/jsQR.js"></script>
    <style>
        .scanner-container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }
        .video-container {
            position: relative;
            background: #000;
            border-radius: 12px;
            overflow: hidden;
            height: 360px;
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
        .controls {
            display: flex;
            gap: 10px;
            margin-top: 15px;
            flex-wrap: wrap;
        }
        .btn {
            padding: 10px 16px;
            border-radius: 8px;
            font-weight: 500;
            cursor: pointer;
            border: none;
            transition: all 0.2s;
        }
        .btn-primary {
            background: #2563eb;
            color: white;
        }
        .btn-primary:hover {
            background: #1d4ed8;
        }
        .btn-danger {
            background: #dc2626;
            color: white;
        }
        .btn-danger:hover {
            background: #b91c1c;
        }
        .btn-warning {
            background: #d97706;
            color: white;
        }
        .btn-warning:hover {
            background: #b45309;
        }
        .btn-secondary {
            background: #e5e7eb;
            color: #374151;
        }
        .btn-secondary:hover {
            background: #d1d5db;
        }
        .btn-success {
            background: #059669;
            color: white;
        }
        .btn-success:hover {
            background: #047857;
        }
        .file-input {
            display: none;
        }
        .result-box {
            background: white;
            border-radius: 12px;
            padding: 20px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
            height: fit-content;
        }
        .qr-result {
            word-break: break-all;
            background: #f3f4f6;
            padding: 12px;
            border-radius: 8px;
            margin: 15px 0;
            border: 1px solid #e5e7eb;
        }
        .zoom-control {
            margin-top: 20px;
        }
        .zoom-slider {
            width: 100%;
            margin: 10px 0;
        }
        .help-text {
            font-size: 0.875rem;
            color: #6b7280;
            margin-top: 5px;
        }
        .error-message {
            background: #fef2f2;
            color: #dc2626;
            padding: 12px;
            border-radius: 8px;
            margin: 15px 0;
            border: 1px solid #fecaca;
        }
        .processing-indicator {
            display: flex;
            align-items: center;
            gap: 10px;
            padding: 12px;
            background: #eff6ff;
            border-radius: 8px;
            margin: 15px 0;
        }
        .spinner {
            width: 20px;
            height: 20px;
            border: 2px solid #d1d5db;
            border-top: 2px solid #3b82f6;
            border-radius: 50%;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        .app-title {
            font-size: 1.5rem;
            font-weight: 600;
            margin-bottom: 1rem;
            color: #1f2937;
        }
        .section-title {
            font-size: 1.125rem;
            font-weight: 600;
            margin-bottom: 0.75rem;
            color: #374151;
        }
        .grid-layout {
            display: grid;
            grid-template-columns: 1fr;
            gap: 20px;
        }
        @media (min-width: 768px) {
            .grid-layout {
                grid-template-columns: 1fr 1fr;
            }
        }
    </style>
</head>
<body class="bg-gray-50 min-h-screen p-4">
    <div id="app" class="scanner-container">
        <h2 class="app-title">QR Scanner Mini App</h2>
        <div class="grid-layout">
            <div>
                <div class="video-container">
                    <video id="video" class="video-element" playsinline muted></video>
                    <canvas id="canvas" class="canvas-overlay"></canvas>
                </div>
                <div class="controls">
                    <button id="startBtn" class="btn btn-primary">Kamerani ishga tushur</button>
                    <button id="stopBtn" class="btn btn-danger" disabled>Kamerani to'xtat</button>

                    <label class="btn btn-secondary">
                        Rasm yuklash
                        <input type="file" id="fileInput" accept="image/*" class="file-input">
                    </label>

                    <button id="torchBtn" class="btn btn-warning">Flash</button>
                </div>

                <div class="zoom-control">
                    <label class="block text-sm">Zoom: <span id="zoomValue" class="font-medium">1.00</span></label>
                    <input type="range" id="zoomSlider" min="1" max="3" step="0.05" value="1" class="zoom-slider">
                    <div id="zoomHelp" class="help-text">(Agar brauzer yoki kamera ichki zoom ni qo'llab-quvvatlamasa, CSS shim ishlatiladi.)</div>
                </div>
            </div>

            <div>
                <div class="result-box">
                    <h3 class="section-title">Natija</h3>
                    <div id="processingIndicator" class="processing-indicator" style="display: none;">
                        <div class="spinner"></div>
                        <span>Qayta ishlash... iltimos kuting.</span>
                    </div>
                    <div id="result">
                        <div class="text-sm text-gray-600 mt-2">Hozircha QR topilmadi — kamerani yaqinlashtirib yoki rasm yuklab ko'ring.</div>
                    </div>

                    <div class="mt-4">
                        <h4 class="font-medium">Yordamchi sozlamalar</h4>
                        <ul class="list-disc pl-5 text-sm text-gray-700">
                            <li>Yaxshi natija uchun yorug'lik yetarliligiga e'tibor bering.</li>
                            <li>Agar QR teskari rangli bo'lsa, ilova avtomatik invert sinab ko'radi.</li>
                            <li>Rasm yuklash orqali eskirgan yoki past sifatli QRlarni ham tekshirish mumkin.</li>
                        </ul>
                    </div>
                </div>
            </div>
        </div>

        <div class="mt-4 text-xs text-gray-500">Eslatma: Ba'zi telefonlar va brauzerlar kamera flash/torch yoki ichki zoom ni cheklashi mumkin. Eng yaxshi ishlash uchun Chrome yoki Edge (Android), Safari (iOS) ning so'nggi versiyasidan foydalaning.</div>
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

        // Elementlar
        const startBtn = document.getElementById('startBtn');
        const stopBtn = document.getElementById('stopBtn');
        const torchBtn = document.getElementById('torchBtn');
        const fileInput = document.getElementById('fileInput');
        const zoomSlider = document.getElementById('zoomSlider');
        const zoomValue = document.getElementById('zoomValue');
        const zoomHelp = document.getElementById('zoomHelp');
        const processingIndicator = document.getElementById('processingIndicator');
        const resultDiv = document.getElementById('result');

        // Event listenerlar
        startBtn.addEventListener('click', startCamera);
        stopBtn.addEventListener('click', stopCamera);
        torchBtn.addEventListener('click', toggleTorch);
        fileInput.addEventListener('change', onFilePicked);
        zoomSlider.addEventListener('input', handleZoom);

        // Kamerani ishga tushirish
        async function startCamera() {
            try {
                const constraints = {
                    video: { 
                        facingMode: 'environment', 
                        width: { ideal: 1280 }, 
                        height: { ideal: 720 } 
                    }
                };
                stream = await navigator.mediaDevices.getUserMedia(constraints);
                
                if (videoRef) {
                    videoRef.srcObject = stream;
                    videoRef.play().catch(() => {});
                }
                
                const tracks = stream.getVideoTracks();
                if (tracks && tracks.length) {
                    trackRef = tracks[0];
                    const caps = trackRef.getCapabilities ? trackRef.getCapabilities() : {};
                    if (caps.zoom) {
                        supportsZoom = true;
                        zoomHelp.style.display = 'none';
                    }
                }
                
                scanning = true;
                startBtn.disabled = true;
                stopBtn.disabled = false;
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
            startBtn.disabled = false;
            stopBtn.disabled = true;
            torchOn = false;
            torchBtn.textContent = 'Flash';
        }

        // Zoom sozlash
        async function handleZoom(event) {
            const z = Number(event.target.value);
            zoom = z;
            zoomValue.textContent = z.toFixed(2);
            
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
                
                // Soddalashtirilgan torch boshqaruvi
                if (trackRef.getCapabilities && trackRef.getCapabilities().torch) {
                    torchOn = !torchOn;
                    await trackRef.applyConstraints({ 
                        advanced: [{ torch: torchOn }] 
                    });
                    torchBtn.textContent = torchOn ? 'Flash O\'chir' : 'Flash';
                } else {
                    alert('Torch (flash) qo\'llab-quvvatlanmaydi.');
                }
            } catch (err) {
                console.warn('Torch error', err);
                alert('Flashni yoqib bo\'lmadi yoki brauzer qo\'llab-quvvatlamaydi.');
            }
        }

        // Tasvir sifatiini yaxshilash
        function enhanceImage(ctx, w, h) {
            const imgData = ctx.getImageData(0, 0, w, h);
            const data = imgData.data;
            const contrast = 1.2;
            const brightness = 10;
            const factor = (259 * (contrast + 255)) / (255 * (259 - contrast));
            
            for (let i = 0; i < data.length; i += 4) {
                for (let c = 0; c < 3; c++) {
                    let v = data[i + c];
                    v = factor * (v - 128) + 128 + brightness;
                    data[i + c] = Math.max(0, Math.min(255, v));
                }
            }
            ctx.putImageData(imgData, 0, 0);
            
            const copy = ctx.getImageData(0, 0, w, h);
            const out = ctx.createImageData(w, h);
            
            for (let y = 1; y < h - 1; y++) {
                for (let x = 1; x < w - 1; x++) {
                    for (let c = 0; c < 3; c++) {
                        const i = (y * w + x) * 4 + c;
                        const center = copy.data[i];
                        const lap = -copy.data[i - 4] - copy.data[i + 4] - copy.data[i - w * 4] - copy.data[i + w * 4] + 4 * center;
                        let val = center + 0.3 * lap;
                        out.data[i] = Math.max(0, Math.min(255, val));
                    }
                    out.data[(y * w + x) * 4 + 3] = 255;
                }
            }
            ctx.putImageData(out, 0, 0);
        }

        // QR kodni skanerlash
        function tick() {
            if (!scanning) return;
            const video = videoRef;
            const canvas = canvasRef;
            
            if (!video || !canvas) {
                animationRef = requestAnimationFrame(tick);
                return;
            }
            
            const ctx = canvas.getContext('2d');
            const w = canvas.width = video.videoWidth || 640;
            const h = canvas.height = video.videoHeight || 480;
            
            try {
                ctx.drawImage(video, 0, 0, w, h);
                enhanceImage(ctx, w, h);
                const imageData = ctx.getImageData(0, 0, w, h);
                const code = jsQR(imageData.data, imageData.width, imageData.height, { 
                    inversionAttempts: 'both' 
                });
                
                if (code) {
                    setDecoded({ text: code.data, location: code.location });
                    drawLocation(ctx, code.location);
                    scanning = false;
                    setTimeout(() => {
                        scanning = true;
                        requestAnimationFrame(tick);
                    }, 1500);
                } else {
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
            ctx.strokeStyle = 'lime';
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
                const canvas = canvasRef;
                const ctx = canvas.getContext('2d');
                const maxW = 1280;
                const scale = Math.min(1, maxW / img.width);
                canvas.width = img.width * scale;
                canvas.height = img.height * scale;
                
                ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
                enhanceImage(ctx, canvas.width, canvas.height);
                
                const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
                const code = jsQR(imageData.data, imageData.width, imageData.height, { 
                    inversionAttempts: 'both' 
                });
                
                if (code) {
                    setDecoded({ text: code.data, location: code.location });
                    drawLocation(ctx, code.location);
                } else {
                    alert('QR topilmadi — tasvirni aniqlashtirish yoki boshqa rasm sinab ko\'ring.');
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
                resultDiv.innerHTML = `
                    <div class="mt-2">
                        <div class="qr-result">${result.text}</div>
                        <div class="mt-2">
                            <button id="copyBtn" class="btn btn-success">Nusxalash</button>
                        </div>
                    </div>
                `;
                
                // Nusxalash tugmasi uchun event listener qo'shish
                document.getElementById('copyBtn').addEventListener('click', () => {
                    navigator.clipboard.writeText(result.text);
                    alert('Matn nusxalandi');
                });
            } else {
                resultDiv.innerHTML = '<div class="text-sm text-gray-600 mt-2">Hozircha QR topilmadi — kamerani yaqinlashtirib yoki rasm yuklab ko\'ring.</div>';
            }
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

        // Dastur yuklanganda zoom yordami yashirish
        zoomHelp.style.display = 'none';
    </script>
</body>
</html>
