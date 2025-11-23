<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Universal Scanner - Telegram Mini App</title>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <script src="https://unpkg.com/quagga/dist/quagga.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/tesseract.js@4/dist/tesseract.min.js"></script>
    <script src="https://unpkg.com/react@17/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@17/umd/react-dom.development.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            background: var(--tg-theme-bg-color, #ffffff);
            color: var(--tg-theme-text-color, #000000);
            line-height: 1.5;
        }

        /* Telegram theme variables */
        :root {
            --tg-theme-bg-color: #ffffff;
            --tg-theme-text-color: #000000;
            --tg-theme-hint-color: #999999;
            --tg-theme-link-color: #2481cc;
            --tg-theme-button-color: #2481cc;
            --tg-theme-button-text-color: #ffffff;
            --tg-theme-secondary-bg-color: #f1f1f1;
        }

        .container {
            max-width: 100%;
            margin: 0 auto;
            padding: 0 16px;
        }

        /* Header styles */
        .header {
            background: var(--tg-theme-bg-color, #ffffff);
            border-bottom: 1px solid var(--tg-theme-secondary-bg-color, #f1f1f1);
            padding: 12px 0;
            position: sticky;
            top: 0;
            z-index: 100;
        }

        .header-content {
            display: flex;
            align-items: center;
            justify-content: space-between;
        }

        .logo {
            display: flex;
            align-items: center;
            gap: 12px;
        }

        .logo-icon {
            width: 40px;
            height: 40px;
            background: linear-gradient(135deg, #2481cc, #6c5ce7);
            border-radius: 10px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .logo-text h1 {
            font-size: 18px;
            font-weight: 700;
            color: var(--tg-theme-text-color, #000000);
        }

        .logo-text p {
            font-size: 12px;
            color: var(--tg-theme-hint-color, #999999);
        }

        .user-info {
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .user-details {
            text-align: right;
        }

        .user-name {
            font-size: 14px;
            font-weight: 600;
            color: var(--tg-theme-text-color, #000000);
        }

        .user-username {
            font-size: 12px;
            color: var(--tg-theme-hint-color, #999999);
        }

        .user-avatar {
            width: 32px;
            height: 32px;
            border-radius: 50%;
            object-fit: cover;
        }

        /* Scanner styles */
        .scanner-section {
            padding: 20px 0;
        }

        .scanner-container {
            position: relative;
            background: #000000;
            border-radius: 16px;
            overflow: hidden;
            margin-bottom: 16px;
            height: 300px;
        }

        #scanner-canvas {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }

        .scanner-overlay {
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .scanner-frame {
            position: relative;
            width: 250px;
            height: 250px;
            border: 2px solid #10b981;
            border-radius: 12px;
            transition: all 0.3s ease;
        }

        .scanner-frame.zoomed {
            width: 200px;
            height: 200px;
            border-color: #f59e0b;
        }

        .scanner-corner {
            position: absolute;
            width: 20px;
            height: 20px;
            border-color: #10b981;
            transition: all 0.3s ease;
        }

        .scanner-frame.zoomed .scanner-corner {
            border-color: #f59e0b;
        }

        .corner-tl {
            top: -2px;
            left: -2px;
            border-top: 3px solid;
            border-left: 3px solid;
            border-radius: 8px 0 0 0;
        }

        .corner-tr {
            top: -2px;
            right: -2px;
            border-top: 3px solid;
            border-right: 3px solid;
            border-radius: 0 8px 0 0;
        }

        .corner-bl {
            bottom: -2px;
            left: -2px;
            border-bottom: 3px solid;
            border-left: 3px solid;
            border-radius: 0 0 0 8px;
        }

        .corner-br {
            bottom: -2px;
            right: -2px;
            border-bottom: 3px solid;
            border-right: 3px solid;
            border-radius: 0 0 8px 0;
        }

        .laser-line {
            position: absolute;
            left: 0;
            right: 0;
            top: 50%;
            height: 2px;
            background: #ef4444;
            animation: laserMove 2s ease-in-out infinite;
        }

        @keyframes laserMove {
            0% { transform: translateY(0%); opacity: 0.8; }
            50% { transform: translateY(100%); opacity: 1; }
            100% { transform: translateY(0%); opacity: 0.8; }
        }

        .scanner-status {
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: rgba(0, 0, 0, 0.7);
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
            text-align: center;
        }

        .status-icon {
            width: 48px;
            height: 48px;
            margin-bottom: 12px;
            opacity: 0.7;
        }

        /* Controls */
        .controls {
            display: flex;
            flex-wrap: wrap;
            gap: 8px;
            justify-content: center;
            margin-bottom: 20px;
        }

        .btn {
            padding: 12px 20px;
            border: none;
            border-radius: 12px;
            font-size: 14px;
            font-weight: 600;
            cursor: pointer;
            display: flex;
            align-items: center;
            gap: 8px;
            transition: all 0.2s ease;
        }

        .btn:active {
            transform: scale(0.98);
        }

        .btn-primary {
            background: var(--tg-theme-button-color, #2481cc);
            color: var(--tg-theme-button-text-color, #ffffff);
        }

        .btn-secondary {
            background: var(--tg-theme-secondary-bg-color, #f1f1f1);
            color: var(--tg-theme-text-color, #000000);
        }

        .btn-danger {
            background: #ef4444;
            color: white;
        }

        .btn-warning {
            background: #f59e0b;
            color: black;
        }

        .btn-success {
            background: #10b981;
            color: white;
        }

        .btn-info {
            background: #3b82f6;
            color: white;
        }

        .btn-icon {
            width: 18px;
            height: 18px;
        }

        /* Settings Panel */
        .settings-panel {
            background: var(--tg-theme-secondary-bg-color, #f1f1f1);
            border-radius: 16px;
            padding: 16px;
            margin-bottom: 20px;
        }

        .settings-header {
            display: flex;
            align-items: center;
            justify-content: space-between;
            margin-bottom: 12px;
        }

        .settings-title {
            display: flex;
            align-items: center;
            gap: 8px;
            font-weight: 600;
            color: #3b82f6;
        }

        .settings-grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 12px;
        }

        .setting-item {
            background: white;
            border-radius: 8px;
            padding: 12px;
            display: flex;
            flex-direction: column;
            gap: 8px;
        }

        .setting-label {
            font-size: 12px;
            color: var(--tg-theme-hint-color, #999999);
            display: flex;
            align-items: center;
            gap: 4px;
        }

        .setting-value {
            font-size: 14px;
            font-weight: 600;
            color: var(--tg-theme-text-color, #000000);
        }

        .slider {
            width: 100%;
            height: 4px;
            background: #e5e7eb;
            border-radius: 2px;
            outline: none;
            -webkit-appearance: none;
        }

        .slider::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 16px;
            height: 16px;
            background: #3b82f6;
            border-radius: 50%;
            cursor: pointer;
        }

        /* Results */
        .result-section {
            background: var(--tg-theme-secondary-bg-color, #f1f1f1);
            border-radius: 16px;
            padding: 16px;
            margin-bottom: 20px;
        }

        .result-header {
            display: flex;
            align-items: center;
            justify-content: space-between;
            margin-bottom: 12px;
        }

        .result-title {
            display: flex;
            align-items: center;
            gap: 8px;
            font-weight: 600;
            color: #059669;
        }

        .result-format {
            background: #10b981;
            color: white;
            padding: 4px 8px;
            border-radius: 6px;
            font-size: 12px;
            font-weight: 600;
        }

        .result-content {
            background: white;
            border-radius: 8px;
            padding: 12px;
            margin-bottom: 12px;
        }

        .result-text {
            font-family: 'Courier New', monospace;
            font-size: 14px;
            word-break: break-all;
            white-space: pre-wrap;
        }

        .result-actions {
            display: flex;
            gap: 8px;
        }

        .result-actions .btn {
            flex: 1;
            justify-content: center;
            padding: 10px 16px;
            font-size: 13px;
        }

        /* History */
        .history-section {
            padding: 20px 0;
        }

        .history-header {
            display: flex;
            align-items: center;
            justify-content: space-between;
            margin-bottom: 16px;
        }

        .history-title {
            font-size: 18px;
            font-weight: 700;
            color: var(--tg-theme-text-color, #000000);
        }

        .clear-btn {
            background: none;
            border: none;
            color: #ef4444;
            font-size: 14px;
            font-weight: 600;
            cursor: pointer;
            display: flex;
            align-items: center;
            gap: 4px;
        }

        .history-list {
            display: flex;
            flex-direction: column;
            gap: 12px;
        }

        .history-item {
            background: white;
            border: 1px solid var(--tg-theme-secondary-bg-color, #f1f1f1);
            border-radius: 12px;
            padding: 16px;
            cursor: pointer;
            transition: all 0.2s ease;
        }

        .history-item:active {
            transform: scale(0.98);
        }

        .history-item-header {
            display: flex;
            align-items: flex-start;
            justify-content: space-between;
            margin-bottom: 8px;
        }

        .history-item-type {
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .type-icon {
            width: 32px;
            height: 32px;
            border-radius: 8px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .type-icon.qr {
            background: #dbeafe;
            color: #1d4ed8;
        }

        .type-icon.barcode {
            background: #dcfce7;
            color: #166534;
        }

        .type-icon.text {
            background: #fef3c7;
            color: #92400e;
        }

        .history-item-info h4 {
            font-size: 14px;
            font-weight: 600;
            color: var(--tg-theme-text-color, #000000);
        }

        .history-item-info p {
            font-size: 12px;
            color: var(--tg-theme-hint-color, #999999);
        }

        .history-item-content {
            background: var(--tg-theme-secondary-bg-color, #f1f1f1);
            border-radius: 6px;
            padding: 8px;
            margin-top: 8px;
        }

        .history-item-text {
            font-size: 13px;
            color: var(--tg-theme-text-color, #000000);
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
        }

        .empty-state {
            text-align: center;
            padding: 40px 20px;
            color: var(--tg-theme-hint-color, #999999);
        }

        .empty-icon {
            width: 64px;
            height: 64px;
            margin: 0 auto 16px;
            opacity: 0.5;
        }

        /* Navigation */
        .navigation {
            position: fixed;
            bottom: 0;
            left: 0;
            right: 0;
            background: white;
            border-top: 1px solid var(--tg-theme-secondary-bg-color, #f1f1f1);
            display: flex;
        }

        .nav-item {
            flex: 1;
            padding: 12px;
            background: none;
            border: none;
            cursor: pointer;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 4px;
            transition: all 0.2s ease;
        }

        .nav-item.active {
            background: var(--tg-theme-button-color, #2481cc);
            color: var(--tg-theme-button-text-color, #ffffff);
        }

        .nav-icon {
            width: 20px;
            height: 20px;
        }

        .nav-label {
            font-size: 11px;
            font-weight: 500;
        }

        /* Instructions */
        .instructions {
            background: #dbeafe;
            border: 1px solid #93c5fd;
            border-radius: 12px;
            padding: 16px;
            margin-top: 20px;
        }

        .instructions-header {
            display: flex;
            align-items: center;
            gap: 8px;
            margin-bottom: 8px;
            color: #1d4ed8;
            font-weight: 600;
        }

        .instructions-list {
            list-style: none;
            color: #1e40af;
            font-size: 14px;
        }

        .instructions-list li {
            margin-bottom: 4px;
            padding-left: 8px;
        }

        /* Error states */
        .error-message {
            background: #fef2f2;
            border: 1px solid #fecaca;
            border-radius: 8px;
            padding: 12px;
            margin-bottom: 16px;
            color: #dc2626;
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .error-icon {
            width: 16px;
            height: 16px;
            flex-shrink: 0;
        }

        /* Progress bar */
        .progress-bar {
            width: 100%;
            height: 4px;
            background: #e5e7eb;
            border-radius: 2px;
            overflow: hidden;
            margin: 8px 0;
        }

        .progress-fill {
            height: 100%;
            background: #10b981;
            transition: width 0.3s ease;
        }

        /* Utility classes */
        .hidden {
            display: none !important;
        }

        .text-center {
            text-align: center;
        }

        .mb-4 {
            margin-bottom: 16px;
        }

        .mt-4 {
            margin-top: 16px;
        }
    </style>
</head>
<body>
    <div id="root"></div>

    <script>
        // Telegram WebApp sozlamalari
        let tg = window.Telegram?.WebApp;
        if (tg) {
            tg.ready();
            tg.expand();
            
            // Apply theme
            document.documentElement.style.setProperty('--tg-theme-bg-color', tg.themeParams.bg_color || '#ffffff');
            document.documentElement.style.setProperty('--tg-theme-text-color', tg.themeParams.text_color || '#000000');
            document.documentElement.style.setProperty('--tg-theme-hint-color', tg.themeParams.hint_color || '#999999');
            document.documentElement.style.setProperty('--tg-theme-link-color', tg.themeParams.link_color || '#2481cc');
            document.documentElement.style.setProperty('--tg-theme-button-color', tg.themeParams.button_color || '#2481cc');
            document.documentElement.style.setProperty('--tg-theme-button-text-color', tg.themeParams.button_text_color || '#ffffff');
            document.documentElement.style.setProperty('--tg-theme-secondary-bg-color', tg.themeParams.secondary_bg_color || '#f1f1f1');
        }

        // Asosiy React komponenti
        const { useState, useEffect, useRef } = React;

        function App() {
            const [activeTab, setActiveTab] = useState('scan');
            const [scanHistory, setScanHistory] = useState([]);
            const [lastResult, setLastResult] = useState(null);
            const [webApp, setWebApp] = useState(tg);
            const [user, setUser] = useState(tg?.initDataUnsafe?.user);
            const [settings, setSettings] = useState({
                zoom: 1.0,
                brightness: 1.0,
                contrast: 1.0,
                autoZoom: true,
                imageEnhancement: true
            });

            useEffect(() => {
                // LocalStorage dan tarixni yuklash
                const savedHistory = localStorage.getItem('scanHistory');
                if (savedHistory) {
                    setScanHistory(JSON.parse(savedHistory));
                }

                // Sozlamalarni yuklash
                const savedSettings = localStorage.getItem('scannerSettings');
                if (savedSettings) {
                    setSettings(JSON.parse(savedSettings));
                }
            }, []);

            const updateSettings = (newSettings) => {
                setSettings(newSettings);
                localStorage.setItem('scannerSettings', JSON.stringify(newSettings));
            };

            const addToHistory = (result) => {
                const newItem = {
                    id: Date.now(),
                    text: result.text,
                    format: result.format,
                    timestamp: new Date().toLocaleString('uz-UZ'),
                    type: result.format.includes('QR') ? 'qr' : 
                          result.format.includes('CODE') ? 'barcode' : 'text'
                };
                
                const newHistory = [newItem, ...scanHistory.slice(0, 49)];
                setScanHistory(newHistory);
                localStorage.setItem('scanHistory', JSON.stringify(newHistory));
            };

            const clearHistory = () => {
                setScanHistory([]);
                localStorage.removeItem('scanHistory');
            };

            const copyToClipboard = (text) => {
                navigator.clipboard?.writeText(text).then(() => {
                    if (webApp) {
                        webApp.showPopup({
                            title: 'Muvaffaqiyatli',
                            message: 'Matn clipboardga nusxalandi'
                        });
                    } else {
                        alert('Matn nusxalandi!');
                    }
                });
            };

            const sendToBot = (result) => {
                if (webApp) {
                    webApp.sendData(JSON.stringify({
                        type: 'scan_result',
                        data: result,
                        timestamp: new Date().toISOString()
                    }));
                } else {
                    alert('Telegram WebApp mavjud emas');
                }
            };

            return React.createElement('div', { className: 'app' }, [
                React.createElement(Header, { key: 'header', user }),
                
                React.createElement('main', { 
                    key: 'main',
                    style: { paddingBottom: '80px' } 
                }, 
                    activeTab === 'scan' ? 
                        React.createElement(ScannerSection, {
                            key: 'scanner',
                            onScanResult: (result) => {
                                setLastResult(result);
                                addToHistory(result);
                            },
                            lastResult,
                            onCopy: copyToClipboard,
                            onSendToBot: sendToBot,
                            webApp,
                            settings,
                            onSettingsChange: updateSettings
                        }) : 
                        React.createElement(HistorySection, {
                            key: 'history',
                            items: scanHistory,
                            onClear: clearHistory,
                            onItemClick: (item) => copyToClipboard(item.text)
                        })
                ),

                React.createElement(Navigation, {
                    key: 'nav',
                    activeTab,
                    onTabChange: setActiveTab
                })
            ]);
        }

        function Header({ user }) {
            return React.createElement('header', { className: 'header' },
                React.createElement('div', { className: 'container' },
                    React.createElement('div', { className: 'header-content' },
                        React.createElement('div', { className: 'logo' },
                            React.createElement('div', { className: 'logo-icon' },
                                React.createElement('svg', { 
                                    width: "20", 
                                    height: "20", 
                                    viewBox: "0 0 24 24", 
                                    fill: "none", 
                                    stroke: "currentColor", 
                                    strokeWidth: "2" 
                                },
                                    React.createElement('path', { 
                                        d: "M12 4v1m6 11h2m-6 0h-2v4m0-11v3m0 0h.01M12 12h4.01M16 20h4M4 12h4m12 0h.01M5 8h2a1 1 0 001-1V5a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1zm12 0h2a1 1 0 001-1V5a1 1 0 00-1-1h-2a1 1 0 00-1 1v2a1 1 0 001 1zM5 20h2a1 1 0 001-1v-2a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1z"
                                    })
                                )
                            ),
                            React.createElement('div', { className: 'logo-text' },
                                React.createElement('h1', null, 'Universal Scanner'),
                                React.createElement('p', null, 'QR va shtrix-kod skaneri')
                            )
                        ),
                        
                        user && React.createElement('div', { className: 'user-info' },
                            React.createElement('div', { className: 'user-details' },
                                React.createElement('div', { className: 'user-name' }, user.first_name || 'Foydalanuvchi'),
                                React.createElement('div', { className: 'user-username' }, '@' + (user.username || 'user'))
                            ),
                            user.photo_url && React.createElement('img', { 
                                src: user.photo_url, 
                                alt: "Profile", 
                                className: "user-avatar" 
                            })
                        )
                    )
                )
            );
        }

        function ScannerSection({ onScanResult, lastResult, onCopy, onSendToBot, webApp, settings, onSettingsChange }) {
            const [scanning, setScanning] = useState(false);
            const [error, setError] = useState(null);
            const [isZoomed, setIsZoomed] = useState(false);
            const [progress, setProgress] = useState(0);
            const videoRef = useRef(null);
            const canvasRef = useRef(null);
            const streamRef = useRef(null);
            const scanIntervalRef = useRef(null);

            const startScanning = async () => {
                setError(null);
                setProgress(0);
                
                try {
                    const stream = await navigator.mediaDevices.getUserMedia({
                        video: {
                            facingMode: 'environment',
                            width: { ideal: 1920 },
                            height: { ideal: 1080 },
                            zoom: settings.zoom
                        }
                    });

                    streamRef.current = stream;
                    const video = videoRef.current;
                    video.srcObject = stream;
                    
                    setScanning(true);
                    setProgress(30);

                    // Video yuklanganidan keyin skanerlashni boshlash
                    video.onloadeddata = () => {
                        setProgress(60);
                        // QR va barcode skanerlash uchun interval
                        scanIntervalRef.current = setInterval(() => {
                            if (!scanning) return;
                            scanForCodes(video);
                        }, 300); // Har 300ms da skanerlash
                        setProgress(100);
                    };

                } catch (err) {
                    setError(`Kamerani ochishda xato: ${err.message}`);
                    setScanning(false);
                    setProgress(0);
                }
            };

            const stopScanning = () => {
                setScanning(false);
                setIsZoomed(false);
                if (scanIntervalRef.current) {
                    clearInterval(scanIntervalRef.current);
                    scanIntervalRef.current = null;
                }
                if (streamRef.current) {
                    streamRef.current.getTracks().forEach(track => track.stop());
                    streamRef.current = null;
                }
                setProgress(0);
            };

            // Tasvirni yaxshilash funksiyasi
            const enhanceImage = (imageData) => {
                const data = imageData.data;
                const length = data.length;
                
                // Kontrast va yorug'likni sozlash
                for (let i = 0; i < length; i += 4) {
                    // Yorug'lik
                    if (settings.brightness !== 1.0) {
                        data[i] = Math.min(255, data[i] * settings.brightness);     // R
                        data[i + 1] = Math.min(255, data[i + 1] * settings.brightness); // G
                        data[i + 2] = Math.min(255, data[i + 2] * settings.brightness); // B
                    }
                    
                    // Kontrast
                    if (settings.contrast !== 1.0) {
                        const factor = (259 * (settings.contrast + 255)) / (255 * (259 - settings.contrast));
                        data[i] = factor * (data[i] - 128) + 128;
                        data[i + 1] = factor * (data[i + 1] - 128) + 128;
                        data[i + 2] = factor * (data[i + 2] - 128) + 128;
                    }
                }
                
                return imageData;
            };

            // QR va barcode kodlarni skanerlash funksiyasi
            const scanForCodes = async (video) => {
                const canvas = canvasRef.current;
                const context = canvas.getContext('2d');
                
                // Video o'lchamlarini olish
                const videoWidth = video.videoWidth;
                const videoHeight = video.videoHeight;
                
                if (videoWidth > 0 && videoHeight > 0) {
                    // Canvas o'lchamlarini sozlash
                    canvas.width = videoWidth;
                    canvas.height = videoHeight;
                    
                    // Videodan rasm chizish
                    context.drawImage(video, 0, 0, canvas.width, canvas.height);
                    
                    // Tasvirni yaxshilash
                    let imageData = context.getImageData(0, 0, canvas.width, canvas.height);
                    if (settings.imageEnhancement) {
                        imageData = enhanceImage(imageData);
                        context.putImageData(imageData, 0, 0);
                    }

                    // QR kodni skanerlash
                    try {
                        const qrCode = jsQR(imageData.data, imageData.width, imageData.height);
                        
                        if (qrCode) {
                            onScanResult({
                                text: qrCode.data,
                                format: 'QR_CODE'
                            });
                            
                            if (webApp) {
                                webApp.HapticFeedback.impactOccurred('medium');
                            }
                            
                            // Avtomatik zoom
                            if (settings.autoZoom && !isZoomed) {
                                setIsZoomed(true);
                            }
                            
                            stopScanning();
                            return;
                        }
                    } catch (e) {
                        console.log('QR kod skanerlash xatosi:', e);
                    }

                    // Barcode skanerlash
                    try {
                        Quagga.decodeSingle({
                            decoder: {
                                readers: ['code_128_reader', 'ean_reader', 'ean_8_reader', 'code_39_reader', 'upc_reader', 'codabar_reader']
                            },
                            locate: true,
                            src: canvas.toDataURL(),
                        }, function(result) {
                            if (result && result.codeResult) {
                                onScanResult({
                                    text: result.codeResult.code,
                                    format: result.codeResult.format
                                });
                                
                                if (webApp) {
                                    webApp.HapticFeedback.impactOccurred('medium');
                                }
                                
                                // Avtomatik zoom
                                if (settings.autoZoom && !isZoomed) {
                                    setIsZoomed(true);
                                }
                                
                                stopScanning();
                            }
                        });
                    } catch (e) {
                        console.log('Barcode skanerlash xatosi:', e);
                    }

                    // OCR bilan matn skanerlash (agar kerak bo'lsa)
                    try {
                        // Bu qismni faqat kerak bo'lganda ishlating
                        // Tesseract.js og'ir kutubxona, shuning uchun kommentga olingan
                        /*
                        const worker = await Tesseract.createWorker('eng+uzb');
                        const { data: { text } } = await worker.recognize(canvas);
                        await worker.terminate();
                        
                        if (text && text.trim().length > 5) {
                            onScanResult({
                                text: text.trim(),
                                format: 'TEXT_OCR'
                            });
                            stopScanning();
                        }
                        */
                    } catch (e) {
                        console.log('OCR xatosi:', e);
                    }
                }
            };

            useEffect(() => {
                // Komponent unmount bo'lganda tozalash
                return () => {
                    if (scanIntervalRef.current) {
                        clearInterval(scanIntervalRef.current);
                    }
                    if (streamRef.current) {
                        streamRef.current.getTracks().forEach(track => track.stop());
                    }
                };
            }, []);

            return React.createElement('div', { className: 'scanner-section' },
                React.createElement('div', { className: 'container' },
                    // Settings Panel
                    React.createElement('div', { className: 'settings-panel' },
                        React.createElement('div', { className: 'settings-header' },
                            React.createElement('div', { className: 'settings-title' },
                                React.createElement('svg', {
                                    width: "18",
                                    height: "18",
                                    fill: "none",
                                    stroke: "currentColor",
                                    viewBox: "0 0 24 24"
                                },
                                    React.createElement('path', {
                                        strokeLinecap: "round",
                                        strokeLinejoin: "round",
                                        strokeWidth: "2",
                                        d: "M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"
                                    }),
                                    React.createElement('path', {
                                        strokeLinecap: "round",
                                        strokeLinejoin: "round",
                                        strokeWidth: "2",
                                        d: "M15 12a3 3 0 11-6 0 3 3 0 016 0z"
                                    })
                                ),
                                'Skanerlash Sozlamalari'
                            )
                        ),
                        React.createElement('div', { className: 'settings-grid' },
                            React.createElement('div', { className: 'setting-item' },
                                React.createElement('div', { className: 'setting-label' },
                                    React.createElement('svg', {
                                        width: "12",
                                        height: "12",
                                        fill: "none",
                                        stroke: "currentColor",
                                        viewBox: "0 0 24 24"
                                    },
                                        React.createElement('path', {
                                            strokeLinecap: "round",
                                            strokeLinejoin: "round",
                                            strokeWidth: "2",
                                            d: "M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"
                                        })
                                    ),
                                    'Zoom'
                                ),
                                React.createElement('input', {
                                    type: 'range',
                                    min: '1',
                                    max: '5',
                                    step: '0.1',
                                    value: settings.zoom,
                                    onChange: (e) => onSettingsChange({...settings, zoom: parseFloat(e.target.value)}),
                                    className: 'slider'
                                }),
                                React.createElement('div', { className: 'setting-value' }, `${settings.zoom}x`)
                            ),
                            React.createElement('div', { className: 'setting-item' },
                                React.createElement('div', { className: 'setting-label' },
                                    React.createElement('svg', {
                                        width: "12",
                                        height: "12",
                                        fill: "none",
                                        stroke: "currentColor",
                                        viewBox: "0 0 24 24"
                                    },
                                        React.createElement('path', {
                                            strokeLinecap: "round",
                                            strokeLinejoin: "round",
                                            strokeWidth: "2",
                                            d: "M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z"
                                        })
                                    ),
                                    'Yorug\'lik'
                                ),
                                React.createElement('input', {
                                    type: 'range',
                                    min: '0.5',
                                    max: '2',
                                    step: '0.1',
                                    value: settings.brightness,
                                    onChange: (e) => onSettingsChange({...settings, brightness: parseFloat(e.target.value)}),
                                    className: 'slider'
                                }),
                                React.createElement('div', { className: 'setting-value' }, `${settings.brightness}x`)
                            ),
                            React.createElement('div', { className: 'setting-item' },
                                React.createElement('div', { className: 'setting-label' },
                                    React.createElement('svg', {
                                        width: "12",
                                        height: "12",
                                        fill: "none",
                                        stroke: "currentColor",
                                        viewBox: "0 0 24 24"
                                    },
                                        React.createElement('path', {
                                            strokeLinecap: "round",
                                            strokeLinejoin: "round",
                                            strokeWidth: "2",
                                            d: "M7 8h10M7 12h4m1 8l-4-4H5a2 2 0 01-2-2V6a2 2 0 012-2h14a2 2 0 012 2v8a2 2 0 01-2 2h-3l-4 4z"
                                        })
                                    ),
                                    'Kontrast'
                                ),
                                React.createElement('input', {
                                    type: 'range',
                                    min: '0.5',
                                    max: '2',
                                    step: '0.1',
                                    value: settings.contrast,
                                    onChange: (e) => onSettingsChange({...settings, contrast: parseFloat(e.target.value)}),
                                    className: 'slider'
                                }),
                                React.createElement('div', { className: 'setting-value' }, `${settings.contrast}x`)
                            ),
                            React.createElement('div', { className: 'setting-item' },
                                React.createElement('div', { className: 'setting-label' },
                                    React.createElement('svg', {
                                        width: "12",
                                        height: "12",
                                        fill: "none",
                                        stroke: "currentColor",
                                        viewBox: "0 0 24 24"
                                    },
                                        React.createElement('path', {
                                            strokeLinecap: "round",
                                            strokeLinejoin: "round",
                                            strokeWidth: "2",
                                            d: "M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12h-1M4 12H3m3.343-5.657l-.707-.707m2.828 9.9a5 5 0 117.072 0l-.548.547A3.374 3.374 0 0014 18.469V19a2 2 0 11-4 0v-.531c0-.895-.356-1.754-.988-2.386l-.548-.547z"
                                        })
                                    ),
                                    'Avtomatik Zoom'
                                ),
                                React.createElement('div', { 
                                    style: { 
                                        display: 'flex', 
                                        alignItems: 'center', 
                                        gap: '8px',
                                        marginTop: '4px'
                                    } 
                                },
                                    React.createElement('label', { 
                                        style: { 
                                            display: 'flex', 
                                            alignItems: 'center', 
                                            gap: '4px',
                                            fontSize: '12px'
                                        } 
                                    },
                                        React.createElement('input', {
                                            type: 'checkbox',
                                            checked: settings.autoZoom,
                                            onChange: (e) => onSettingsChange({...settings, autoZoom: e.target.checked}),
                                            style: { margin: 0 }
                                        }),
                                        'Yoqilgan'
                                    )
                                )
                            )
                        )
                    ),

                    // Progress Bar
                    progress > 0 && React.createElement('div', { className: 'progress-bar' },
                        React.createElement('div', { 
                            className: 'progress-fill',
                            style: { width: `${progress}%` }
                        })
                    ),

                    // Scanner View
                    React.createElement('div', { className: 'scanner-container' },
                        React.createElement('video', {
                            ref: videoRef,
                            autoPlay: true,
                            playsInline: true,
                            muted: true,
                            style: {
                                width: '100%',
                                height: '100%',
                                objectFit: 'cover',
                                display: scanning ? 'block' : 'none',
                                filter: `brightness(${settings.brightness}) contrast(${settings.contrast})`
                            }
                        }),
                        React.createElement('canvas', {
                            ref: canvasRef,
                            style: { display: 'none' }
                        }),
                        
                        scanning && React.createElement('div', { className: 'scanner-overlay' },
                            React.createElement('div', { className: `scanner-frame ${isZoomed ? 'zoomed' : ''}` },
                                React.createElement('div', { className: 'scanner-corner corner-tl' }),
                                React.createElement('div', { className: 'scanner-corner corner-tr' }),
                                React.createElement('div', { className: 'scanner-corner corner-bl' }),
                                React.createElement('div', { className: 'scanner-corner corner-br' }),
                                React.createElement('div', { className: 'laser-line' })
                            )
                        ),

                        !scanning && React.createElement('div', { className: 'scanner-status' },
                            React.createElement('div', null,
                                React.createElement('svg', {
                                    className: 'status-icon',
                                    fill: "none",
                                    stroke: "currentColor",
                                    viewBox: "0 0 24 24"
                                }, [
                                    React.createElement('path', {
                                        strokeLinecap: "round",
                                        strokeLinejoin: "round",
                                        strokeWidth: "2",
                                        d: "M3 9a2 2 0 012-2h.93a2 2 0 001.664-.89l.812-1.22A2 2 0 0110.07 4h3.86a2 2 0 011.664.89l.812 1.22A2 2 0 0018.07 7H19a2 2 0 012 2v9a2 2 0 01-2 2H5a2 2 0 01-2-2V9z"
                                    }),
                                    React.createElement('path', {
                                        strokeLinecap: "round",
                                        strokeLinejoin: "round",
                                        strokeWidth: "2",
                                        d: "M15 13a3 3 0 11-6 0 3 3 0 016 0z"
                                    })
                                ]),
                                React.createElement('p', { 
                                    style: { fontSize: '18px', fontWeight: '600', marginBottom: '4px' } 
                                }, 'Kamera tayyor'),
                                React.createElement('p', { 
                                    style: { fontSize: '14px', opacity: '0.8' } 
                                }, 'Skanerlashni boshlash uchun tugmani bosing')
                            )
                        )
                    ),

                    // Error Display
                    error && React.createElement('div', { className: 'error-message' },
                        React.createElement('svg', {
                            className: 'error-icon',
                            fill: "none",
                            stroke: "currentColor",
                            viewBox: "0 0 24 24"
                        },
                            React.createElement('path', {
                                strokeLinecap: "round",
                                strokeLinejoin: "round",
                                strokeWidth: "2",
                                d: "M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
                            })
                        ),
                        error
                    ),

                    // Control Buttons
                    React.createElement('div', { className: 'controls' },
                        !scanning ? 
                            React.createElement('button', {
                                onClick: startScanning,
                                className: 'btn btn-primary'
                            }, [
                                React.createElement('svg', {
                                    className: 'btn-icon',
                                    fill: "none",
                                    stroke: "currentColor",
                                    viewBox: "0 0 24 24"
                                },
                                    React.createElement('path', {
                                        strokeLinecap: "round",
                                        strokeLinejoin: "round",
                                        strokeWidth: "2",
                                        d: "M12 4v1m6 11h2m-6 0h-2v4m0-11v3m0 0h.01M12 12h4.01M16 20h4M4 12h4m12 0h.01M5 8h2a1 1 0 001-1V5a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1zm12 0h2a1 1 0 001-1V5a1 1 0 00-1-1h-2a1 1 0 00-1 1v2a1 1 0 001 1zM5 20h2a1 1 0 001-1v-2a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1z"
                                    })
                                ),
                                'Skanerlashni Boshlash'
                            ]) :
                            React.createElement('button', {
                                onClick: stopScanning,
                                className: 'btn btn-danger'
                            }, [
                                React.createElement('svg', {
                                    className: 'btn-icon',
                                    fill: "none",
                                    stroke: "currentColor",
                                    viewBox: "0 0 24 24"
                                }, [
                                    React.createElement('path', {
                                        strokeLinecap: "round",
                                        strokeLinejoin: "round",
                                        strokeWidth: "2",
                                        d: "M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
                                    }),
                                    React.createElement('path', {
                                        strokeLinecap: "round",
                                        strokeLinejoin: "round",
                                        strokeWidth: "2",
                                        d: "M9 10a1 1 0 011-1h4a1 1 0 011 1v4a1 1 0 01-1 1h-4a1 1 0 01-1-1v-4z"
                                    })
                                ]),
                                'To\'xtatish'
                            ])
                    ),

                    // Last Result
                    lastResult && React.createElement('div', { className: 'result-section' },
                        React.createElement('div', { className: 'result-header' },
                            React.createElement('div', { className: 'result-title' },
                                React.createElement('svg', {
                                    width: "20",
                                    height: "20",
                                    fill: "none",
                                    stroke: "currentColor",
                                    viewBox: "0 0 24 24"
                                },
                                    React.createElement('path', {
                                        strokeLinecap: "round",
                                        strokeLinejoin: "round",
                                        strokeWidth: "2",
                                        d: "M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"
                                    })
                                ),
                                'Kod Topildi!'
                            ),
                            React.createElement('span', { className: 'result-format' }, lastResult.format)
                        ),
                        
                        React.createElement('div', { className: 'result-content' },
                            React.createElement('pre', { className: 'result-text' }, lastResult.text)
                        ),

                        React.createElement('div', { className: 'result-actions' },
                            React.createElement('button', {
                                onClick: () => onCopy(lastResult.text),
                                className: 'btn btn-primary'
                            }, 'Nusxalash'),
                            React.createElement('button', {
                                onClick: () => onSendToBot(lastResult),
                                className: 'btn btn-success'
                            }, 'Botga Yuborish'),
                            React.createElement('button', {
                                onClick: () => {
                                    setLastResult(null);
                                    startScanning();
                                },
                                className: 'btn btn-secondary'
                            }, 'Yana Skanerlash')
                        )
                    ),

                    // Instructions
                    React.createElement('div', { className: 'instructions' },
                        React.createElement('div', { className: 'instructions-header' },
                            React.createElement('svg', {
                                width: "18",
                                height: "18",
                                fill: "none",
                                stroke: "currentColor",
                                viewBox: "0 0 24 24"
                            },
                                React.createElement('path', {
                                    strokeLinecap: "round",
                                    strokeLinejoin: "round",
                                    strokeWidth: "2",
                                    d: "M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
                                })
                            ),
                            'Qanday ishlatish'
                        ),
                        React.createElement('ul', { className: 'instructions-list' }, [
                            React.createElement('li', { key: 1 }, '• QR kod yoki shtrix-kodni ramkaga joylashtiring'),
                            React.createElement('li', { key: 2 }, '• Yorug\'lik yetarli bo\'lishiga ishonch hosil qiling'),
                            React.createElement('li', { key: 3 }, '• Kod avtomatik ravishda skanerlanadi'),
                            React.createElement('li', { key: 4 }, '• Avtomatik zoom kodni topganda faollashadi')
                        ])
                    )
                )
            );
        }

        function HistorySection({ items, onClear, onItemClick }) {
            if (items.length === 0) {
                return React.createElement('div', { className: 'history-section' },
                    React.createElement('div', { className: 'container' },
                        React.createElement('div', { className: 'empty-state' },
                            React.createElement('svg', {
                                className: 'empty-icon',
                                fill: "none",
                                stroke: "currentColor",
                                viewBox: "0 0 24 24"
                            },
                                React.createElement('path', {
                                    strokeLinecap: "round",
                                    strokeLinejoin: "round",
                                    strokeWidth: "2",
                                    d: "M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"
                                })
                            ),
                            React.createElement('h3', { 
                                style: { marginBottom: '8px', color: 'var(--tg-theme-text-color)' } 
                            }, 'Hali skanerlar mavjud emas'),
                            React.createElement('p', { 
                                style: { color: 'var(--tg-theme-hint-color)' } 
                            }, 'Skanerlash natijalari shu yerda ko\'rinadi')
                        )
                    )
                );
            }

            return React.createElement('div', { className: 'history-section' },
                React.createElement('div', { className: 'container' },
                    React.createElement('div', { className: 'history-header' },
                        React.createElement('h2', { className: 'history-title' }, 'Skanerlar Tarixi'),
                        React.createElement('button', { onClick: onClear, className: 'clear-btn' }, [
                            React.createElement('svg', {
                                width: "16",
                                height: "16",
                                fill: "none",
                                stroke: "currentColor",
                                viewBox: "0 0 24 24"
                            },
                                React.createElement('path', {
                                    strokeLinecap: "round",
                                    strokeLinejoin: "round",
                                    strokeWidth: "2",
                                    d: "M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"
                                })
                            ),
                            'Tozalash'
                        ])
                    ),

                    React.createElement('div', { className: 'history-list' },
                        items.map((item) => 
                            React.createElement('div', {
                                key: item.id,
                                className: 'history-item',
                                onClick: () => onItemClick(item)
                            }, [
                                React.createElement('div', { className: 'history-item-header' }, [
                                    React.createElement('div', { className: 'history-item-type' }, [
                                        React.createElement('div', { className: `type-icon ${item.type}` },
                                            item.type === 'qr' ? 
                                                React.createElement('svg', {
                                                    width: "16",
                                                    height: "16",
                                                    fill: "none",
                                                    stroke: "currentColor",
                                                    viewBox: "0 0 24 24"
                                                },
                                                    React.createElement('path', {
                                                        strokeLinecap: "round",
                                                        strokeLinejoin: "round",
                                                        strokeWidth: "2",
                                                        d: "M12 4v1m6 11h2m-6 0h-2v4m0-11v3m0 0h.01M12 12h4.01M16 20h4M4 12h4m12 0h.01M5 8h2a1 1 0 001-1V5a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1zm12 0h2a1 1 0 001-1V5a1 1 0 00-1-1h-2a1 1 0 00-1 1v2a1 1 0 001 1zM5 20h2a1 1 0 001-1v-2a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1z"
                                                    })
                                                ) :
                                                item.type === 'barcode' ?
                                                React.createElement('svg', {
                                                    width: "16",
                                                    height: "16",
                                                    fill: "none",
                                                    stroke: "currentColor",
                                                    viewBox: "0 0 24 24"
                                                },
                                                    React.createElement('path', {
                                                        strokeLinecap: "round",
                                                        strokeLinejoin: "round",
                                                        strokeWidth: "2",
                                                        d: "M10 20l4-16m4 4l4 4-4 4M6 16l-4-4 4-4"
                                                    })
                                                ) :
                                                React.createElement('svg', {
                                                    width: "16",
                                                    height: "16",
                                                    fill: "none",
                                                    stroke: "currentColor",
                                                    viewBox: "0 0 24 24"
                                                },
                                                    React.createElement('path', {
                                                        strokeLinecap: "round",
                                                        strokeLinejoin: "round",
                                                        strokeWidth: "2",
                                                        d: "M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"
                                                    })
                                                )
                                        ),
                                        React.createElement('div', { className: 'history-item-info' }, [
                                            React.createElement('h4', null, item.format),
                                            React.createElement('p', null, item.timestamp)
                                        ])
                                    ]),
                                    React.createElement('svg', {
                                        width: "16",
                                        height: "16",
                                        fill: "none",
                                        stroke: "currentColor",
                                        viewBox: "0 0 24 24"
                                    },
                                        React.createElement('path', {
                                            strokeLinecap: "round",
                                            strokeLinejoin: "round",
                                            strokeWidth: "2",
                                            d: "M9 5l7 7-7 7"
                                        })
                                    )
                                ]),
                                
                                React.createElement('div', { className: 'history-item-content' },
                                    React.createElement('div', { className: 'history-item-text' }, item.text)
                                )
                            ])
                        )
                    ),

                    React.createElement('div', { 
                        style: { 
                            textAlign: 'center', 
                            marginTop: '16px', 
                            color: 'var(--tg-theme-hint-color)', 
                            fontSize: '14px' 
                        } 
                    }, `${items.length} ta skaner`)
                )
            );
        }

        function Navigation({ activeTab, onTabChange }) {
            return React.createElement('nav', { className: 'navigation' }, [
                React.createElement('button', {
                    key: 'scan',
                    className: `nav-item ${activeTab === 'scan' ? 'active' : ''}`,
                    onClick: () => onTabChange('scan')
                }, [
                    React.createElement('svg', {
                        className: 'nav-icon',
                        fill: "none",
                        stroke: "currentColor",
                        viewBox: "0 0 24 24"
                    },
                        React.createElement('path', {
                            strokeLinecap: "round",
                            strokeLinejoin: "round",
                            strokeWidth: "2",
                            d: "M12 4v1m6 11h2m-6 0h-2v4m0-11v3m0 0h.01M12 12h4.01M16 20h4M4 12h4m12 0h.01M5 8h2a1 1 0 001-1V5a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1zm12 0h2a1 1 0 001-1V5a1 1 0 00-1-1h-2a1 1 0 00-1 1v2a1 1 0 001 1zM5 20h2a1 1 0 001-1v-2a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1z"
                        })
                    ),
                    React.createElement('span', { className: 'nav-label' }, 'Skaner')
                ]),
                
                React.createElement('button', {
                    key: 'history',
                    className: `nav-item ${activeTab === 'history' ? 'active' : ''}`,
                    onClick: () => onTabChange('history')
                }, [
                    React.createElement('svg', {
                        className: 'nav-icon',
                        fill: "none",
                        stroke: "currentColor",
                        viewBox: "0 0 24 24"
                    },
                        React.createElement('path', {
                            strokeLinecap: "round",
                            strokeLinejoin: "round",
                            strokeWidth: "2",
                            d: "M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"
                        })
                    ),
                    React.createElement('span', { className: 'nav-label' }, 'Tarix')
                ])
            ]);
        }

        // React ilovasini ishga tushirish
        ReactDOM.render(React.createElement(App), document.getElementById('root'));
    </script>
</body>
</html>
