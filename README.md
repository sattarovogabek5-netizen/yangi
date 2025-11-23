<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>QR Skaner</title>
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
    </style>
</head>
<body>
    <div class="container">
        <h1>QR Kod Skaneri</h1>
        
        <div id="video-container">
            <video id="video" playsinline autoplay muted></video>
            <div id="scan-box"></div>
        </div>
        
        <div id="status" class="scanning">Kamera ochilmoqda...</div>
        
        <div>
            <button id="start-btn">Skanerlashni boshlash</button>
            <button id="stop-btn">To'xtatish</button>
        </div>
        
        <div id="result">
            <h3>QR kod ma'lumoti:</h3>
            <div id="result-text"></div>
            <button id="copy-btn">Nusxa olish</button>
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
        const newScanBtn = document.getElementById('new-scan-btn');

        let stream = null;
        let scanning = false;
        let scanInterval = null;

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
                        width: { ideal: 1280 },
                        height: { ideal: 720 }
                    } 
                });
                
                video.srcObject = stream;
                await video.play();
                
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
            }
            status.textContent = 'Skanerlash to\'xtatildi';
            status.className = 'status error';
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
            const imageData = context.getImageData(0, 0, canvas.width, canvas.height);
            
            // QR kodni dekod qilish
            const code = jsQR(imageData.data, imageData.width, imageData.height, {
                inversionAttempts: 'dontInvert',
            });
            
            // Agar QR kod topilsa
            if (code) {
                stopScanner();
                
                // Natijani ko'rsatish
                resultText.textContent = code.data;
                result.style.display = 'block';
                
                status.textContent = 'QR kod topildi!';
                status.className = 'status success';
                
                // Agar URL bo'lsa, avtomatik ochish taklifi
                try {
                    new URL(code.data);
                    if (confirm('Bu URL. Ochishni xohlaysizmi?')) {
                        window.open(code.data, '_blank');
                    }
                } catch (e) {
                    // URL emas
                }
            }
        }

        // Dastur yuklanganda avtomatik boshlash
        window.addEventListener('load', startScanner);
        
        // Sahifa yopilganda kamerani to'xtatish
        window.addEventListener('beforeunload', stopScanner);
    </script>
</body>
</html>
