<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Chill Guy — Immersive 3D Spatial Viewer | Steven Valentino</title>
  <meta name="description" content="Interactive 3D GLB / WebGL showcase of Chill Guy by Triative Studio. Featured in Steven Valentino's Spatial & 3D portfolio.">

  <!-- Favicon -->
  <link rel="icon" type="image/jpeg" href="./assets/chill-guy-preview.jpeg">

  <!-- Google Fonts -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500;600;700&family=Space+Grotesk:wght@500;700&display=swap" rel="stylesheet">

  <!-- Sketchfab Viewer API -->
  <script src="https://static.sketchfab.com/api/sketchfab-viewer-1.12.1.js"></script>

  <!-- Google model-viewer for local GLB / AR fallback -->
  <script type="module" src="https://ajax.googleapis.com/ajax/libs/model-viewer/3.5.0/model-viewer.min.js"></script>

  <style>
    :root {
      --bg: #090a0f;
      --surface: rgba(18, 20, 29, 0.7);
      --surface-border: rgba(255, 255, 255, 0.1);
      --surface-hover: rgba(255, 255, 255, 0.12);
      --text: #f5f5f7;
      --text-muted: #8e8ea0;
      --accent: #ffb443;
      --accent-glow: rgba(255, 180, 67, 0.25);
      --accent-cyan: #38bdf8;
      --radius-pill: 9999px;
      --radius-panel: 20px;
      --blur: blur(24px);
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      -webkit-tap-highlight-color: transparent;
    }

    body, html {
      width: 100%;
      height: 100%;
      overflow: hidden;
      background-color: var(--bg);
      color: var(--text);
      font-family: 'Plus Jakarta Sans', -apple-system, BlinkMacSystemFont, sans-serif;
      user-select: none;
    }

    /* Ambient Spatial Lighting */
    .ambient-bg {
      position: absolute;
      inset: 0;
      pointer-events: none;
      z-index: 0;
      background:
        radial-gradient(circle at 50% 30%, rgba(255, 180, 67, 0.08) 0%, transparent 60%),
        radial-gradient(circle at 20% 80%, rgba(56, 189, 248, 0.06) 0%, transparent 50%),
        radial-gradient(circle at 80% 80%, rgba(168, 85, 247, 0.06) 0%, transparent 50%);
    }

    /* Grid overlay for spatial room feel */
    .spatial-grid {
      position: absolute;
      inset: 0;
      pointer-events: none;
      z-index: 1;
      background-image: 
        linear-gradient(to right, rgba(255, 255, 255, 0.02) 1px, transparent 1px),
        linear-gradient(to bottom, rgba(255, 255, 255, 0.02) 1px, transparent 1px);
      background-size: 60px 60px;
      mask-image: radial-gradient(circle at 50% 50%, black 30%, transparent 80%);
      -webkit-mask-image: radial-gradient(circle at 50% 50%, black 30%, transparent 80%);
    }

    /* 3D Viewport container */
    #viewer-container {
      position: absolute;
      inset: 0;
      z-index: 2;
      width: 100%;
      height: 100%;
    }

    #sketchfab-frame {
      width: 100%;
      height: 100%;
      border: 0;
      display: block;
    }

    model-viewer {
      width: 100%;
      height: 100%;
      display: none;
      --poster-color: transparent;
    }

    /* Top Navigation Header */
    .header {
      position: absolute;
      top: 20px;
      left: 20px;
      right: 20px;
      z-index: 10;
      display: flex;
      justify-content: space-between;
      align-items: center;
      pointer-events: none;
    }

    .header > * {
      pointer-events: auto;
    }

    .glass-pill {
      background: var(--surface);
      backdrop-filter: var(--blur);
      -webkit-backdrop-filter: var(--blur);
      border: 1px solid var(--surface-border);
      border-radius: var(--radius-pill);
      padding: 8px 16px;
      display: flex;
      align-items: center;
      gap: 12px;
      box-shadow: 0 10px 30px -5px rgba(0, 0, 0, 0.5);
      transition: all 0.25s cubic-bezier(0.16, 1, 0.3, 1);
      text-decoration: none;
      color: var(--text);
    }

    .glass-pill:hover {
      background: var(--surface-hover);
      border-color: rgba(255, 255, 255, 0.2);
      transform: translateY(-1px);
    }

    .creator-avatar {
      width: 28px;
      height: 28px;
      border-radius: 50%;
      border: 1px solid var(--surface-border);
      object-fit: cover;
    }

    .model-info h1 {
      font-size: 14px;
      font-weight: 600;
      letter-spacing: -0.01em;
      display: flex;
      align-items: center;
      gap: 8px;
    }

    .badge-3d {
      font-size: 10px;
      font-weight: 700;
      text-transform: uppercase;
      padding: 2px 7px;
      border-radius: var(--radius-pill);
      background: linear-gradient(135deg, #ffb443, #ff7e40);
      color: #000;
      letter-spacing: 0.05em;
    }

    .model-info p {
      font-size: 11px;
      color: var(--text-muted);
    }

    .top-actions {
      display: flex;
      gap: 10px;
    }

    .btn-glass {
      background: var(--surface);
      backdrop-filter: var(--blur);
      -webkit-backdrop-filter: var(--blur);
      border: 1px solid var(--surface-border);
      border-radius: var(--radius-pill);
      padding: 8px 14px;
      color: var(--text);
      font-size: 12px;
      font-weight: 500;
      cursor: pointer;
      display: inline-flex;
      align-items: center;
      gap: 8px;
      text-decoration: none;
      box-shadow: 0 8px 24px -4px rgba(0, 0, 0, 0.4);
      transition: all 0.2s ease;
    }

    .btn-glass:hover {
      background: var(--surface-hover);
      border-color: rgba(255, 255, 255, 0.25);
      color: #fff;
    }

    .btn-glass svg {
      width: 14px;
      height: 14px;
      stroke-width: 2;
    }

    /* Floating Bottom Interaction Dock */
    .dock-container {
      position: absolute;
      bottom: 24px;
      left: 50%;
      transform: translateX(-50%);
      z-index: 10;
      display: flex;
      flex-direction: column;
      align-items: center;
      gap: 12px;
      pointer-events: none;
    }

    .dock-container > * {
      pointer-events: auto;
    }

    .dock {
      background: rgba(14, 16, 24, 0.75);
      backdrop-filter: blur(28px);
      -webkit-backdrop-filter: blur(28px);
      border: 1px solid rgba(255, 255, 255, 0.12);
      border-radius: var(--radius-pill);
      padding: 6px 10px;
      display: flex;
      align-items: center;
      gap: 6px;
      box-shadow: 0 20px 40px -10px rgba(0, 0, 0, 0.7);
    }

    .dock-btn {
      background: transparent;
      border: none;
      color: var(--text-muted);
      width: 40px;
      height: 40px;
      border-radius: 50%;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: all 0.2s cubic-bezier(0.16, 1, 0.3, 1);
      position: relative;
    }

    .dock-btn svg {
      width: 18px;
      height: 18px;
    }

    .dock-btn:hover {
      color: var(--text);
      background: rgba(255, 255, 255, 0.08);
      transform: scale(1.08);
    }

    .dock-btn.active {
      color: var(--accent);
      background: rgba(255, 180, 67, 0.15);
    }

    .dock-divider {
      width: 1px;
      height: 20px;
      background: rgba(255, 255, 255, 0.1);
      margin: 0 2px;
    }

    /* Tooltip */
    .dock-btn::after {
      content: attr(data-tooltip);
      position: absolute;
      bottom: calc(100% + 12px);
      left: 50%;
      transform: translateX(-50%) translateY(4px);
      background: rgba(12, 14, 20, 0.92);
      backdrop-filter: blur(12px);
      border: 1px solid rgba(255, 255, 255, 0.15);
      padding: 4px 10px;
      font-size: 11px;
      font-weight: 500;
      color: #fff;
      border-radius: 6px;
      white-space: nowrap;
      pointer-events: none;
      opacity: 0;
      transition: all 0.18s ease;
      box-shadow: 0 6px 16px rgba(0,0,0,0.5);
    }

    .dock-btn:hover::after {
      opacity: 1;
      transform: translateX(-50%) translateY(0);
    }

    /* Spatial Gestures Helper Pill */
    .gesture-hint {
      font-size: 11px;
      color: rgba(255, 255, 255, 0.45);
      background: rgba(0, 0, 0, 0.4);
      backdrop-filter: blur(8px);
      padding: 4px 12px;
      border-radius: var(--radius-pill);
      border: 1px solid rgba(255, 255, 255, 0.05);
      display: flex;
      align-items: center;
      gap: 6px;
    }

    .gesture-hint span {
      display: inline-block;
      width: 6px;
      height: 6px;
      border-radius: 50%;
      background: #10b981;
    }

    /* Drop Zone / Overlay */
    .drop-overlay {
      position: absolute;
      inset: 20px;
      border: 2px dashed rgba(255, 180, 67, 0.5);
      border-radius: var(--radius-panel);
      background: rgba(9, 10, 15, 0.85);
      backdrop-filter: blur(20px);
      z-index: 100;
      display: none;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      gap: 16px;
      pointer-events: none;
    }

    .drop-overlay.active {
      display: flex;
    }

    .drop-icon {
      width: 64px;
      height: 64px;
      border-radius: 50%;
      background: rgba(255, 180, 67, 0.12);
      color: var(--accent);
      display: flex;
      align-items: center;
      justify-content: center;
    }

    /* QR / AR Modal */
    .modal-backdrop {
      position: fixed;
      inset: 0;
      background: rgba(0, 0, 0, 0.75);
      backdrop-filter: blur(16px);
      z-index: 1000;
      display: none;
      align-items: center;
      justify-content: center;
      padding: 20px;
    }

    .modal-backdrop.open {
      display: flex;
    }

    .modal-content {
      background: rgba(18, 20, 30, 0.95);
      border: 1px solid rgba(255, 255, 255, 0.15);
      border-radius: 24px;
      padding: 28px;
      max-width: 380px;
      width: 100%;
      text-align: center;
      box-shadow: 0 25px 60px -15px rgba(0, 0, 0, 0.8);
      position: relative;
    }

    .modal-close {
      position: absolute;
      top: 16px;
      right: 16px;
      background: transparent;
      border: none;
      color: var(--text-muted);
      cursor: pointer;
      width: 32px;
      height: 32px;
      border-radius: 50%;
      display: flex;
      align-items: center;
      justify-content: center;
    }

    .modal-close:hover {
      background: rgba(255, 255, 255, 0.1);
      color: #fff;
    }

    .qr-frame {
      background: #fff;
      padding: 16px;
      border-radius: 16px;
      display: inline-block;
      margin: 16px 0;
    }

    .qr-frame img {
      width: 180px;
      height: 180px;
      display: block;
    }

    /* Responsive */
    @media (max-width: 640px) {
      .header {
        top: 14px;
        left: 14px;
        right: 14px;
        flex-direction: column;
        align-items: flex-start;
        gap: 10px;
      }
      .top-actions {
        align-self: flex-end;
      }
      .dock-container {
        bottom: 16px;
      }
      .gesture-hint {
        display: none;
      }
    }
  </style>
</head>
<body>

  <div class="ambient-bg"></div>
  <div class="spatial-grid"></div>

  <!-- Top Bar -->
  <header class="header">
    <a href="https://github.com/sevet15" class="glass-pill" title="Steven Valentino GitHub Profile">
      <img src="./assets/chill-guy-preview.jpeg" alt="Chill Guy" class="creator-avatar">
      <div class="model-info">
        <h1>Chill Guy <span class="badge-3d">3D GLB</span></h1>
        <p>Spatial Asset · Triative Studio</p>
      </div>
    </a>

    <div class="top-actions">
      <a href="https://skfb.ly/p9DrQ" target="_blank" rel="noopener noreferrer" class="btn-glass" title="Open source model on Sketchfab">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor"><path d="M18 13v6a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h6"></path><polyline points="15 3 21 3 21 9"></polyline><line x1="10" y1="14" x2="21" y2="3"></line></svg>
        <span>Sketchfab</span>
      </a>

      <a href="https://sevetportfolio.vercel.app" target="_blank" rel="noopener noreferrer" class="btn-glass" title="Visit Steven Valentino's Design Portfolio">
        <span>Portfolio ↗</span>
      </a>
    </div>
  </header>

  <!-- 3D Viewport -->
  <main id="viewer-container">
    <!-- Sketchfab WebGL Embed Frame -->
    <iframe 
      id="sketchfab-frame"
      src="https://sketchfab.com/models/a6ff1cc81fb5436097e4e2319a25c32d/embed?autostart=1&autospin=0.3&camera=0&preload=1&ui_controls=1&ui_infos=0&ui_inspector=0&ui_stop=0&ui_watermark=0&ui_hint=2"
      allow="autoplay; fullscreen; xr-spatial-tracking" 
      execution-while-out-of-viewport 
      execution-while-not-rendered 
      web-share 
      allowfullscreen 
      mozallowfullscreen="true" 
      webkitallowfullscreen="true">
    </iframe>

    <!-- Model Viewer for Custom GLB Drag & Drop Fallback -->
    <model-viewer 
      id="local-model-viewer" 
      camera-controls 
      auto-rotate 
      ar 
      shadow-intensity="1.5" 
      exposure="1" 
      shadow-softness="0.8">
    </model-viewer>
  </main>

  <!-- Bottom Interaction Dock -->
  <div class="dock-container">
    <div class="gesture-hint">
      <span></span> Drag to orbit · Scroll to zoom · Two-finger pan
    </div>

    <div class="dock">
      <button class="dock-btn active" id="btn-spin" data-tooltip="Auto-Rotate">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21.5 2v6h-6M21.34 15.57a10 10 0 1 1-.57-8.38l5.67-5.67"/></svg>
      </button>

      <button class="dock-btn" id="btn-reset" data-tooltip="Reset Camera">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="12" r="10"/><path d="M12 8v8M8 12h8"/></svg>
      </button>

      <div class="dock-divider"></div>

      <button class="dock-btn" id="btn-upload" data-tooltip="Load Custom GLB">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4M17 8l-5-5-5 5M12 3v12"/></svg>
      </button>
      <input type="file" id="file-input" accept=".glb,.gltf" style="display: none;">

      <button class="dock-btn" id="btn-ar" data-tooltip="Augmented Reality / Mobile">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M5 3H3v2M19 3h2v2M5 21H3v-2M19 21h2v-2M12 7l5 3v6l-5 3-5-3v-6l5-3z"/></svg>
      </button>

      <div class="dock-divider"></div>

      <button class="dock-btn" id="btn-fullscreen" data-tooltip="Toggle Fullscreen">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M8 3H5a2 2 0 0 0-2 2v3m18 0V5a2 2 0 0 0-2-2h-3m0 18h3a2 2 0 0 0 2-2v-3M3 16v3a2 2 0 0 0 2 2h3"/></svg>
      </button>
    </div>
  </div>

  <!-- Drag & Drop Overlay Zone -->
  <div class="drop-overlay" id="drop-overlay">
    <div class="drop-icon">
      <svg width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4M17 8l-5-5-5 5M12 3v12"/></svg>
    </div>
    <div style="text-align: center;">
      <h2 style="font-size: 18px; margin-bottom: 6px;">Drop your .GLB or .GLTF model</h2>
      <p style="font-size: 13px; color: var(--text-muted);">Inspect your custom 3D files inside the spatial viewport</p>
    </div>
  </div>

  <!-- AR / QR Code Modal -->
  <div class="modal-backdrop" id="ar-modal">
    <div class="modal-content">
      <button class="modal-close" id="modal-close">&times;</button>
      <h2 style="font-size: 18px; font-weight: 700; margin-bottom: 6px;">Experience in AR</h2>
      <p style="font-size: 13px; color: var(--text-muted);">Scan with your iPhone, iPad, or Android camera to place Chill Guy in your physical space.</p>
      
      <div class="qr-frame">
        <!-- Live QR Code linking to the Sketchfab AR / WebGL experience -->
        <img id="qr-img" src="https://api.qrserver.com/v1/create-qr-code/?size=200x200&data=https://sketchfab.com/3d-models/chill-guy-a6ff1cc81fb5436097e4e2319a25c32d" alt="AR QR Code">
      </div>

      <div style="display: flex; gap: 8px; justify-content: center;">
        <a href="https://sketchfab.com/3d-models/chill-guy-a6ff1cc81fb5436097e4e2319a25c32d" target="_blank" rel="noopener noreferrer" class="btn-glass" style="width: 100%; justify-content: center;">
          Open Mobile AR ↗
        </a>
      </div>
    </div>
  </div>

  <script>
    // Elements
    const sfIframe = document.getElementById('sketchfab-frame');
    const localViewer = document.getElementById('local-model-viewer');
    const btnSpin = document.getElementById('btn-spin');
    const btnReset = document.getElementById('btn-reset');
    const btnUpload = document.getElementById('btn-upload');
    const fileInput = document.getElementById('file-input');
    const btnAr = document.getElementById('btn-ar');
    const btnFullscreen = document.getElementById('btn-fullscreen');
    const arModal = document.getElementById('ar-modal');
    const modalClose = document.getElementById('modal-close');
    const dropOverlay = document.getElementById('drop-overlay');

    let sketchfabApi = null;
    let isSpinning = true;
    let isUsingLocalModel = false;

    // Initialize Sketchfab Viewer API
    if (window.Sketchfab) {
      const client = new window.Sketchfab(sfIframe);
      client.init('a6ff1cc81fb5436097e4e2319a25c32d', {
        autostart: 1,
        autospin: 0.3,
        camera: 0,
        preload: 1,
        ui_controls: 1,
        ui_infos: 0,
        ui_watermark: 0,
        ui_inspector: 0,
        success: function(api) {
          sketchfabApi = api;
          api.start();
          api.addEventListener('viewerready', function() {
            console.log('Sketchfab 3D Scene Ready');
          });
        },
        error: function() {
          console.warn('Sketchfab API init error - using default iframe embed');
        }
      });
    }

    // Toggle Auto-Rotate
    btnSpin.addEventListener('click', () => {
      isSpinning = !isSpinning;
      btnSpin.classList.toggle('active', isSpinning);

      if (isUsingLocalModel) {
        localViewer.autoRotate = isSpinning;
      } else if (sketchfabApi) {
        sketchfabApi.setAutorotate(isSpinning ? 0.3 : 0);
      }
    });

    // Reset Camera
    btnReset.addEventListener('click', () => {
      if (isUsingLocalModel) {
        localViewer.cameraOrbit = "0deg 75deg 105%";
        localViewer.resetTurntableRotation();
      } else if (sketchfabApi) {
        sketchfabApi.recenterCamera(() => {
          console.log('Camera recentered');
        });
      } else {
        // Reload iframe to restore default view
        const currentSrc = sfIframe.src;
        sfIframe.src = currentSrc;
      }
    });

    // Custom GLB File Upload
    btnUpload.addEventListener('click', () => fileInput.click());

    fileInput.addEventListener('change', (e) => {
      const file = e.target.files[0];
      if (file) loadLocalFile(file);
    });

    // Drag and Drop Handling
    window.addEventListener('dragover', (e) => {
      e.preventDefault();
      dropOverlay.classList.add('active');
    });

    window.addEventListener('dragleave', (e) => {
      if (e.relatedTarget === null) {
        dropOverlay.classList.remove('active');
      }
    });

    window.addEventListener('drop', (e) => {
      e.preventDefault();
      dropOverlay.classList.remove('active');
      if (e.dataTransfer.files && e.dataTransfer.files[0]) {
        loadLocalFile(e.dataTransfer.files[0]);
      }
    });

    function loadLocalFile(file) {
      if (!file.name.match(/\.(glb|gltf)$/i)) {
        alert('Please drop a valid .GLB or .GLTF 3D model file.');
        return;
      }

      const fileUrl = URL.createObjectURL(file);
      sfIframe.style.display = 'none';
      localViewer.style.display = 'block';
      localViewer.src = fileUrl;
      isUsingLocalModel = true;

      // Update badge text
      const badge = document.querySelector('.badge-3d');
      if (badge) badge.textContent = 'Custom GLB';
      const title = document.querySelector('.model-info h1');
      if (title) title.firstChild.textContent = file.name.replace(/\.[^/.]+$/, '') + ' ';
    }

    // AR / QR Modal
    btnAr.addEventListener('click', () => {
      arModal.classList.add('open');
    });

    modalClose.addEventListener('click', () => {
      arModal.classList.remove('open');
    });

    arModal.addEventListener('click', (e) => {
      if (e.target === arModal) arModal.classList.remove('open');
    });

    // Fullscreen Toggle
    btnFullscreen.addEventListener('click', () => {
      if (!document.fullscreenElement) {
        document.documentElement.requestFullscreen().catch(err => {
          console.warn('Fullscreen error:', err);
        });
      } else {
        if (document.exitFullscreen) {
          document.exitFullscreen();
        }
      }
    });
  </script>
</body>
</html>
