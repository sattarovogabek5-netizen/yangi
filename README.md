<!doctype html>
<html lang="uz">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Mini App — Avto QR Scanner + Editor</title>
  <style>
    :root{--bg:#fff;--muted:#666;--accent:#0a84ff}
    body{font-family:system-ui,-apple-system,Segoe UI,Roboto; margin:10px; background:var(--bg); color:#111}
    h1{font-size:18px;margin:6px 0}
    .controls{display:flex;flex-wrap:wrap;gap:8px;align-items:center}
    #viewer{width:340px;height:420px;border:1px solid #ddd;display:flex;align-items:center;justify-content:center;overflow:hidden;position:relative;background:#000}
    video{width:100%;height:100%;object-fit:cover}
    #scanOverlay{position:absolute;inset:0;pointer-events:none}
    .centerBox{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);width:220px;height:220px;border:2px dashed rgba(255,255,255,0.25);box-sizing:border-box;border-radius:8px}
    #controlsBar{margin-top:8px;display:flex;gap:8px;align-items:center;flex-wrap:wrap}
    input[type="range"]{width:180px}
    button{padding:8px 10px;border-radius:8px;border:1px solid #ccc;background:#fff;cursor:pointer}
    #resultPanel{position:fixed;right:18px;top:18px;background:#fff;border:1px solid #ddd;padding:10px;border-radius:8px;max-width:320px;box-shadow:0 6px 20px rgba(0,0,0,0.08);}
    #qrPreview{width:240px;height:240px;display:block;margin-bottom:8px;object-fit:contain;filter:brightness(1.35) contrast(1.15);border-radius:6px;background:#fff}
    .small{font-size:13px;color:var(--muted)}
    #loader{display:none}
    #manualTools{margin-top:12px}
    .hint{font-size:13px;color:var(--muted);margin-top:8px}
    /* detection box lines */
    .line{position:absolute;background:lime;opacity:0.9}
  </style>
</head>
<body>
  <h1>Mini-app: avtomatik QR skaner + rasm → QR</h1>

  <div id="viewer">
    <video id="video" playsinline autoplay muted></video>
    <div id="scanOverlay">
      <div class="centerBox"></div>
      <!-- dynamic drawing (canvas overlay) -->
      <canvas id="overlayCanvas" width="340" height="420" style="position:absolute;inset:0;pointer-events:none"></canvas>
    </div>
  </div>

  <div id="controlsBar" class="controls">
    <label class="small">Zoom</label>
    <input id="zoomRange" type="range" min="1" max="4" step="0.01" value="1">
    <label class="small">Torch</label>
    <button id="toggleTorch">Torch: O'ch</button>
    <label class="small">Auto-focus</label>
    <button id="focusBtn">Focus sinov</button>
    <button id="stopBtn">Kamerani to'xtat</button>
    <span id="loader" class="small">Skanner yuklanmoqda...</span>
  </div>

  <div class="hint">Ishga tushganda kamera avtomatik ochiladi va atrofdagi QRlarni qidiradi. Agar telefon zoom yoki torch qo'llab-quvvatlasa, zoom/torch ishlaydi.</div>

  <!-- Result panel (ko'rinadi QR topilganda) -->
  <div id="resultPanel" style="display:none;">
    <img id="qrPreview" src="" alt="QR snapshot"/>
    <div><strong>Decoded:</strong></div>
    <div id="decodedText" class="small" style="word-break:break-word"></div>
    <div style="margin-top:8px;display:flex;gap:8px">
      <button id="copyBtn">Nusxa olish</button>
      <button id="openBtn">Ochish (URL bo'lsa)</button>
      <button id="resumeBtn">Yana skan qilish</button>
    </div>
  </div>

  <hr style="margin:12px 0">

  <!-- eski rasm→QR vositalari (iste'fo uchun) -->
  <div id="manualTools">
    <div class="controls">
      <input id="fileInput" type="file" accept="image/*" />
      <button id="genFromCanvas">Hoziroq QR yarat (hozirgi snapshot)</button>
      <input id="qrText" placeholder="QR uchun matn yoki URL" style="width:46%" />
      <label><input id="encodeImage" type="checkbox" /> Rasmni base64 sifatida QRga joylash</label>
      <button id="genQrBtn">QR yarat</button>
    </div>
    <div id="qrholder" style="margin-top:8px"></div>
  </div>

  <!-- kutubxonalar -->
  <script src="https://cdn.jsdelivr.net/npm/jsqr@1.4.0/dist/jsQR.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
  <script>
    // DOM
    const video = document.getElementById('video');
    const overlayCanvas = document.getElementById('overlayCanvas');
    const octx = overlayCanvas.getContext('2d');
    const zoomRange = document.getElementById('zoomRange');
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

    // manual tools
    const fileInput = document.getElementById('fileInput');
    const genQrBtn = document.getElementById('genQrBtn');
    const qrholder = document.getElementById('qrholder');
    const qrText = document.getElementById('qrText');
    const encodeImage = document.getElementById('encodeImage');
    const genFromCanvasBtn = document.getElementById('genFromCanvas');

    let stream = null;
    let scanning = false;
    let scanLoopId = null;
    let track = null;
    let torchOn = false;
    let useHardwareZoom = false;
    let lastDetected = null;

    // start on load
    window.addEventListener('load', ()=>{ startScanner(); });

    async function startScanner(){
      loader.style.display='inline';
      try {
        stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'environment', width: { ideal: 1280 }, height: { ideal: 720 } }, audio: false });
        video.srcObject = stream;
        await video.play();
        track = stream.getVideoTracks()[0];

        // attempt to enable continuous focus / exposure for better results
        try {
          const capabilities = track.getCapabilities ? track.getCapabilities() : {};
          const settings = track.getSettings ? track.getSettings() : {};
          // Focus mode
          if (capabilities.focusMode && capabilities.focusMode.includes('continuous')) {
            await track.applyConstraints({ advanced: [{ focusMode: 'continuous' }] });
            console.log('Applied continuous focus');
          }
          // exposure mode
          if (capabilities.exposureMode && capabilities.exposureMode.includes('continuous')) {
            await track.applyConstraints({ advanced: [{ exposureMode: 'continuous' }] });
            console.log('Applied continuous exposure');
          }
          // zoom
          if (capabilities.zoom) {
            useHardwareZoom = true;
            zoomRange.min = capabilities.zoom.min || 1;
            zoomRange.max = capabilities.zoom.max || 4;
            zoomRange.step = capabilities.zoom.step || 0.01;
            zoomRange.value = settings.zoom || capabilities.zoom.min || 1;
          } else {
            // keep default slidable digital zoom range
            useHardwareZoom = false;
            zoomRange.min = 1; zoomRange.max = 3; zoomRange.step = 0.01; zoomRange.value = 1;
          }
        } catch (e){
          console.warn('Capability set failed', e);
        }

        scanning = true;
        scanLoop();
      } catch (err) {
        alert('Kamera ochilmadi: '+err.message);
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
        stream = null; track = null;
      }
      video.pause();
      video.srcObject = null;
    }

    stopBtn.addEventListener('click', ()=>{ stopScanner(); });

    // zoom handling
    zoomRange.addEventListener('input', async (e)=>{
      const val = parseFloat(e.target.value);
      if (track && useHardwareZoom) {
        try {
          await track.applyConstraints({ advanced: [{ zoom: val }] });
        } catch(e){ console.warn('hardware zoom failed', e); }
      } else {
        // digital zoom via CSS transform
        video.style.transform = `scale(${val})`;
      }
    });

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

    // manual focus try
    focusBtn.addEventListener('click', async ()=>{
      if (!track) return;
      try {
        const caps = track.getCapabilities();
        if (caps.focusDistance) {
          // set focus to mid-range to try
          const min = caps.focusDistance.min || 0;
          const max = caps.focusDistance.max || 1;
          const mid = (min + max) / 2;
          await track.applyConstraints({ advanced: [{ focusMode: 'manual', focusDistance: mid }]});
          alert('Focus sozlandi (sinov).');
        } else if (caps.focusMode && caps.focusMode.includes('continuous')) {
          await track.applyConstraints({ advanced: [{ focusMode: 'continuous' }]});
          alert('Continuous focus yoqildi.');
        } else {
          alert('Focus boshqaruvi qurilmangizda cheklangan.');
        }
      } catch (e){ console.warn(e); alert('Focus sozlashda xatolik.'); }
    });

    // scanning loop
    const scanCanvas = document.createElement('canvas');
    const sctx = scanCanvas.getContext('2d');

    async function scanLoop(){
      if (!scanning) return;
      if (video.readyState === video.HAVE_ENOUGH_DATA) {
        // set canvas to video size
        const vw = video.videoWidth;
        const vh = video.videoHeight;
        const vwScaled = overlayCanvas.width = 340;
        const vhScaled = overlayCanvas.height = 420;
        scanCanvas.width = vw;
        scanCanvas.height = vh;
        // draw current frame to scan canvas
        sctx.drawImage(video, 0, 0, vw, vh);
        const imageData = sctx.getImageData(0,0,scanCanvas.width,scanCanvas.height);
        const code = jsQR(imageData.data, imageData.width, imageData.height, { inversionAttempts: "attemptBoth" });

        // clear overlay
        octx.clearRect(0,0,overlayCanvas.width, overlayCanvas.height);

        if (code) {
          // draw detection box scaled to overlay size
          // compute scale between scanCanvas and overlayCanvas
          const scaleX = overlayCanvas.width / scanCanvas.width;
          const scaleY = overlayCanvas.height / scanCanvas.height;
          drawLine(code.location.topLeftCorner, code.location.topRightCorner, scaleX, scaleY);
          drawLine(code.location.topRightCorner, code.location.bottomRightCorner, scaleX, scaleY);
          drawLine(code.location.bottomRightCorner, code.location.bottomLeftCorner, scaleX, scaleY);
          drawLine(code.location.bottomLeftCorner, code.location.topLeftCorner, scaleX, scaleY);

          // take snapshot of the QR area for preview (crop from scanCanvas)
          const minX = Math.max(0, Math.floor(Math.min(code.location.topLeftCorner.x, code.location.bottomLeftCorner.x, code.location.topRightCorner.x, code.location.bottomRightCorner.x)));
          const minY = Math.max(0, Math.floor(Math.min(code.location.topLeftCorner.y, code.location.topRightCorner.y, code.location.bottomLeftCorner.y, code.location.bottomRightCorner.y)));
          const maxX = Math.min(scanCanvas.width, Math.ceil(Math.max(code.location.topLeftCorner.x, code.location.bottomLeftCorner.x, code.location.topRightCorner.x, code.location.bottomRightCorner.x)));
          const maxY = Math.min(scanCanvas.height, Math.ceil(Math.max(code.location.topLeftCorner.y, code.location.topRightCorner.y, code.location.bottomLeftCorner.y, code.location.bottomRightCorner.y)));
          const w = maxX - minX, h = maxY - minY;
          // create temporary canvas to crop and scale for preview
          const cropCanvas = document.createElement('canvas');
          cropCanvas.width = w; cropCanvas.height = h;
          cropCanvas.getContext('2d').putImageData(sctx.getImageData(minX, minY, w, h), 0, 0);
          // upscale for preview
          const previewCanvas = document.createElement('canvas');
          const maxPreview = 600;
          const scaleUp = Math.min(maxPreview / w, maxPreview / h, 3);
          previewCanvas.width = Math.round(w * scaleUp);
          previewCanvas.height = Math.round(h * scaleUp);
          const pctx = previewCanvas.getContext('2d');
          // apply simple brightness/contrast by drawing onto white then blending (we'll use CSS filter later)
          pctx.drawImage(cropCanvas, 0, 0, previewCanvas.width, previewCanvas.height);

          // show result panel with brightened preview and decoded text
          qrPreview.src = previewCanvas.toDataURL('image/png');
          decodedText.textContent = code.data;
          resultPanel.style.display = 'block';

          // stop scanning to avoid repeated triggers; user can resume
          scanning = false;
          if (scanLoopId) cancelAnimationFrame(scanLoopId);
          lastDetected = code.data;
        } else {
          // draw center box hint
          octx.strokeStyle = 'rgba(255,255,255,0.35)';
          octx.lineWidth = 2;
          octx.strokeRect((overlayCanvas.width-220)/2, (overlayCanvas.height-220)/2, 220, 220);
        }
      }
      scanLoopId = requestAnimationFrame(scanLoop);
    }

    function drawLine(p1, p2, scaleX, scaleY){
      octx.strokeStyle = 'lime';
      octx.lineWidth = 4;
      octx.beginPath();
      octx.moveTo(p1.x * scaleX, p1.y * scaleY);
      octx.lineTo(p2.x * scaleX, p2.y * scaleY);
      octx.stroke();
    }

    // resume scanning
    resumeBtn.addEventListener('click', ()=>{
      resultPanel.style.display = 'none';
      lastDetected = null;
      scanning = true;
      scanLoop();
    });

    copyBtn.addEventListener('click', async ()=>{
      const text = decodedText.textContent || '';
      if (!text) return;
      try {
        await navigator.clipboard.writeText(text);
        alert('Nusxa olindi');
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

    // QR creation (with brightened preview)
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
      if (!text && !encodeImage.checked) { alert('QR uchun matn yoki rasm yuklang / tanlang'); return; }
      if (encodeImage.checked) {
        // use currently shown preview or snapshot
        if (!video || video.readyState < 2) { alert('Kamera tayyor emas'); return; }
        const c = document.createElement('canvas');
        c.width = video.videoWidth; c.height = video.videoHeight;
        c.getContext('2d').drawImage(video, 0, 0);
        const dataUrl = c.toDataURL('image/jpeg', 0.7);
        createQR(dataUrl);
      } else {
        createQR(text);
      }
    });

    // cleanup on unload
    window.addEventListener('beforeunload', ()=>{ stopScanner(); });
  </script>
</body>
</html>
