<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Universal Scanner - Telegram Mini App</title>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://unpkg.com/@zxing/library@0.20.0"></script>
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
        }

        .scanner-video {
            width: 100%;
            height: 300px;
            object-fit: cover;
            display: block;
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
        }

        .scanner-corner {
            position: absolute;
            width: 20px;
            height: 20px;
            border-color: #10b981;
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

        .btn-icon {
            width: 18px;
            height: 18px;
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

    <script type="text/babel">
        const { useState, useEffect, useRef } = React;
        const { BrowserMultiFormatReader, BarcodeFormat, DecodeHintType } = ZXing;

        function App() {
            const [activeTab, setActiveTab] = useState('scan');
            const [scanHistory, setScanHistory] = useState([]);
            const [lastResult, setLastResult] = useState(null);
            const [webApp, setWebApp] = useState(null);
            const [user, setUser] = useState(null);

            useEffect(() => {
                const tg = window.Telegram?.WebApp;
                if (tg) {
                    setWebApp(tg);
                    setUser(tg.initDataUnsafe?.user);
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
            }, []);

            const addToHistory = (result) => {
                const newItem = {
                    id: Date.now(),
                    text: result.text,
                    format: result.format,
                    timestamp: new Date().toLocaleString('uz-UZ'),
                    type: result.format.toLowerCase().includes('qr') ? 'qr' : 'barcode'
                };
                
                setScanHistory(prev => [newItem, ...prev.slice(0, 49)]);
            };

            const clearHistory = () => {
                setScanHistory([]);
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

            return (
                <div className="app">
                    <Header user={user} />
                    
                    <main style={{ paddingBottom: '80px' }}>
                        {activeTab === 'scan' && (
                            <ScannerSection 
                                onScanResult={(result) => {
                                    setLastResult(result);
                                    addToHistory(result);
                                }}
                                lastResult={lastResult}
                                onCopy={copyToClipboard}
                                onSendToBot={sendToBot}
                                webApp={webApp}
                            />
                        )}
                        
                        {activeTab === 'history' && (
                            <HistorySection 
                                items={scanHistory}
                                onClear={clearHistory}
                                onItemClick={(item) => copyToClipboard(item.text)}
                            />
                        )}
                    </main>

                    <Navigation activeTab={activeTab} onTabChange={setActiveTab} />
                </div>
            );
        }

        function Header({ user }) {
            return (
                <header className="header">
                    <div className="container">
                        <div className="header-content">
                            <div className="logo">
                                <div className="logo-icon">
                                    <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2">
                                        <path d="M12 4v1m6 11h2m-6 0h-2v4m0-11v3m0 0h.01M12 12h4.01M16 20h4M4 12h4m12 0h.01M5 8h2a1 1 0 001-1V5a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1zm12 0h2a1 1 0 001-1V5a1 1 0 00-1-1h-2a1 1 0 00-1 1v2a1 1 0 001 1zM5 20h2a1 1 0 001-1v-2a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1z"/>
                                    </svg>
                                </div>
                                <div className="logo-text">
                                    <h1>Universal Scanner</h1>
                                    <p>QR va shtrix-kod skaneri</p>
                                </div>
                            </div>
                            
                            {user && (
                                <div className="user-info">
                                    <div className="user-details">
                                        <div className="user-name">{user.first_name || 'Foydalanuvchi'}</div>
                                        <div className="user-username">@{user.username || 'user'}</div>
                                    </div>
                                    {user.photo_url && (
                                        <img src={user.photo_url} alt="Profile" className="user-avatar" />
                                    )}
                                </div>
                            )}
                        </div>
                    </div>
                </header>
            );
        }

        function ScannerSection({ onScanResult, lastResult, onCopy, onSendToBot, webApp }) {
            const videoRef = useRef(null);
            const readerRef = useRef(null);
            const [scanning, setScanning] = useState(false);
            const [error, setError] = useState(null);
            const [torchOn, setTorchOn] = useState(false);
            const [torchAvailable, setTorchAvailable] = useState(false);

            useEffect(() => {
                // Initialize scanner
                const formats = [
                    BarcodeFormat.QR_CODE,
                    BarcodeFormat.DATA_MATRIX,
                    BarcodeFormat.AZTEC,
                    BarcodeFormat.PDF_417,
                    BarcodeFormat.CODE_128,
                    BarcodeFormat.CODE_39,
                    BarcodeFormat.EAN_13,
                    BarcodeFormat.EAN_8,
                    BarcodeFormat.UPC_A,
                    BarcodeFormat.UPC_E,
                ].filter(Boolean);

                const hints = new Map();
                hints.set(DecodeHintType.POSSIBLE_FORMATS, formats);

                readerRef.current = new BrowserMultiFormatReader(hints);

                return () => {
                    stopScanning();
                };
            }, []);

            const startScanning = async () => {
                setError(null);
                
                try {
                    const stream = await navigator.mediaDevices.getUserMedia({
                        video: {
                            facingMode: 'environment',
                            width: { ideal: 1280 },
                            height: { ideal: 720 }
                        }
                    });

                    videoRef.current.srcObject = stream;
                    
                    const track = stream.getVideoTracks()[0];
                    const capabilities = track.getCapabilities?.() || {};
                    setTorchAvailable(!!capabilities.torch);

                    setScanning(true);

                    const decodeLoop = async () => {
                        if (!videoRef.current || videoRef.current.readyState < 2) {
                            requestAnimationFrame(decodeLoop);
                            return;
                        }

                        try {
                            const result = await readerRef.current.decodeFromVideoElement(videoRef.current);
                            if (result) {
                                onScanResult({
                                    text: result.getText(),
                                    format: result.getBarcodeFormat()
                                });
                                
                                // Haptic feedback
                                if (webApp) {
                                    webApp.HapticFeedback.impactOccurred('medium');
                                }
                            }
                        } catch (err) {
                            // Ignore decoding errors
                        }

                        if (scanning) {
                            requestAnimationFrame(decodeLoop);
                        }
                    };

                    decodeLoop();
                } catch (err) {
                    setError(`Kamerani ochishda xato: ${err.message}`);
                    setScanning(false);
                }
            };

            const stopScanning = () => {
                setScanning(false);
                if (videoRef.current?.srcObject) {
                    videoRef.current.srcObject.getTracks().forEach(track => track.stop());
                    videoRef.current.srcObject = null;
                }
                setTorchOn(false);
            };

            const toggleTorch = async () => {
                try {
                    const track = videoRef.current?.srcObject?.getVideoTracks()[0];
                    if (!track) return;

                    await track.applyConstraints({
                        advanced: [{ torch: !torchOn }]
                    });
                    setTorchOn(!torchOn);
                } catch (err) {
                    console.error('Torch toggle error:', err);
                }
            };

            return (
                <div className="scanner-section">
                    <div className="container">
                        {/* Scanner View */}
                        <div className="scanner-container">
                            <video 
                                ref={videoRef}
                                autoPlay 
                                playsInline 
                                muted 
                                className="scanner-video"
                            />
                            
                            {scanning && (
                                <div className="scanner-overlay">
                                    <div className="scanner-frame">
                                        <div className="scanner-corner corner-tl"></div>
                                        <div className="scanner-corner corner-tr"></div>
                                        <div className="scanner-corner corner-bl"></div>
                                        <div className="scanner-corner corner-br"></div>
                                        <div className="laser-line"></div>
                                    </div>
                                </div>
                            )}

                            {!scanning && (
                                <div className="scanner-status">
                                    <div>
                                        <svg className="status-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M3 9a2 2 0 012-2h.93a2 2 0 001.664-.89l.812-1.22A2 2 0 0110.07 4h3.86a2 2 0 011.664.89l.812 1.22A2 2 0 0018.07 7H19a2 2 0 012 2v9a2 2 0 01-2 2H5a2 2 0 01-2-2V9z"/>
                                            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M15 13a3 3 0 11-6 0 3 3 0 016 0z"/>
                                        </svg>
                                        <p style={{ fontSize: '18px', fontWeight: '600', marginBottom: '4px' }}>Kamera tayyor</p>
                                        <p style={{ fontSize: '14px', opacity: '0.8' }}>Skanerlashni boshlash uchun tugmani bosing</p>
                                    </div>
                                </div>
                            )}
                        </div>

                        {/* Error Display */}
                        {error && (
                            <div className="error-message">
                                <svg className="error-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/>
                                </svg>
                                {error}
                            </div>
                        )}

                        {/* Control Buttons */}
                        <div className="controls">
                            {!scanning ? (
                                <button onClick={startScanning} className="btn btn-primary">
                                    <svg className="btn-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                        <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M12 4v1m6 11h2m-6 0h-2v4m0-11v3m0 0h.01M12 12h4.01M16 20h4M4 12h4m12 0h.01M5 8h2a1 1 0 001-1V5a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1zm12 0h2a1 1 0 001-1V5a1 1 0 00-1-1h-2a1 1 0 00-1 1v2a1 1 0 001 1zM5 20h2a1 1 0 001-1v-2a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1z"/>
                                    </svg>
                                    Skanerlashni Boshlash
                                </button>
                            ) : (
                                <button onClick={stopScanning} className="btn btn-danger">
                                    <svg className="btn-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                        <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/>
                                        <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M9 10a1 1 0 011-1h4a1 1 0 011 1v4a1 1 0 01-1 1h-4a1 1 0 01-1-1v-4z"/>
                                    </svg>
                                    To'xtatish
                                </button>
                            )}

                            {torchAvailable && scanning && (
                                <button onClick={toggleTorch} className={`btn ${torchOn ? 'btn-warning' : 'btn-secondary'}`}>
                                    <svg className="btn-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                        <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12h-1M4 12H3m3.343-5.657l-.707-.707m2.828 9.9a5 5 0 117.072 0l-.548.547A3.374 3.374 0 0014 18.469V19a2 2 0 11-4 0v-.531c0-.895-.356-1.754-.988-2.386l-.548-.547z"/>
                                    </svg>
                                    {torchOn ? 'O\'chirish' : 'Chiroq'}
                                </button>
                            )}
                        </div>

                        {/* Last Result */}
                        {lastResult && (
                            <div className="result-section">
                                <div className="result-header">
                                    <div className="result-title">
                                        <svg width="20" height="20" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"/>
                                        </svg>
                                        Kod Topildi!
                                    </div>
                                    <span className="result-format">{lastResult.format}</span>
                                </div>
                                
                                <div className="result-content">
                                    <pre className="result-text">{lastResult.text}</pre>
                                </div>

                                <div className="result-actions">
                                    <button onClick={() => onCopy(lastResult.text)} className="btn btn-primary">
                                        Nusxalash
                                    </button>
                                    <button onClick={() => onSendToBot(lastResult)} className="btn btn-success">
                                        Botga Yuborish
                                    </button>
                                </div>
                            </div>
                        )}

                        {/* Instructions */}
                        <div className="instructions">
                            <div className="instructions-header">
                                <svg width="18" height="18" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"/>
                                </svg>
                                Qanday ishlatish
                            </div>
                            <ul className="instructions-list">
                                <li>• QR kod yoki shtrix-kodni ramkaga joylashtiring</li>
                                <li>• Yorug'lik yetarli bo'lishiga ishonch hosil qiling</li>
                                <li>• Kod avtomatik ravishda skanerlanadi</li>
                            </ul>
                        </div>
                    </div>
                </div>
            );
        }

        function HistorySection({ items, onClear, onItemClick }) {
            if (items.length === 0) {
                return (
                    <div className="history-section">
                        <div className="container">
                            <div className="empty-state">
                                <svg className="empty-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"/>
                                </svg>
                                <h3 style={{ marginBottom: '8px', color: 'var(--tg-theme-text-color)' }}>Hali skanerlar mavjud emas</h3>
                                <p style={{ color: 'var(--tg-theme-hint-color)' }}>Skanerlash natijalari shu yerda ko'rinadi</p>
                            </div>
                        </div>
                    </div>
                );
            }

            return (
                <div className="history-section">
                    <div className="container">
                        <div className="history-header">
                            <h2 className="history-title">Skanerlar Tarixi</h2>
                            <button onClick={onClear} className="clear-btn">
                                <svg width="16" height="16" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"/>
                                </svg>
                                Tozalash
                            </button>
                        </div>

                        <div className="history-list">
                            {items.map((item) => (
                                <div key={item.id} className="history-item" onClick={() => onItemClick(item)}>
                                    <div className="history-item-header">
                                        <div className="history-item-type">
                                            <div className={`type-icon ${item.type}`}>
                                                {item.type === 'qr' ? (
                                                    <svg width="16" height="16" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                                        <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M12 4v1m6 11h2m-6 0h-2v4m0-11v3m0 0h.01M12 12h4.01M16 20h4M4 12h4m12 0h.01M5 8h2a1 1 0 001-1V5a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1zm12 0h2a1 1 0 001-1V5a1 1 0 00-1-1h-2a1 1 0 00-1 1v2a1 1 0 001 1zM5 20h2a1 1 0 001-1v-2a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1z"/>
                                                    </svg>
                                                ) : (
                                                    <svg width="16" height="16" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                                        <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M10 20l4-16m4 4l4 4-4 4M6 16l-4-4 4-4"/>
                                                    </svg>
                                                )}
                                            </div>
                                            <div className="history-item-info">
                                                <h4>{item.format}</h4>
                                                <p>{item.timestamp}</p>
                                            </div>
                                        </div>
                                        <svg width="16" height="16" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M9 5l7 7-7 7"/>
                                        </svg>
                                    </div>
                                    
                                    <div className="history-item-content">
                                        <div className="history-item-text">{item.text}</div>
                                    </div>
                                </div>
                            ))}
                        </div>

                        <div style={{ textAlign: 'center', marginTop: '16px', color: 'var(--tg-theme-hint-color)', fontSize: '14px' }}>
                            {items.length} ta skaner
                        </div>
                    </div>
                </div>
            );
        }

        function Navigation({ activeTab, onTabChange }) {
            return (
                <nav className="navigation">
                    <button 
                        className={`nav-item ${activeTab === 'scan' ? 'active' : ''}`}
                        onClick={() => onTabChange('scan')}
                    >
                        <svg className="nav-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M12 4v1m6 11h2m-6 0h-2v4m0-11v3m0 0h.01M12 12h4.01M16 20h4M4 12h4m12 0h.01M5 8h2a1 1 0 001-1V5a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1zm12 0h2a1 1 0 001-1V5a1 1 0 00-1-1h-2a1 1 0 00-1 1v2a1 1 0 001 1zM5 20h2a1 1 0 001-1v-2a1 1 0 00-1-1H5a1 1 0 00-1 1v2a1 1 0 001 1z"/>
                        </svg>
                        <span className="nav-label">Skaner</span>
                    </button>
                    
                    <button 
                        className={`nav-item ${activeTab === 'history' ? 'active' : ''}`}
                        onClick={() => onTabChange('history')}
                    >
                        <svg className="nav-icon" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z"/>
                        </svg>
                        <span className="nav-label">Tarix</span>
                    </button>
                </nav>
            );
        }

        ReactDOM.render(<App />, document.getElementById('root'));
    </script>
</body>
</html>
