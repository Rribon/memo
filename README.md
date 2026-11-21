[gemini-code-1795221589514.html](https://github.com/user-attachments/files/35190818/gemini-code-1795221589514.html)
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>나만의 메모장 (펜 필기 지원)</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link rel="stylesheet" as="style" crossorigin href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/static/pretendard.min.css" />
  <style>
    * { box-sizing: border-box; }
    body, html {
      margin: 0;
      padding: 0;
      width: 100vw;
      height: 100vh;
      overflow: hidden;
      font-family: 'Pretendard', -apple-system, BlinkMacSystemFont, sans-serif;
      background-color: #faf9f6;
      color: #1f2937;
    }
    body.dark {
      background-color: #161618;
      color: #e5e7eb;
    }
    #main-notepad {
      position: fixed;
      top: 0;
      left: 0;
      width: 100vw;
      height: 100vh;
      max-width: none;
      max-height: none;
      padding: 24px 28px 84px 28px;
      border: none;
      outline: none;
      resize: none;
      background: transparent;
      font-size: 16px;
      line-height: 1.75;
      color: inherit;
      font-family: inherit;
    }
    #boxes-layer {
      position: fixed;
      inset: 0;
      width: 100vw;
      height: 100vh;
      pointer-events: none;
      z-index: 200;
    }
    .floating-box {
      position: absolute;
      min-width: 160px;
      min-height: 90px;
      border-radius: 12px;
      box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.12), 0 4px 10px -2px rgba(0, 0, 0, 0.06);
      display: flex;
      flex-direction: column;
      overflow: hidden;
      transition: box-shadow 0.15s ease;
      border: 1px solid rgba(0, 0, 0, 0.08);
      pointer-events: auto;
    }
    body.dark .floating-box {
      border-color: rgba(255, 255, 255, 0.12);
      box-shadow: 0 12px 28px -5px rgba(0, 0, 0, 0.45);
    }
    .floating-box.pinned {
      border: 1.5px solid #ef4444;
    }
    .box-header {
      height: 30px;
      padding: 0 8px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      cursor: grab;
      user-select: none;
      background: rgba(0, 0, 0, 0.04);
      border-bottom: 1px solid rgba(0, 0, 0, 0.05);
      font-size: 11px;
    }
    body.dark .box-header {
      background: rgba(255, 255, 255, 0.06);
      border-bottom-color: rgba(255, 255, 255, 0.06);
    }
    .floating-box.pinned .box-header {
      cursor: default;
    }
    .box-header:active {
      cursor: grabbing;
    }
    .box-content {
      flex: 1;
      width: 100%;
      height: 100%;
      position: relative;
      overflow: hidden;
      display: flex;
      flex-direction: column;
    }
    .box-textarea {
      width: 100%;
      height: 100%;
      border: none;
      outline: none;
      resize: none;
      background: transparent;
      padding: 12px;
      font-size: 14px;
      line-height: 1.6;
      color: #1f2937;
      font-family: inherit;
    }
    .box-image-view {
      width: 100%;
      height: 100%;
      object-fit: contain;
      cursor: grab;
      user-select: none;
      display: block;
    }
    .floating-box.pinned .box-image-view {
      cursor: default;
    }
    .box-pdf-view {
      width: 100%;
      height: 100%;
      border: none;
      background: #fff;
    }
    .resize-handle {
      position: absolute;
      right: 0;
      bottom: 0;
      width: 16px;
      height: 16px;
      cursor: nwse-resize;
      background: linear-gradient(135deg, transparent 50%, rgba(100, 116, 139, 0.45) 50%);
      z-index: 10;
    }
    .floating-box.pinned .resize-handle {
      display: none;
    }

    /* 전체 화면 필기 캔버스 레이어 */
    #drawing-canvas {
      position: fixed;
      inset: 0;
      width: 100vw;
      height: 100vh;
      z-index: 4000;
      pointer-events: none;
      touch-action: none;
    }
    #drawing-canvas.active {
      pointer-events: auto;
      cursor: crosshair;
    }

    /* 펜 모드 전용 팔레트 */
    #pen-palette {
      position: fixed;
      bottom: 74px;
      left: 50%;
      transform: translateX(-50%);
      z-index: 5100;
      display: none;
      align-items: center;
      gap: 8px;
      padding: 8px 14px;
      border-radius: 9999px;
      background: rgba(255, 255, 255, 0.95);
      backdrop-filter: blur(12px);
      border: 1px solid rgba(0, 0, 0, 0.12);
      box-shadow: 0 10px 28px rgba(0, 0, 0, 0.14);
      font-size: 13px;
    }
    body.dark #pen-palette {
      background: rgba(34, 34, 38, 0.95);
      border-color: rgba(255, 255, 255, 0.14);
      box-shadow: 0 10px 28px rgba(0, 0, 0, 0.5);
    }
    .pen-mode-btn {
      padding: 4px 10px;
      border-radius: 9999px;
      font-weight: 600;
      cursor: pointer;
      transition: all 0.15s;
      border: 1px solid transparent;
    }
    .pen-mode-btn.selected {
      background: #2563eb;
      color: #ffffff;
    }
    .color-swatch {
      width: 22px;
      height: 22px;
      border-radius: 9999px;
      border: 2px solid rgba(0, 0, 0, 0.15);
      cursor: pointer;
      transition: transform 0.12s;
    }
    .color-swatch:hover {
      transform: scale(1.15);
    }
    .color-swatch.selected {
      outline: 2px solid #2563eb;
      outline-offset: 2px;
    }
    #rgb-picker-wrapper {
      position: relative;
      width: 26px;
      height: 26px;
      border-radius: 9999px;
      overflow: hidden;
      border: 2px solid rgba(0, 0, 0, 0.2);
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      background: conic-gradient(red, yellow, lime, aqua, blue, magenta, red);
    }
    #rgb-color-input {
      position: absolute;
      inset: -6px;
      width: 40px;
      height: 40px;
      opacity: 0;
      cursor: pointer;
    }

    #context-menu {
      position: fixed;
      z-index: 9999;
      min-width: 210px;
      background: rgba(255, 255, 255, 0.96);
      backdrop-filter: blur(10px);
      border: 1px solid rgba(0, 0, 0, 0.1);
      border-radius: 10px;
      box-shadow: 0 10px 25px rgba(0, 0, 0, 0.15);
      padding: 6px;
      display: none;
      font-size: 13px;
      color: #1f2937;
    }
    body.dark #context-menu {
      background: rgba(34, 34, 38, 0.96);
      border-color: rgba(255, 255, 255, 0.12);
      color: #f3f4f6;
    }
    .ctx-item {
      padding: 8px 12px;
      border-radius: 6px;
      cursor: pointer;
      display: flex;
      align-items: center;
      gap: 8px;
    }
    .ctx-item:hover {
      background: rgba(59, 130, 246, 0.12);
      color: #2563eb;
    }
    body.dark .ctx-item:hover {
      background: rgba(96, 165, 250, 0.18);
      color: #60a5fa;
    }
    .toolbar-pill {
      position: fixed;
      bottom: 22px;
      left: 50%;
      transform: translateX(-50%);
      z-index: 5000;
      display: flex;
      align-items: center;
      gap: 6px;
      padding: 6px 14px;
      border-radius: 9999px;
      background: rgba(255, 255, 255, 0.88);
      backdrop-filter: blur(12px);
      border: 1px solid rgba(0, 0, 0, 0.08);
      box-shadow: 0 8px 24px rgba(0, 0, 0, 0.08);
      font-size: 13px;
      opacity: 0.7;
      transition: opacity 0.2s ease;
    }
    .toolbar-pill:hover {
      opacity: 1;
    }
    body.dark .toolbar-pill {
      background: rgba(30, 30, 34, 0.88);
      border-color: rgba(255, 255, 255, 0.1);
      box-shadow: 0 8px 24px rgba(0, 0, 0, 0.4);
    }
    .tool-btn {
      padding: 5px 10px;
      border-radius: 9999px;
      cursor: pointer;
      transition: background 0.15s, color 0.15s;
      white-space: nowrap;
      font-weight: 500;
    }
    .tool-btn:hover {
      background: rgba(0, 0, 0, 0.06);
    }
    body.dark .tool-btn:hover {
      background: rgba(255, 255, 255, 0.1);
    }
    .tool-btn.pen-active {
      background: #2563eb;
      color: #ffffff;
      font-weight: 600;
    }
    #drop-overlay {
      position: fixed;
      inset: 0;
      z-index: 8000;
      background: rgba(59, 130, 246, 0.12);
      border: 3px dashed #3b82f6;
      display: none;
      align-items: center;
      justify-content: center;
      font-size: 20px;
      font-weight: 600;
      color: #2563eb;
      pointer-events: none;
      backdrop-filter: blur(2px);
    }
  </style>
