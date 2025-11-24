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
            const [botToken, setBotToken] = useState('');
            const [chatId, setChatId] = useState('');
            const [scanHistory, setScanHistory] = useState([]);
            const [showSettings, setShowSettings] = useState(false);
            const [showHistory, setShowHistory] = useState(false);
            const [zoomLevel, setZoomLevel] = useState(1);
            const [brightness, setBrightness] = useState(100);
            const [contrast, setContrast] = useState(100);
            const videoRef = useRef(null);
            const canvasRef = useRef(null);
            const streamRef = useRef(null);
            const html5QrCodeRef = useRef(null);

            // Sozlamalarni yuklash
            useEffect(() => {
                const savedToken = localStorage.getItem('botToken') || '';
                const savedChatId = localStorage.getItem('chatId') || '';
                const savedHistory = JSON.parse(localStorage.getItem('scanHistory') || '[]');
                
                setBotToken(savedToken);
                setChatId(savedChatId);
                setScanHistory(savedHistory);
            }, []);

            // Sozlamalarni saqlash
            const saveSettings = (token, chat) => {
                setBotToken(token);
                setChatId(chat);
                localStorage.setItem('botToken', token);
                localStorage.setItem('chatId', chat);
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
                        focusMode: "continuous"
                    };
                    
                    await html5QrCodeRef.current.start(
                        { facingMode: "environment" },
                        config,
                        onScanSuccess,
                        onScanFailure
                    );
                    
                    setScanning(true);
                    
                } catch (err) {
                    setError('Kameraga kirish xato: ' + err.message);
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

            const sendToBot = async () => {
                if (!botToken || !chatId || !result) {
                    setError('Iltimos, barcha maydonlarni to\'ldiring');
                    return;
                }

                try {
                    setError('');
                    setSuccess('');
                    
                    const response = await fetch(`https://api.telegram.org/bot${botToken}/sendMessage`, {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                        },
                        body: JSON.stringify({
                            chat_id: chatId,
                            text: `🔍 QR Code Natijasi\n━━━━━━━━━━━━━━━\n\n${result}\n\n📅 Vaqt: ${new Date().toLocaleString('uz-UZ')}\n\n❤️ Avantika Shifokor`,
                            parse_mode: 'HTML'
                        })
                    });

                    const data = await response.json();

                    if (data.ok) {
                        setSuccess('✅ Xabar muvaffaqiyatli yuborildi!');
                        setTimeout(() => setSuccess(''), 3000);
                    } else {
                        setError('Xatolik: ' + data.description);
                    }
                } catch (err) {
                    setError('Yuborishda xatolik: ' + err.message);
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
                                    "Tarix"
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
                                    React.createElement('p', { className: "text-gray-600" }, "Hozircha tarix bo'sh")
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
                                            className: "text-xs text-emerald-600 font-semibold" 
                                        }, "📋 Nusxalash")
                                    )
                                )
                        )
                    )
                );
            }

            if (showSettings) {
                return React.createElement('div', { className: "min-h-screen bg-gradient-to-br from-emerald-50 via-teal-50 to-green-50" },
                    React.createElement('div', { className: "max-w-md mx-auto min-h-screen flex flex-col" },
                        React.createElement('div', { className: "bg-gradient-to-r from-emerald-600 to-teal-600 px-4 py-6 shadow-lg" },
                            React.createElement('div', { className: "flex items-center gap-3" },
                                React.createElement('button', { onClick: () => setShowSettings(false), className: "text-white" },
                                    React.createElement('svg', { xmlns: "http://www.w3.org/2000/svg", width: "24", height: "24", viewBox: "0 0 24 24", fill: "none", stroke: "currentColor", strokeWidth: "2", strokeLinecap: "round", strokeLinejoin: "round" },
                                        React.createElement('line', { x1: "18", y1: "6", x2: "6", y2: "18" }),
                                        React.createElement('line', { x1: "6", y1: "6", x2: "18", y2: "18" })
                                    )
                                ),
                                React.createElement('h1', { className: "text-2xl font-bold text-white flex items-center gap-2" },
                                    React.createElement('svg', { xmlns: "http://www.w3.org/2000/svg", width: "28", height: "28", viewBox: "0 0 24 24", fill: "none", stroke: "currentColor", strokeWidth: "2", strokeLinecap: "round", strokeLinejoin: "round" },
                                        React.createElement('path', { d: "M12.22 2h-.44a2 2 0 0 0-2 2v.18a2 2 0 0 1-1 1.73l-.43.25a2 2 0 0 1-2 0l-.15-.08a2 2 0 0 0-2.73.73l-.22.38a2 2 0 0 0 .73 2.73l.15.1a2 2 0 0 1 1 1.72v.51a2 2 0 0 1-1 1.74l-.15.09a2 2 0 0 0-.73 2.73l.22.38a2 2 0 0 0 2.73.73l.15-.08a2 2 0 0 1 2 0l.43.25a2 2 0 0 1 1 1.73V20a2 2 0 0 0 2 2h.44a2 2 0 0 0 2-2v-.18a2 2 0 0 1 1-1.73l.43-.25a2 2 0 0 1 2 0l.15.08a2 2 0 0 0 2.73-.73l.22-.39a2 2 0 0 0-.73-2.73l-.15-.08a2 2 0 0 1-1-1.74v-.5a2 2 0 0 1 1-1.74l.15-.09a2 2 0 0 0 .73-2.73l-.22-.38a2 2 0 0 0-2.73-.73l-.15.08a2 2 0 0 1-2 0l-.43-.25a2 2 0 0 1-1-1.73V4a2 2 0 0 0-2-2z" }),
                                        React.createElement('circle', { cx: "12", cy: "12", r: "3" })
                                    ),
                                    "Sozlamalar"
                                )
                            )
                        ),
                        React.createElement('div', { className: "flex-1 p-4 space-y-4" },
                            React.createElement('div', { className: "bg-white rounded-2xl shadow-xl p-4" },
                                React.createElement('h3', { className: "font-bold text-gray-900 text-lg mb-4" }, "Bot Sozlamalari"),
                                React.createElement('div', { className: "space-y-4" },
                                    React.createElement('div', null,
                                        React.createElement('label', { className: "block text-sm font-semibold text-gray-700 mb-2" }, "Bot Token"),
                                        React.createElement('input', {
                                            type: "text",
                                            value: botToken,
                                            onChange: (e) => setBotToken(e.target.value),
                                            placeholder: "1234567890:ABC...",
                                            className: "w-full px-4 py-3 border-2 border-gray-200 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 text-sm"
                                        })
                                    ),
                                    React.createElement('div', null,
                                        React.createElement('label', { className: "block text-sm font-semibold text-gray-700 mb-2" }, "Chat ID"),
                                        React.createElement('input', {
                                            type: "text",
                                            value: chatId,
                                            onChange: (e) => setChatId(e.target.value),
                                            placeholder: "123456789",
                                            className: "w-full px-4 py-3 border-2 border-gray-200 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 text-sm"
                                        })
                                    ),
                                    React.createElement('button', {
                                        onClick: () => {
                                            saveSettings(botToken, chatId);
                                            setShowSettings(false);
                                            setSuccess('✅ Sozlamalar saqlandi!');
                                            setTimeout(() => setSuccess(''), 2000);
                                        },
                                        className: "w-full bg-gradient-to-r from-emerald-600 to-teal-600 text-white py-3 rounded-xl font-bold"
                                    }, "Saqlash")
                                )
                            ),
                            React.createElement('div', { className: "bg-blue-50 rounded-xl p-4 text-sm" },
                                React.createElement('h4', { className: "font-bold text-blue-900 mb-2" }, "Yo'riqnoma"),
                                React.createElement('ul', { className: "space-y-1 text-blue-800 text-xs" },
                                    React.createElement('li', null, "• Bot Token: @BotFather dan oling"),
                                    React.createElement('li', null, "• Chat ID: @userinfobot dan oling"),
                                    React.createElement('li', null, "• Sozlamalar avtomatik saqlanadi")
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
                            React.createElement('button', { onClick: () => setShowSettings(true), className: "text-white p-2" },
                                React.createElement('svg', { xmlns: "http://www.w3.org/2000/svg", width: "24", height: "24", viewBox: "0 0 24 24", fill: "none", stroke: "currentColor", strokeWidth: "2", strokeLinecap: "round", strokeLinejoin: "round" },
                                    React.createElement('path', { d: "M12.22 2h-.44a2 2 0 0 0-2 2v.18a2 2 0 0 1-1 1.73l-.43.25a2 2 0 0 1-2 0l-.15-.08a2 2 0 0 0-2.73.73l-.22.38a2 2 0 0 0 .73 2.73l.15.1a2 2 0 0 1 1 1.72v.51a2 2 0 0 1-1 1.74l-.15.09a2 2 0 0 0-.73 2.73l.22.38a2 2 0 0 0 2.73.73l.15-.08a2 2 0 0 1 2 0l.43.25a2 2 0 0 1 1 1.73V20a2 2 0 0 0 2 2h.44a2 2 0 0 0 2-2v-.18a2 2 0 0 1 1-1.73l.43-.25a2 2 0 0 1 2 0l.15.08a2 2 0 0 0 2.73-.73l.22-.39a2 2 0 0 0-.73-2.73l-.15-.08a2 2 0 0 1-1-1.74v-.5a2 2 0 0 1 1-1.74l.15-.09a2 2 0 0 0 .73-2.73l-.22-.38a2 2 0 0 0-2.73-.73l-.15.08a2 2 0 0 1-2 0l-.43-.25a2 2 0 0 1-1-1.73V4a2 2 0 0 0-2-2z" }),
                                    React.createElement('circle', { cx: "12", cy: "12", r: "3" })
                                )
                            )
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
                                className: `w-full h-64 ${scanning ? 'block' : 'hidden'}`
                            }),
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
                                        React.createElement('span', { className: "text-sm font-medium text-gray-700" }, "Zoom: {Math.round(zoomLevel * 100)}%" ),
                                        React.createElement('button', {
                                            onClick: () => setZoomLevel(1),
                                            className: "text-xs text-emerald-600 font-semibold"
                                        }, "Reset")
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
                                        React.createElement('span', { className: "text-sm font-medium text-gray-700" }, "Yorqinlik: {brightness}%" ),
                                        React.createElement('button', {
                                            onClick: () => setBrightness(100),
                                            className: "text-xs text-emerald-600 font-semibold"
                                        }, "Reset")
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
                                        React.createElement('span', { className: "text-sm font-medium text-gray-700" }, "Kontrast: {contrast}%" ),
                                        React.createElement('button', {
                                            onClick: () => setContrast(100),
                                            className: "text-xs text-emerald-600 font-semibold"
                                        }, "Reset")
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
                            className: "w-full bg-gradient-to-r from-emerald-600 to-teal-600 text-white py-4 rounded-xl font-bold hover:from-emerald-700 hover:to-teal-700 transition-all shadow-lg hover:shadow-xl active:scale-95 flex items-center justify-center gap-2 text-lg"
                        },
                            React.createElement('svg', { xmlns: "http://www.w3.org/2000/svg", width: "24", height: "24", viewBox: "0 0 24 24", fill: "none", stroke: "currentColor", strokeWidth: "2", strokeLinecap: "round", strokeLinejoin: "round" },
                                React.createElement('path', { d: "M14.5 4h-5L7 7H4a2 2 0 0 0-2 2v9a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V9a2 2 0 0 0-2-2h-3l-2.5-3z" }),
                                React.createElement('circle', { cx: "12", cy: "13", r: "3" })
                            ),
                            "Skanerlashni Boshlash"
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
                            botToken && chatId && React.createElement('button', {
                                onClick: sendToBot,
                                className: "w-full bg-gradient-to-r from-teal-600 to-emerald-600 text-white py-4 rounded-xl font-bold hover:from-teal-700 hover:to-emerald-700 transition-all shadow-lg hover:shadow-xl active:scale-95 flex items-center justify-center gap-2 text-lg"
                            },
                                React.createElement('svg', { xmlns: "http://www.w3.org/2000/svg", width: "24", height: "24", viewBox: "0 0 24 24", fill: "none", stroke: "currentColor", strokeWidth: "2", strokeLinecap: "round", strokeLinejoin: "round" },
                                    React.createElement('line', { x1: "22", y1: "2", x2: "11", y2: "13" }),
                                    React.createElement('polygon', { points: "22 2 15 22 11 13 2 9 22 2" })
                                ),
                                "Botga Yuborish"
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
