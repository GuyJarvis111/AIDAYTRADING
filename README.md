<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>Pixel-Perfect Image Viewer</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <style>
    :root {
      --bg: #0b0d10;
      --fg: #e7e9ee;
      --muted: #9aa3ad;
      --accent: #5aa9ff;
      --border: #1a2028;
      --radius: 14px;
      --shadow: 0 10px 30px rgba(0,0,0,.35);
    }
    * { box-sizing: border-box; }
    html, body {
      height: 100%;
      margin: 0;
      background: radial-gradient(1200px 800px at 20% 10%, #11141a 0%, #0b0d10 60%, #0b0d10 100%);
      color: var(--fg);
      font: 14px/1.4 system-ui, -apple-system, Segoe UI, Roboto, Inter, "Helvetica Neue", Arial, "Apple Color Emoji", "Segoe UI Emoji";
    }
    .wrap {
      min-height: 100%;
      display: grid;
      grid-template-rows: auto 1fr;
      gap: 14px;
      padding: 24px;
    }
    header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 12px;
      padding: 14px 18px;
      border: 1px solid var(--border);
      background: rgba(15,18,24,.6);
      backdrop-filter: blur(8px);
      border-radius: var(--radius);
      box-shadow: var(--shadow);
    }
    header h1 {
      font-size: 16px;
      margin: 0;
      letter-spacing: .3px;
      font-weight: 600;
    }
    .controls {
      display: flex;
      align-items: center;
      gap: 10px;
      flex-wrap: wrap;
    }
    .btn, label.file {
      display: inline-flex;
      align-items: center;
      gap: 8px;
      padding: 9px 12px;
      border-radius: 10px;
      border: 1px solid var(--border);
      background: #121620;
      color: var(--fg);
      text-decoration: none;
      cursor: pointer;
      user-select: none;
      box-shadow: inset 0 1px 0 rgba(255,255,255,.02);
    }
    .btn:hover, label.file:hover { border-color: #2a3240; }
    .btn:active, label.file:active { transform: translateY(1px); }
    .btn[disabled] { opacity: .6; cursor: not-allowed; }
    label.file input { display: none; }

    .stage {
      position: relative;
      border: 1px dashed #2a3240;
      border-radius: var(--radius);
      background:
        linear-gradient(45deg, rgba(255,255,255,.04) 25%, transparent 25%) -10px 0/20px 20px,
        linear-gradient(-45deg, rgba(255,255,255,.04) 25%, transparent 25%) -10px 0/20px 20px,
        linear-gradient(45deg, transparent 75%, rgba(255,255,255,.04) 75%)/20px 20px,
        linear-gradient(-45deg, transparent 75%, rgba(255,255,255,.04) 75%)/20px 20px,
        #0f1420;
      box-shadow: var(--shadow);
      overflow: auto;
      display: grid;
      place-items: center;
      padding: 18px;
      outline: none;
    }
    .stage.drag {
      border-style: solid;
      border-color: var(--accent);
      box-shadow: 0 0 0 4px rgba(90,169,255,.2), var(--shadow);
    }
    figure {
      margin: 0;
      position: relative;
      display: grid;
      place-items: center;
    }

    /* Native-pixel rendering */
    img.native {
      image-rendering: -moz-crisp-edges;
      image-rendering: pixelated; /* Chrome/Edge/Firefox */
      image-rendering: crisp-edges; /* Safari fallback */
      max-width: none;    /* prevent browser from shrinking */
      max-height: none;   /* prevent browser from shrinking */
      width: auto;        /* use intrinsic width */
      height: auto;       /* use intrinsic height */
      display: block;
      box-shadow: 0 0 0 1px rgba(255,255,255,.05);
    }

    /* Fit-to-window mode */
    .fit .native {
      image-rendering: auto;
      max-width: calc(100% - 2px);
      max-height: calc(100% - 2px);
    }

    .meta {
      position: absolute;
      bottom: 10px;
      left: 12px;
      font-size: 12px;
      color: var(--muted);
      background: rgba(13,16,22,.7);
      padding: 6px 8px;
      border-radius: 8px;
      border: 1px solid var(--border);
    }

    .hint {
      text-align: center;
      color: var(--muted);
      font-size: 13px;
      line-height: 1.6;
      padding: 40px 20px;
    }
    kbd {
      font: 12px/1.2 ui-monospace, SFMono-Regular, Menlo, Consolas, "Liberation Mono", monospace;
      background: #0f1520;
      border: 1px solid var(--border);
      border-bottom-color: #1e2632;
      padding: 2px 6px;
      border-radius: 6px;
      box-shadow: inset 0 -1px 0 rgba(0,0,0,.15);
      color: var(--fg);
    }

    @media print {
      body { background: white; }
      header { display: none; }
      .stage {
        border: none;
        box-shadow: none;
        background: white !important;
        overflow: visible;
        padding: 0;
      }
      .meta { display: none; }
      .fit .native {
        max-width: 100%;
        max-height: 100%;
      }
    }
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <h1>Pixel-Perfect Image Viewer</h1>
      <div class="controls">
        <label class="file">
          <input id="file" type="file" accept="image/*" />
          Choose image…
        </label>
        <button id="toggleFit" class="btn" disabled>Fit to window</button>
        <button id="download" class="btn" disabled>Download copy</button>
        <button id="print" class="btn" disabled>Print</button>
      </div>
    </header>

    <main id="stage" class="stage" tabindex="0" aria-label="Drop an image here">
      <div id="hint" class="hint">
        <p>Drag & drop your image here or click <em>Choose image…</em></p>
        <p>
          Default view is <strong>1:1 native pixels</strong>. Toggle <kbd>F</kbd> or the button to switch between
          <em>fit to window</em> and <em>exact size</em>.
        </p>
        <p>Pan with scrollbars. Press <kbd>Ctrl/Cmd +</kbd> / <kbd>−</kbd> if you want browser zoom (not exact).</p>
      </div>
      <figure id="figure" aria-live="polite"></figure>
      <div id="meta" class="meta" hidden></div>
    </main>
  </div>

  <script>
    const fileInput = document.getElementById('file');
    const stage = document.getElementById('stage');
    const figure = document.getElementById('figure');
    const hint = document.getElementById('hint');
    const meta = document.getElementById('meta');
    const toggleFitBtn = document.getElementById('toggleFit');
    const downloadBtn = document.getElementById('download');
    const printBtn = document.getElementById('print');

    let fit = false;
    let currentBlobUrl = null;
    let currentFileName = null;

    function setFitMode(next) {
      fit = next;
      stage.classList.toggle('fit', fit);
      toggleFitBtn.textContent = fit ? 'Exact size (1:1)' : 'Fit to window';
    }

    function setImage(src, fileName) {
      // Clear previous
      figure.innerHTML = '';
      if (currentBlobUrl) URL.revokeObjectURL(currentBlobUrl);
      currentBlobUrl = null;
      currentFileName = fileName || null;

      // Create <img>
      const img = new Image();
      img.className = 'native';
      img.alt = fileName || 'Image';
      img.onload = () => {
        hint.hidden = true;
        meta.hidden = false;
        meta.textContent = `${img.naturalWidth} × ${img.naturalHeight}px` + (fileName ? ` • ${fileName}` : '');
        toggleFitBtn.disabled = false;
        downloadBtn.disabled = false;
        printBtn.disabled = false;
      };
      img.src = src;
      figure.appendChild(img);
    }

    function handleFiles(files) {
      if (!files || !files[0]) return;
      const file = files[0];
      currentBlobUrl = URL.createObjectURL(file);
      setImage(currentBlobUrl, file.name);
    }

    // Drag & drop
    stage.addEventListener('dragover', (e) => {
      e.preventDefault();
      stage.classList.add('drag');
    });
    stage.addEventListener('dragleave', () => stage.classList.remove('drag'));
    stage.addEventListener('drop', (e) => {
      e.preventDefault();
      stage.classList.remove('drag');
      if (e.dataTransfer.files && e.dataTransfer.files.length) {
        handleFiles(e.dataTransfer.files);
      } else {
        const url = e.dataTransfer.getData('text/uri-list') || e.dataTransfer.getData('text/plain');
        if (url) setImage(url);
      }
    });

    // File picker
    fileInput.addEventListener('change', (e) => handleFiles(e.target.files));

    // Keyboard toggle (F)
    window.addEventListener('keydown', (e) => {
      if (e.key.toLowerCase() === 'f') {
        setFitMode(!fit);
      }
    });

    toggleFitBtn.addEventListener('click', () => setFitMode(!fit));

    downloadBtn.addEventListener('click', async () => {
      try {
        let blob;
        if (currentBlobUrl) {
          blob = await (await fetch(currentBlobUrl)).blob();
        } else {
          const img = figure.querySelector('img');
          const res = await fetch(img.src, { mode: 'cors' });
          blob = await res.blob();
        }
        const a = document.createElement('a');
        a.href = URL.createObjectURL(blob);
        a.download = currentFileName || 'image';
        document.body.appendChild(a);
        a.click();
        a.remove();
      } catch (err) {
        alert('Could not download this image (CORS or data URL).');
      }
    });

    printBtn.addEventListener('click', () => {
      // For printing, switch to fit mode so it scales to page neatly
      const wasFit = fit;
      setFitMode(true);
      setTimeout(() => {
        window.print();
        setTimeout(() => setFitMode(wasFit), 50);
      }, 30);
    });

    // Optional: if you want to “hardcode” a specific image file you already have locally,
    // uncomment the next line and replace with your path:
    // setImage('./YOUR_IMAGE_FILENAME.png', 'YOUR_IMAGE_FILENAME.png');
  </script>
</body>
</html>
