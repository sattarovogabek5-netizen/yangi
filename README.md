import React, { useState, useRef, useEffect } from 'react';
import { Camera, Send, AlertCircle, CheckCircle2, X, Smartphone } from 'lucide-react';

export default function QRScanner() {
  const [scanning, setScanning] = useState(false);
  const [result, setResult] = useState('');
  const [error, setError] = useState('');
  const [success, setSuccess] = useState('');
  const [botToken, setBotToken] = useState('');
  const [chatId, setChatId] = useState('');
  const videoRef = useRef(null);
  const canvasRef = useRef(null);
  const streamRef = useRef(null);
  const scanIntervalRef = useRef(null);

  // QR code o'qish uchun jsQR kutubxonasini yuklash
  useEffect(() => {
    const script = document.createElement('script');
    script.src = 'https://cdnjs.cloudflare.com/ajax/libs/jsQR/1.4.0/jsQR.min.js';
    script.async = true;
    document.body.appendChild(script);
    
    return () => {
      if (document.body.contains(script)) {
        document.body.removeChild(script);
      }
    };
  }, []);

  const startScanning = async () => {
    try {
      setError('');
      setResult('');
      setSuccess('');
      
      const stream = await navigator.mediaDevices.getUserMedia({
        video: { facingMode: 'environment' }
      });
      
      if (videoRef.current) {
        videoRef.current.srcObject = stream;
        streamRef.current = stream;
        setScanning(true);
        
        videoRef.current.onloadedmetadata = () => {
          videoRef.current.play();
          scanQRCode();
        };
      }
    } catch (err) {
      setError('Kameraga kirishda xatolik: ' + err.message);
    }
  };

  const stopScanning = () => {
    if (streamRef.current) {
      streamRef.current.getTracks().forEach(track => track.stop());
      streamRef.current = null;
    }
    if (scanIntervalRef.current) {
      clearInterval(scanIntervalRef.current);
      scanIntervalRef.current = null;
    }
    setScanning(false);
  };

  const scanQRCode = () => {
    if (scanIntervalRef.current) {
      clearInterval(scanIntervalRef.current);
    }

    scanIntervalRef.current = setInterval(() => {
      if (videoRef.current && canvasRef.current && window.jsQR) {
        const canvas = canvasRef.current;
        const video = videoRef.current;
        const context = canvas.getContext('2d');

        if (video.readyState === video.HAVE_ENOUGH_DATA) {
          canvas.width = video.videoWidth;
          canvas.height = video.videoHeight;
          context.drawImage(video, 0, 0, canvas.width, canvas.height);
          
          const imageData = context.getImageData(0, 0, canvas.width, canvas.height);
          const code = window.jsQR(imageData.data, imageData.width, imageData.height);

          if (code) {
            setResult(code.data);
            stopScanning();
          }
        }
      }
    }, 300);
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
          text: `🔍 QR Code natijasi:\n\n${result}`,
          parse_mode: 'HTML'
        })
      });

      const data = await response.json();

      if (data.ok) {
        setSuccess('Xabar muvaffaqiyatli yuborildi!');
        setTimeout(() => setSuccess(''), 3000);
      } else {
        setError('Xatolik: ' + data.description);
      }
    } catch (err) {
      setError('Yuborishda xatolik: ' + err.message);
    }
  };

  useEffect(() => {
    return () => {
      stopScanning();
    };
  }, []);

  return (
    <div className="min-h-screen bg-gradient-to-br from-emerald-50 via-teal-50 to-green-50">
      {/* Mobile optimized container */}
      <div className="max-w-md mx-auto min-h-screen flex flex-col">
        
        {/* Header with Logo */}
        <div className="bg-gradient-to-r from-emerald-600 via-teal-600 to-green-600 px-4 py-6 shadow-lg">
          {/* Logo */}
          <div className="flex justify-center mb-4">
            <div className="bg-white rounded-2xl p-4 shadow-xl">
              <svg width="200" height="80" viewBox="0 0 280 100" className="w-full h-auto">
                {/* Stylized figures */}
                <g transform="translate(10, 20)">
                  <circle cx="8" cy="8" r="6" fill="#10b981" />
                  <path d="M 8 15 L 8 35 M 3 22 L 13 22 M 3 35 L 8 35 M 13 35 L 8 35" 
                        stroke="#10b981" strokeWidth="2.5" fill="none" strokeLinecap="round"/>
                </g>
                <g transform="translate(25, 20)">
                  <circle cx="8" cy="8" r="6" fill="#059669" />
                  <path d="M 8 15 L 8 35 M 3 22 L 13 22 M 3 35 L 8 35 M 13 35 L 8 35" 
                        stroke="#059669" strokeWidth="2.5" fill="none" strokeLinecap="round"/>
                </g>
                <g transform="translate(40, 20)">
                  <circle cx="8" cy="8" r="6" fill="#047857" />
                  <path d="M 8 15 L 8 35 M 3 22 L 13 22 M 3 35 L 8 35 M 13 35 L 8 35" 
                        stroke="#047857" strokeWidth="2.5" fill="none" strokeLinecap="round"/>
                </g>
                
                {/* Avantika text */}
                <text x="65" y="40" fontFamily="Arial, sans-serif" fontSize="26" fontWeight="bold" fill="#dc2626">
                  Avantika
                </text>
                
                {/* Shifokor text */}
                <text x="170" y="40" fontFamily="Arial, sans-serif" fontSize="26" fontWeight="bold" fill="#059669">
                  Shifokor
                </text>
                
                {/* Tagline */}
                <rect x="65" y="50" width="200" height="22" rx="11" fill="#dc2626"/>
                <text x="165" y="65" fontFamily="Arial, sans-serif" fontSize="11" fontWeight="600" fill="white" textAnchor="middle">
                  Committed towards good health
                </text>
              </svg>
            </div>
          </div>

          {/* Title */}
          <div className="text-center">
            <h1 className="text-2xl font-bold text-white flex items-center justify-center gap-2 mb-2">
              <Camera size={28} />
              QR Code Skaneri
            </h1>
            <p className="text-emerald-100 text-sm">Tez va oson QR code o'qish</p>
          </div>
        </div>

        {/* Main Content */}
        <div className="flex-1 p-4 space-y-4">
          
          {/* Xatolik xabari */}
          {error && (
            <div className="bg-red-50 border-l-4 border-red-500 rounded-lg p-3 flex items-start gap-2 animate-pulse">
              <AlertCircle className="text-red-600 flex-shrink-0 mt-0.5" size={18} />
              <p className="text-red-800 text-sm">{error}</p>
            </div>
          )}

          {/* Muvaffaqiyat xabari */}
          {success && (
            <div className="bg-green-50 border-l-4 border-green-500 rounded-lg p-3 flex items-start gap-2 animate-pulse">
              <CheckCircle2 className="text-green-600 flex-shrink-0 mt-0.5" size={18} />
              <p className="text-green-800 text-sm">{success}</p>
            </div>
          )}

          {/* Video va Canvas - Mobile optimized */}
          <div className="relative bg-white rounded-2xl shadow-xl overflow-hidden">
            <video
              ref={videoRef}
              className={`w-full h-64 object-cover ${scanning ? 'block' : 'hidden'}`}
              playsInline
              autoPlay
            />
            <canvas ref={canvasRef} className="hidden" />
            
            {!scanning && !result && (
              <div className="h-64 bg-gradient-to-br from-gray-100 to-gray-200 flex flex-col items-center justify-center p-6">
                <div className="bg-white rounded-full p-6 shadow-lg mb-4">
                  <Camera size={48} className="text-emerald-600" />
                </div>
                <p className="text-gray-700 text-center font-medium">QR code skanerlash uchun<br/>kamerani yoqing</p>
              </div>
            )}

            {/* Scanning overlay */}
            {scanning && (
              <div className="absolute inset-0 pointer-events-none">
                <div className="absolute inset-0 border-4 border-emerald-500 animate-pulse"></div>
                <div className="absolute top-1/2 left-1/2 transform -translate-x-1/2 -translate-y-1/2 w-48 h-48 border-4 border-red-500 rounded-lg"></div>
              </div>
            )}
          </div>

          {/* Skaner tugmasi */}
          {!scanning && !result && (
            <button
              onClick={startScanning}
              className="w-full bg-gradient-to-r from-emerald-600 to-teal-600 text-white py-4 rounded-xl font-bold hover:from-emerald-700 hover:to-teal-700 transition-all shadow-lg hover:shadow-xl active:scale-95 flex items-center justify-center gap-2 text-lg"
            >
              <Camera size={24} />
              Skanerlashni Boshlash
            </button>
          )}

          {/* To'xtatish tugmasi */}
          {scanning && (
            <button
              onClick={stopScanning}
              className="w-full bg-red-600 text-white py-4 rounded-xl font-bold hover:bg-red-700 transition-all shadow-lg hover:shadow-xl active:scale-95 flex items-center justify-center gap-2 text-lg"
            >
              <X size={24} />
              To'xtatish
            </button>
          )}

          {/* Natija */}
          {result && (
            <div className="space-y-4">
              <div className="bg-white rounded-2xl shadow-xl p-4 border-2 border-emerald-200">
                <div className="flex items-center gap-2 mb-3">
                  <CheckCircle2 className="text-emerald-600" size={24} />
                  <h3 className="font-bold text-gray-900 text-lg">Natija topildi!</h3>
                </div>
                <div className="bg-emerald-50 rounded-lg p-3 border border-emerald-200">
                  <p className="text-emerald-900 break-all font-mono text-sm">{result}</p>
                </div>
              </div>

              {/* Bot sozlamalari - Mobile optimized */}
              <div className="space-y-3 bg-white rounded-2xl shadow-xl p-4">
                <h3 className="font-bold text-gray-900 text-lg flex items-center gap-2">
                  <Send size={20} className="text-teal-600" />
                  Telegram Bot
                </h3>
                
                <div>
                  <label className="block text-sm font-semibold text-gray-700 mb-2">
                    Bot Token
                  </label>
                  <input
                    type="text"
                    value={botToken}
                    onChange={(e) => setBotToken(e.target.value)}
                    placeholder="1234567890:ABC..."
                    className="w-full px-4 py-3 border-2 border-gray-200 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 text-sm"
                  />
                </div>

                <div>
                  <label className="block text-sm font-semibold text-gray-700 mb-2">
                    Chat ID
                  </label>
                  <input
                    type="text"
                    value={chatId}
                    onChange={(e) => setChatId(e.target.value)}
                    placeholder="123456789"
                    className="w-full px-4 py-3 border-2 border-gray-200 rounded-xl focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 text-sm"
                  />
                </div>

                <button
                  onClick={sendToBot}
                  className="w-full bg-gradient-to-r from-teal-600 to-emerald-600 text-white py-4 rounded-xl font-bold hover:from-teal-700 hover:to-emerald-700 transition-all shadow-lg hover:shadow-xl active:scale-95 flex items-center justify-center gap-2 text-lg"
                >
                  <Send size={24} />
                  Botga Yuborish
                </button>
              </div>

              <button
                onClick={() => {
                  setResult('');
                  setError('');
                  setSuccess('');
                }}
                className="w-full bg-gray-600 text-white py-3 rounded-xl font-semibold hover:bg-gray-700 transition-all active:scale-95 flex items-center justify-center gap-2"
              >
                <Camera size={20} />
                Yangi Skaner
              </button>
            </div>
          )}

          {/* Yo'riqnoma - Mobile friendly */}
          <div className="bg-gradient-to-br from-blue-50 to-indigo-50 rounded-2xl p-4 text-sm border-2 border-blue-200">
            <h4 className="font-bold text-blue-900 mb-3 flex items-center gap-2">
              <Smartphone size={18} />
              Qo'llanma
            </h4>
            <ol className="list-decimal list-inside space-y-2 text-blue-800">
              <li className="leading-relaxed">Kamerani yoqing va QR codeni ko'rsating</li>
              <li className="leading-relaxed">Bot token va Chat ID ni kiriting</li>
              <li className="leading-relaxed">Natijani botga yuboring</li>
            </ol>
            <div className="mt-3 pt-3 border-t border-blue-200 text-xs text-blue-700 space-y-1">
              <p><strong>Bot token:</strong> @BotFather</p>
              <p><strong>Chat ID:</strong> @userinfobot</p>
            </div>
          </div>
        </div>

        {/* Footer */}
        <div className="bg-gradient-to-r from-emerald-600 to-teal-600 px-4 py-4 text-center shadow-lg">
          <p className="text-white font-semibold text-sm leading-relaxed">
            ❤️ Avantika Shifokor hamisha sizni g'amxo'rlik qiladi<br/>
            Hech qachon kasal bo'lmang!
          </p>
        </div>
      </div>
    </div>
  );
}