</head>
<body>
  <!-- 전체 배경 메모장 -->
  <textarea id="main-notepad" placeholder="여기에 자유롭게 메모하세요...&#10;&#10;• 글씨를 드래그한 뒤 [우클릭]하면 그림 스티커로 바로 변환할 수 있습니다.&#10;• 사진(JPG, PNG, GIF)이나 PDF 파일을 화면에 끌어다 놓거나 붙여넣기(Ctrl+V) 하세요.&#10;• 하단 [✏️ 펜 모드]를 켜면 메모장, 사진, PDF 위에 직접 펜/형광펜으로 필기할 수 있습니다."></textarea>

  <!-- 플로팅 박스들이 올라가는 레이어 -->
  <div id="boxes-layer"></div>

  <!-- 전체 화면 필기 캔버스 레이어 -->
  <canvas id="drawing-canvas"></canvas>

  <!-- 펜 도구 팔레트 -->
  <div id="pen-palette">
    <button id="mode-pen" class="pen-mode-btn selected">✏️ 펜</button>
    <button id="mode-highlighter" class="pen-mode-btn">🖍️ 형광펜</button>
    <button id="mode-eraser" class="pen-mode-btn">🧹 지우개</button>
    <span class="text-gray-300 dark:text-gray-600">|</span>

    <!-- 기본 프리셋 색상 -->
    <button class="color-swatch selected" data-color="#1f2937" style="background:#1f2937;" title="검정"></button>
    <button class="color-swatch" data-color="#ef4444" style="background:#ef4444;" title="빨강"></button>
    <button class="color-swatch" data-color="#2563eb" style="background:#2563eb;" title="파랑"></button>
    <button class="color-swatch" data-color="#16a34a" style="background:#16a34a;" title="초록"></button>
    <button class="color-swatch" data-color="#eab308" style="background:#eab308;" title="노랑"></button>

    <!-- RGB 자유 색상 선택기 -->
    <div id="rgb-picker-wrapper" title="RGB 색상 직접 선택">
      <input type="color" id="rgb-color-input" value="#1f2937" />
    </div>
    <span id="current-rgb-label" class="text-xs font-mono text-gray-500 dark:text-gray-400">#1F2937</span>

    <span class="text-gray-300 dark:text-gray-600">|</span>
    <!-- 굵기 조절 -->
    <input type="range" id="pen-size" min="1" max="30" value="3" style="width:65px; cursor:pointer;" title="펜 굵기 조절" />
    <span id="pen-size-label" class="text-xs w-6 text-center">3px</span>

    <span class="text-gray-300 dark:text-gray-600">|</span>
    <button id="btn-undo-stroke" class="tool-btn text-xs" title="실행 취소 (Ctrl+Z)">↩️ 취소</button>
    <button id="btn-clear-ink" class="tool-btn text-xs text-red-500" title="필기만 모두 지우기">필기 삭제</button>
  </div>

  <!-- 파일 드래그 앤 드롭 안내 오버레이 -->
  <div id="drop-overlay">사진(JPG, PNG, GIF) 또는 PDF 파일을 여기에 놓으세요</div>

  <!-- 숨겨진 파일 선택 인풋 -->
  <input type="file" id="file-input" accept="image/jpeg,image/png,image/jpg,image/gif,application/pdf" multiple style="display:none;" />

  <!-- 우클릭 컨텍스트 메뉴 -->
  <div id="context-menu">
    <div id="ctx-text-to-img" class="ctx-item" style="display:none;">
      <span>🖼️</span> <span>선택한 글씨를 그림 스티커로 변환</span>
    </div>
    <div id="ctx-text-to-box" class="ctx-item" style="display:none;">
      <span>📝</span> <span>선택한 글씨를 새 텍스트 박스로 분리</span>
    </div>
    <div id="ctx-add-box" class="ctx-item">
      <span>➕</span> <span>여기에 새 텍스트 박스 추가</span>
    </div>
    <div id="ctx-add-file" class="ctx-item">
      <span>📎</span> <span>사진 / PDF 파일 넣기</span>
    </div>
  </div>

  <!-- 하단 유틸리티 바 -->
  <div class="toolbar-pill">
    <span id="save-status" class="text-xs text-gray-400 px-2">저장됨</span>
    <span class="text-gray-300 dark:text-gray-600">|</span>
    <button id="btn-toggle-pen" class="tool-btn">✏️ 펜 모드</button>
    <button id="btn-add-box" class="tool-btn">+ 텍스트 박스</button>
    <button id="btn-add-file" class="tool-btn">+ 사진/PDF</button>
    <button id="btn-theme" class="tool-btn" title="테마 전환">🌓</button>
    <button id="btn-clear" class="tool-btn text-red-500">전체 초기화</button>
  </div>

  <script>
    (function () {
      const STORAGE_KEY_TEXT = 'my_memo_main_text_v4';
      const STORAGE_KEY_BOXES = 'my_memo_boxes_v4';
      const STORAGE_KEY_THEME = 'my_memo_theme_v4';
      const STORAGE_KEY_INK = 'my_memo_ink_strokes_v4';

      const notepad = document.getElementById('main-notepad');
      const boxesLayer = document.getElementById('boxes-layer');
      const saveStatus = document.getElementById('save-status');
      const contextMenu = document.getElementById('context-menu');
      const fileInput = document.getElementById('file-input');
      const dropOverlay = document.getElementById('drop-overlay');

      const drawCanvas = document.getElementById('drawing-canvas');
      const drawCtx = drawCanvas.getContext('2d');
      const penPalette = document.getElementById('pen-palette');
      const btnTogglePen = document.getElementById('btn-toggle-pen');
      const rgbColorInput = document.getElementById('rgb-color-input');
      const currentRgbLabel = document.getElementById('current-rgb-label');
      const penSizeInput = document.getElementById('pen-size');
      const penSizeLabel = document.getElementById('pen-size-label');

      let boxes = [];
      let topZIndex = 100;
      let lastRightClickPos = { x: 150, y: 150 };
      let currentSelectedText = '';

      let isPenMode = false;
      let currentTool = 'pen'; // 'pen' | 'highlighter' | 'eraser'
      let currentColor = '#1f2937';
      let currentSize = 3;
      let strokes = [];
      let isDrawing = false;
      let currentStroke = null;

      const PASTEL_COLORS = ['#fef9c3', '#ffffff', '#e0f2fe', '#fce7f3', '#dcfce7', '#f3e8ff'];

      function showSaved() {
        saveStatus.textContent = '저장 중...';
        clearTimeout(window._saveTimer);
        window._saveTimer = setTimeout(() => {
          saveStatus.textContent = '저장됨';
        }, 400);
      }

      function saveBoxes() {
        try {
          localStorage.setItem(STORAGE_KEY_BOXES, JSON.stringify(boxes));
          showSaved();
        } catch (e) {
          saveStatus.textContent = '용량 초과(일부 미저장)';
          alert('브라우저 저장 용량 한도에 도달했습니다. 불필요한 고화질 이미지나 PDF 박스를 삭제해 주세요.');
        }
      }

      function saveInk() {
        try {
          localStorage.setItem(STORAGE_KEY_INK, JSON.stringify(strokes));
          showSaved();
        } catch (e) {
          saveStatus.textContent = '필기 용량 초과';
        }
      }

      function hexToRgba(hex, alpha) {
        const cleanHex = hex.replace('#', '');
        const r = parseInt(cleanHex.substring(0, 2), 16);
        const g = parseInt(cleanHex.substring(2, 4), 16);
        const b = parseInt(cleanHex.substring(4, 6), 16);
        return `rgba(${r}, ${g}, ${b}, ${alpha})`;
      }

      function resizeDrawCanvas() {
        const dpr = window.devicePixelRatio || 1;
        drawCanvas.width = window.innerWidth * dpr;
        drawCanvas.height = window.innerHeight * dpr;
        drawCtx.setTransform(dpr, 0, 0, dpr, 0, 0);
        redrawAllStrokes();
      }

      function drawStrokeOnContext(ctx, stroke, offsetX = 0, offsetY = 0, scaleX = 1, scaleY = 1) {
        const pts = stroke.points;
        if (!pts || pts.length === 0) return;

        ctx.save();
        ctx.lineCap = 'round';
        ctx.lineJoin = 'round';

        const avgScale = (scaleX + scaleY) / 2;

        if (stroke.tool === 'eraser') {
          ctx.globalCompositeOperation = 'destination-out';
          ctx.strokeStyle = 'rgba(0,0,0,1)';
          ctx.lineWidth = stroke.size * 2.5 * avgScale;
        } else if (stroke.tool === 'highlighter') {
          ctx.globalCompositeOperation = 'source-over';
          ctx.strokeStyle = hexToRgba(stroke.color, 0.35);
          ctx.lineWidth = Math.max(stroke.size * 3.5, 14) * avgScale;
        } else {
          ctx.globalCompositeOperation = 'source-over';
          ctx.strokeStyle = stroke.color;
          ctx.lineWidth = stroke.size * avgScale;
        }

        const mapX = x => (x - offsetX) * scaleX;
        const mapY = y => (y - offsetY) * scaleY;

        if (pts.length === 1) {
          ctx.beginPath();
          ctx.arc(mapX(pts[0].x), mapY(pts[0].y), ctx.lineWidth / 2, 0, Math.PI * 2);
          ctx.fillStyle = ctx.strokeStyle;
          ctx.fill();
        } else {
          for (let i = 1; i < pts.length; i++) {
            const prev = pts[i - 1];
            const curr = pts[i];
            ctx.beginPath();
            if (stroke.tool === 'pen' && curr.p && curr.p !== 0.5) {
              ctx.lineWidth = Math.max(1, stroke.size * (0.5 + curr.p)) * avgScale;
            }
            ctx.moveTo(mapX(prev.x), mapY(prev.y));
            ctx.lineTo(mapX(curr.x), mapY(curr.y));
            ctx.stroke();
          }
        }
        ctx.restore();
      }

      function redrawAllStrokes() {
        drawCtx.clearRect(0, 0, window.innerWidth, window.innerHeight);
        for (const stroke of strokes) {
          drawStrokeOnContext(drawCtx, stroke);
        }
      }

      // 이미지 박스 위에 그린 획을 해당 이미지 데이터 자체에 합성(bake)
      function bakeStrokeIntoMatchingImages(stroke) {
        if (!stroke || stroke.tool === 'eraser' || !stroke.points || stroke.points.length === 0) return;

        const imgBoxes = boxes.filter(b => b.type === 'image');
        for (const boxData of imgBoxes) {
          const boxEl = boxesLayer.querySelector(`.floating-box[data-id="${boxData.id}"]`);
          const imgEl = boxEl && boxEl.querySelector('.box-image-view');
          if (!imgEl) continue;

          const rect = imgEl.getBoundingClientRect();
          if (rect.width <= 0 || rect.height <= 0) continue;

          const touchesBox = stroke.points.some(
            pt => pt.x >= rect.left && pt.x <= rect.right && pt.y >= rect.top && pt.y <= rect.bottom
          );
          if (!touchesBox) continue;

          const baseImg = new Image();
          baseImg.onload = () => {
            const canvas = document.createElement('canvas');
            canvas.width = baseImg.naturalWidth || Math.round(rect.width);
            canvas.height = baseImg.naturalHeight || Math.round(rect.height);
            const ctx = canvas.getContext('2d');
            ctx.drawImage(baseImg, 0, 0, canvas.width, canvas.height);

            const natRatio = canvas.width / canvas.height;
            const boxRatio = rect.width / rect.height;
            let drawW = rect.width;
            let drawH = rect.height;
            let drawLeft = rect.left;
            let drawTop = rect.top;

            if (natRatio > boxRatio) {
              drawH = rect.width / natRatio;
              drawTop = rect.top + (rect.height - drawH) / 2;
            } else {
              drawW = rect.height * natRatio;
              drawLeft = rect.left + (rect.width - drawW) / 2;
            }

            const scaleX = canvas.width / drawW;
            const scaleY = canvas.height / drawH;

            drawStrokeOnContext(ctx, stroke, drawLeft, drawTop, scaleX, scaleY);

            boxData.dataUrl = canvas.toDataURL('image/png');
            imgEl.src = boxData.dataUrl;
            saveBoxes();
          };
          baseImg.src = boxData.dataUrl;
        }
      }

      drawCanvas.addEventListener('pointerdown', e => {
        if (!isPenMode || e.button !== 0) return;
        drawCanvas.setPointerCapture(e.pointerId);
        isDrawing = true;
        currentStroke = {
          tool: currentTool,
          color: currentColor,
          size: currentSize,
          points: [{ x: e.clientX, y: e.clientY, p: e.pressure || 0.5 }]
        };
        strokes.push(currentStroke);
        drawStrokeOnContext(drawCtx, currentStroke);
      });

      drawCanvas.addEventListener('pointermove', e => {
        if (!isDrawing || !currentStroke) return;
        const pts = currentStroke.points;
        const prev = pts[pts.length - 1];
        const curr = { x: e.clientX, y: e.clientY, p: e.pressure || 0.5 };
        pts.push(curr);

        drawCtx.save();
        drawCtx.lineCap = 'round';
        drawCtx.lineJoin = 'round';
        if (currentStroke.tool === 'eraser') {
          drawCtx.globalCompositeOperation = 'destination-out';
          drawCtx.strokeStyle = 'rgba(0,0,0,1)';
          drawCtx.lineWidth = currentStroke.size * 2.5;
        } else if (currentStroke.tool === 'highlighter') {
          drawCtx.globalCompositeOperation = 'source-over';
          drawCtx.strokeStyle = hexToRgba(currentStroke.color, 0.35);
          drawCtx.lineWidth = Math.max(currentStroke.size * 3.5, 14);
        } else {
          drawCtx.globalCompositeOperation = 'source-over';
          drawCtx.strokeStyle = currentStroke.color;
          drawCtx.lineWidth = (curr.p && curr.p !== 0.5)
            ? Math.max(1, currentStroke.size * (0.5 + curr.p))
            : currentStroke.size;
        }
        drawCtx.beginPath();
        drawCtx.moveTo(prev.x, prev.y);
        drawCtx.lineTo(curr.x, curr.y);
        drawCtx.stroke();
        drawCtx.restore();
      });

      function endStroke() {
        if (!isDrawing) return;
        isDrawing = false;
        const finished = currentStroke;
        currentStroke = null;
        redrawAllStrokes();
        saveInk();

        if (finished) {
          bakeStrokeIntoMatchingImages(finished);
        }
      }

      drawCanvas.addEventListener('pointerup', endStroke);
      drawCanvas.addEventListener('pointercancel', endStroke);

      function togglePenMode(forceState) {
        isPenMode = typeof forceState === 'boolean' ? forceState : !isPenMode;
        drawCanvas.classList.toggle('active', isPenMode);
        btnTogglePen.classList.toggle('pen-active', isPenMode);
        btnTogglePen.textContent = isPenMode ? '✏️ 펜 모드 ON' : '✏️ 펜 모드';
        penPalette.style.display = isPenMode ? 'flex' : 'none';
      }

      btnTogglePen.onclick = () => togglePenMode();

      const modeBtns = {
        pen: document.getElementById('mode-pen'),
        highlighter: document.getElementById('mode-highlighter'),
        eraser: document.getElementById('mode-eraser')
      };

      function selectTool(tool) {
        currentTool = tool;
        Object.keys(modeBtns).forEach(k => {
          modeBtns[k].classList.toggle('selected', k === tool);
        });
        if (tool === 'highlighter' && currentSize < 6) {
          currentSize = 6;
          penSizeInput.value = 6;
          penSizeLabel.textContent = '6px';
        } else if (tool === 'pen' && currentSize > 10) {
          currentSize = 3;
          penSizeInput.value = 3;
          penSizeLabel.textContent = '3px';
        }
      }

      modeBtns.pen.onclick = () => selectTool('pen');
      modeBtns.highlighter.onclick = () => selectTool('highlighter');
      modeBtns.eraser.onclick = () => selectTool('eraser');

      function setPenColor(hexColor) {
        currentColor = hexColor;
        rgbColorInput.value = hexColor;
        currentRgbLabel.textContent = hexColor.toUpperCase();
        document.querySelectorAll('.color-swatch').forEach(sw => {
          sw.classList.toggle('selected', sw.dataset.color.toLowerCase() === hexColor.toLowerCase());
        });
        if (currentTool === 'eraser') selectTool('pen');
      }

      document.querySelectorAll('.color-swatch').forEach(sw => {
        sw.onclick = () => setPenColor(sw.dataset.color);
      });

      rgbColorInput.addEventListener('input', e => {
        setPenColor(e.target.value);
      });

      penSizeInput.addEventListener('input', e => {
        currentSize = parseInt(e.target.value, 10);
        penSizeLabel.textContent = currentSize + 'px';
      });

      function undoStroke() {
        if (strokes.length === 0) return;
        strokes.pop();
        redrawAllStrokes();
        saveInk();
      }

      document.getElementById('btn-undo-stroke').onclick = undoStroke;
      window.addEventListener('keydown', e => {
        if (isPenMode && (e.ctrlKey || e.metaKey) && e.key.toLowerCase() === 'z') {
          e.preventDefault();
          undoStroke();
        }
      });

      document.getElementById('btn-clear-ink').onclick = () => {
        if (strokes.length === 0) return;
        strokes = [];
        redrawAllStrokes();
        saveInk();
      };

      function renderTextToImage(text, bgColor = '#fef9c3') {
        const lines = (text || ' ').split('\n');
        const fontSize = 16;
        const lineHeight = 25;
        const padding = 20;
        const scale = 2;

        const tempCanvas = document.createElement('canvas');
        const tCtx = tempCanvas.getContext('2d');
        tCtx.font = `500 ${fontSize}px Pretendard, sans-serif`;

        let maxWidth = 80;
        lines.forEach(line => {
          const w = tCtx.measureText(line).width;
          if (w > maxWidth) maxWidth = w;
        });

        const width = Math.min(Math.max(maxWidth + padding * 2, 140), 650);
        const height = Math.max(lines.length * lineHeight + padding * 2, 70);

        const canvas = document.createElement('canvas');
        canvas.width = width * scale;
        canvas.height = height * scale;
        const ctx = canvas.getContext('2d');
        ctx.scale(scale, scale);

        ctx.fillStyle = bgColor === '#ffffff' ? '#ffffff' : bgColor;
        ctx.fillRect(0, 0, width, height);

        ctx.fillStyle = '#1f2937';
        ctx.font = `500 ${fontSize}px Pretendard, sans-serif`;
        ctx.textBaseline = 'top';

        lines.forEach((line, idx) => {
          ctx.fillText(line, padding, padding + idx * lineHeight);
        });

        return {
          dataUrl: canvas.toDataURL('image/png'),
          width: Math.round(width),
          height: Math.round(height)
        };
      }

      function compressImageFile(file, callback) {
        if (file.type === 'image/gif') {
          const reader = new FileReader();
          reader.onload = e => callback(e.target.result, 280, 220);
          reader.readAsDataURL(file);
          return;
        }
        const reader = new FileReader();
        reader.onload = e => {
          const img = new Image();
          img.onload = () => {
            const MAX_DIM = 1200;
            let w = img.width;
            let h = img.height;
            if (w > MAX_DIM || h > MAX_DIM) {
              if (w > h) {
                h = Math.round((h * MAX_DIM) / w);
                w = MAX_DIM;
              } else {
                w = Math.round((w * MAX_DIM) / h);
                h = MAX_DIM;
              }
            }
            const canvas = document.createElement('canvas');
            canvas.width = w;
            canvas.height = h;
            const ctx = canvas.getContext('2d');
            ctx.drawImage(img, 0, 0, w, h);
            const dataUrl = canvas.toDataURL('image/jpeg', 0.85);
            const displayW = Math.min(w, 340);
            const displayH = Math.round((h / w) * displayW) + 30;
            callback(dataUrl, displayW, displayH);
          };
          img.src = e.target.result;
        };
        reader.readAsDataURL(file);
      }

      function createBoxElement(boxData) {
        const el = document.createElement('div');
        el.className = 'floating-box' + (boxData.pinned ? ' pinned' : '');
        el.dataset.id = boxData.id;
        el.style.left = boxData.x + 'px';
        el.style.top = boxData.y + 'px';
        el.style.width = boxData.width + 'px';
        el.style.height = boxData.height + 'px';
        el.style.zIndex = boxData.zIndex || ++topZIndex;
        el.style.backgroundColor = boxData.bgColor || '#fef9c3';

        const header = document.createElement('div');
        header.className = 'box-header';

        const leftControls = document.createElement('div');
        leftControls.style.display = 'flex';
        leftControls.style.alignItems = 'center';
        leftControls.style.gap = '6px';

        if (boxData.type === 'text') {
          const colorBtn = document.createElement('button');
          colorBtn.textContent = '🎨';
          colorBtn.title = '배경색 변경';
          colorBtn.onclick = e => {
            e.stopPropagation();
            const curIdx = PASTEL_COLORS.indexOf(boxData.bgColor);
            boxData.bgColor = PASTEL_COLORS[(curIdx + 1) % PASTEL_COLORS.length];
            el.style.backgroundColor = boxData.bgColor;
            if (boxData.isImageMode) {
              const rendered = renderTextToImage(boxData.text, boxData.bgColor);
              boxData.imageDataUrl = rendered.dataUrl;
              renderBoxBody();
            }
            saveBoxes();
          };
          leftControls.appendChild(colorBtn);

          const modeBtn = document.createElement('button');
          modeBtn.textContent = boxData.isImageMode ? '✏️ 편집' : '🖼️ 그림화';
          modeBtn.style.fontWeight = '600';
          modeBtn.style.padding = '1px 5px';
          modeBtn.style.borderRadius = '4px';
          modeBtn.style.background = 'rgba(0,0,0,0.06)';
          modeBtn.onclick = e => {
            e.stopPropagation();
            boxData.isImageMode = !boxData.isImageMode;
            if (boxData.isImageMode) {
              const rendered = renderTextToImage(boxData.text, boxData.bgColor);
              boxData.imageDataUrl = rendered.dataUrl;
            }
            modeBtn.textContent = boxData.isImageMode ? '✏️ 편집' : '🖼️ 그림화';
            renderBoxBody();
            saveBoxes();
          };
          leftControls.appendChild(modeBtn);
        } else {
          const label = document.createElement('span');
          label.textContent = boxData.type === 'pdf' ? '📄 PDF' : '🖼️ 이미지';
          label.style.opacity = '0.75';
          leftControls.appendChild(label);
        }

        const rightControls = document.createElement('div');
        rightControls.style.display = 'flex';
        rightControls.style.alignItems = 'center';
        rightControls.style.gap = '6px';

        const pinBtn = document.createElement('button');
        pinBtn.textContent = boxData.pinned ? '📌 고정됨' : '📍 고정';
        pinBtn.title = '위치 및 크기 고정/해제';
        pinBtn.style.color = boxData.pinned ? '#ef4444' : 'inherit';
        pinBtn.onclick = e => {
          e.stopPropagation();
          boxData.pinned = !boxData.pinned;
          el.classList.toggle('pinned', boxData.pinned);
          pinBtn.textContent = boxData.pinned ? '📌 고정됨' : '📍 고정';
          pinBtn.style.color = boxData.pinned ? '#ef4444' : 'inherit';
          saveBoxes();
        };

        const delBtn = document.createElement('button');
        delBtn.textContent = '✕';
        delBtn.title = '삭제';
        delBtn.style.padding = '0 4px';
        delBtn.onclick = e => {
          e.stopPropagation();
          boxes = boxes.filter(b => b.id !== boxData.id);
          el.remove();
          saveBoxes();
        };

        rightControls.appendChild(pinBtn);
        rightControls.appendChild(delBtn);
        header.appendChild(leftControls);
        header.appendChild(rightControls);

        const body = document.createElement('div');
        body.className = 'box-content';

        function renderBoxBody() {
          body.innerHTML = '';
          if (boxData.type === 'text') {
            if (boxData.isImageMode) {
              const img = document.createElement('img');
              img.src = boxData.imageDataUrl || renderTextToImage(boxData.text, boxData.bgColor).dataUrl;
              img.className = 'box-image-view';
              img.ondblclick = () => {
                boxData.isImageMode = false;
                renderBoxBody();
                saveBoxes();
              };
              enableDrag(img);
              body.appendChild(img);
            } else {
              const ta = document.createElement('textarea');
              ta.className = 'box-textarea';
              ta.value = boxData.text || '';
              ta.placeholder = '내용을 입력하세요...';
              ta.oninput = () => {
                boxData.text = ta.value;
                saveBoxes();
              };
              body.appendChild(ta);
            }
          } else if (boxData.type === 'image') {
            const img = document.createElement('img');
            img.src = boxData.dataUrl;
            img.className = 'box-image-view';
            enableDrag(img);
            body.appendChild(img);
          } else if (boxData.type === 'pdf') {
            const iframe = document.createElement('iframe');
            iframe.src = boxData.dataUrl;
            iframe.className = 'box-pdf-view';
            body.appendChild(iframe);
          }
        }

        function enableDrag(handleEl) {
          handleEl.addEventListener('mousedown', e => {
            if (boxData.pinned || e.button !== 0 || isPenMode) return;
            e.preventDefault();
            el.style.zIndex = ++topZIndex;
            boxData.zIndex = topZIndex;

            const startX = e.clientX;
            const startY = e.clientY;
            const origX = boxData.x;
            const origY = boxData.y;

            function onMove(ev) {
              boxData.x = Math.max(0, origX + (ev.clientX - startX));
              boxData.y = Math.max(0, origY + (ev.clientY - startY));
              el.style.left = boxData.x + 'px';
              el.style.top = boxData.y + 'px';
            }
            function onUp() {
              document.removeEventListener('mousemove', onMove);
              document.removeEventListener('mouseup', onUp);
              saveBoxes();
            }
            document.addEventListener('mousemove', onMove);
            document.addEventListener('mouseup', onUp);
          });
        }

        enableDrag(header);

        const resizer = document.createElement('div');
        resizer.className = 'resize-handle';
        resizer.addEventListener('mousedown', e => {
          if (boxData.pinned || e.button !== 0 || isPenMode) return;
          e.stopPropagation();
          e.preventDefault();
          el.style.zIndex = ++topZIndex;
          boxData.zIndex = topZIndex;

          const startX = e.clientX;
          const startY = e.clientY;
          const origW = boxData.width;
          const origH = boxData.height;

          function onResize(ev) {
            boxData.width = Math.max(150, origW + (ev.clientX - startX));
            boxData.height = Math.max(85, origH + (ev.clientY - startY));
            el.style.width = boxData.width + 'px';
            el.style.height = boxData.height + 'px';
          }
          function onUp() {
            document.removeEventListener('mousemove', onResize);
            document.removeEventListener('mouseup', onUp);
            saveBoxes();
          }
          document.addEventListener('mousemove', onResize);
          document.addEventListener('mouseup', onUp);
        });

        el.addEventListener('mousedown', () => {
          if (isPenMode) return;
          el.style.zIndex = ++topZIndex;
          boxData.zIndex = topZIndex;
        });

        renderBoxBody();
        el.appendChild(header);
        el.appendChild(body);
        el.appendChild(resizer);
        boxesLayer.appendChild(el);
      }

      function addTextBox(x, y, text = '', asImage = false) {
        let width = 260;
        let height = 170;
        let imageDataUrl = null;
        const bgColor = '#fef9c3';

        if (asImage && text.trim()) {
          const rendered = renderTextToImage(text, bgColor);
          imageDataUrl = rendered.dataUrl;
          width = Math.max(180, rendered.width + 16);
          height = Math.max(95, rendered.height + 36);
        }

        const boxData = {
          id: 'box_' + Date.now() + '_' + Math.random().toString(36).slice(2, 6),
          type: 'text',
          x: Math.min(x, window.innerWidth - 280),
          y: Math.min(y, window.innerHeight - 200),
          width,
          height,
          text,
          bgColor,
          isImageMode: asImage,
          imageDataUrl,
          pinned: false,
          zIndex: ++topZIndex
        };
        boxes.push(boxData);
        createBoxElement(boxData);
        saveBoxes();
      }

      function handleFilesUpload(files, dropX = 160, dropY = 140) {
        Array.from(files).forEach((file, idx) => {
          const offsetX = dropX + idx * 24;
          const offsetY = dropY + idx * 24;

          if (file.type.startsWith('image/')) {
            compressImageFile(file, (dataUrl, w, h) => {
              const boxData = {
                id: 'img_' + Date.now() + '_' + Math.random().toString(36).slice(2, 6),
                type: 'image',
                x: Math.min(offsetX, window.innerWidth - 320),
                y: Math.min(offsetY, window.innerHeight - 240),
                width: Math.max(180, w),
                height: Math.max(120, h),
                dataUrl,
                bgColor: '#ffffff',
                pinned: false,
                zIndex: ++topZIndex
              };
              boxes.push(boxData);
              createBoxElement(boxData);
              saveBoxes();
            });
          } else if (file.type === 'application/pdf') {
            const reader = new FileReader();
            reader.onload = e => {
              const boxData = {
                id: 'pdf_' + Date.now() + '_' + Math.random().toString(36).slice(2, 6),
                type: 'pdf',
                x: Math.min(offsetX, window.innerWidth - 420),
                y: Math.min(offsetY, window.innerHeight - 480),
                width: 400,
                height: 480,
                dataUrl: e.target.result,
                bgColor: '#ffffff',
                pinned: false,
                zIndex: ++topZIndex
              };
              boxes.push(boxData);
              createBoxElement(boxData);
              saveBoxes();
            };
            reader.readAsDataURL(file);
          }
        });
      }

      notepad.addEventListener('input', () => {
        localStorage.setItem(STORAGE_KEY_TEXT, notepad.value);
        showSaved();
      });

      notepad.addEventListener('keydown', e => {
        if (e.key === 'Tab') {
          e.preventDefault();
          const start = notepad.selectionStart;
          const end = notepad.selectionEnd;
          notepad.value = notepad.value.substring(0, start) + '    ' + notepad.value.substring(end);
          notepad.selectionStart = notepad.selectionEnd = start + 4;
          localStorage.setItem(STORAGE_KEY_TEXT, notepad.value);
        }
      });

      document.addEventListener('contextmenu', e => {
        if (isPenMode || e.target.closest('.floating-box')) return;

        e.preventDefault();
        lastRightClickPos = { x: e.clientX, y: e.clientY };

        const sel = window.getSelection().toString().trim();
        const taSel = notepad.value.substring(notepad.selectionStart, notepad.selectionEnd).trim();
        currentSelectedText = taSel || sel;

        const itemImg = document.getElementById('ctx-text-to-img');
        const itemBox = document.getElementById('ctx-text-to-box');

        if (currentSelectedText) {
          itemImg.style.display = 'flex';
          itemBox.style.display = 'flex';
        } else {
          itemImg.style.display = 'none';
          itemBox.style.display = 'none';
        }

        contextMenu.style.left = Math.min(e.clientX, window.innerWidth - 230) + 'px';
        contextMenu.style.top = Math.min(e.clientY, window.innerHeight - 160) + 'px';
        contextMenu.style.display = 'block';
      });

      document.addEventListener('click', () => {
        contextMenu.style.display = 'none';
      });

      document.getElementById('ctx-text-to-img').onclick = () => {
        if (currentSelectedText) {
          addTextBox(lastRightClickPos.x, lastRightClickPos.y, currentSelectedText, true);
        }
      };

      document.getElementById('ctx-text-to-box').onclick = () => {
        if (currentSelectedText) {
          addTextBox(lastRightClickPos.x, lastRightClickPos.y, currentSelectedText, false);
        }
      };

      document.getElementById('ctx-add-box').onclick = () => {
        addTextBox(lastRightClickPos.x, lastRightClickPos.y, '', false);
      };

      document.getElementById('ctx-add-file').onclick = () => {
        fileInput.click();
      };

      document.getElementById('btn-add-box').onclick = () => {
        togglePenMode(false);
        addTextBox(140 + (boxes.length % 6) * 25, 110 + (boxes.length % 6) * 25, '', false);
      };

      document.getElementById('btn-add-file').onclick = () => {
        togglePenMode(false);
        fileInput.click();
      };

      fileInput.onchange = e => {
        if (e.target.files.length) {
          handleFilesUpload(e.target.files, lastRightClickPos.x, lastRightClickPos.y);
          fileInput.value = '';
        }
      };

      window.addEventListener('dragover', e => {
        e.preventDefault();
        dropOverlay.style.display = 'flex';
      });
      window.addEventListener('dragleave', e => {
        if (e.relatedTarget === null) dropOverlay.style.display = 'none';
      });
      window.addEventListener('drop', e => {
        e.preventDefault();
        dropOverlay.style.display = 'none';
        if (e.dataTransfer.files && e.dataTransfer.files.length > 0) {
          togglePenMode(false);
          handleFilesUpload(e.dataTransfer.files, e.clientX, e.clientY);
        }
      });

      window.addEventListener('paste', e => {
        const items = e.clipboardData && e.clipboardData.items;
        if (!items) return;
        const files = [];
        for (let i = 0; i < items.length; i++) {
          if (items[i].kind === 'file') {
            files.push(items[i].getAsFile());
          }
        }
        if (files.length > 0) {
          e.preventDefault();
          togglePenMode(false);
          handleFilesUpload(files, 180, 140);
        }
      });

      document.getElementById('btn-theme').onclick = () => {
        document.body.classList.toggle('dark');
        localStorage.setItem(STORAGE_KEY_THEME, document.body.classList.contains('dark') ? 'dark' : 'light');
      };

      let clearConfirm = false;
      const btnClear = document.getElementById('btn-clear');
      btnClear.onclick = () => {
        if (!clearConfirm) {
          clearConfirm = true;
          btnClear.textContent = '정말 모두 지울까요?';
          setTimeout(() => {
            clearConfirm = false;
            btnClear.textContent = '전체 초기화';
          }, 3000);
        } else {
          notepad.value = '';
          boxes = [];
          strokes = [];
          boxesLayer.innerHTML = '';
          redrawAllStrokes();
          localStorage.removeItem(STORAGE_KEY_TEXT);
          localStorage.removeItem(STORAGE_KEY_BOXES);
          localStorage.removeItem(STORAGE_KEY_INK);
          clearConfirm = false;
          btnClear.textContent = '전체 초기화';
          showSaved();
        }
      };

      window.addEventListener('resize', resizeDrawCanvas);

      function init() {
        const savedTheme = localStorage.getItem(STORAGE_KEY_THEME);
        if (savedTheme === 'dark') document.body.classList.add('dark');

        const savedText = localStorage.getItem(STORAGE_KEY_TEXT);
        if (savedText !== null) notepad.value = savedText;

        try {
          const savedBoxes = JSON.parse(localStorage.getItem(STORAGE_KEY_BOXES) || '[]');
          boxes = savedBoxes;
          boxes.forEach(b => {
            if (b.zIndex && b.zIndex > topZIndex) topZIndex = b.zIndex;
            createBoxElement(b);
          });
        } catch (e) {
          boxes = [];
        }

        try {
          strokes = JSON.parse(localStorage.getItem(STORAGE_KEY_INK) || '[]');
        } catch (e) {
          strokes = [];
        }

        resizeDrawCanvas();
      }

      init();
    })();
  </script>
</body>
</html>
