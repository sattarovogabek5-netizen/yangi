import React, { useState, useEffect, useRef } from 'react';
import { Camera, Upload, X, CheckCircle, Scan } from 'lucide-react';

export default function DataMatrixScanner() {
  const [scanning, setScanning] = useState(false);
  const [result, setResult] = useState('');
  const [error, setError] = useState('');
  const [codeType, setCodeType] = useState('');
  const videoRef = useRef(null);
  const canvasRef = useRef(null);
  const streamRef = useRef(null);
  const scanIntervalRef = useRef(null);

  const loadZXing = async () => {
    if (!window.ZXing) {
      const script = document.createElement('script');
      script.src = 'https://cdnjs.cloudflare.com/ajax/libs/zxing-library/0.19.1/zxing.min.js';
      document.head.appendChild(script);
      await new Promise((resolve) => {
        script.onload = resolve;
      });
    }
  };

  const startCamera = async () => {
    try {
      await loadZXing();
      const stream = await navigator.mediaDevices.getUserMedia({
        video: { 
          facingMode: 'environment',
          width: { ideal: 1280 },
          height: { ideal: 720 }
        }
      });
      
      if (videoRef.current) {
        videoRef.current.srcObject = stream;
        streamRef.current = stream;
        setScanning(true);
        setError('');
        
        videoRef.current.onloadedmetadata = () => {
          videoRef.current.play();
          scanCode();
        };
      }
    } catch (err) {
      setError('Kameraga kirish imkoni yo\'q. Iltimos, ruxsat bering.');
      console.error('Camera error:', err);
    }
  };

  const stopCamera = () => {
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

  const scanCode = async () => {
    if (scanIntervalRef.current) {
      clearInterval(scanIntervalRef.current);
    }

    const codeReader = new window.ZXing.BrowserMultiFormatReader();

    scanIntervalRef.current = setInterval(async () => {
      if (videoRef.current && canvasRef.current && videoRef.current.readyState === videoRef.current.HAVE_ENOUGH_DATA) {
        try {
          const canvas = canvasRef.current;
          const video = videoRef.current;
          const context = canvas.getContext('2d');

          canvas.width = video.videoWidth;
          canvas.height = video.videoHeight;
          context.drawImage(video, 0, 0, canvas.width, canvas.height);

          const imageData = context.getImageData(0, 0, canvas.width, canvas.height);
          const code = await codeReader.decodeFromImageData(imageData);

          if (code) {
            setResult(code.text);
            setCodeType(code.format);
            stopCamera();
          }
        } catch (err) {
          // Kod topilmadi, davom etamiz
        }
      }
    }, 200);
  };

  const handleFileUpload = async (e) => {
    const file = e.target.files[0];
    if (!file) return;

    try {
      await loadZXing();
      const reader = new FileReader();
      
      reader.onload = async (event) => {
        const img = new Image();
        img.onload = async () => {
          const canvas = canvasRef.current;
          const context = canvas.getContext('2d');
          
          canvas.width = img.width;
          canvas.height = img.height;
          context.drawImage(img, 0, 0);
          
          const imageData = context.getImageData(0, 0, canvas.width, canvas.height);
          const codeReader = new window.ZXing.BrowserMultiFormatReader();
          
          try {
            const code = await codeReader.decodeFromImageData(imageData);
            setResult(code.text);
            setCodeType(code.format);
            setError('');
          } catch (err) {
            setError('Kod topilmadi. Iltimos, aniqroq rasm yuklang.');
          }
        };
        img.src = event.target.result;
      };
      
      reader.readAsDataURL(file);
    } catch (err) {
      setError('Rasmni o\'qishda xatolik yuz berdi.');
      console.error('File upload error:', err);
    }
  };

  const resetScanner = () => {
    setResult('');
    setError('');
    setCodeType('');
  };

  const copyToClipboard = () => {
    navigator.clipboard.writeText(result);
    alert('Nusxa olindi!');
  };

  useEffect(() => {
    return () => {
      stopCamera();
    };
  }, []);

  const getCodeTypeLabel = (format) => {
    const labels = {
      'QR_CODE': 'QR Kod',
      'DATA_MATRIX': 'DataMatrix',
      'CODE_128': 'Code 128',
      'CODE_39': 'Code 39',
      'EAN_13': 'EAN-13',
      'EAN_8': 'EAN-8',
      'UPC_A': 'UPC-A',
      'UPC_E': 'UPC-E',
      'PDF_417': 'PDF417',
      'AZTEC': 'Aztec'
    };
    return labels[format] || format;
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-violet-50 via-purple-50 to-indigo-100 p-4">
      <div className="max-w-2xl mx-auto">
        <div className="bg-white rounded-3xl shadow-2xl p-6 mb-6">
          <div className="flex items-center justify-center mb-2">
            <Scan className="text-violet-600 mr-3" size={36} />
            <h1 className="text-3xl font-bold text-gray-800">
              Universal Skaner
            </h1>
          </div>
          <p className="text-gray-600 text-center mb-2">
            DataMatrix, QR va boshqa kodlarni skanerlash
          </p>
          <div className="flex flex-wrap justify-center gap-2 mb-6">
            <span className="bg-violet-100 text-violet-700 text-xs font-semibold px-3 py-1 rounded-full">DataMatrix</span>
            <span className="bg-indigo-100 text-indigo-700 text-xs font-semibold px-3 py-1 rounded-full">QR Code</span>
            <span className="bg-purple-100 text-purple-700 text-xs font-semibold px-3 py-1 rounded-full">Barcode</span>
            <span className="bg-pink-100 text-pink-700 text-xs font-semibold px-3 py-1 rounded-full">PDF417</span>
          </div>

          {!result && !scanning && (
            <div className="space-y-4">
              <button
                onClick={startCamera}
                className="w-full bg-gradient-to-r from-violet-600 to-indigo-600 hover:from-violet-700 hover:to-indigo-700 text-white font-semibold py-5 px-6 rounded-2xl flex items-center justify-center gap-3 transition-all shadow-lg hover:shadow-xl transform hover:scale-105"
              >
                <Camera size={28} />
                <span className="text-lg">Kamerani yoqish</span>
              </button>

              <div className="relative">
                <input
                  type="file"
                  accept="image/*"
                  onChange={handleFileUpload}
                  className="hidden"
                  id="file-upload"
                />
                <label
                  htmlFor="file-upload"
                  className="w-full bg-gradient-to-r from-gray-100 to-gray-200 hover:from-gray-200 hover:to-gray-300 text-gray-700 font-semibold py-5 px-6 rounded-2xl flex items-center justify-center gap-3 transition-all cursor-pointer shadow hover:shadow-md"
                >
                  <Upload size={28} />
                  <span className="text-lg">Rasm yuklash</span>
                </label>
              </div>
            </div>
          )}

          {scanning && (
            <div className="space-y-4">
              <div className="relative bg-black rounded-2xl overflow-hidden shadow-2xl">
                <video
                  ref={videoRef}
                  className="w-full h-auto"
                  playsInline
                  muted
                />
                <div className="absolute inset-0 border-4 border-violet-500 m-8 rounded-2xl pointer-events-none animate-pulse"></div>
                <div className="absolute top-4 left-4 bg-violet-600 text-white px-4 py-2 rounded-full text-sm font-semibold shadow-lg">
                  Skanerlash...
                </div>
              </div>
              <button
                onClick={stopCamera}
                className="w-full bg-gradient-to-r from-red-600 to-red-700 hover:from-red-700 hover:to-red-800 text-white font-semibold py-4 px-6 rounded-2xl flex items-center justify-center gap-2 transition-all shadow-lg"
              >
                <X size={24} />
                To'xtatish
              </button>
            </div>
          )}

          {result && (
            <div className="space-y-4 animate-fadeIn">
              <div className="bg-gradient-to-br from-green-50 to-emerald-50 border-2 border-green-500 rounded-2xl p-6 shadow-lg">
                <div className="flex items-center gap-2 mb-3">
                  <CheckCircle className="text-green-600" size={28} />
                  <div>
                    <h2 className="text-xl font-bold text-green-800">
                      Muvaffaqiyatli o'qildi!
                    </h2>
                    {codeType && (
                      <p className="text-green-700 text-sm font-semibold">
                        Turi: {getCodeTypeLabel(codeType)}
                      </p>
                    )}
                  </div>
                </div>
                <div className="bg-white rounded-xl p-5 break-all shadow-inner">
                  <p className="text-gray-800 font-mono text-base leading-relaxed">{result}</p>
                </div>
              </div>
              
              <div className="grid grid-cols-2 gap-3">
                <button
                  onClick={copyToClipboard}
                  className="bg-blue-600 hover:bg-blue-700 text-white font-semibold py-3 px-4 rounded-xl transition-all shadow-md hover:shadow-lg"
                >
                  Nusxa olish
                </button>
                <button
                  onClick={resetScanner}
                  className="bg-gradient-to-r from-violet-600 to-indigo-600 hover:from-violet-700 hover:to-indigo-700 text-white font-semibold py-3 px-4 rounded-xl transition-all shadow-md hover:shadow-lg"
                >
                  Yana skanerlash
                </button>
              </div>
            </div>
          )}

          {error && (
            <div className="bg-gradient-to-r from-red-50 to-pink-50 border-2 border-red-500 rounded-2xl p-5 mt-4 shadow-lg">
              <p className="text-red-800 font-semibold">{error}</p>
            </div>
          )}
        </div>

        <div className="bg-white bg-opacity-80 backdrop-blur rounded-2xl shadow-lg p-5">
          <h3 className="font-bold text-gray-800 mb-3 text-center">Qo'llab-quvvatlanadigan kodlar</h3>
          <div className="grid grid-cols-2 gap-3 text-sm text-gray-700">
            <div className="flex items-center gap-2">
              <div className="w-2 h-2 bg-violet-500 rounded-full"></div>
              <span>DataMatrix</span>
            </div>
            <div className="flex items-center gap-2">
              <div className="w-2 h-2 bg-indigo-500 rounded-full"></div>
              <span>QR Code</span>
            </div>
            <div className="flex items-center gap-2">
              <div className="w-2 h-2 bg-purple-500 rounded-full"></div>
              <span>Code 128/39</span>
            </div>
            <div className="flex items-center gap-2">
              <div className="w-2 h-2 bg-pink-500 rounded-full"></div>
              <span>EAN-13/8</span>
            </div>
            <div className="flex items-center gap-2">
              <div className="w-2 h-2 bg-blue-500 rounded-full"></div>
              <span>UPC-A/E</span>
            </div>
            <div className="flex items-center gap-2">
              <div className="w-2 h-2 bg-cyan-500 rounded-full"></div>
              <span>PDF417, Aztec</span>
            </div>
          </div>
        </div>

        <canvas ref={canvasRef} className="hidden" />
      </div>
    </div>
  );
}
