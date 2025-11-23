<!doctype html>
<html lang="uz">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Takomillashtirilgan QR Scanner</title>
  <style>
    :root{--bg:#fff;--muted:#666;--accent:#0a84ff}
    body{font-family:system-ui,-apple-system,Segoe UI,Roboto; margin:10px; background:var(--bg); color:#111}
    h1{font-size:18px;margin:6px 0}
    .controls{display:flex;flex-wrap:wrap;gap:8px;align-items:center}
    #viewer{width:100%;max-width:400px;height:500px;border:1px solid #ddd;display:flex;align-items:center;justify-content:center;overflow:hidden;position:relative;background:#000;margin:0 auto}
    video{width:100%;height:100%;object-fit:cover}
    #scanOverlay{position:absolute;inset:0;pointer-events:none}
    .centerBox{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);width:220px;height:220px;border:2px dashed rgba(255,255,255,0.25);box-sizing:border-box;border-radius:8px}
    #controlsBar{margin-top:8px;display:flex;gap:8px;align-items:center;flex-wrap:wrap;justify-content:center}
    input[type="range"]{width:180px}
    button{padding:8px 10px;border-radius:8px;border:1px solid #ccc;background:#fff;cursor:pointer}
    #resultPanel{position:fixed;right:18px;top:18px;background:#fff;border:1px solid #ddd;padding:10px;border-radius:8px;max-width:320px;box-shadow:0 6px 20px rgba(0,0,0,0.08);z-index:1000}
    #qrPreview{width:240px;height:240px;display:block;margin-bottom:8px;object-fit:contain;filter:brightness(1.35) contrast(1.15);border-radius:6px;background:#fff}
    .small{font-size:13px;color:var(--muted)}
    #loader{display:none}
    #manualTools{margin-top:12px}
    .hint{font-size:13px;color:var(--muted);margin-top:8px;text-align:center}
    /* detection box lines */
    .line{position:absolute;background:lime;opacity:0.9}
    /* Image enhancement controls */
    .enhancement-controls {margin-top:12px;padding:10px;border:1px solid #eee;border-radius:8px;}
    .enhancement-controls h3 {margin:0 0 8px 0;font-size:14px;}
    .enhancement-sliders {display:flex;flex-direction:column;gap:8px;}
    .slider-container {display:flex;align-items:center;gap:8px;}
    .slider-container label {min-width:80px;font-size:13px;}
    .slider-container input {flex:1;}
    .slider-value {min-width:30px;text-align:right;font-size:12px;}
    .status-info {margin-top:8px;padding:6px;background:#f0f0f0;border-radius:4px;font-size:12px;}
    .auto-enhance {margin-top:8px;padding:8px;background:#e8f4ff;border-radius:6px;border:1px solid #cce5ff;}
    .auto-enhance h4 {margin:0 0 6px 0;font-size:13px;}
  </style>
</head>
<body>
  <h1 style="text-align:center">Takomillashtirilgan QR Scanner</h1>

  <div id="viewer">
    <video id="video" playsinline autoplay muted></video>
    <div id="scanOverlay">
      <div class="centerBox"></div>
      <!-- dynamic drawing (canvas overlay) -->
      <canvas id="overlayCanvas" style="position:absolute;inset:0;pointer-events:none"></canvas>
    </div>
  </div>

  <div id="controlsBar" class="controls">
    <label class="small">Zoom</label>
    <input id="zoomRange" type="range" min="1" max="10" step="0.1" value="1">
    <span id="zoomValue" class="small">1x</span>
    <label class="small">Torch</label>
    <button id="toggleTorch">Torch: O'ch</button>
    <button id="focusBtn">Auto Focus</button>
    <button id="stopBtn">Kamerani to'xtat</button>
    <span id="loader" class="small">Skanner yuklanmoqda...</span>
  </div>

  <div class="status-info" id="statusInfo">
    Kamera yuklanmoqda...
  </div>

  <div class="auto-enhance">
    <h4>Avtomatik Tasvir Yaxshilash</h4>
    <div class="controls">
      <label><input type="checkbox" id="autoEnhance" checked> Avtomatik yaxshilash (zoomda)</label>
      <label><input type="checkbox" id="autoFocus" checked> Avtomatik fokus (zoomda)</label>
    </div>
  </div>

  <div class="hint">
    Zoom qilganda, sistem avtomatik ravishda fokusni sozlaydi va tasvirni tiniqlashtiradi.<br>
    Mayda QR kodlarni o'qish uchun zoom qiling va QR kodni ramka markaziga joylashtiring.
  </div>

  <!-- Result panel (ko'rinadi QR topilganda) -->
  <div id="resultPanel" style="display:none;">
    <img id="qrPreview" src="" alt="QR snapshot"/>
    
    <!-- Image enhancement controls -->
    <div class="enhancement-controls">
      <h3>Tasvirni yaxshilash</h3>
      <div class="enhancement-sliders">
        <div class="slider-container">
          <label for="brightnessSlider">Yorqinlik:</label>
          <input type="range" id="brightnessSlider" min="0.5" max="2" step="0.05" value="1.35">
          <span class="slider-value" id="brightnessValue">1.35</span>
        </div>
        <div class="slider-container">
          <label for="contrastSlider">Kontrast:</label>
          <input type="range" id="contrastSlider" min="0.5" max="2" step="0.05" value="1.15">
          <span class="slider-value" id="contrastValue">1.15</span>
        </div>
        <div class="slider-container">
          <label for="sharpnessSlider">O'tkir:</label>
          <input type="range" id="sharpnessSlider" min="0" max="2" step="0.1" value="0.5">
          <span class="slider-value" id="sharpnessValue">0.5</span>
        </div>
        <button id="enhanceBtn">Tasvirni yaxshilash</button>
        <button id="resetEnhancement">Asl holatga qaytarish</button>
      </div>
    </div>
    
    <div><strong>QR kod ma'lumoti:</strong></div>
    <div id="decodedText" class="small" style="word-break:break-word; margin:5px 0"></div>
    
    <!-- Botga ulanish uchun qism -->
    <div id="botSection" style="margin-top:10px; padding:8px; background:#f0f8ff; border-radius:6px; display:none;">
      <div><strong>Botga ulanish:</strong></div>
      <div id="botInfo" class="small" style="margin:5px 0"></div>
      <button id="connectBotBtn">Botga ulanish</button>
    </div>
    
    <div style="margin-top:8px;display:flex;gap:8px; flex-wrap:wrap">
      <button id="copyBtn">Nusxa olish</button>
      <button id="openBtn">Ochish (URL bo'lsa)</button>
      <button id="resumeBtn">Yana skan qilish</button>
    </div>
  </div>

  <hr style="margin:12px 0">

  <!-- QR yaratish qismi -->
  <div id="manualTools">
    <div class="controls" style="justify-content:center">
      <input id="fileInput" type="file" accept="image/*" />
      <button id="genFromCanvas">QR yarat (hozirgi kadrdan)</button>
      <input id="qrText" placeholder="QR uchun matn yoki URL" style="width:200px" />
      <button id="genQrBtn">QR yarat</button>
    </div>
    <div id="qrholder" style="margin-top:8px; text-align:center"></div>
  </div>

  <!-- kutubxonalar -->
  <script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
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
    const qrPreview = document.getElementById('qrPreview');
    const decodedText = document.getElementById('decodedText');
    const copyBtn = document.getElementById('copyBtn');
    const openBtn = document.getElementById('openBtn');
    const resumeBtn = document.getElementById('resumeBtn');
    const loader = document.getElementById('loader');
    const stopBtn = document.getElementById('stopBtn');
    const statusInfo = document.getElementById('statusInfo');
    const autoEnhance = document.getElementById('autoEnhance');
    const autoFocus = document.getElementById('autoFocus');
    const botSection = document.getElementById('botSection');
    const botInfo = document.getElementById('botInfo');
    const connectBotBtn = document.getElementById('connectBotBtn');

    // manual tools
    const fileInput = document.getElementById('fileInput');
    const genQrBtn = document.getElementById('genQrBtn');
    const qrholder = document.getElementById('qrholder');
    const qrText = document.getElementById('qrText');
    const genFromCanvasBtn = document.getElementById('genFromCanvas');

    // image enhancement controls
    const brightnessSlider = document.getElementById('brightnessSlider');
    const contrastSlider = document.getElementById('contrastSlider');
    const sharpnessSlider = document.getElementById('sharpnessSlider');
    const brightnessValue = document.getElementById('brightnessValue');
    const contrastValue = document.getElementById('contrastValue');
    const sharpnessValue = document.getElementById('sharpnessValue');
    const enhanceBtn = document.getElementById('enhanceBtn');
    const resetEnhancementBtn = document.getElementById('resetEnhancement');

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

    // start on load
    window.addEventListener('load', ()=>{ startScanner(); });

    async function startScanner(){
      loader.style.display='inline';
      statusInfo.textContent = 'Kamera ochilmoqda...';
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

        // Kamera imkoniyatlarini sozlash
        try {
          const capabilities = track.getCapabilities ? track.getCapabilities() : {};
          const settings = track.getSettings ? track.getSettings() : {};
          
          // Focus mode
          if (capabilities.focusMode && capabilities.focusMode.includes('continuous')) {
            await track.applyConstraints({ advanced: [{ focusMode: 'continuous' }] });
            console.log('Applied continuous focus');
          }
          
          // Zoom sozlamalari
          if (capabilities.zoom) {
            useHardwareZoom = true;
            zoomRange.min = capabilities.zoom.min || 1;
            zoomRange.max = Math.min(capabilities.zoom.max || 10, 10);
            zoomRange.step = capabilities.zoom.step || 0.1;
            zoomRange.value = settings.zoom || capabilities.zoom.min || 1;
            currentZoom = parseFloat(zoomRange.value);
            zoomValue.textContent = currentZoom.toFixed(1) + 'x';
          } else {
            useHardwareZoom = false;
            zoomRange.min = 1; 
            zoomRange.max = 5; 
            zoomRange.step = 0.1; 
            zoomRange.value = 1;
          }
        } catch (e){
          console.warn('Capability set failed', e);
        }

        scanning = true;
        scanLoop();
      } catch (err) {
        statusInfo.textContent = 'Kamera ochilmadi: ' + err.message;
        console.error(err);
      } finally {
        loader.style.display='none';
      }
    }

    function stopScanner(){
      scanning = false;
      if (scanLoopId) cancelAnimationFrame(scanLoopId);
      if (stream) {
        stream.getTracks().forEach(t=>t.stop());
        stream = null; 
        track = null;
      }
      video.pause();
      video.srcObject = null;
      statusInfo.textContent = 'Kamera to\'xtatildi.';
    }

    stopBtn.addEventListener('click', ()=>{ stopScanner(); });

    // Zoom handling with auto-focus and enhancement
    zoomRange.addEventListener('input', async (e)=>{
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
        } catch(e){ 
          console.warn('hardware zoom failed', e); 
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
          // Try to set focus to a closer distance for better QR reading
          const min = caps.focusDistance.min || 0;
          const max = caps.focusDistance.max || 1;
          const focusValue = Math.min(max, min + (max - min) * 0.3); // 30% towards min for closer focus
          await track.applyConstraints({ advanced: [{ focusMode: 'manual', focusDistance: focusValue }]});
          statusInfo.textContent = 'Fokus yaqinroq sozlandi.';
        }
      } catch (e){ 
        console.warn('Auto focus failed', e); 
      }
    }

    focusBtn.addEventListener('click', applyAutoFocus);

    // torch
    toggleTorchBtn.addEventListener('click', async ()=>{
      if (!track) return;
      const capabilities = track.getCapabilities ? track.getCapabilities() : {};
      if (!capabilities.torch) {
        alert('Ushbu qurilmada torch (flesh) qoʻllab-quvvatlanmaydi.');
        return;
      }
      torchOn = !torchOn;
      try {
        await track.applyConstraints({ advanced: [{ torch: torchOn }] });
        toggleTorchBtn.textContent = `Torch: ${torchOn ? "Yo'qildi" : "O'ch"}`;
      } catch(e){
        console.warn('Torch control failed', e);
      }
    });

    // scanning loop with enhanced detection
    const scanCanvas = document.createElement('canvas');
    const sctx = scanCanvas.getContext('2d');

    async function scanLoop(){
      if (!scanning) return;
      if (video.readyState === video.HAVE_ENOUGH_DATA) {
        // set canvas to video size
        const vw = video.videoWidth;
        const vh = video.videoHeight;
        
        // Update overlay canvas size to match video display
        overlayCanvas.width = video.clientWidth;
        overlayCanvas.height = video.clientHeight;
        
        scanCanvas.width = vw;
        scanCanvas.height = vh;
        
        // draw current frame to scan canvas
        sctx.drawImage(video, 0, 0, vw, vh);
        
        // Apply enhancement if needed
        let imageData;
        if (isEnhancing && currentZoom > 2) {
          imageData = enhanceImage(sctx, vw, vh);
          statusInfo.textContent = `Zoom: ${currentZoom.toFixed(1)}x - Tasvir yaxshilandi. QR qidirilmoqda...`;
        } else {
          imageData = sctx.getImageData(0, 0, vw, vh);
          statusInfo.textContent = `Zoom: ${currentZoom.toFixed(1)}x - QR qidirilmoqda...`;
        }
        
        const code = jsQR(imageData.data, imageData.width, imageData.height, { 
          inversionAttempts: "attemptBoth" 
        });

        // clear overlay
        octx.clearRect(0, 0, overlayCanvas.width, overlayCanvas.height);

        if (code) {
          // draw detection box scaled to overlay size
          const scaleX = overlayCanvas.width / imageData.width;
          const scaleY = overlayCanvas.height / imageData.height;
          drawLine(code.location.topLeftCorner, code.location.topRightCorner, scaleX, scaleY);
          drawLine(code.location.topRightCorner, code.location.bottomRightCorner, scaleX, scaleY);
          drawLine(code.location.bottomRightCorner, code.location.bottomLeftCorner, scaleX, scaleY);
          drawLine(code.location.bottomLeftCorner, code.location.topLeftCorner, scaleX, scaleY);

          // take snapshot of the QR area for preview
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
          
          // create temporary canvas to crop and scale for preview
          const cropCanvas = document.createElement('canvas');
          cropCanvas.width = w; 
          cropCanvas.height = h;
          cropCanvas.getContext('2d').putImageData(
            sctx.getImageData(minX, minY, w, h), 0, 0
          );
          
          // upscale for preview
          const previewCanvas = document.createElement('canvas');
          const maxPreview = 600;
          const scaleUp = Math.min(maxPreview / w, maxPreview / h, 4); // Increased max scale
          previewCanvas.width = Math.round(w * scaleUp);
          previewCanvas.height = Math.round(h * scaleUp);
          const pctx = previewCanvas.getContext('2d');
          
          // Apply high-quality image scaling
          pctx.imageSmoothingEnabled = true;
          pctx.imageSmoothingQuality = 'high';
          pctx.drawImage(cropCanvas, 0, 0, previewCanvas.width, previewCanvas.height);

          // Save original image data for enhancement
          originalImageData = pctx.getImageData(0, 0, previewCanvas.width, previewCanvas.height);
          
          // show result panel
          qrPreview.src = previewCanvas.toDataURL('image/png');
          decodedText.textContent = code.data;
          resultPanel.style.display = 'block';
          
          // Check if it's a bot link and show bot section
          checkForBotLink(code.data);
          
          // stop scanning to avoid repeated triggers
          scanning = false;
          if (scanLoopId) cancelAnimationFrame(scanLoopId);
          lastDetected = code.data;
          statusInfo.textContent = 'QR kod muvaffaqiyatli o\'qildi!';
        } else {
          // draw center box hint
          octx.strokeStyle = 'rgba(255,255,255,0.35)';
          octx.lineWidth = 2;
          const centerX = overlayCanvas.width / 2;
          const centerY = overlayCanvas.height / 2;
          const boxSize = Math.min(overlayCanvas.width, overlayCanvas.height) * 0.6;
          octx.strokeRect(centerX - boxSize/2, centerY - boxSize/2, boxSize, boxSize);
        }
      }
      scanLoopId = requestAnimationFrame(scanLoop);
    }

    // Image enhancement function for scanning
    function enhanceImage(ctx, width, height) {
      const imageData = ctx.getImageData(0, 0, width, height);
      const data = imageData.data;
      
      // Apply brightness and contrast enhancement
      const brightness = 1.3;
      const contrast = 1.2;
      
      for (let i = 0; i < data.length; i += 4) {
        // Brightness adjustment
        data[i] = Math.min(255, Math.max(0, (data[i] - 128) * brightness + 128));
        data[i+1] = Math.min(255, Math.max(0, (data[i+1] - 128) * brightness + 128));
        data[i+2] = Math.min(255, Math.max(0, (data[i+2] - 128) * brightness + 128));
        
        // Contrast adjustment
        data[i] = Math.min(255, Math.max(0, (data[i] - 128) * contrast + 128));
        data[i+1] = Math.min(255, Math.max(0, (data[i+1] - 128) * contrast + 128));
        data[i+2] = Math.min(255, Math.max(0, (data[i+2] - 128) * contrast + 128));
      }
      
      return imageData;
    }

    function drawLine(p1, p2, scaleX, scaleY){
      octx.strokeStyle = 'lime';
      octx.lineWidth = 4;
      octx.beginPath();
      octx.moveTo(p1.x * scaleX, p1.y * scaleY);
      octx.lineTo(p2.x * scaleX, p2.y * scaleY);
      octx.stroke();
    }

    // Check if QR contains a bot link
    function checkForBotLink(text) {
      const telegramBotRegex = /(t\.me\/|telegram\.me\/|https?:\/\/t\.me\/|https?:\/\/telegram\.me\/)/i;
      const whatsappRegex = /(wa\.me\/|https?:\/\/wa\.me\/)/i;
      
      if (telegramBotRegex.test(text)) {
        botInfo.textContent = 'Telegram bot topildi';
        botSection.style.display = 'block';
      } else if (whatsappRegex.test(text)) {
        botInfo.textContent = 'WhatsApp chat topildi';
        botSection.style.display = 'block';
      } else {
        botSection.style.display = 'none';
      }
    }

    // Bot connection
    connectBotBtn.addEventListener('click', () => {
      const txt = decodedText.textContent || '';
      if (!txt) return;
      
      try {
        window.open(txt, '_blank');
      } catch(e) {
        alert('Botga ulanishda xatolik: ' + e.message);
      }
    });

    // resume scanning
    resumeBtn.addEventListener('click', ()=>{
      resultPanel.style.display = 'none';
      botSection.style.display = 'none';
      lastDetected = null;
      scanning = true;
      isEnhancing = false;
      scanLoop();
      statusInfo.textContent = 'QR qidirilmoqda...';
    });

    copyBtn.addEventListener('click', async ()=>{
      const text = decodedText.textContent || '';
      if (!text) return;
      try {
        await navigator.clipboard.writeText(text);
        alert('Matn nusxalandi');
      } catch(e){
        alert('Clipboardga yozishda xatolik');
      }
    });

    openBtn.addEventListener('click', ()=>{
      const txt = decodedText.textContent || '';
      if (!txt) return;
      try {
        // if looks like URL, open in new tab
        const url = new URL(txt);
        window.open(url.toString(), '_blank');
      } catch(e){
        alert('Bu URL emas: ' + txt);
      }
    });

    // Image enhancement functions for preview
    function applyImageEnhancement() {
      if (!originalImageData) return;
      
      const brightness = parseFloat(brightnessSlider.value);
      const contrast = parseFloat(contrastSlider.value);
      const sharpness = parseFloat(sharpnessSlider.value);
      
      // Create a copy of the original image data
      const enhancedData = new ImageData(
        new Uint8ClampedArray(originalImageData.data),
        originalImageData.width,
        originalImageData.height
      );
      
      const data = enhancedData.data;
      
      // Apply brightness and contrast
      for (let i = 0; i < data.length; i += 4) {
        // Brightness adjustment
        data[i] = Math.min(255, Math.max(0, (data[i] - 128) * brightness + 128));
        data[i+1] = Math.min(255, Math.max(0, (data[i+1] - 128) * brightness + 128));
        data[i+2] = Math.min(255, Math.max(0, (data[i+2] - 128) * brightness + 128));
        
        // Contrast adjustment
        data[i] = Math.min(255, Math.max(0, (data[i] - 128) * contrast + 128));
        data[i+1] = Math.min(255, Math.max(0, (data[i+1] - 128) * contrast + 128));
        data[i+2] = Math.min(255, Math.max(0, (data[i+2] - 128) * contrast + 128));
      }
      
      // Apply sharpness if needed
      if (sharpness > 0) {
        applySharpness(enhancedData, sharpness);
      }
      
      // Create a canvas to display the enhanced image
      const canvas = document.createElement('canvas');
      canvas.width = enhancedData.width;
      canvas.height = enhancedData.height;
      const ctx = canvas.getContext('2d');
      ctx.putImageData(enhancedData, 0, 0);
      
      // Update the preview image
      qrPreview.src = canvas.toDataURL('image/png');
    }
    
    function applySharpness(imageData, strength) {
      const data = imageData.data;
      const width = imageData.width;
      const height = imageData.height;
      
      // Create a copy for the sharpening kernel
      const tempData = new Uint8ClampedArray(data);
      
      // Simple sharpening kernel
      const kernel = [
        0, -1, 0,
        -1, 5, -1,
        0, -1, 0
      ];
      
      // Apply convolution
      for (let y = 1; y < height - 1; y++) {
        for (let x = 1; x < width - 1; x++) {
          for (let c = 0; c < 3; c++) { // RGB channels only
            let sum = 0;
            for (let ky = -1; ky <= 1; ky++) {
              for (let kx = -1; kx <= 1; kx++) {
                const idx = ((y + ky) * width + (x + kx)) * 4 + c;
                const kernelIdx = (ky + 1) * 3 + (kx + 1);
                sum += tempData[idx] * kernel[kernelIdx];
              }
            }
            const idx = (y * width + x) * 4 + c;
            data[idx] = Math.min(255, Math.max(0, 
              (1 - strength) * tempData[idx] + strength * sum
            ));
          }
        }
      }
    }
    
    // Update slider values display
    brightnessSlider.addEventListener('input', () => {
      brightnessValue.textContent = brightnessSlider.value;
    });
    
    contrastSlider.addEventListener('input', () => {
      contrastValue.textContent = contrastSlider.value;
    });
    
    sharpnessSlider.addEventListener('input', () => {
      sharpnessValue.textContent = sharpnessSlider.value;
    });
    
    // Apply enhancement when button is clicked
    enhanceBtn.addEventListener('click', applyImageEnhancement);
    
    // Reset enhancement
    resetEnhancementBtn.addEventListener('click', () => {
      if (!originalImageData) return;
      
      // Reset sliders to default values
      brightnessSlider.value = 1.35;
      contrastSlider.value = 1.15;
      sharpnessSlider.value = 0.5;
      
      // Update displayed values
      brightnessValue.textContent = brightnessSlider.value;
      contrastValue.textContent = contrastSlider.value;
      sharpnessValue.textContent = sharpnessSlider.value;
      
      // Create a canvas to display the original image
      const canvas = document.createElement('canvas');
      canvas.width = originalImageData.width;
      canvas.height = originalImageData.height;
      const ctx = canvas.getContext('2d');
      ctx.putImageData(originalImageData, 0, 0);
      
      // Update the preview image
      qrPreview.src = canvas.toDataURL('image/png');
    });

    // manual snapshot -> QR generation (from current video frame)
    genFromCanvasBtn.addEventListener('click', ()=>{
      if (!video || video.readyState < 2) { alert('Kamera tayyor emas'); return; }
      const c = document.createElement('canvas');
      c.width = video.videoWidth; c.height = video.videoHeight;
      const ctx = c.getContext('2d');
      ctx.drawImage(video, 0, 0, c.width, c.height);
      const dataUrl = c.toDataURL('image/jpeg', 0.9);
      createQR(dataUrl);
    });

    // file input -> show on canvas and optionally make QR
    fileInput.addEventListener('change', (e)=>{
      const f = e.target.files && e.target.files[0];
      if (!f) return;
      const img = new Image();
      img.onload = ()=> {
        // draw image on hidden canvas to ensure proper dataURL
        const c = document.createElement('canvas');
        c.width = img.width; c.height = img.height;
        c.getContext('2d').drawImage(img,0,0);
        createQR(c.toDataURL('image/jpeg', 0.85));
      };
      img.src = URL.createObjectURL(f);
    });

    // QR creation
    function createQR(payload){
      qrholder.innerHTML = '';
      const qdiv = document.createElement('div');
      qdiv.id = 'qrcode';
      qdiv.style.filter = 'brightness(1.4) contrast(1.12)';
      qrholder.appendChild(qdiv);

      try {
        new QRCode(qdiv, {
          text: payload,
          width: 260,
          height: 260,
          correctLevel: QRCode.CorrectLevel.H
        });
      } catch(e){
        qdiv.innerText = 'QR yaratishda xato: ' + e;
      }

      // save button
      const saveBtn = document.createElement('button');
      saveBtn.textContent = 'QRni yuklab olish';
      saveBtn.onclick = ()=> {
        const qrCanvas = qdiv.querySelector('canvas');
        if (qrCanvas) {
          const a = document.createElement('a');
          a.href = qrCanvas.toDataURL('image/png');
          a.download = 'qr.png';
          a.click();
        } else alert('QR topilmadi');
      };
      qrholder.appendChild(document.createElement('br'));
      qrholder.appendChild(saveBtn);
    }

    genQrBtn.addEventListener('click', ()=>{
      const text = (qrText.value || '').trim();
      if (!text) { alert('QR uchun matn kiriting'); return; }
      createQR(text);
    });

    // cleanup on unload
    window.addEventListener('beforeunload', ()=>{ stopScanner(); });
  </script>
</body>
</html>
