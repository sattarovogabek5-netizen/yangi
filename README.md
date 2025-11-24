<!DOCTYPE html>
<html lang="uz">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>QR Code Skaneri - Avantika Shifokor</title>
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://unpkg.com/html5-qrcode"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        * {
            font-family: 'Inter', sans-serif;
        }
        .html5-qrcode-element {
            display: block;
            width: 100%;
            padding: 12px 16px;
            background: linear-gradient(to right, #059669, #047857);
            color: white;
            border: none;
            border-radius: 12px;
            font-weight: bold;
            font-size: 16px;
            cursor: pointer;
            transition: all 0.3s ease;
            box-shadow: 0 4px 6px rgba(5, 150, 105, 0.3);
        }
        .html5-qrcode-element:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 8px rgba(5, 150, 105, 0.4);
        }
        .html5-qrcode-element:active {
            transform: translateY(0);
        }
        #reader video {
            width: 100% !important;
            height: auto !important;
            border-radius: 12px;
        }
        #reader {
            position: relative;
        }
        .scanner-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 10;
        }
        .scanner-frame {
            width: 250px;
            height: 250px;
            border: 2px solid #10b981;
            border-radius: 12px;
            box-shadow: 0 0 0 9999px rgba(0, 0, 0, 0.3);
            position: relative;
        }
        .scanner-frame::before, .scanner-frame::after {
            content: '';
            position: absolute;
            width: 30px;
            height: 30px;
            border: 3px solid #10b981;
        }
        .scanner-frame::before {
            top: -3px;
            left: -3px;
            border-right: none;
            border-bottom: none;
        }
        .scanner-frame::after {
            bottom: -3px;
            right: -3px;
            border-left: none;
            border-top: none;
        }
        .scanner-line {
            position: absolute;
            width: 100%;
            height: 2px;
            background: #10b981;
            top: 50%;
            animation: scan 2s linear infinite;
        }
        @keyframes scan {
            0% { top: 10%; }
            50% { top: 90%; }
            100% { top: 10%; }
        }
    </style>
