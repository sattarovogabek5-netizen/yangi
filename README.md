<!doctype html>
<html lang="uz">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Kuchli QR Scanner</title>
  <style>
    :root{--bg:#fff;--muted:#666;--accent:#0a84ff;--success:#00c853;--warning:#ff9100;--error:#ff1744}
    body{font-family:system-ui,-apple-system,Segoe UI,Roboto; margin:10px; background:var(--bg); color:#111; max-width:500px; margin:0 auto; padding:10px}
    h1{font-size:18px;margin:6px 0; text-align:center}
    .controls{display:flex;flex-wrap:wrap;gap:8px;align-items:center; justify-content:center}
    #viewer{width:100%;height:400px;border:1px solid #ddd;display:flex;align-items:center;justify-content:center;overflow:hidden;position:relative;background:#000; border-radius:12px; margin:10px 0}
    video{width:100%;height:100%;object-fit:cover}
    #scanOverlay{position:absolute;inset:0;pointer-events:none}
    .centerBox{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);width:220px;height:220px;border:2px dashed rgba(255,255,255,0.25);box-sizing:border-box;border-radius:8px}
    #controlsBar{margin-top:8px;display:flex;gap:8px;align-items:center;flex-wrap:wrap;justify-content:center}
    input[type="range"]{width:150px}
    button{padding:8px 12px;border-radius:8px;border:none;background:var(--accent);color:white;cursor:pointer;font-weight:500}
    button.secondary{background:#f0f0f0;color:#333}
    #resultPanel{position:fixed;left:50%;top:50%;transform:translate(-50%,-50%);background:#fff;border:1px solid #ddd;padding:15px;border-radius:12px;max-width:90%;width:350px;box-shadow:0 10px 30px rgba(0,0,0,0.2);z-index:1000; text-align:center}
    #qrPreview{width:200px;height:200px;display:block;margin:0 auto 10px;object-fit:contain;border-radius:6px;background:#fff; border:1px solid #eee}
    .small{font-size:13px;color:var(--muted)}
    #loader{display:none}
    .hint{font-size:13px;color:var(--muted);margin-top:8px;text-align:center; line-height:1.4}
    /* detection box lines */
    .line{position:absolute;background:lime;opacity:0.9}
    .status-info {margin-top:8px;padding:8px;background:#f0f8ff;border-radius:8px;font-size:13px;text-align:center; border:1px solid #d0e8ff}
    .auto-enhance {margin-top:12px;padding:10px;background:#e8f4ff;border-radius:8px;border:1px solid #cce5ff;}
    .auto-enhance h4 {margin:0 0 6px 0;font-size:14px; text-align:center}
    .sensitivity-controls {margin-top:12px; padding:10px; background:#fff8e1; border-radius:8px; border:1px solid #ffecb3;}
    .sensitivity-controls h4 {margin:0 0 6px 0;font-size:14px; text-align:center}
    .backdrop{position:fixed;inset:0;background:rgba(0,0,0,0.7);z-index:999;display:none}
    .manual-input {margin-top:15px; padding:12px; background:#f9f9f9; border-radius:8px; text-align:center}
    .manual-input h4 {margin:0 0 8px 0; font-size:14px}
  </style>
</head>
<body>
  <h1>Kuchli QR Scanner</h1>

  <div class="backdrop" id="backdrop"></div>

  <div id="viewer">
    <video id="video" playsinline autoplay muted></video>
    <div id="scanOverlay">
      <div class="centerBox"></div>
      <canvas id="overlayCanvas" style="position:absolute;inset:0;pointer-events:none"></canvas>
    </div>
  </div>

  <div class="status-info" id="statusInfo">
    Kamera ishga tushmoqda...
  </div>

  <div id="controlsBar" class="controls">
    <label class="small">Zoom</label>
    <input id="zoomRange" type="range" min="1" max="8" step="0.1" value="1">
    <span id="zoomValue" class="small">1x</span>
    <button id="toggleTorch">Torch: O'ch</button>
    <button id="focusBtn" class="secondary">Auto Focus</button>
    <button id="stopBtn" class="secondary">To'xtatish</button>
  </div>

  <div class="auto-enhance">
    <h4>Avtomatik Sozlamalar</h4>
    <div class="controls">
      <label><input type="checkbox" id="autoEnhance" checked> Avtomatik tasvir yaxshilash</label>
      <label><input type="checkbox" id="autoFocus" checked> Avtomatik fokus</label>
    </div>
  </div>

  <div class="sensitivity-controls">
    <h4>QR Topish Sezgirligi</h4>
    <div class="controls">
      <label class="small">Sezgirlik:</label>
      <input id="sensitivityRange" type="range" min="1" max="10" step="1" value="7">
      <span id="sensitivityValue" class="small">Yuqori</span>
    </div>
  </div>

  <div class="hint">
    <strong>Maslahat:</strong> QR kodni ramka markaziga joylashtiring.<br>
    Agar QR kodni o'qiy olmasangiz, quyidagi "Telefon skaneri" tugmasini bosing.
  </div>

  <div class="manual-input">
    <h4>Agar QR kodni o'qiy olmasangiz</h4>
    <button id="usePhoneScanner">Telefon skaneridan foydalaning</button>
    <div class="hint" style="margin-top:8px">
      Telefoningizdagi QR skaner dasturidan foydalanib, natijani shu yuboring
    </div>
  </div>

  <!-- Result panel -->
  <div id="resultPanel" style="display:none;">
    <img id="qrPreview" src="" alt="QR snapshot"/>
    
    <div><strong>QR kod ma'lumoti:</strong></div>
    <div id="decodedText" class="small" style="word-break:break-word; margin:8px 0; padding:8px; background:#f5f5f5; border-radius:4px"></div>
    
    <div style="margin-top:12px;display:flex;gap:8px; flex-wrap:wrap; justify-content:center">
      <button id="copyBtn">Nusxa olish</button>
      <button id="openBtn" class="secondary">Ochish</button>
      <button id="sendToBotBtn" style="background:var(--success)">Botga yuborish</button>
      <button id="resumeBtn" class="secondary">Davom etish</button>
    </div>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.js"></script>
  <script>
    // DOM elementlari
    const video = document.getElementById('video');
    const overlayCanvas = document.getElementById('overlayCanvas');
    const octx = overlayCanvas.getContext('2d');
    const zoomRange = document.getElementById('zoomRange');
    const zoomValue = document.getElementById('zoomValue');
    const toggleTorchBtn = document.getElementById('toggleTorch');
    const focusBtn = document.getElementById('focusBtn');
    const resultPanel = document.getElementById('resultPanel');
    const backdrop = document.getElementById('backdrop');
    const qrPreview = document.getElementById('qrPreview');
    const decodedText = document.getElementById('decodedText');
    const copyBtn = document.getElementById('copyBtn');
    const openBtn = document.getElementById('openBtn');
    const sendToBotBtn = document.getElementById('sendToBotBtn');
    const resumeBtn = document.getElementById('resumeBtn');
    const statusInfo = document.getElementById('statusInfo');
    const autoEnhance = document.getElementById('autoEnhance');
    const autoFocus = document.getElementById('autoFocus');
    const stopBtn = document.getElementById('stopBtn');
    const sensitivityRange = document.getElementById('sensitivityRange');
    const sensitivityValue = document.getElementById('sensitivityValue');
    const usePhoneScanner = document.getElementById('usePhoneScanner');

    let stream = null;
    let scanning = false;
    let scanLoopId = null;
    let track = null;
    let torchOn = false;
    let useHardwareZoom = false;
    let lastDetected = null;
    let originalImageData = null;
    let currentZoom = 1;
    let isEnhancing = false;
    let sensitivity = 7;

    // Start scanner
    window.addEventListener('load', () => { startScanner(); });

    async function startScanner() {
      statusInfo.textContent = 'Kamera ochilmoqda...';
      statusInfo.style.background = '#fff8e1';
      statusInfo.style.border = '1px solid #ffecb3';
      
      try {
        stream = await navigator.mediaDevices.getUserMedia({ 
          video: { 
            facingMode: 'environment', 
            width: { ideal: 1920 }, 
            height: { ideal: 1080 } 
          }, 
          audio: false 
        });
        
        video.srcObject = stream;
        await video.play();
        
        // Canvas o'lchamlarini sozlash
        overlayCanvas.width = video.clientWidth;
        overlayCanvas.height = video.clientHeight;
        
        track = stream.getVideoTracks()[0];
        statusInfo.textContent = 'Kamera tayyor. QR kodni skanerlang.';
        statusInfo.style.background = '#f0f8ff';
        statusInfo.style.border = '1px solid #d0e8ff';

        // Kamera imkoniyatlarini sozlash
        try {
          const capabilities = track.getCapabilities ? track.getCapabilities() : {};
          const settings = track.getSettings ? track.getSettings() : {};
          
          // Focus mode
          if (capabilities.focusMode && capabilities.focusMode.includes('continuous')) {
            await track.applyConstraints({ advanced: [{ focusMode: 'continuous' }] });
          }
          
          // Zoom sozlamalari
          if (capabilities.zoom) {
            useHardwareZoom = true;
            zoomRange.min = capabilities.zoom.min || 1;
            zoomRange.max = Math.min(capabilities.zoom.max || 8, 8);
            zoomRange.step = capabilities.zoom.step || 0.1;
            zoomRange.value = settings.zoom || capabilities.zoom.min || 1;
            currentZoom = parseFloat(zoomRange.value);
            zoomValue.textContent = currentZoom.toFixed(1) + 'x';
          }
        } catch (e) {
          console.warn('Camera capabilities error:', e);
        }

        scanning = true;
        scanLoop();
      } catch (err) {
        statusInfo.textContent = 'Kamera ochilmadi: ' + err.message;
        statusInfo.style.background = '#ffebee';
        statusInfo.style.border = '1px solid #ffcdd2';
        console.error(err);
      }
    }

    function stopScanner() {
      scanning = false;
      if (scanLoopId) cancelAnimationFrame(scanLoopId);
      if (stream) {
        stream.getTracks().forEach(t => t.stop());
        stream = null; 
        track = null;
      }
      video.pause();
      video.srcObject = null;
      statusInfo.textContent = 'Kamera to\'xtatildi.';
      statusInfo.style.background = '#f5f5f5';
      statusInfo.style.border = '1px solid #e0e0e0';
    }

    stopBtn.addEventListener('click', () => { stopScanner(); });

    // Sezgirlik sozlamalari
    sensitivityRange.addEventListener('input', (e) => {
      sensitivity = parseInt(e.target.value);
      updateSensitivityDisplay();
    });

    function updateSensitivityDisplay() {
      let levelText = "";
      let color = "";
      
      switch(sensitivity) {
        case 1:
        case 2:
          levelText = "Juda Past";
          color = "#ffcdd2";
          break;
        case 3:
        case 4:
          levelText = "Past";
          color = "#ffecb3";
          break;
        case 5:
        case 6:
          levelText = "O'rta";
          color = "#fff8e1";
          break;
        case 7:
        case 8:
          levelText = "Yuqori";
          color = "#f0f8ff";
          break;
        case 9:
        case 10:
          levelText = "Juda Yuqori";
          color = "#e8f5e8";
          break;
      }
      
      sensitivityValue.textContent = levelText;
      document.querySelector('.sensitivity-controls').style.background = color;
    }

    // Zoom handling with auto-focus and enhancement
    zoomRange.addEventListener('input', async (e) => {
      const val = parseFloat(e.target.value);
      currentZoom = val;
      zoomValue.textContent = val.toFixed(1) + 'x';
      
      if (track && useHardwareZoom) {
        try {
          await track.applyConstraints({ advanced: [{ zoom: val }] });
          
          // Auto focus when zooming
          if (autoFocus.checked && val > 1.5) {
            setTimeout(() => {
              applyAutoFocus();
            }, 300);
          }
          
          // Auto enhance when zooming
          if (autoEnhance.checked && val > 2) {
            isEnhancing = true;
            statusInfo.textContent = 'Zoom: ' + val.toFixed(1) + 'x - Tasvir yaxshilanmoqda...';
          }
        } catch(e) { 
          console.warn('Hardware zoom failed', e); 
        }
      } else {
        // digital zoom via CSS transform
        video.style.transform = `scale(${val})`;
        
        // Auto enhance for digital zoom too
        if (autoEnhance.checked && val > 2) {
          isEnhancing = true;
          statusInfo.textContent = 'Zoom: ' + val.toFixed(1) + 'x - Tasvir yaxshilanmoqda...';
        }
      }
    });

    // Auto focus function
    async function applyAutoFocus() {
      if (!track) return;
      try {
        const caps = track.getCapabilities();
        if (caps.focusMode && caps.focusMode.includes('continuous')) {
          await track.applyConstraints({ advanced: [{ focusMode: 'continuous' }]});
          statusInfo.textContent = 'Avtomatik fokus faollashtirildi.';
        } else if (caps.focusDistance) {
          const min = caps.focusDistance.min || 0;
          const max = caps.focusDistance.max || 1;
          const focusValue = Math.min(max, min + (max - min) * 0.3);
          await track.applyConstraints({ advanced: [{ focusMode: 'manual', focusDistance: focusValue }]});
          statusInfo.textContent = 'Fokus yaqinroq sozlandi.';
        }
      } catch (e) { 
        console.warn('Auto focus failed', e); 
      }
    }

    focusBtn.addEventListener('click', applyAutoFocus);

    // Torch control
    toggleTorchBtn.addEventListener('click', async () => {
      if (!track) return;
      const capabilities = track.getCapabilities ? track.getCapabilities() : {};
      if (!capabilities.torch) {
        alert('Ushbu qurilmada torch (flesh) qoʻllab-quvvatlanmaydi.');
        return;
      }
      torchOn = !torchOn;
      try {
        await track.applyConstraints({ advanced: [{ torch: torchOn }] });
        toggleTorchBtn.textContent = `Torch: ${torchOn ? "Yoqilgan" : "O'ch"}`;
      } catch(e) {
        console.warn('Torch control failed', e);
      }
    });

    // Scanning loop with enhanced detection
    const scanCanvas = document.createElement('canvas');
    const sctx = scanCanvas.getContext('2d');

    async function scanLoop() {
      if (!scanning) return;
      if (video.readyState === video.HAVE_ENOUGH_DATA) {
        // Set canvas to video size
        const vw = video.videoWidth;
        const vh = video.videoHeight;
        
        // Update overlay canvas size to match video display
        overlayCanvas.width = video.clientWidth;
        overlayCanvas.height = video.clientHeight;
        
        scanCanvas.width = vw;
        scanCanvas.height = vh;
        
        // Draw current frame to scan canvas
        sctx.drawImage(video, 0, 0, vw, vh);
        
        // Apply enhancement if needed
        let imageData;
        if (isEnhancing && currentZoom > 2) {
          imageData = enhanceImage(sctx, vw, vh);
        } else {
          imageData = sctx.getImageData(0, 0, vw, vh);
        }
        
        // Multiple detection attempts with different settings
        let code = null;
        
        // Try normal detection first
        code = jsQR(imageData.data, imageData.width, imageData.height, { 
          inversionAttempts: "dontInvert"
        });
        
        // If not found, try with inversion based on sensitivity
        if (!code && sensitivity > 5) {
          code = jsQR(imageData.data, imageData.width, imageData.height, { 
            inversionAttempts: "onlyInvert"
          });
        }
        
        // If still not found and high sensitivity, try both
        if (!code && sensitivity > 7) {
          code = jsQR(imageData.data, imageData.width, imageData.height, { 
            inversionAttempts: "attemptBoth"
          });
        }
        
        // Clear overlay
        octx.clearRect(0, 0, overlayCanvas.width, overlayCanvas.height);

        if (code) {
          // Draw detection box
          const scaleX = overlayCanvas.width / imageData.width;
          const scaleY = overlayCanvas.height / imageData.height;
          drawLine(code.location.topLeftCorner, code.location.topRightCorner, scaleX, scaleY);
          drawLine(code.location.topRightCorner, code.location.bottomRightCorner, scaleX, scaleY);
          drawLine(code.location.bottomRightCorner, code.location.bottomLeftCorner, scaleX, scaleY);
          drawLine(code.location.bottomLeftCorner, code.location.topLeftCorner, scaleX, scaleY);

          // Take snapshot for preview
          const minX = Math.max(0, Math.floor(Math.min(
            code.location.topLeftCorner.x, code.location.bottomLeftCorner.x, 
            code.location.topRightCorner.x, code.location.bottomRightCorner.x
          )));
          const minY = Math.max(0, Math.floor(Math.min(
            code.location.topLeftCorner.y, code.location.topRightCorner.y, 
            code.location.bottomLeftCorner.y, code.location.bottomRightCorner.y
          )));
          const maxX = Math.min(imageData.width, Math.ceil(Math.max(
            code.location.topLeftCorner.x, code.location.bottomLeftCorner.x, 
            code.location.topRightCorner.x, code.location.bottomRightCorner.x
          )));
          const maxY = Math.min(imageData.height, Math.ceil(Math.max(
            code.location.topLeftCorner.y, code.location.topRightCorner.y, 
            code.location.bottomLeftCorner.y, code.location.bottomRightCorner.y
          )));
          
          const w = maxX - minX, h = maxY - minY;
          
          // Create preview
          const cropCanvas = document.createElement('canvas');
          cropCanvas.width = w; 
          cropCanvas.height = h;
          cropCanvas.getContext('2d').putImageData(
            sctx.getImageData(minX, minY, w, h), 0, 0
          );
          
          const previewCanvas = document.createElement('canvas');
          const maxPreview = 600;
          const scaleUp = Math.min(maxPreview / w, maxPreview / h, 4);
          previewCanvas.width = Math.round(w * scaleUp);
          previewCanvas.height = Math.round(h * scaleUp);
          const pctx = previewCanvas.getContext('2d');
          
          pctx.imageSmoothingEnabled = true;
          pctx.imageSmoothingQuality = 'high';
          pctx.drawImage(cropCanvas, 0, 0, previewCanvas.width, previewCanvas.height);

          // Save original image data
          originalImageData = pctx.getImageData(0, 0, previewCanvas.width, previewCanvas.height);
          
          // Show result
          qrPreview.src = previewCanvas.toDataURL('image/png');
          decodedText.textContent = code.data;
          resultPanel.style.display = 'block';
          backdrop.style.display = 'block';
          
          // Stop scanning
          scanning = false;
          if (scanLoopId) cancelAnimationFrame(scanLoopId);
          lastDetected = code.data;
          statusInfo.textContent = 'QR kod topildi!';
          statusInfo.style.background = '#e8f5e8';
          statusInfo.style.border = '1px solid #c8e6c9';
        } else {
          // Draw center box
          octx.strokeStyle = 'rgba(255,255,255,0.35)';
          octx.lineWidth = 2;
          const centerX = overlayCanvas.width / 2;
          const centerY = overlayCanvas.height / 2;
          const boxSize = Math.min(overlayCanvas.width, overlayCanvas.height) * 0.6;
          octx.strokeRect(centerX - boxSize/2, centerY - boxSize/2, boxSize, boxSize);
          
          // Update status based on sensitivity
          let statusMsg = `QR qidirilmoqda (Sezgirlik: ${sensitivityValue.textContent})`;
          if (sensitivity < 5) {
            statusInfo.style.background = '#fff8e1';
            statusInfo.style.border = '1px solid #ffecb3';
          } else {
            statusInfo.style.background = '#f0f8ff';
            statusInfo.style.border = '1px solid #d0e8ff';
          }
          statusInfo.textContent = statusMsg;
        }
      }
      scanLoopId = requestAnimationFrame(scanLoop);
    }

    // Image enhancement for scanning
    function enhanceImage(ctx, width, height) {
      const imageData = ctx.getImageData(0, 0, width, height);
      const data = imageData.data;
      
      // Apply enhancement based on sensitivity
      const brightness = 1 + (sensitivity * 0.05);
      const contrast = 1 + (sensitivity * 0.03);
      
      for (let i = 0; i < data.length; i += 4) {
        // Brightness
        data[i] = Math.min(255, Math.max(0, (data[i] - 128) * brightness + 128));
        data[i+1] = Math.min(255, Math.max(0, (data[i+1] - 128) * brightness + 128));
        data[i+2] = Math.min(255, Math.max(0, (data[i+2] - 128) * brightness + 128));
        
        // Contrast
        data[i] = Math.min(255, Math.max(0, (data[i] - 128) * contrast + 128));
        data[i+1] = Math.min(255, Math.max(0, (data[i+1] - 128) * contrast + 128));
        data[i+2] = Math.min(255, Math.max(0, (data[i+2] - 128) * contrast + 128));
      }
      
      return imageData;
    }

    function drawLine(p1, p2, scaleX, scaleY) {
      octx.strokeStyle = 'lime';
      octx.lineWidth = 4;
      octx.beginPath();
      octx.moveTo(p1.x * scaleX, p1.y * scaleY);
      octx.lineTo(p2.x * scaleX, p2.y * scaleY);
      octx.stroke();
    }

    // Phone scanner fallback
    usePhoneScanner.addEventListener('click', () => {
      alert("Iltimos, telefoningizdagi QR skaner dasturidan foydalanib, QR kodni skanerlang va natijani shu yerga yuboring.\n\nSkaner dasturi yo'q bo'lsa, AppStore yoki Google Play'dan yuklab olishingiz mumkin.");
    });

    // Resume scanning
    resumeBtn.addEventListener('click', () => {
      resultPanel.style.display = 'none';
      backdrop.style.display = 'none';
      lastDetected = null;
      scanning = true;
      isEnhancing = false;
      scanLoop();
      statusInfo.textContent = 'QR qidirilmoqda...';
      statusInfo.style.background = '#f0f8ff';
      statusInfo.style.border = '1px solid #d0e8ff';
    });

    // Copy to clipboard
    copyBtn.addEventListener('click', async () => {
      const text = decodedText.textContent || '';
      if (!text) return;
      try {
        await navigator.clipboard.writeText(text);
        alert('Matn nusxalandi!');
      } catch(e) {
        alert('Nusxalashda xatolik: ' + e.message);
      }
    });

    // Open URL
    openBtn.addEventListener('click', () => {
      const txt = decodedText.textContent || '';
      if (!txt) return;
      try {
        const url = new URL(txt);
        window.open(url.toString(), '_blank');
      } catch(e) {
        alert('Bu URL emas: ' + txt);
      }
    });

    // Send to bot
    sendToBotBtn.addEventListener('click', () => {
      const txt = decodedText.textContent || '';
      if (!txt) return;
      
      // Simulate sending to bot (in real app, this would be an API call)
      alert(`QR kod ma'lumoti botga yuborildi:\n\n${txt}\n\nBot javobini kuting...`);
      
      // Close the result panel
      resultPanel.style.display = 'none';
      backdrop.style.display = 'none';
      scanning = true;
      scanLoop();
    });

    // Initialize sensitivity display
    updateSensitivityDisplay();
    
    // Cleanup
    window.addEventListener('beforeunload', () => { stopScanner(); });
  </script>
</body>
</html>
