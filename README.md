<!doctype html>
<html lang="uz">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Mini App — Foto → QR</title>
  <style>
    body{font-family:system-ui,-apple-system,Segoe UI,Roboto; margin:12px; color:#111}
    h1{font-size:18px;margin-bottom:6px}
    .row{display:flex;gap:8px;align-items:center;flex-wrap:wrap}
    #viewer{width:320px;height:420px;border:1px solid #ddd;display:flex;align-items:center;justify-content:center;overflow:hidden;position:relative;background:#fafafa}
    #photoCanvas{transform-origin:center center; touch-action:none; background:#fff}
    .controls{margin-top:8px;display:flex;gap:8px;flex-wrap:wrap}
    button{padding:8px 10px;border-radius:8px;border:1px solid #ccc;background:#fff;cursor:pointer}
    input[type="range"]{width:180px}
    #qrholder{margin-top:12px}
    #loader{display:none}
    .small{font-size:12px;color:#555}
  </style>
</head>
<body>
  <h1>Mini-app: rasm oling/sozlang → QR yarating</h1>

  <div class="row">
    <input id="fileInput" type="file" accept="image/*" />
    <button id="camBtn">Kameradan olish</button>
    <button id="autoFaceBtn">Auto-Face (markaz)</button>
    <button id="cropBtn">Crop va Saqlash</button>
  </div>

  <div id="viewer" style="margin-top:10px;">
    <canvas id="photoCanvas" width="320" height="420"></canvas>
    <video id="video" style="display:none; position:absolute; inset:0; width:100%; height:100%; object-fit:cover"></video>
  </div>

  <div class="controls">
    <label class="small">Zoom</label>
    <input id="zoomRange" type="range" min="0.5" max="3" step="0.01" value="1">
    <label class="small">Rotate</label>
    <button id="rotLeft">⟲</button>
    <button id="rotRight">⟳</button>
    <label style="margin-left:8px;">
      <input id="centerToggle" type="checkbox" checked/> Fit to view
    </label>
  </div>

  <div style="margin-top:10px;">
    <input id="qrText" placeholder="QR uchun matn yoki URL (masalan: https://example.com)" style="width:60%;" />
    <label style="margin-left:8px"><input id="encodeImage" type="checkbox" /> Rasmni base64 sifatida QRga joylash (kattaligi muhim)</label>
    <button id="genQrBtn">QR yarat</button>
  </div>

  <div id="qrholder"></div>
  <div id="loader">Model yuklanmoqda, iltimos bir necha soniya kuting...</div>

  <!-- kutubxonalar: face-api va qrcodejs -->
  <script src="https://unpkg.com/face-api.js@0.22.2/dist/face-api.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>

  <script>
    // --- Telegram WebApp init (agar WebApp ichida ochilsa) ---
    try {
      if (window.Telegram && window.Telegram.WebApp) {
        window.Telegram.WebApp.init();
      }
    } catch(e){ console.log('tg init err', e) }

    // --- DOM refs ---
    const fileInput = document.getElementById('fileInput');
    const camBtn = document.getElementById('camBtn');
    const video = document.getElementById('video');
    const canvas = document.getElementById('photoCanvas');
    const ctx = canvas.getContext('2d');
    const zoomRange = document.getElementById('zoomRange');
    const rotLeft = document.getElementById('rotLeft');
    const rotRight = document.getElementById('rotRight');
    const autoFaceBtn = document.getElementById('autoFaceBtn');
    const cropBtn = document.getElementById('cropBtn');
    const genQrBtn = document.getElementById('genQrBtn');
    const qrHolder = document.getElementById('qrholder');
    const qrText = document.getElementById('qrText');
    const encodeImage = document.getElementById('encodeImage');
    const loader = document.getElementById('loader');
    const centerToggle = document.getElementById('centerToggle');

    // state
    let img = new Image();
    let scale = 1;
    let rotation = 0; // degrees
    let offsetX = 0, offsetY = 0;
    let isPanning = false, startPan = null;
    let stream = null;

    // draw function
    function redraw() {
      ctx.clearRect(0,0,canvas.width,canvas.height);
      if (!img.src) {
        ctx.fillStyle = '#f5f5f5';
        ctx.fillRect(0,0,canvas.width,canvas.height);
        ctx.fillStyle = '#999';
        ctx.font = '14px sans-serif';
        ctx.textAlign = 'center';
        ctx.fillText('Rasm yuklanmadi', canvas.width/2, canvas.height/2);
        return;
      }
      // save + center transform
      ctx.save();
      ctx.translate(canvas.width/2 + offsetX, canvas.height/2 + offsetY);
      ctx.rotate(rotation * Math.PI/180);
      ctx.scale(scale, scale);

      // choose image draw size to preserve aspect
      const iw = img.width, ih = img.height;
      const ratio = Math.min( (canvas.width*0.9)/iw, (canvas.height*0.9)/ih );
      const drawW = iw * ratio;
      const drawH = ih * ratio;
      ctx.drawImage(img, -drawW/2, -drawH/2, drawW, drawH);
      ctx.restore();
    }

    // load file
    fileInput.addEventListener('change', (e)=>{
      const f = e.target.files && e.target.files[0];
      if (!f) return;
      const url = URL.createObjectURL(f);
      stopCamera();
      img = new Image();
      img.onload = ()=> {
        scale = 1; rotation = 0; offsetX=0; offsetY=0;
        redraw();
      };
      img.src = url;
    });

    // camera
    camBtn.addEventListener('click', async ()=>{
      try {
        stopCamera();
        stream = await navigator.mediaDevices.getUserMedia({video:{facingMode:'environment'}, audio:false});
        video.srcObject = stream;
        video.play();
        video.style.display = 'block';
        // show live and capture after click
        camBtn.textContent = 'Yozib olish (bosish rasm olish uchun)';
        camBtn.onclick = () => {
          // draw current video frame to canvas (as image)
          ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
          // convert to image
          const data = canvas.toDataURL('image/jpeg');
          img = new Image();
          img.onload = ()=> { video.style.display='none'; stopCamera(); redraw(); camBtn.textContent='Kameradan olish'; camBtn.onclick = cameraDefault; }
          img.src = data;
        };
      } catch(err){ alert('Kamera ochilmadi: '+err.message); }
    });
    function cameraDefault(){ /* placeholder */ }

    function stopCamera(){
      if (stream){
        stream.getTracks().forEach(t=>t.stop());
        stream = null;
      }
      video.pause(); video.srcObject = null;
      video.style.display='none';
      camBtn.textContent='Kameradan olish';
      camBtn.onclick = cameraDefault;
    }

    // zoom
    zoomRange.addEventListener('input', (e)=>{
      scale = parseFloat(e.target.value);
      redraw();
    });

    // rotate
    rotLeft.addEventListener('click', ()=>{ rotation = (rotation-90)%360; redraw(); });
    rotRight.addEventListener('click', ()=>{ rotation = (rotation+90)%360; redraw(); });

    // panning (mouse / touch)
    canvas.addEventListener('pointerdown', (e)=>{
      isPanning = true;
      startPan = {x:e.clientX, y:e.clientY};
      canvas.setPointerCapture(e.pointerId);
    });
    canvas.addEventListener('pointermove', (e)=>{
      if (!isPanning) return;
      const dx = e.clientX - startPan.x;
      const dy = e.clientY - startPan.y;
      offsetX += dx; offsetY += dy;
      startPan = {x:e.clientX, y:e.clientY};
      redraw();
    });
    canvas.addEventListener('pointerup', (e)=>{ isPanning=false; canvas.releasePointerCapture(e.pointerId); });

    // face-api load models (tiny face detector)
    async function loadModels(){
      loader.style.display='block';
      try {
        const MODEL_URL = 'https://raw.githubusercontent.com/justadudewhohacks/face-api.js/master/weights';
        // use CDN models host — smaller tiny face detector
        await faceapi.nets.tinyFaceDetector.loadFromUri(MODEL_URL).catch(()=>{ /* fallback */});
      } catch(e){ console.warn('model load err', e) }
      loader.style.display='none';
    }
    loadModels();

    // auto-face: detect and center
    autoFaceBtn.addEventListener('click', async ()=>{
      if (!img.src) { alert('Avvalo rasm yuklang.'); return; }
      loader.style.display='block';
      // draw current state into a tmp canvas and run detection
      const tmp = document.createElement('canvas');
      tmp.width = img.width; tmp.height = img.height;
      const tctx = tmp.getContext('2d');
      tctx.drawImage(img,0,0);
      // run tiny face detector
      const detections = await faceapi.detectAllFaces(tmp, new faceapi.TinyFaceDetectorOptions());
      loader.style.display='none';
      if (!detections || detections.length===0){ alert('Yuz topilmadi.'); return; }
      // pick largest box
      const box = detections.reduce((a,b)=> (a.box.width*a.box.height > b.box.width*b.box.height)?a:b).box;
      // compute face center in image coords
      const faceCx = box.x + box.width/2;
      const faceCy = box.y + box.height/2;
      // Map image coords to canvas transform to center face
      // We used drawW/drawH ratio in redraw; reuse logic to compute offsets
      const iw = img.width, ih = img.height;
      const ratio = Math.min( (canvas.width*0.9)/iw, (canvas.height*0.9)/ih );
      const drawW = iw * ratio;
      const drawH = ih * ratio;
      // After scaling by 'scale', the pixel of face on canvas (before rotation) is:
      const sx = (faceCx - iw/2) * ratio * scale;
      const sy = (faceCy - ih/2) * ratio * scale;
      // To put this point at canvas center, set offset opposite of sx,sy
      offsetX = -sx;
      offsetY = -sy;
      // optionally reset rotation
      if (centerToggle.checked) rotation = 0;
      redraw();
    });

    // crop to current view and replace image with cropped image
    cropBtn.addEventListener('click', ()=>{
      // create temporary canvas of visible area and draw current transformed image there
      const tmp = document.createElement('canvas');
      tmp.width = canvas.width; tmp.height = canvas.height;
      const tctx = tmp.getContext('2d');
      // draw background white
      tctx.fillStyle = '#fff'; tctx.fillRect(0,0,tmp.width,tmp.height);

      // replicate the same transform as redraw
      tctx.save();
      tctx.translate(tmp.width/2 + offsetX, tmp.height/2 + offsetY);
      tctx.rotate(rotation * Math.PI/180);
      tctx.scale(scale, scale);

      const iw = img.width, ih = img.height;
      const ratio = Math.min( (canvas.width*0.9)/iw, (canvas.height*0.9)/ih );
      const drawW = iw * ratio;
      const drawH = ih * ratio;
      tctx.drawImage(img, -drawW/2, -drawH/2, drawW, drawH);
      tctx.restore();

      // replace image with cropped
      const data = tmp.toDataURL('image/jpeg', 0.9);
      img = new Image();
      img.onload = ()=> {
        scale = 1; rotation = 0; offsetX=0; offsetY=0;
        redraw();
      };
      img.src = data;
    });

    // generate QR
    genQrBtn.addEventListener('click', ()=>{
      qrHolder.innerHTML = '';
      const text = (qrText.value || '').trim();
      const doImage = encodeImage.checked;

      if (!text && !doImage) { alert('QR uchun matn yoki "Rasmni base64 sifatida" ni tanlang.'); return; }

      if (doImage) {
        // small size check: convert current canvas to compressed dataURL and use as payload
        const dataUrl = canvas.toDataURL('image/jpeg', 0.6);
        // check length (base64 string length)
        if (dataUrl.length > 2000) {
          // warn user: large
          if (!confirm('Rasm juda katta bo‘lishi mumkin va QR o‘qilmasligi mumkin. Davom etilsinmi?')) return;
        }
        createQR(dataUrl);
      } else {
        createQR(text);
      }
    });

    function createQR(payload){
      qrHolder.innerHTML = '';
      const qdiv = document.createElement('div');
      qdiv.id = 'qrcode';
      qrHolder.appendChild(qdiv);

      // use QRCode library
      try {
        new QRCode(qdiv, {
          text: payload,
          width: 240,
          height: 240,
          correctLevel: QRCode.CorrectLevel.L
        });
      } catch(e){
        qdiv.innerText = 'QR hosil qilishda xato: '+e;
      }

      // add save button
      const saveBtn = document.createElement('button');
      saveBtn.textContent = 'QRni yuklab olish';
      saveBtn.onclick = ()=> {
        // convert qrcode canvas/svg to image and download
        const qrCanvas = qdiv.querySelector('canvas');
        if (qrCanvas) {
          const a = document.createElement('a');
          a.href = qrCanvas.toDataURL('image/png');
          a.download = 'qr.png';
          a.click();
        } else {
          alert('QR topilmadi');
        }
      };
      qrHolder.appendChild(document.createElement('br'));
      qrHolder.appendChild(saveBtn);
    }

    // initial empty redraw
    redraw();

    // optional: adapt size on orientation change
    window.addEventListener('resize', ()=> { redraw(); });
  </script>
</body>
</html>