</head>
<body>
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useRef, useEffect } = React;

        function QRScanner() {
            const [scanning, setScanning] = useState(false);
            const [result, setResult] = useState('');
            const [error, setError] = useState('');
            const [success, setSuccess] = useState('');
            const [scanHistory, setScanHistory] = useState([]);
            const [showHistory, setShowHistory] = useState(false);
            const [zoomLevel, setZoomLevel] = useState(1);
            const [brightness, setBrightness] = useState(100);
            const [contrast, setContrast] = useState(100);
            const [cameraPermission, setCameraPermission] = useState(false);
            const html5QrCodeRef = useRef(null);
            const readerRef = useRef(null);

            // Tarixni yuklash
            useEffect(() => {
                const savedHistory = JSON.parse(localStorage.getItem('scanHistory') || '[]');
                setScanHistory(savedHistory);
            }, []);

            // Kameraga ruxsatni tekshirish
            useEffect(() => {
                checkCameraPermission();
            }, []);

            const checkCameraPermission = async () => {
                try {
                    const stream = await navigator.mediaDevices.getUserMedia({ video: true });
                    // Agar ruxsat mavjud bo'lsa, streamni to'xtatamiz
                    stream.getTracks().forEach(track => track.stop());
                    setCameraPermission(true);
                } catch (err) {
                    setCameraPermission(false);
                    console.error('Camera permission denied:', err);
                }
            };

            const startScanning = async () => {
                try {
                    setError('');
                    setResult('');
                    setSuccess('');
                    
                    // HTML5 QR Code scannerini ishga tushirish
                    html5QrCodeRef.current = new Html5Qrcode("reader");
                    
                    const config = {
                        fps: 10,
                        qrbox: { width: 250, height: 250 },
                        aspectRatio: 1.0,
                        focusMode: "continuous",
                        supportedScanTypes: [
                            Html5QrcodeScanType.SCAN_TYPE_CAMERA
                        ]
                    };
                    
                    await html5QrCodeRef.current.start(
                        { facingMode: "environment" },
                        config,
                        onScanSuccess,
                        onScanFailure
                    );
                    
                    setScanning(true);
                    
                } catch (err) {
                    console.error('Camera error:', err);
                    if (err.name === 'NotAllowedError') {
                        setError('Kameraga ruxsat berilmagan. Iltimos, brauzer sozlamalaridan kameraga ruxsat bering.');
                    } else if (err.name === 'NotFoundError') {
                        setError('Kamera topilmadi. Iltimos, qurilmangizda kamera mavjudligini tekshiring.');
                    } else if (err.name === 'NotSupportedError') {
                        setError('Bu funksiya brauzeringiz tomonidan qo\'llab-quvvatlanmaydi.');
                    } else if (err.name === 'NotReadableError') {
                        setError('Kamera band yoki ishlamayapti. Boshqa dastur kamerani ishlatayotgan bo\'lishi mumkin.');
                    } else {
                        setError('Kamerani ochishda xatolik yuz berdi: ' + err.message);
                    }
                }
            };

            const onScanSuccess = (decodedText, decodedResult) => {
                setResult(decodedText);
                stopScanning();
                
                // Tarixga qo'shish
                const newHistory = [{
                    data: decodedText,
                    timestamp: new Date().toLocaleString('uz-UZ'),
                    id: Date.now()
                }, ...scanHistory].slice(0, 20);
                setScanHistory(newHistory);
                localStorage.setItem('scanHistory', JSON.stringify(newHistory));
                
                // Ovozli signal
                playBeep();
            };

            const onScanFailure = (error) => {
                // Xatolarni ko'rsatmaslik mumkin, chunki bu doimiy ravishda chaqiriladi
                // Faqatgina debug uchun
                // console.log("Scan failure:", error);
            };

            const stopScanning = async () => {
                if (html5QrCodeRef.current && scanning) {
                    try {
                        await html5QrCodeRef.current.stop();
                        html5QrCodeRef.current.clear();
                    } catch (err) {
                        console.log("Scanner to'xtatishda xatolik:", err);
                    }
                }
                setScanning(false);
            };

            const playBeep = () => {
                try {
                    const audioContext = new (window.AudioContext || window.webkitAudioContext)();
                    const oscillator = audioContext.createOscillator();
                    const gainNode = audioContext.createGain();
                    
                    oscillator.connect(gainNode);
                    gainNode.connect(audioContext.destination);
                    
                    oscillator.frequency.value = 800;
                    oscillator.type = 'sine';
                    
                    gainNode.gain.setValueAtTime(0.3, audioContext.currentTime);
                    gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.1);
                    
                    oscillator.start(audioContext.currentTime);
                    oscillator.stop(audioContext.currentTime + 0.1);
                } catch (e) {
                    console.log('Audio error:', e);
                }
            };

            const copyToClipboard = (text) => {
                navigator.clipboard.writeText(text).then(() => {
                    setSuccess('📋 Nusxalandi!');
                    setTimeout(() => setSuccess(''), 2000);
                });
            };

            const deleteHistoryItem = (id) => {
                const newHistory = scanHistory.filter(item => item.id !== id);
                setScanHistory(newHistory);
                localStorage.setItem('scanHistory', JSON.stringify(newHistory));
            };

            const applyVideoFilters = () => {
                const video = document.querySelector('#reader video');
                if (video) {
                    video.style.transform = `scale(${zoomLevel})`;
                    video.style.filter = `brightness(${brightness}%) contrast(${contrast}%)`;
                }
            };

            useEffect(() => {
                applyVideoFilters();
            }, [zoomLevel, brightness, contrast]);

            useEffect(() => {
                return () => {
                    stopScanning();
                };
            }, []);

            if (showHistory) {
                return React.createElement('div', { className: "min-h-screen bg-gradient-to-br from-emerald-50 via-teal-50 to-green-50" },
                    React.createElement('div', { className: "max-w-md mx-auto min-h-screen flex flex-col" },
                        React.createElement('div', { className: "bg-gradient-to-r from-emerald-600 to-teal-600 px-4 py-6 shadow-lg" },
                            React.createElement('div', { className: "flex items-center gap-3" },
                                React.createElement('button', { onClick: () => setShowHistory(false), className: "text-white" },
                                    React.createElement('svg', { xmlns: "http://www.w3.org/2000/svg", width: "24", height: "24", viewBox: "0 0 24 24", fill: "none", stroke: "currentColor", strokeWidth: "2", strokeLinecap: "round", strokeLinejoin: "round" },
                                        React.createElement('line', { x1: "18", y1: "6", x2: "6", y2: "18" }),
                                        React.createElement('line', { x1: "6", y1: "6", x2: "18", y2: "18" })
                                    )
                                ),
                                React.createElement('h1', { className: "text-2xl font-bold text-white flex items-center gap-2" },
                                    React.createElement('svg', { xmlns: "http://www.w3.org/2000/svg", width: "28", height: "28", viewBox: "0 0 24 24", fill: "none", stroke: "currentColor", strokeWidth: "2", strokeLinecap: "round", strokeLinejoin: "round" },
                                        React.createElement('path', { d: "M3 3v5h5" }),
                                        React.createElement('path', { d: "M3.05 13A9 9 0 1 0 12 3" }),
                                        React.createElement('path', { d: "M12 7v5l2 2" })
                                    ),
                                    "Skaner Tarixi"
                                )
                            )
                        ),
                        React.createElement('div', { className: "flex-1 p-4 space-y-3" },
                            scanHistory.length === 0 ? 
                                React.createElement('div', { className: "text-center py-12" },
                                    React.createElement('svg', { xmlns: "http://www.w3.org/2000/svg", width: "64", height: "64", viewBox: "0 0 24 24", fill: "none", stroke: "currentColor", strokeWidth: "2", strokeLinecap: "round", strokeLinejoin: "round", className: "mx-auto text-gray-400 mb-4" },
                                        React.createElement('path', { d: "M3 3v5h5" }),
                                        React.createElement('path', { d: "M3.05 13A9 9 0 1 0 12 3" }),
                                        React.createElement('path', { d: "M12 7v5l2 2" })
                                    ),
                                    React.createElement('p', { className: "text-gray-600" }, "Hozircha skaner tarixi bo'sh")
                                ) :
                                scanHistory.map((item) => 
                                    React.createElement('div', { key: item.id, className: "bg-white rounded-xl shadow p-4" },
                                        React.createElement('div', { className: "flex justify-between items-start mb-2" },
                                            React.createElement('span', { className: "text-xs text-gray-500" }, item.timestamp),
                                            React.createElement('button', { 
                                                onClick: () => deleteHistoryItem(item.id),
                                                className: "text-red-500 hover:text-red-700" 
                                            },
                                                React.createElement('svg', { xmlns: "http://www.w3.org/2000/svg", width: "16", height: "16", viewBox: "0 0 24 24", fill: "none", stroke: "currentColor", strokeWidth: "2", strokeLinecap: "round", strokeLinejoin: "round" },
                                                    React.createElement('line', { x1: "18", y1: "6", x2: "6", y2: "18" }),
                                                    React.createElement('line', { x1: "6", y1: "6", x2: "18", y2: "18" })
                                                )
                                            )
                                        ),
                                        React.createElement('p', { className: "text-sm font-mono text-gray-800 break-all mb-3" }, item.data),
                                        React.createElement('button', { 
                                            onClick: () => copyToClipboard(item.data),
                                            className: "text-xs bg-emerald-100 text-emerald-700 px-3 py-1 rounded-lg font-semibold" 
                                        }, "📋 Nusxalash")
                                    )
                                )
                        )
                    )
                );
            }

            return React.createElement('div', { className: "min-h-screen bg-gradient-to-br from-emerald-50 via-teal-50 to-green-50" },
                React.createElement('div', { className: "max-w-md mx-auto min-h-screen flex flex-col" },
                    React.createElement('div', { className: "bg-gradient-to-r from-emerald-600 via-teal-600 to-green-600 px-4 py-6 shadow-lg" },
                        React.createElement('div', { className: "flex justify-between items-center mb-4" },
                            React.createElement('button', { onClick: () => setShowHistory(true), className: "text-white p-2" },
                                React.createElement('svg', { xmlns: "http://www.w3.org/2000/svg", width: "24", height: "24", viewBox: "0 0 24 24", fill: "none", stroke: "currentColor", strokeWidth: "2", strokeLinecap: "round", strokeLinejoin: "round" },
                                    React.createElement('path', { d: "M3 3v5h5" }),
                                    React.createElement('path', { d: "M3.05 13A9 9 0 1 0 12 3" }),
                                    React.createElement('path', { d: "M12 7v5l2 2" })
                                )
                            ),
                            React.createElement('div', { className: "w-8" }) // Balans uchun bo'sh div
                        ),
                        React.createElement('div', { className: "flex justify-center mb-4" },
                            React.createElement('div', { className: "bg-white rounded-2xl p-4 shadow-xl" },
                                React.createElement('svg', { width: "200", height: "80", viewBox: "0 0 280 100", className: "w-full h-auto" },
                                    React.createElement('g', { transform: "translate(10, 20)" },
                                        React.createElement('circle', { cx: "8", cy: "8", r: "6", fill: "#10b981" }),
                                        React.createElement('path', { 
                                            d: "M 8 15 L 8 35 M 3 22 L 13 22 M 3 35 L 8 35 M 13 35 L 8 35", 
                                            stroke: "#10b981", 
                                            strokeWidth: "2.5", 
                                            fill: "none", 
                                            strokeLinecap: "round" 
                                        })
                                    ),
                                    React.createElement('g', { transform: "translate(25, 20)" },
                                        React.createElement('circle', { cx: "8", cy: "8", r: "6", fill: "#059669" }),
                                        React.createElement('path', { 
                                            d: "M 8 15 L 8 35 M 3 22 L 13 22 M 3 35 L 8 35 M 13 35 L 8 35", 
                                            stroke: "#059669", 
                                            strokeWidth: "2.5", 
                                            fill: "none", 
                                            strokeLinecap: "round" 
                                        })
                                    ),
                                    React.createElement('g', { transform: "translate(40, 20)" },
                                        React.createElement('circle', { cx: "8", cy: "8", r: "6", fill: "#047857" }),
                                        React.createElement('path', { 
                                            d: "M 8 15 L 8 35 M 3 22 L 13 22 M 3 35 L 8 35 M 13 35 L 8 35", 
                                            stroke: "#047857", 
                                            strokeWidth: "2.5", 
                                            fill: "none", 
                                            strokeLinecap: "round" 
                                        })
                                    ),
                                    React.createElement('text', { x: "65", y: "40", fontFamily: "Arial, sans-serif", fontSize: "26", fontWeight: "bold", fill: "#dc2626" }, "Avantika"),
                                    React.createElement('text', { x: "170", y: "40", fontFamily: "Arial, sans-serif", fontSize: "26", fontWeight: "bold", fill: "#059669" }, "Shifokor"),
                                    React.createElement('rect', { x: "65", y: "50", width: "200", height: "22", rx: "11", fill: "#dc2626" }),
                                    React.createElement('text', { x: "165", y: "65", fontFamily: "Arial, sans-serif", fontSize: "11", fontWeight: "600", fill: "white", textAnchor: "middle" }, "Committed towards good health")
                                )
                            )
                        ),
                        React.createElement('div', { className: "text-center" },
                            React.createElement('h1', { className: "text-2xl font-bold text-white flex items-center justify-center gap-2 mb-2" },
                                React.createElement('svg', { xmlns: "http://www.w3.org/2000/svg", width: "28", height: "28", viewBox: "0 0 24 24", fill: "none", stroke: "currentColor", strokeWidth: "2", strokeLinecap: "round", strokeLinejoin: "round" },
                                    React.createElement('path', { d: "M14.5 4h-5L7 7H4a2 2 0 0 0-2 2v9a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V9a2 2 0 0 0-2-2h-3l-2.5-3z" }),
                                    React.createElement('circle', { cx: "12", cy: "13", r: "3" })
                                ),
                                "QR Code Skaneri"
                            ),
                            React.createElement('p', { className: "text-emerald-100 text-sm" }, "Tez va oson QR code o'qish")
                        )
                    ),
                    React.createElement('div', { className: "flex-1 p-4 space-y-4" },
                        !cameraPermission && React.createElement('div', { className: "bg-yellow-50 border-l-4 border-yellow-500 rounded-lg p-3 flex items-start gap-2" },
                            React.createElement('svg', { xmlns: "http://www.w3.org/2000/svg", width: "18", height: "18", viewBox: "0 0 24 24", fill: "none", stroke: "currentColor", strokeWidth: "2", strokeLinecap: "round", strokeLinejoin: "round", className: "text-yellow-600 flex-shrink-0 mt-0.5" },
                                React.createElement('circle', { cx: "12", cy: "12", r: "10" }),
                                React.createElement('line', { x1: "12", y1: "8", x2: "12", y2: "12" }),
                                React.createElement('line', { x1: "12", y1: "16", x2: "12.01", y2: "16" })
                            ),
                            React.createElement('p', { className: "text-yellow-800 text-sm" }, "Kameraga ruxsat berilmagan. Skanerlash uchun kameraga ruxsat bering.")
                        ),
                        error && React.createElement('div', { className: "bg-red-50 border-l-4 border-red-500 rounded-lg p-3 flex items-start gap-2 animate-pulse" },
                            React.createElement('svg', { xmlns: "http://www.w3.org/2000/svg", width: "18", height: "18", viewBox: "0 0 24 24", fill: "none", stroke: "currentColor", strokeWidth: "2", strokeLinecap: "round", strokeLinejoin: "round", className: "text-red-600 flex-shrink-0 mt-0.5" },
                                React.createElement('circle', { cx: "12", cy: "12", r: "10" }),
                                React.createElement('line', { x1: "12", y1: "8", x2: "12", y2: "12" }),
                                React.createElement('line', { x1: "12", y1: "16", x2: "12.01", y2: "16" })
                            ),
                            React.createElement('p', { className: "text-red-800 text-sm" }, error)
                        ),
                        success && React.createElement('div', { className: "bg-green-50 border-l-4 border-green-500 rounded-lg p-3 flex items-start gap-2 animate-pulse" },
                            React.createElement('svg', { xmlns: "http://www.w3.org/2000/svg", width: "18", height: "18", viewBox: "0 0 24 24", fill: "none", stroke: "currentColor", strokeWidth: "2", strokeLinecap: "round", strokeLinejoin: "round", className: "text-green-600 flex-shrink-0 mt-0.5" },
                                React.createElement('path', { d: "M22 11.08V12a10 10 0 1 1-5.93-9.14" }),
                                React.createElement('polyline', { points: "22 4 12 14.01 9 11.01" })
                            ),
                            React.createElement('p', { className: "text-green-800 text-sm" }, success)
                        ),
                        React.createElement('div', { className: "relative bg-white rounded-2xl shadow-xl overflow-hidden" },
                            React.createElement('div', {
                                id: "reader",
                                ref: readerRef,
                                className: `w-full h-64 ${scanning ? 'block' : 'hidden'}`
                            }),
                            scanning && React.createElement('div', { className: "scanner-overlay" },
                                React.createElement('div', { className: "scanner-frame" },
                                    React.createElement('div', { className: "scanner-line" })
                                )
                            ),
                            !scanning && !result && React.createElement('div', { className: "h-64 bg-gradient-to-br from-gray-100 to-gray-200 flex flex-col items-center justify-center p-6" },
                                React.createElement('div', { className: "bg-white rounded-full p-6 shadow-lg mb-4" },
                                    React.createElement('svg', { xmlns: "http://www.w3.org/2000/svg", width: "48", height: "48", viewBox: "0 0 24 24", fill: "none", stroke: "currentColor", strokeWidth: "2", strokeLinecap: "round", strokeLinejoin: "round", className: "text-emerald-600" },
                                        React.createElement('path', { d: "M14.5 4h-5L7 7H4a2 2 0 0 0-2 2v9a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V9a2 2 0 0 0-2-2h-3l-2.5-3z" }),
                                        React.createElement('circle', { cx: "12", cy: "13", r: "3" })
                                    )
                                ),
                                React.createElement('p', { className: "text-gray-700 text-center font-medium" }, "QR code skanerlash uchun\nkamerani yoqing")
                            )
                        ),
                        scanning && React.createElement('div', { className: "bg-white rounded-2xl shadow-xl p-4" },
                            React.createElement('h3', { className: "font-bold text-gray-900 text-lg mb-3" }, "Kamera Sozlamalari"),
                            React.createElement('div', { className: "space-y-4" },
                                React.createElement('div', null,
                                    React.createElement('div', { className: "flex justify-between mb-1" },
                                        React.createElement('span', { className: "text-sm font-medium text-gray-700" }, "Zoom: " + Math.round(zoomLevel * 100) + "%" ),
                                        React.createElement('button', {
                                            onClick: () => setZoomLevel(1),
                                            className: "text-xs text-emerald-600 font-semibold"
                                        }, "Asl holat")
                                    ),
                                    React.createElement('input', {
                                        type: "range",
                                        min: "1",
                                        max: "3",
                                        step: "0.1",
                                        value: zoomLevel,
                                        onChange: (e) => setZoomLevel(parseFloat(e.target.value)),
                                        className: "w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer"
                                    })
                                ),
                                React.createElement('div', null,
                                    React.createElement('div', { className: "flex justify-between mb-1" },
                                        React.createElement('span', { className: "text-sm font-medium text-gray-700" }, "Yorqinlik: " + brightness + "%" ),
                                        React.createElement('button', {
                                            onClick: () => setBrightness(100),
                                            className: "text-xs text-emerald-600 font-semibold"
                                        }, "Asl holat")
                                    ),
                                    React.createElement('input', {
                                        type: "range",
                                        min: "50",
                                        max: "200",
                                        step: "5",
                                        value: brightness,
                                        onChange: (e) => setBrightness(parseInt(e.target.value)),
                                        className: "w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer"
                                    })
                                ),
                                React.createElement('div', null,
                                    React.createElement('div', { className: "flex justify-between mb-1" },
                                        React.createElement('span', { className: "text-sm font-medium text-gray-700" }, "Kontrast: " + contrast + "%" ),
                                        React.createElement('button', {
                                            onClick: () => setContrast(100),
                                            className: "text-xs text-emerald-600 font-semibold"
                                        }, "Asl holat")
                                    ),
                                    React.createElement('input', {
                                        type: "range",
                                        min: "50",
                                        max: "200",
                                        step: "5",
                                        value: contrast,
                                        onChange: (e) => setContrast(parseInt(e.target.value)),
                                        className: "w-full h-2 bg-gray-200 rounded-lg appearance-none cursor-pointer"
                                    })
                                )
                            )
                        ),
                        !scanning && !result && React.createElement('button', {
                            onClick: startScanning,
                            disabled: !cameraPermission,
                            className: `w-full ${cameraPermission ? 'bg-gradient-to-r from-emerald-600 to-teal-600 hover:from-emerald-700 hover:to-teal-700' : 'bg-gray-400'} text-white py-4 rounded-xl font-bold transition-all shadow-lg hover:shadow-xl active:scale-95 flex items-center justify-center gap-2 text-lg`
                        },
                            React.createElement('svg', { xmlns: "http://www.w3.org/2000/svg", width: "24", height: "24", viewBox: "0 0 24 24", fill: "none", stroke: "currentColor", strokeWidth: "2", strokeLinecap: "round", strokeLinejoin: "round" },
                                React.createElement('path', { d: "M14.5 4h-5L7 7H4a2 2 0 0 0-2 2v9a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V9a2 2 0 0 0-2-2h-3l-2.5-3z" }),
                                React.createElement('circle', { cx: "12", cy: "13", r: "3" })
                            ),
                            cameraPermission ? "Skanerlashni Boshlash" : "Kameraga Ruxsat Kerak"
                        ),
                        scanning && React.createElement('button', {
                            onClick: stopScanning,
                            className: "w-full bg-red-600 text-white py-4 rounded-xl font-bold hover:bg-red-700 transition-all shadow-lg hover:shadow-xl active:scale-95 flex items-center justify-center gap-2 text-lg"
                        },
                            React.createElement('svg', { xmlns: "http://www.w3.org/2000/svg", width: "24", height: "24", viewBox: "0 0 24 24", fill: "none", stroke: "currentColor", strokeWidth: "2", strokeLinecap: "round", strokeLinejoin: "round" },
                                React.createElement('line', { x1: "18", y1: "6", x2: "6", y2: "18" }),
                                React.createElement('line', { x1: "6", y1: "6", x2: "18", y2: "18" })
                            ),
                            "To'xtatish"
                        ),
                        result && React.createElement('div', { className: "space-y-4" },
                            React.createElement('div', { className: "bg-white rounded-2xl shadow-xl p-4 border-2 border-emerald-200" },
                                React.createElement('div', { className: "flex items-center justify-between mb-3" },
                                    React.createElement('div', { className: "flex items-center gap-2" },
                                        React.createElement('svg', { xmlns: "http://www.w3.org/2000/svg", width: "24", height: "24", viewBox: "0 0 24 24", fill: "none", stroke: "currentColor", strokeWidth: "2", strokeLinecap: "round", strokeLinejoin: "round", className: "text-emerald-600" },
                                            React.createElement('path', { d: "M22 11.08V12a10 10 0 1 1-5.93-9.14" }),
                                            React.createElement('polyline', { points: "22 4 12 14.01 9 11.01" })
                                        ),
                                        React.createElement('h3', { className: "font-bold text-gray-900 text-lg" }, "Natija topildi!")
                                    ),
                                    React.createElement('button', {
                                        onClick: () => copyToClipboard(result),
                                        className: "text-xs bg-emerald-100 text-emerald-700 px-3 py-1 rounded-lg font-semibold"
                                    }, "📋 Nusxalash")
                                ),
                                React.createElement('div', { className: "bg-emerald-50 rounded-lg p-3 border border-emerald-200" },
                                    React.createElement('p', { className: "text-emerald-900 break-all font-mono text-sm" }, result)
                                )
                            ),
                            React.createElement('button', {
                                onClick: () => {
                                    setResult('');
                                    setError('');
                                    setSuccess('');
                                },
                                className: "w-full bg-gray-600 text-white py-3 rounded-xl font-semibold hover:bg-gray-700 transition-all active:scale-95 flex items-center justify-center gap-2"
                            },
                                React.createElement('svg', { xmlns: "http://www.w3.org/2000/svg", width: "20", height: "20", viewBox: "0 0 24 24", fill: "none", stroke: "currentColor", strokeWidth: "2", strokeLinecap: "round", strokeLinejoin: "round" },
                                    React.createElement('path', { d: "M14.5 4h-5L7 7H4a2 2 0 0 0-2 2v9a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V9a2 2 0 0 0-2-2h-3l-2.5-3z" }),
                                    React.createElement('circle', { cx: "12", cy: "13", r: "3" })
                                ),
                                "Yangi Skaner"
                            )
                        )
                    ),
                    React.createElement('div', { className: "bg-gradient-to-r from-emerald-600 to-teal-600 px-4 py-4 text-center shadow-lg" },
                        React.createElement('p', { className: "text-white font-semibold text-sm leading-relaxed" },
                            "❤️ Avantika Shifokor hamisha sizni g'amxo'rlik qiladi\nHech qachon kasal bo'lmang!"
                        )
                    )
                )
            );
        }

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(React.createElement(QRScanner));
    </script>
</body>
</html>
