// QR Scanner Mini App (React)
// Fayl: QRScannerMiniApp.jsx
// Tavsif: Bu single-file React komponenti - kamera orqali QR kodni aniqlash, tasvirni avtomatik yaxshilash (kontrast/yoritish/sharpen), zoom va rasm yuklash orqali skan qilishni qo'llab-quvvatlaydi.
// Talablar: npm install jsqr
// Qo'llanma:
// 1) React loyihangizga ushbu faylni qo'shing (masalan src/QRScannerMiniApp.jsx)
// 2) `npm install jsqr` bajarib oling
// 3) App ichida: import QRScannerMiniApp from './QRScannerMiniApp'; va <QRScannerMiniApp /> qo'ying

import React, { useRef, useEffect, useState } from 'react';
import jsQR from 'jsqr';

export default function QRScannerMiniApp() {
  const videoRef = useRef(null);
  const canvasRef = useRef(null);
  const [stream, setStream] = useState(null);
  const [scanning, setScanning] = useState(false);
  const [decoded, setDecoded] = useState(null);
  const [zoom, setZoom] = useState(1);
  const [supportsZoom, setSupportsZoom] = useState(false);
  const trackRef = useRef(null);
  const [torchOn, setTorchOn] = useState(false);
  const [processing, setProcessing] = useState(false);
  const animationRef = useRef(null);

  useEffect(() => {
    return () => stopCamera();
  }, []);

  async function startCamera() {
    try {
      const constraints = {
        video: { facingMode: 'environment', width: { ideal: 1280 }, height: { ideal: 720 } }
      };
      const s = await navigator.mediaDevices.getUserMedia(constraints);
      setStream(s);
      if (videoRef.current) {
        videoRef.current.srcObject = s;
        videoRef.current.play().catch(() => {});
      }
      const tracks = s.getVideoTracks();
      if (tracks && tracks.length) {
        trackRef.current = tracks[0];
        const caps = trackRef.current.getCapabilities ? trackRef.current.getCapabilities() : {};
        if (caps.zoom) setSupportsZoom(true);
      }
      setScanning(true);
      requestAnimationFrame(tick);
    } catch (err) {
      console.error('Camera start error', err);
      alert('Kamera ochib bo\'lmadi: iltimos brauzerga ruxsat bering yoki HTTPS orqali oching.');
    }
  }

  function stopCamera() {
    setScanning(false);
    if (animationRef.current) cancelAnimationFrame(animationRef.current);
    if (stream) {
      stream.getTracks().forEach(t => t.stop());
      setStream(null);
    }
    if (trackRef.current) trackRef.current = null;
  }

  async function setCameraZoom(z) {
    setZoom(z);
    try {
      if (trackRef.current && trackRef.current.applyConstraints) {
        await trackRef.current.applyConstraints({ advanced: [{ zoom: z }] });
      } else if (videoRef.current) {
        videoRef.current.style.transform = `scale(${z})`;
      }
    } catch (e) {
      if (videoRef.current) videoRef.current.style.transform = `scale(${z})`;
    }
  }

  async function toggleTorch() {
    try {
      if (!trackRef.current) return;
      const imageCapture = new window.ImageCapture(trackRef.current);
      const capabilities = await imageCapture.getPhotoCapabilities();
      if (capabilities.torch || (trackRef.current.getCapabilities && trackRef.current.getCapabilities().torch)) {
        torchOn ? await trackRef.current.applyConstraints({ advanced: [{ torch: false }] }) : await trackRef.current.applyConstraints({ advanced: [{ torch: true }] });
        setTorchOn(!torchOn);
      } else {
        alert('Torch (flash) qo\'llab-quvvatlanmaydi.');
      }
    } catch (err) {
      console.warn('Torch error', err);
      alert('Flashni yoqib bo\'lmadi yoki brauzer qo\'llab-quvvatlamaydi.');
    }
  }

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

  function tick() {
    if (!scanning) return;
    const video = videoRef.current;
    const canvas = canvasRef.current;
    if (!video || !canvas) {
      animationRef.current = requestAnimationFrame(tick);
      return;
    }
    const ctx = canvas.getContext('2d');
    const w = canvas.width = video.videoWidth || 640;
    const h = canvas.height = video.videoHeight || 480;
    try {
      ctx.drawImage(video, 0, 0, w, h);
      enhanceImage(ctx, w, h);
      const imageData = ctx.getImageData(0, 0, w, h);
      const code = jsQR(imageData.data, imageData.width, imageData.height, { inversionAttempts: 'both' });
      if (code) {
        setDecoded({ text: code.data, location: code.location });
        drawLocation(ctx, code.location);
        setScanning(false);
        setTimeout(() => setScanning(true), 1500);
      } else {
        setDecoded(null);
      }
    } catch (e) {
      console.warn('Tick error', e);
    }
    animationRef.current = requestAnimationFrame(tick);
  }

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

  async function onFilePicked(e) {
    const file = e.target.files && e.target.files[0];
    if (!file) return;
    setProcessing(true);
    const img = new Image();
    img.onload = () => {
      const canvas = canvasRef.current;
      const ctx = canvas.getContext('2d');
      const maxW = 1280;
      const scale = Math.min(1, maxW / img.width);
      canvas.width = img.width * scale;
      canvas.height = img.height * scale;
      ctx.drawImage(img, 0, 0, canvas.width, canvas.height);
      enhanceImage(ctx, canvas.width, canvas.height);
      const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
      const code = jsQR(imageData.data, imageData.width, imageData.height, { inversionAttempts: 'both' });
      if (code) setDecoded({ text: code.data, location: code.location });
      else alert('QR topilmadi — tasvirni aniqlashtirish yoki boshqa rasm sinab ko\'ring.');
      setProcessing(false);
    };
    img.onerror = () => { setProcessing(false); alert('Rasm yuklashda xatolik.'); };
    img.src = URL.createObjectURL(file);
  }

  return (
    <div className="p-4 max-w-3xl mx-auto">
      <h2 className="text-xl font-semibold mb-3">QR Scanner Mini App</h2>
      <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
        <div>
          <div className="relative bg-black overflow-hidden rounded-lg" style={{ height: 360 }}>
            <video ref={videoRef} className="w-full h-full object-cover" playsInline muted />
            <canvas ref={canvasRef} className="absolute top-0 left-0 w-full h-full pointer-events-none" />
          </div>
          <div className="flex gap-2 mt-2">
            {!stream ? (
              <button onClick={startCamera} className="px-3 py-2 rounded bg-blue-600 text-white">Kamerani ishga tushur</button>
            ) : (
              <button onClick={stopCamera} className="px-3 py-2 rounded bg-red-600 text-white">Kamerani to'xtat</button>
            )}

            <label className="px-3 py-2 rounded bg-gray-200 cursor-pointer">
              Rasm yuklash
              <input type="file" accept="image/*" onChange={onFilePicked} className="hidden" />
            </label>

            <button onClick={toggleTorch} className="px-3 py-2 rounded bg-yellow-500">Flash</button>
          </div>

          <div className="mt-3">
            <label className="block text-sm">Zoom: <span className="font-medium">{zoom.toFixed(2)}</span></label>
            <input type="range" min={1} max={3} step={0.05} value={zoom} onChange={(e) => setCameraZoom(Number(e.target.value))} />
            {!supportsZoom && <div className="text-xs text-gray-500">(Agar brauzer yoki kamera ichki zoom ni qo'llab-quvvatlamasa, CSS shim ishlatiladi.)</div>}
          </div>
        </div>

        <div>
          <div className="bg-white rounded p-3 shadow">
            <h3 className="font-semibold">Natija</h3>
            {processing && <div>Qayta ishlash... iltimos kuting.</div>}
            {decoded ? (
              <div className="mt-2">
                <div className="break-words p-2 bg-gray-100 rounded">{decoded.text}</div>
                <div className="mt-2">
                  <button className="px-3 py-2 rounded bg-green-600 text-white mt-2" onClick={() => { navigator.clipboard.writeText(decoded.text); alert('Matn nusxalandi'); }}>Nusxalash</button>
                </div>
              </div>
            ) : (
              <div className="text-sm text-gray-600 mt-2">Hozircha QR topilmadi — kamerani yaqinlashtirib yoki rasm yuklab ko'ring.</div>
            )}

            <div className="mt-4">
              <h4 className="font-medium">Yordamchi sozlamalar</h4>
              <ul className="list-disc pl-5 text-sm text-gray-700">
                <li>Yaxshi natija uchun yorug'lik yetarliligiga e'tibor bering.</li>
                <li>Agar QR teskari rangli bo'lsa, ilova avtomatik invert sinab ko'radi.</li>
                <li>Rasm yuklash orqali eskirgan yoki past sifatli QRlarni ham tekshirish mumkin.</li>
              </ul>
            </div>

          </div>
        </div>
      </div>

      <div className="mt-4 text-xs text-gray-500">Eslatma: Ba'zi telefonlar va brauzerlar kamera flash/torch yoki ichki zoom ni cheklashi mumkin. Eng yaxshi ishlash uchun Chrome yoki Edge (Android), Safari (iOS) ning so'nggi versiyasidan foydalaning.</div>
    </div>
  );
}
