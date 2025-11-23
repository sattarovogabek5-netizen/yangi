<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>QR Scanner Mini App</title>
    <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://unpkg.com/jsqr@1.4.0/dist/jsQR.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
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
    <div id="root"></div>

    <script type="text/babel">
        const { useRef, useEffect, useState } = React;

        function QRScannerMiniApp() {
            const videoRef = useRef(null);
            const canvasRef = useRef(null);
            const [stream, setStream] = useState(null);
            const [scanning, setScanning] = useState(false);
            const [decoded, setDecoded] = useState(null);
            const [zoom, setZoom] = useState(1);
            const [supportsZoom, setSupportsZoom] = useState(false);
            const [supportsTorch, setSupportsTorch] = useState(false);
            const trackRef = useRef(null);
            const [torchOn, setTorchOn] = useState(false);
            const [processing, setProcessing] = useState(false);
            const [error, setError] = useState(null);
            const animationRef = useRef(null);

            // Kamera o'chirish
            useEffect(() => {
                return () => stopCamera();
            }, []);

            // Kamerani ishga tushirish
            async function startCamera() {
                try {
                    setError(null);
                    const constraints = {
                        video: { 
                            facingMode: 'environment', 
                            width: { ideal: 1280 }, 
                            height: { ideal: 720 },
                            zoom: { ideal: zoom }
                        }
                    };
                    
                    const s = await navigator.mediaDevices.getUserMedia(constraints);
                    setStream(s);
                    
                    if (videoRef.current) {
                        videoRef.current.srcObject = s;
                        await videoRef.current.play();
                    }
                    
                    // Kamera qobiliyatlarini tekshirish
                    const tracks = s.getVideoTracks();
                    if (tracks && tracks.length) {
                        trackRef.current = tracks[0];
                        const caps = trackRef.current.getCapabilities ? trackRef.current.getCapabilities() : {};
                        
                        if (caps.zoom) setSupportsZoom(true);
                        if (caps.torch) setSupportsTorch(true);
                    }
                    
                    setScanning(true);
                    requestAnimationFrame(tick);
                } catch (err) {
                    console.error('Camera start error', err);
                    setError('Kamera ochib bo\'lmadi: iltimos brauzerga ruxsat bering yoki HTTPS orqali oching.');
                }
            }

            // Kamerani to'xtatish
            function stopCamera() {
                setScanning(false);
                if (animationRef.current) {
                    cancelAnimationFrame(animationRef.current);
                    animationRef.current = null;
                }
                if (stream) {
                    stream.getTracks().forEach(track => track.stop());
                    setStream(null);
                }
                if (trackRef.current) {
                    trackRef.current = null;
                }
                setTorchOn(false);
            }

            // Zoom sozlash
            async function setCameraZoom(value) {
                const newZoom = Math.max(1, Math.min(5, value));
                setZoom(newZoom);
                
                try {
                    if (trackRef.current && trackRef.current.applyConstraints) {
                        await trackRef.current.applyConstraints({ 
                            advanced: [{ zoom: newZoom }] 
                        });
                    }
                    // Agar kamera zoom qo'llab-quvvatlamasa, CSS transform orqali
                    if (videoRef.current && !supportsZoom) {
                        videoRef.current.style.transform = `scale(${newZoom})`;
                    }
                } catch (e) {
                    console.warn('Zoom sozlashda xatolik:', e);
                    if (videoRef.current) {
                        videoRef.current.style.transform = `scale(${newZoom})`;
                    }
                }
            }

            // Flash (torch) ni boshqarish
            async function toggleTorch() {
                if (!trackRef.current || !supportsTorch) {
                    alert('Ushbu qurilma yoki brauzer flashni qo\'llab-quvvatlamaydi.');
                    return;
                }

                try {
                    const newTorchState = !torchOn;
                    await trackRef.current.applyConstraints({ 
                        advanced: [{ torch: newTorchState }] 
                    });
                    setTorchOn(newTorchState);
                } catch (err) {
                    console.warn('Flashni boshqarishda xatolik:', err);
                    alert('Flashni o\'zgartirib bo\'lmadi.');
                }
            }

            // Tasvir sifatiini yaxshilash
            function enhanceImage(ctx, width, height) {
                const imageData = ctx.getImageData(0, 0, width, height);
                const data = imageData.data;
                
                // Kontrast va yorug'likni sozlash
                const contrast = 1.3;
                const brightness = 15;
                const factor = (259 * (contrast + 255)) / (255 * (259 - contrast));
                
                for (let i = 0; i < data.length; i += 4) {
                    // RGB kanallari uchun
                    for (let c = 0; c < 3; c++) {
                        let value = data[i + c];
                        value = factor * (value - 128) + 128 + brightness;
                        data[i + c] = Math.max(0, Math.min(255, value));
                    }
                    // Alpha kanalini saqlab qolish
                    data[i + 3] = 255;
                }
                
                ctx.putImageData(imageData, 0, 0);
                
                // Sharpening (aniqlikni oshirish)
                const sharpenedData = ctx.getImageData(0, 0, width, height);
                const output = ctx.createImageData(width, height);
                
                for (let y = 1; y < height - 1; y++) {
                    for (let x = 1; x < width - 1; x++) {
                        for (let c = 0; c < 3; c++) {
                            const index = (y * width + x) * 4 + c;
                            const center = sharpenedData.data[index];
                            
                            // Laplacian filtr
                            const laplacian = -sharpenedData.data[index - 4] 
                                            - sharpenedData.data[index + 4] 
                                            - sharpenedData.data[index - width * 4] 
                                            - sharpenedData.data[index + width * 4] 
                                            + 4 * center;
                            
                            let sharpenedValue = center + 0.3 * laplacian;
                            output.data[index] = Math.max(0, Math.min(255, sharpenedValue));
                        }
                        output.data[(y * width + x) * 4 + 3] = 255;
                    }
                }
                
                ctx.putImageData(output, 0, 0);
            }

            // QR kodni skan qilish
            function tick() {
                if (!scanning || !videoRef.current || !canvasRef.current) {
                    return;
                }

                const video = videoRef.current;
                const canvas = canvasRef.current;
                const ctx = canvas.getContext('2d', { willReadFrequently: true });
                
                // Video o'lchamlarini olish
                const videoWidth = video.videoWidth || 640;
                const videoHeight = video.videoHeight || 480;
                
                // Canvas o'lchamlarini yangilash
                if (canvas.width !== videoWidth || canvas.height !== videoHeight) {
                    canvas.width = videoWidth;
                    canvas.height = videoHeight;
                }

                try {
                    // Videoni canvasga chizish
                    ctx.drawImage(video, 0, 0, videoWidth, videoHeight);
                    
                    // Tasvirni yaxshilash
                    enhanceImage(ctx, videoWidth, videoHeight);
                    
                    // QR kodni aniqlash
                    const imageData = ctx.getImageData(0, 0, videoWidth, videoHeight);
                    const code = jsQR(imageData.data, imageData.width, imageData.height, { 
                        inversionAttempts: 'dontInvert',
                        canOverwriteImage: false
                    });
                    
                    if (code) {
                        setDecoded({ text: code.data, location: code.location });
                        drawLocation(ctx, code.location);
                        setScanning(false);
                        setTimeout(() => setScanning(true), 1500);
                    } else {
                        setDecoded(null);
                    }
                } catch (e) {
                    console.warn('Skanerlashda xatolik:', e);
                }
                
                animationRef.current = requestAnimationFrame(tick);
            }

            // QR kod joylashuvini chizish
            function drawLocation(ctx, location) {
                if (!location) return;
                
                ctx.lineWidth = 4;
                ctx.strokeStyle = '#00ff00';
                ctx.beginPath();
                ctx.moveTo(location.topLeftCorner.x, location.topLeftCorner.y);
                ctx.lineTo(location.topRightCorner.x, location.topRightCorner.y);
                ctx.lineTo(location.bottomRightCorner.x, location.bottomRightCorner.y);
                ctx.lineTo(location.bottomLeftCorner.x, location.bottomLeftCorner.y);
                ctx.closePath();
                ctx.stroke();
            }

            // Fayl yuklash orqali QR kodni skanerlash
            async function onFilePicked(e) {
                const file = e.target.files && e.target.files[0];
                if (!file) return;
                
                // Fayl turini tekshirish
                if (!file.type.startsWith('image/')) {
                    alert('Iltimos, faqat rasm fayllarini yuklang.');
                    return;
                }
                
                setProcessing(true);
                setDecoded(null);
                
                const img = new Image();
                img.onload = () => {
                    const canvas = canvasRef.current;
                    const ctx = canvas.getContext('2d', { willReadFrequently: true });
                    
                    // Rasm o'lchamini sozlash
                    const maxWidth = 1280;
                    const scale = Math.min(1, maxWidth / img.width);
                    canvas.width = img.width * scale;
                    canvas.height = img.height * scale;
                    
                    // Rasmni chizish
                    ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
                    
                    // Tasvirni yaxshilash
                    enhanceImage(ctx, canvas.width, canvas.height);
                    
                    // QR kodni aniqlash
                    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
                    const code = jsQR(imageData.data, imageData.width, imageData.height, { 
                        inversionAttempts: 'both' 
                    });
                    
                    if (code) {
                        setDecoded({ text: code.data, location: code.location });
                        drawLocation(ctx, code.location);
                    } else {
                        alert('QR kod topilmadi. Boshqa rasm yoki yaxshiroq sifattagi QR kodni sinab ko\'ring.');
                    }
                    
                    setProcessing(false);
                };
                
                img.onerror = () => {
                    setProcessing(false);
                    alert('Rasm yuklashda xatolik yuz berdi.');
                };
                
                img.src = URL.createObjectURL(file);
            }

            // Matnni nusxalash
            function copyToClipboard() {
                if (decoded && decoded.text) {
                    navigator.clipboard.writeText(decoded.text)
                        .then(() => {
                            alert('Matn nusxalandi!');
                        })
                        .catch(err => {
                            console.error('Nusxalashda xatolik:', err);
                            alert('Nusxalash muvaffaqiyatsiz. Qo\'lda nusxalashni sinab ko\'ring.');
                        });
                }
            }

            return (
                <div className="scanner-container">
                    <h1 className="app-title">QR Scanner Mini App</h1>
                    
                    {error && (
                        <div className="error-message">
                            {error}
                        </div>
                    )}
                    
                    <div className="grid-layout">
                        <div>
                            <div className="video-container">
                                <video 
                                    ref={videoRef} 
                                    className="video-element" 
                                    playsInline 
                                    muted 
                                />
                                <canvas 
                                    ref={canvasRef} 
                                    className="canvas-overlay" 
                                />
                            </div>
                            
                            <div className="controls">
                                {!stream ? (
                                    <button 
                                        onClick={startCamera} 
                                        className="btn btn-primary"
                                    >
                                        Kamerani ishga tushur
                                    </button>
                                ) : (
                                    <button 
                                        onClick={stopCamera} 
                                        className="btn btn-danger"
                                    >
                                        Kamerani to'xtat
                                    </button>
                                )}
                                
                                <label className="btn btn-secondary">
                                    Rasm yuklash
                                    <input 
                                        type="file" 
                                        accept="image/*" 
                                        onChange={onFilePicked} 
                                        className="file-input" 
                                    />
                                </label>
                                
                                <button 
                                    onClick={toggleTorch} 
                                    className="btn btn-warning"
                                    disabled={!supportsTorch}
                                >
                                    {torchOn ? 'Flash O\'chir' : 'Flash Yoq'}
                                </button>
                            </div>
                            
                            <div className="zoom-control">
                                <label className="section-title">
                                    Zoom: <span>{zoom.toFixed(2)}x</span>
                                </label>
                                <input 
                                    type="range" 
                                    min="1" 
                                    max="5" 
                                    step="0.1" 
                                    value={zoom} 
                                    onChange={(e) => setCameraZoom(Number(e.target.value))}
                                    className="zoom-slider"
                                    disabled={!stream}
                                />
                                {!supportsZoom && (
                                    <div className="help-text">
                                        (Agar brauzer yoki kamera ichki zoom ni qo'llab-quvvatlamasa, CSS shim ishlatiladi.)
                                    </div>
                                )}
                            </div>
                        </div>
                        
                        <div className="result-box">
                            <h3 className="section-title">Natija</h3>
                            
                            {processing && (
                                <div className="processing-indicator">
                                    <div className="spinner"></div>
                                    <span>Qayta ishlash... iltimos kuting.</span>
                                </div>
                            )}
                            
                            {decoded ? (
                                <div>
                                    <div className="qr-result">
                                        {decoded.text}
                                    </div>
                                    <button 
                                        onClick={copyToClipboard} 
                                        className="btn btn-success"
                                    >
                                        Matnni nusxalash
                                    </button>
                                </div>
                            ) : (
                                <div className="help-text">
                                    Hozircha QR kod topilmadi. Kamerani QR kodga yaqinlashtiring yoki rasm yuklab ko'ring.
                                </div>
                            )}
                            
                            <div style={{marginTop: '20px'}}>
                                <h4 className="section-title">Yordamchi sozlamalar</h4>
                                <ul style={{listStyleType: 'disc', paddingLeft: '20px', fontSize: '0.875rem', color: '#6b7280'}}>
                                    <li>Yaxshi natija uchun yorug'lik yetarliligiga e'tibor bering.</li>
                                    <li>Agar QR teskari rangli bo'lsa, ilova avtomatik invert sinab ko'radi.</li>
                                    <li>Rasm yuklash orqali eskirgan yoki past sifatli QRlarni ham tekshirish mumkin.</li>
                                </ul>
                            </div>
                        </div>
                    </div>
                    
                    <div className="help-text" style={{marginTop: '20px'}}>
                        Eslatma: Ba'zi telefonlar va brauzerlar kamera flash/torch yoki ichki zoom ni cheklashi mumkin. 
                        Eng yaxshi ishlash uchun Chrome yoki Edge (Android), Safari (iOS) ning so'nggi versiyasidan foydalaning.
                    </div>
                </div>
            );
        }

        // React ilovasini ishga tushirish
        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<QRScannerMiniApp />);
    </script>
</body>
</html>
