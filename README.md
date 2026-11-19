<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>나만의 메모장</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link rel="stylesheet" as="style" crossorigin href="https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/static/pretendard.min.css" />
  <style>
    * { box-sizing: border-box; }
    body, html {
      margin: 0;
      padding: 0;
      width: 100%;
      height: 100%;
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
      width: 100%;
      height: 100%;
      padding: 40px 48px 100px 48px;
      border: none;
      outline: none;
      resize: none;
      background: transparent;
      font-size: 16px;
      line-height: 1.75;
      color: inherit;
      font-family: inherit;
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
      background: rgba(255, 255, 255, 0.85);
      backdrop-filter: blur(12px);
      border: 1px solid rgba(0, 0, 0, 0.08);
      box-shadow: 0 8px 24px rgba(0, 0, 0, 0.08);
      font-size: 13px;
      opacity: 0.55;
      transition: opacity 0.2s ease;
    }
    .toolbar-pill:hover {
      opacity: 1;
    }
    body.dark .toolbar-pill {
      background: rgba(30, 30, 34, 0.85);
      border-color: rgba(255, 255, 255, 0.1);
      box-shadow: 0 8px 24px rgba(0, 0, 0, 0.4);
    }
    .tool-btn {
      padding: 5px 10px;
      border-radius: 9999px;
      cursor: pointer;
      transition: background 0.15s;
      white-space: nowrap;
      font-weight: 500;
    }
    .tool-btn:hover {
      background: rgba(0, 0, 0, 0.06);
    }
    body.dark .tool-btn:hover {
      background: rgba(255, 255, 255, 0.1);
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
  <textarea id="main-notepad" placeholder="여기에 자유롭게 메모하세요...&#10;&#10;• 글씨를 드래그한 뒤 [우클릭]하면 그림 스티커로 바로 변환할 수 있습니다.&#10;• 사진(JPG, PNG, GIF)이나 PDF 파일을 화면에 끌어다 놓거나 붙여넣기(Ctrl+V) 하세요."></textarea>

  <!-- 플로팅 박스들이 올라가는 레이어 -->
  <div id="boxes-layer"></div>

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
    <button id="btn-add-box" class="tool-btn">+ 텍스트 박스</button>
    <button id="btn-add-file" class="tool-btn">+ 사진/PDF</button>
    <button id="btn-theme" class="tool-btn" title="테마 전환">🌓</button>
    <button id="btn-clear" class="tool-btn text-red-500">전체 초기화</button>
  </div>

  <script>
    (function () {
      const STORAGE_KEY_TEXT = 'my_memo_main_text_v3';
      const STORAGE_KEY_BOXES = 'my_memo_boxes_v3';
      const STORAGE_KEY_THEME = 'my_memo_theme_v3';

      const notepad = document.getElementById('main-notepad');
      const boxesLayer = document.getElementById('boxes-layer');
      const saveStatus = document.getElementById('save-status');
      const contextMenu = document.getElementById('context-menu');
      const fileInput = document.getElementById('file-input');
      const dropOverlay = document.getElementById('drop-overlay');

      let boxes = [];
      let topZIndex = 100;
      let lastRightClickPos = { x: 150, y: 150 };
      let currentSelectedText = '';

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

      // 텍스트를 고해상도 이미지(DataURL)로 변환하는 함수
      function renderTextToImage(text, bgColor = '#fef9c3') {
        const lines = (text || ' ').split('\n');
        const fontSize = 16;
        const lineHeight = 25;
        const padding = 20;
        const scale = 2; // 고해상도 레티나

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

        // 배경
        ctx.fillStyle = bgColor === '#ffffff' ? '#ffffff' : bgColor;
        ctx.fillRect(0, 0, width, height);

        // 텍스트
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

      // 이미지 용량 최적화 리사이즈 (저장공간 보호)
      function compressImageFile(file, callback) {
        if (file.type === 'image/gif') {
          // GIF 애니메이션 유지를 위해 원본 그대로 읽기
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

        // 상단 컨트롤 바
        const header = document.createElement('div');
        header.className = 'box-header';

        const leftControls = document.createElement('div');
        leftControls.style.display = 'flex';
        leftControls.style.alignItems = 'center';
        leftControls.style.gap = '6px';

        // 색상 변경 버튼 (텍스트 박스용)
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

          // 그림화 / 편집 토글 버튼
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

        // 핀 고정 버튼
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

        // 삭제 버튼
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

        // 드래그 이동 로직
        function enableDrag(handleEl) {
          handleEl.addEventListener('mousedown', e => {
            if (boxData.pinned || e.button !== 0) return;
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

        // 우측 하단 크기 조절 핸들
        const resizer = document.createElement('div');
        resizer.className = 'resize-handle';
        resizer.addEventListener('mousedown', e => {
          if (boxData.pinned || e.button !== 0) return;
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

      // 배경 메모장 자동 저장
      notepad.addEventListener('input', () => {
        localStorage.setItem(STORAGE_KEY_TEXT, notepad.value);
        showSaved();
      });

      // 탭 들여쓰기 지원
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

      // 우클릭 커스텀 메뉴
      document.addEventListener('contextmenu', e => {
        // 플로팅 박스 내부에서 우클릭한 경우는 기본 브라우저 메뉴(이미지 복사 등) 허용
        if (e.target.closest('.floating-box')) return;

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

      // 하단 버튼 이벤트
      document.getElementById('btn-add-box').onclick = () => {
        addTextBox(140 + (boxes.length % 6) * 25, 110 + (boxes.length % 6) * 25, '', false);
      };

      document.getElementById('btn-add-file').onclick = () => {
        fileInput.click();
      };

      fileInput.onchange = e => {
        if (e.target.files.length) {
          handleFilesUpload(e.target.files, lastRightClickPos.x, lastRightClickPos.y);
          fileInput.value = '';
        }
      };

      // 화면 전체 드래그 앤 드롭 지원
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
          handleFilesUpload(e.dataTransfer.files, e.clientX, e.clientY);
        }
      });

      // 클립보드 이미지 붙여넣기(Ctrl+V) 지원
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
          handleFilesUpload(files, 180, 140);
        }
      });

      // 테마 전환
      document.getElementById('btn-theme').onclick = () => {
        document.body.classList.toggle('dark');
        localStorage.setItem(STORAGE_KEY_THEME, document.body.classList.contains('dark') ? 'dark' : 'light');
      };

      // 전체 초기화 (2단계 확인)
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
          boxesLayer.innerHTML = '';
          localStorage.removeItem(STORAGE_KEY_TEXT);
          localStorage.removeItem(STORAGE_KEY_BOXES);
          clearConfirm = false;
          btnClear.textContent = '전체 초기화';
          showSaved();
        }
      };

      // 초기 로드
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
      }

      init();
    })();
  </script>
</body>
</html>
