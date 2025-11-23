<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>DataMatrix Skaner</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        
        body {
            background: linear-gradient(135deg, #6a11cb 0%, #2575fc 100%);
            color: #fff;
            min-height: 100vh;
            padding: 20px;
        }
        
        .container {
            max-width: 800px;
            margin: 0 auto;
        }
        
        header {
            text-align: center;
            padding: 20px 0;
            margin-bottom: 30px;
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
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(10px);
            border-radius: 20px;
            padding: 25px;
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
            margin-bottom: 30px;
        }
        
        .scanner-section {
            text-align: center;
            margin-bottom: 30px;
        }
        
        .scanner-placeholder {
            width: 100%;
            height: 300px;
            background: rgba(255, 255, 255, 0.1);
            border-radius: 15px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            margin: 20px 0;
            position: relative;
            overflow: hidden;
        }
        
        .scanner-animation {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 4px;
            background: linear-gradient(90deg, transparent, #00ff88, transparent);
            animation: scan 2s linear infinite;
        }
        
        @keyframes scan {
            0% { top: 0; }
            50% { top: 100%; }
            100% { top: 0; }
        }
        
        .scanner-icon {
            font-size: 4rem;
            margin-bottom: 15px;
            color: #00ff88;
        }
        
        .btn {
            background: linear-gradient(90deg, #00ff88, #00ccff);
            border: none;
            padding: 15px 30px;
            border-radius: 50px;
            color: #1a1a2e;
            font-size: 1.1rem;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.3s ease;
            box-shadow: 0 4px 15px rgba(0, 255, 136, 0.3);
            display: inline-flex;
            align-items: center;
            justify-content: center;
            gap: 10px;
        }
        
        .btn:hover {
            transform: translateY(-3px);
            box-shadow: 0 6px 20px rgba(0, 255, 136, 0.5);
        }
        
        .btn:active {
            transform: translateY(1px);
        }
        
        .results-section {
            background: rgba(255, 255, 255, 0.1);
            border-radius: 15px;
            padding: 20px;
            margin-top: 20px;
        }
        
        .results-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
        }
        
        .results-content {
            background: rgba(0, 0, 0, 0.2);
            border-radius: 10px;
            padding: 15px;
            min-height: 100px;
            font-family: monospace;
            white-space: pre-wrap;
            word-break: break-all;
        }
        
        .history-section {
            margin-top: 30px;
        }
        
        .history-list {
            list-style-type: none;
            max-height: 200px;
            overflow-y: auto;
        }
        
        .history-item {
            background: rgba(255, 255, 255, 0.1);
            border-radius: 10px;
            padding: 12px 15px;
            margin-bottom: 10px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        
        .history-data {
            font-family: monospace;
            max-width: 70%;
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
        }
        
        .history-time {
            font-size: 0.8rem;
            opacity: 0.7;
        }
        
        .action-buttons {
            display: flex;
            gap: 10px;
            margin-top: 15px;
        }
        
        .btn-secondary {
            background: rgba(255, 255, 255, 0.2);
            color: white;
        }
        
        footer {
            text-align: center;
            margin-top: 40px;
            padding-top: 20px;
            border-top: 1px solid rgba(255, 255, 255, 0.2);
            font-size: 0.9rem;
            opacity: 0.8;
        }
        
        @media (max-width: 600px) {
            h1 {
                font-size: 2rem;
            }
            
            .scanner-placeholder {
                height: 250px;
            }
            
            .btn {
                padding: 12px 25px;
                font-size: 1rem;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <header>
            <h1>DataMatrix Skaner</h1>
            <p class="subtitle">DataMatrix kodlarini tez va oson skaner qiling</p>
        </header>
        
        <div class="app-container">
            <div class="scanner-section">
                <h2>Kodni Skaner Qiling</h2>
                <div class="scanner-placeholder">
                    <div class="scanner-animation"></div>
                    <div class="scanner-icon">📷</div>
                    <p>Kamerani DataMatrix kodga qarating</p>
                </div>
                
                <button class="btn" id="scanBtn">
                    <span>📷</span> Skanerlashni Boshlang
                </button>
                
                <div class="action-buttons">
                    <button class="btn btn-secondary" id="uploadBtn">
                        <span>📁</span> Fayldan Yuklash
                    </button>
                    <button class="btn btn-secondary" id="manualBtn">
                        <span>⌨️</span> Qo'lda Kiritish
                    </button>
                </div>
            </div>
            
            <div class="results-section">
                <div class="results-header">
                    <h2>Skaner Natijalari</h2>
                    <button class="btn btn-secondary" id="copyBtn">
                        <span>📋</span> Nusxalash
                    </button>
                </div>
                <div class="results-content" id="results">
                    Skaner qilingan ma'lumotlar shu yerda paydo bo'ladi...
                </div>
            </div>
            
            <div class="history-section">
                <h2>Skaner Tarixi</h2>
                <ul class="history-list" id="historyList">
                    <!-- Tarix elementlari JavaScript orqali qo'shiladi -->
                </ul>
            </div>
        </div>
        
        <footer>
            <p>DataMatrix Skaner &copy; 2023 - Barcha huquqlar himoyalangan</p>
        </footer>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            const scanBtn = document.getElementById('scanBtn');
            const uploadBtn = document.getElementById('uploadBtn');
            const manualBtn = document.getElementById('manualBtn');
            const copyBtn = document.getElementById('copyBtn');
            const results = document.getElementById('results');
            const historyList = document.getElementById('historyList');
            
            // Skanerlash tarixi
            let scanHistory = JSON.parse(localStorage.getItem('scanHistory')) || [];
            
            // Tarixni ekranga chiqarish
            function renderHistory() {
                historyList.innerHTML = '';
                scanHistory.forEach((item, index) => {
                    const li = document.createElement('li');
                    li.className = 'history-item';
                    li.innerHTML = `
                        <div class="history-data">${item.data}</div>
                        <div class="history-time">${item.time}</div>
                    `;
                    historyList.appendChild(li);
                });
            }
            
            // Dastlabki tarixni yuklash
            renderHistory();
            
            // Skanerlash boshlash
            scanBtn.addEventListener('click', function() {
                // Bu yerda haqiqiy skanerlash logikasi bo'ladi
                // Hozircha demo natija ko'rsatamiz
                
                const demoData = [
                    "Product: SmartPhone-X2023\nSerial: DMX-887654321\nManufacturer: TechCorp Inc.",
                    "Order: #ORD-789123\nCustomer: John Smith\nTotal: $1,299.99",
                    "Tracking: 1Z987XYZ654321\nStatus: In Transit\nEst. Delivery: 2023-12-15",
                    "Inventory: WAREHOUSE-A\nSKU: DMX-456789\nQty: 150 units",
                    "URL: https://example.com/product/datamatrix-info\nExpires: 2024-06-30"
                ];
                
                const randomData = demoData[Math.floor(Math.random() * demoData.length)];
                
                // Natijalarni ko'rsatish
                results.textContent = randomData;
                
                // Tarixga qo'shish
                const now = new Date();
                const timeString = now.toLocaleTimeString('uz-UZ', { 
                    hour: '2-digit', 
                    minute: '2-digit',
                    second: '2-digit'
                });
                
                const dateString = now.toLocaleDateString('uz-UZ');
                
                scanHistory.unshift({
                    data: randomData.split('\n')[0],
                    time: `${dateString} ${timeString}`,
                    fullData: randomData
                });
                
                // Tarixni saqlash (localStorage da)
                localStorage.setItem('scanHistory', JSON.stringify(scanHistory));
                
                // Tarixni yangilash
                renderHistory();
                
                // Animatsiya effekti
                scanBtn.innerHTML = '<span>✅</span> Kod Skaner Qilindi!';
                scanBtn.style.background = 'linear-gradient(90deg, #00ff88, #00ccff)';
                
                setTimeout(() => {
                    scanBtn.innerHTML = '<span>📷</span> Skanerlashni Boshlang';
                }, 2000);
            });
            
            // Fayl yuklash
            uploadBtn.addEventListener('click', function() {
                alert('Fayl yuklash funksiyasi ishga tushirildi. Haqiqiy ilovada bu faylni yuklab, DataMatrix kodini ajratib olish imkoniyatiga ega bo\'lardi.');
            });
            
            // Qo'lda kiritish
            manualBtn.addEventListener('click', function() {
                const manualData = prompt('DataMatrix kod ma\'lumotlarini kiriting:');
                if (manualData) {
                    results.textContent = manualData;
                    
                    // Tarixga qo'shish
                    const now = new Date();
                    const timeString = now.toLocaleTimeString('uz-UZ', { 
                        hour: '2-digit', 
                        minute: '2-digit',
                        second: '2-digit'
                    });
                    
                    const dateString = now.toLocaleDateString('uz-UZ');
                    
                    scanHistory.unshift({
                        data: manualData.length > 30 ? manualData.substring(0, 30) + '...' : manualData,
                        time: `${dateString} ${timeString}`,
                        fullData: manualData
                    });
                    
                    // Tarixni saqlash
                    localStorage.setItem('scanHistory', JSON.stringify(scanHistory));
                    
                    // Tarixni yangilash
                    renderHistory();
                }
            });
            
            // Nusxalash tugmasi
            copyBtn.addEventListener('click', function() {
                const textToCopy = results.textContent;
                
                // Clipboard API yordamida nusxalash
                navigator.clipboard.writeText(textToCopy).then(() => {
                    // Muvaffaqiyatli nusxalandi
                    const originalText = copyBtn.innerHTML;
                    copyBtn.innerHTML = '<span>✅</span> Nusxalandi!';
                    
                    setTimeout(() => {
                        copyBtn.innerHTML = originalText;
                    }, 2000);
                }).catch(err => {
                    // Xatolik yuz berdi
                    console.error('Nusxalashda xatolik: ', err);
                    alert('Matnni nusxalash muvaffaqiyatsiz tugadi. Qaytadan urinib ko\'ring.');
                });
            });
        });
    </script>
</body>
</html>
