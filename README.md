<!DOCTYPE html>
<html lang="es" class="h-full bg-slate-950 text-slate-100 select-none">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Explorador Interactivo de Fractales y Autosemejanza</title>

  <!-- Tailwind CSS via CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- FontAwesome Icons -->
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

  <style>
    /* Styling canvas and custom interactive scrollbar */
    canvas {
      touch-action: none;
      cursor: grab;
    }
    canvas:active {
      cursor: grabbing;
    }

    ::-webkit-scrollbar {
      width: 5px;
      height: 5px;
    }
    ::-webkit-scrollbar-track {
      background: #0f172a;
    }
    ::-webkit-scrollbar-thumb {
      background: #334155;
      border-radius: 4px;
    }

    /* Custom touch-friendly sliders */
    input[type=range] {
      -webkit-appearance: none;
      background: transparent;
    }
    input[type=range]::-webkit-slider-track {
      height: 8px;
      background: #1e293b;
      border-radius: 4px;
      border: 1px solid #334155;
    }
    input[type=range]::-webkit-slider-thumb {
      -webkit-appearance: none;
      width: 22px;
      height: 22px;
      border-radius: 50%;
      background: #6366f1;
      border: 2px solid #ffffff;
      cursor: pointer;
      margin-top: -7px;
      box-shadow: 0 2px 6px rgba(0, 0, 0, 0.4);
    }
    input[type=range]:focus {
      outline: none;
    }

    /* Floating animation for mobile controls toggle */
    .drawer-transition {
      transition: transform 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    }
  </style>
</head>
<body class="h-full w-full flex flex-col md:flex-row overflow-hidden font-sans bg-slate-950 text-slate-100">


  <!-- Canvas Container (Fills screen, behind controls on mobile) -->
  <main class="relative flex-1 w-full h-full overflow-hidden flex items-center justify-center bg-slate-950">
    
    <!-- Canvas for fractal rendering -->
    <canvas id="fractalCanvas" class="w-full h-full block"></canvas>

    <!-- Top Mobile Control Bar & Floating Controls -->
    <div class="absolute top-3 left-3 right-3 flex items-center justify-between pointer-events-none z-10">
      
      <!-- App Title Badge (Mobile / Desktop) -->
      <div class="bg-slate-900/85 backdrop-blur-md border border-slate-800/80 px-3.5 py-2 rounded-xl shadow-xl flex items-center space-x-2.5 pointer-events-auto">
        <div class="p-1.5 bg-indigo-600/30 text-indigo-400 rounded-lg border border-indigo-500/30">
          <i class="fa-solid fa-cubes-stacked text-base"></i>
        </div>
        <div>
          <h1 class="text-xs md:text-sm font-bold text-white leading-tight">Explorador Fractal</h1>
          <p class="text-[10px] text-indigo-400 font-medium hidden sm:block">Autosemejanza Interactiva</p>
        </div>
      </div>

      <!-- Quick Action Floating Buttons (Zoom In/Out, Reset) -->
      <div class="flex items-center space-x-2 pointer-events-auto">
        <button id="btnZoomIn" title="Acercar Zoom" class="w-9 h-9 bg-slate-900/85 backdrop-blur-md border border-slate-800 text-slate-200 rounded-xl hover:bg-slate-800 active:scale-95 transition flex items-center justify-center shadow-lg">
          <i class="fa-solid fa-plus text-xs"></i>
        </button>
        <button id="btnZoomOut" title="Alejar Zoom" class="w-9 h-9 bg-slate-900/85 backdrop-blur-md border border-slate-800 text-slate-200 rounded-xl hover:bg-slate-800 active:scale-95 transition flex items-center justify-center shadow-lg">
          <i class="fa-solid fa-minus text-xs"></i>
        </button>
        <button id="resetViewBtn" title="Restablecer Vista" class="w-9 h-9 bg-slate-900/85 backdrop-blur-md border border-slate-800 text-indigo-400 rounded-xl hover:bg-slate-800 active:scale-95 transition flex items-center justify-center shadow-lg">
          <i class="fa-solid fa-rotate-left text-xs"></i>
        </button>
      </div>
    </div>

    <!-- Instructions Overlay for Desktop & Mobile Touch -->
    <div class="absolute top-16 right-3 hidden lg:flex bg-slate-900/80 backdrop-blur border border-slate-800/80 px-3 py-1.5 rounded-xl text-[11px] text-slate-400 items-center gap-2.5 shadow-lg pointer-events-none z-10">
      <span class="flex items-center gap-1"><i class="fa-solid fa-computer-mouse text-slate-300"></i> Arrastrar / Rueda</span>
      <span class="border-r border-slate-700 h-3"></span>
      <span class="flex items-center gap-1"><i class="fa-solid fa-hand-pointer text-slate-300"></i> Táctil: 1 Dedo Pan, 2 Pellizcar</span>
    </div>

    <!-- Self-Similarity Badge Overlay -->
    <div id="similarityBadge" class="absolute bottom-20 left-3 right-3 md:bottom-4 md:left-auto md:right-4 md:max-w-md bg-slate-900/90 backdrop-blur-md border border-amber-500/40 p-3 rounded-2xl shadow-2xl flex items-center space-x-3 transition-all duration-300 z-10">
      <div class="p-2.5 bg-amber-500/20 text-amber-400 rounded-xl shrink-0">
        <i class="fa-solid fa-wand-magic-sparkles text-base"></i>
      </div>
      <div class="text-xs">
        <span class="font-bold text-amber-300 block mb-0.5">Demostración de Autosimilitud</span>
        <span id="similarityDetailText" class="text-slate-300 text-[11px] leading-tight block">
          La zona resaltada con resplandor dorado representa un subconjunto idéntico a la estructura macro.
        </span>
      </div>
    </div>

    <!-- Mobile Toggle Button for Controls Drawer -->
    <button id="toggleControlsBtn" class="md:hidden fixed bottom-4 left-1/2 -translate-x-1/2 bg-indigo-600 hover:bg-indigo-500 text-white font-semibold text-xs py-2.5 px-5 rounded-full shadow-2xl border border-indigo-400/40 flex items-center gap-2 z-30 transition active:scale-95">
      <i class="fa-solid fa-sliders text-sm"></i>
      <span id="toggleBtnText">Ajustar Parámetros</span>
    </button>
  </main>


  <!-- Control Panel: Responsive Drawer (Bottom Sheet on Mobile, Sidebar on Desktop) -->
  <aside id="controlsPanel" class="drawer-transition fixed md:relative bottom-0 left-0 right-0 md:right-auto w-full md:w-80 lg:w-96 bg-slate-900/95 md:bg-slate-900 border-t md:border-t-0 md:border-r border-slate-800 flex flex-col max-h-[82vh] md:max-h-full h-auto md:h-full z-20 shadow-2xl rounded-t-3xl md:rounded-none overflow-hidden translate-y-full md:translate-y-0">
    
    <!-- Mobile Drawer Pull Handle / Header -->
    <div id="drawerHeader" class="p-3 md:p-5 border-b border-slate-800 bg-slate-900 flex items-center justify-between cursor-pointer md:cursor-default">
      <div class="flex items-center space-x-2">
        <div class="w-8 h-1 bg-slate-700 rounded-full mx-auto md:hidden absolute top-2 left-1/2 -translate-x-1/2"></div>
        <i class="fa-solid fa-sliders text-indigo-400 text-sm hidden md:inline"></i>
        <h2 class="text-sm font-bold text-slate-100 uppercase tracking-wider">Panel de Control</h2>
      </div>

      <button id="closeDrawerBtn" class="md:hidden p-1.5 text-slate-400 hover:text-white rounded-lg">
        <i class="fa-solid fa-xmark text-lg"></i>
      </button>
    </div>

    <!-- Scrollable Controls Content -->
    <div class="p-4 md:p-5 space-y-5 overflow-y-auto flex-1">
      
      <!-- 1. Fractal Selector -->
      <div>
        <label class="text-[11px] font-bold uppercase tracking-wider text-slate-400 mb-2 block flex items-center justify-between">
          <span><i class="fa-solid fa-shapes mr-1.5 text-indigo-400"></i> Seleccionar Fractal</span>
          <span class="text-[10px] text-indigo-400 bg-indigo-950/80 px-2 py-0.5 rounded-md border border-indigo-800/40">6 Opciónes</span>
        </label>
        <select id="fractalSelect" class="w-full bg-slate-800 border border-slate-700 text-slate-100 text-xs md:text-sm rounded-xl p-3 focus:ring-2 focus:ring-indigo-500 focus:outline-none cursor-pointer">
          <option value="pythagoras">1. Árbol de Pitágoras</option>
          <option value="koch">2. Copo de Nieve de Koch</option>
          <option value="cesaro">3. Copo de Nieve de Cesàro</option>
          <option value="sierpinski">4. Triángulo de Sierpinski</option>
          <option value="mandelbrot">5. Conjunto de Mandelbrot</option>
          <option value="julia">6. Conjunto de Julia</option>
        </select>
      </div>

      <!-- 2. Self-Similarity Toggle Switch -->
      <div class="bg-indigo-950/40 border border-indigo-800/40 rounded-2xl p-3.5 space-y-2">
        <div class="flex items-center justify-between">
          <div class="flex items-center space-x-2">
            <i class="fa-solid fa-wand-magic-sparkles text-amber-400 text-xs"></i>
            <span class="text-xs font-bold text-slate-200">Ver Autosemejanza</span>
          </div>
          <label class="relative inline-flex items-center cursor-pointer">
            <input type="checkbox" id="selfSimilarityToggle" class="sr-only peer" checked>
            <div class="w-11 h-6 bg-slate-800 peer-focus:outline-none rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:border-slate-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-indigo-600"></div>
          </label>
        </div>
        <p id="similarityHelpText" class="text-[11px] text-slate-400 leading-tight">
          Ilumina en tono brillante las sub-estructuras secundarias idénticas al fractal original.
        </p>
      </div>


      <!-- 3. General Iteration Controls -->
      <div class="space-y-4 pt-1">
        <div class="flex justify-between items-center">
          <label id="iterLabel" for="iterSlider" class="text-xs font-semibold text-slate-300">
            Iteraciones / Profundidad
          </label>
          <span id="iterVal" class="text-xs font-bold text-indigo-400 bg-indigo-950 px-2 py-0.5 rounded-lg border border-indigo-800/50">5</span>
        </div>
        <input type="range" id="iterSlider" min="1" max="10" value="5" class="w-full h-2 bg-slate-800 rounded-lg appearance-none cursor-pointer">

        <!-- Dynamic Controls according to selected fractal -->
        <!-- Árbol de Pitágoras -->
        <div id="pythagorasControls" class="space-y-4 pt-2 border-t border-slate-800">
          <div>
            <div class="flex justify-between items-center mb-1.5">
              <label for="angleSlider" class="text-xs text-slate-300">Ángulo de Ramificación</label>
              <span id="angleVal" class="text-xs font-bold text-indigo-400 bg-indigo-950 px-2 py-0.5 rounded-lg border border-indigo-800/50">45°</span>
            </div>
            <input type="range" id="angleSlider" min="10" max="80" value="45" class="w-full h-2 bg-slate-800 rounded-lg appearance-none cursor-pointer">
          </div>
          <div>
            <div class="flex justify-between items-center mb-1.5">
              <label for="scaleSlider" class="text-xs text-slate-300">Factor de Escala de Ramas</label>
              <span id="scaleVal" class="text-xs font-bold text-indigo-400 bg-indigo-950 px-2 py-0.5 rounded-lg border border-indigo-800/50">0.70</span>
            </div>
            <input type="range" id="scaleSlider" min="0.4" max="0.8" step="0.01" value="0.70" class="w-full h-2 bg-slate-800 rounded-lg appearance-none cursor-pointer">
          </div>
        </div>

        <!-- Cesàro Snowflake Controls -->
        <div id="cesaroControls" class="space-y-4 pt-2 border-t border-slate-800 hidden">
          <div>
            <div class="flex justify-between items-center mb-1.5">
              <label for="cesaroAngleSlider" class="text-xs text-slate-300">Ángulo de Inflexión Interior</label>
              <span id="cesaroAngleVal" class="text-xs font-bold text-indigo-400 bg-indigo-950 px-2 py-0.5 rounded-lg border border-indigo-800/50">85°</span>
            </div>
            <input type="range" id="cesaroAngleSlider" min="50" max="88" value="85" class="w-full h-2 bg-slate-800 rounded-lg appearance-none cursor-pointer">
          </div>
        </div>

        <!-- Julia Set Complex Parameters -->
        <div id="complexControls" class="space-y-4 pt-2 border-t border-slate-800 hidden">
          <div>
            <div class="flex justify-between items-center mb-1.5">
              <label for="cxSlider" class="text-xs text-slate-300">Constante C (Real)</label>
              <span id="cxVal" class="text-xs font-bold text-indigo-400 bg-indigo-950 px-2 py-0.5 rounded-lg border border-indigo-800/50">-0.700</span>
            </div>
            <input type="range" id="cxSlider" min="-1.5" max="1.5" step="0.005" value="-0.7" class="w-full h-2 bg-slate-800 rounded-lg appearance-none cursor-pointer">
          </div>
          <div>
            <div class="flex justify-between items-center mb-1.5">
              <label for="cySlider" class="text-xs text-slate-300">Constante C (Imaginaria)</label>
              <span id="cyVal" class="text-xs font-bold text-indigo-400 bg-indigo-950 px-2 py-0.5 rounded-lg border border-indigo-800/50">0.270</span>
            </div>
            <input type="range" id="cySlider" min="-1.5" max="1.5" step="0.005" value="0.27" class="w-full h-2 bg-slate-800 rounded-lg appearance-none cursor-pointer">
          </div>
        </div>
      </div>

      <!-- 4. Color Scheme Selector -->
      <div class="pt-2 border-t border-slate-800">
        <label class="text-[11px] font-bold uppercase tracking-wider text-slate-400 mb-2 block">
          Esquema de Color
        </label>
        <div class="grid grid-cols-3 gap-2">
          <button data-theme="neon" class="theme-btn py-2 px-2 text-xs rounded-xl border border-indigo-500 bg-indigo-900/40 text-indigo-300 font-semibold hover:bg-indigo-800/40 transition text-center">Neón</button>
          <button data-theme="fire" class="theme-btn py-2 px-2 text-xs rounded-xl border border-slate-700 bg-slate-800 text-slate-300 font-semibold hover:bg-slate-700 transition text-center">Fuego</button>
          <button data-theme="emerald" class="theme-btn py-2 px-2 text-xs rounded-xl border border-slate-700 bg-slate-800 text-slate-300 font-semibold hover:bg-slate-700 transition text-center">Esmeralda</button>
        </div>
      </div>

      <!-- 5. Educational Concept Description -->
      <div class="bg-slate-800/50 rounded-2xl p-3.5 border border-slate-700/50 space-y-1.5">
        <h3 class="text-[11px] font-bold uppercase tracking-wider text-indigo-400 flex items-center">
          <i class="fa-solid fa-circle-info mr-1.5"></i> Concepto
        </h3>
        <p id="fractalDescription" class="text-xs text-slate-300 leading-relaxed">
          El Árbol de Pitágoras es un fractal construido a partir de cuadrados. Cada par de ramas crea una copia reducida de la figura entera a escala.
        </p>
      </div>

    </div>
  </aside>


  <script>
    // Canvas & Context Setup
    const canvas = document.getElementById('fractalCanvas');
    const ctx = canvas.getContext('2d');

    // UI Panel & Elements
    const controlsPanel = document.getElementById('controlsPanel');
    const toggleControlsBtn = document.getElementById('toggleControlsBtn');
    const toggleBtnText = document.getElementById('toggleBtnText');
    const closeDrawerBtn = document.getElementById('closeDrawerBtn');

    const fractalSelect = document.getElementById('fractalSelect');
    const iterSlider = document.getElementById('iterSlider');
    const iterVal = document.getElementById('iterVal');
    const selfSimilarityToggle = document.getElementById('selfSimilarityToggle');
    const pythagorasControls = document.getElementById('pythagorasControls');
    const cesaroControls = document.getElementById('cesaroControls');
    const complexControls = document.getElementById('complexControls');
    const angleSlider = document.getElementById('angleSlider');
    const angleVal = document.getElementById('angleVal');
    const scaleSlider = document.getElementById('scaleSlider');
    const scaleVal = document.getElementById('scaleVal');
    const cesaroAngleSlider = document.getElementById('cesaroAngleSlider');
    const cesaroAngleVal = document.getElementById('cesaroAngleVal');
    const cxSlider = document.getElementById('cxSlider');
    const cxVal = document.getElementById('cxVal');
    const cySlider = document.getElementById('cySlider');
    const cyVal = document.getElementById('cyVal');
    const fractalDescription = document.getElementById('fractalDescription');
    const similarityDetailText = document.getElementById('similarityDetailText');
    const similarityBadge = document.getElementById('similarityBadge');
    const resetViewBtn = document.getElementById('resetViewBtn');
    const btnZoomIn = document.getElementById('btnZoomIn');
    const btnZoomOut = document.getElementById('btnZoomOut');

    // Camera State (Pan & Zoom)
    let camera = {
      x: 0,
      y: 0,
      zoom: 1,
      isDragging: false,
      dragStart: { x: 0, y: 0 }
    };

    // Touch Gesture State
    let touchState = {
      initialPinchDist: 0,
      initialZoom: 1
    };

    // Fractal Parameters State
    let currentFractal = 'pythagoras';
    let maxIterations = 5;
    let highlightSimilarity = true;
    let colorTheme = 'neon';
    let isDrawerOpen = false;

    let pythagorasAngle = Math.PI / 4;
    let pythagorasScale = 0.70;
    let cesaroAngleRad = (85 * Math.PI) / 180;
    let juliaC = { r: -0.7, i: 0.27 };

    // Palettes
    const palettes = {
      neon: (t) => `hsl(${t * 360}, 85%, 60%)`,
      fire: (t) => `hsl(${t * 60}, 100%, ${15 + t * 75}%)`,
      emerald: (t) => `hsl(${140 + t * 100}, 80%, ${25 + t * 50}%)`
    };

    // Descriptions
    const descriptions = {
      pythagoras: "El Árbol de Pitágoras se construye con cuadrados recurrentes. Cada par de ramas crea una copia reducida idéntica de la figura entera.",
      koch: "El Copo de Nieve de Koch sustituye el tercio central de cada lado por una punta triangular. Tiene longitud infinita encorvada en un área finita.",
      cesaro: "El Copo de Nieve de Cesàro pliega los bordes hacia adentro formando estrellas autosimilares complejas con ángulo personalizable.",
      sierpinski: "El Triángulo de Sierpinski se subdivide en 3 triángulos congruentes. Cada subsistema es una réplica exacta del todo.",
      mandelbrot: "El Conjunto de Mandelbrot en el plano complejo $z_{n+1} = z_n^2 + c$. Al acercarte, emergen 'mini-Mandelbrots' autosemejantes.",
      julia: "El Conjunto de Julia muestra intrincadas espirales simétricas que repiten los mismos patrones dinámicos en cualquier nivel de amplificación."
    };

    const similarityTexts = {
      pythagoras: "La rama dorada resaltada es geométricamente indistinguible de la estructura del árbol completo.",
      koch: "Cada tramo resplandeciente contiene exactamente la misma estructura de picos que la figura completa.",
      cesaro: "El pliegue resaltado replica la estrella completa en escala menor.",
      sierpinski: "El triángulo destacado es una réplica exacta conteniendo infinitas sub-estructuras.",
      mandelbrot: "La zona resaltada contiene un bulbo secundario que repite la forma cardioide original.",
      julia: "Las ramificaciones secundarias replican exactamente la dinámica fractal del todo."
    };


    function resizeCanvas() {
      const rect = canvas.getBoundingClientRect();
      canvas.width = rect.width * window.devicePixelRatio;
      canvas.height = rect.height * window.devicePixelRatio;
      ctx.scale(window.devicePixelRatio, window.devicePixelRatio);
      render();
    }

    function resetCamera() {
      const width = canvas.width / window.devicePixelRatio;
      const height = canvas.height / window.devicePixelRatio;

      camera.zoom = 1;
      if (currentFractal === 'pythagoras') {
        camera.x = width / 2;
        camera.y = height - (window.innerWidth < 768 ? 120 : 80);
      } else if (currentFractal === 'sierpinski' || currentFractal === 'koch' || currentFractal === 'cesaro') {
        camera.x = width / 2;
        camera.y = height / 2 + (currentFractal === 'sierpinski' ? 30 : 0);
      } else {
        camera.x = width / 2;
        camera.y = height / 2;
      }
      render();
    }

    // Pythagoras Tree
    function drawPythagorasTree(x, y, size, angle, depth, isSubBranch = false) {
      if (depth === 0) return;

      ctx.save();
      ctx.translate(x, y);
      ctx.rotate(angle);

      const isHighlighted = isSubBranch && highlightSimilarity;
      
      if (isHighlighted) {
        ctx.fillStyle = 'rgba(251, 191, 36, 0.85)';
        ctx.strokeStyle = '#fbbf24';
        ctx.shadowColor = '#fbbf24';
        ctx.shadowBlur = 12;
      } else {
        const t = depth / maxIterations;
        ctx.fillStyle = palettes[colorTheme](t);
        ctx.strokeStyle = '#020617';
        ctx.shadowBlur = 0;
      }

      ctx.beginPath();
      ctx.rect(-size / 2, -size, size, size);
      ctx.fill();
      ctx.stroke();

      const nextSize = size * pythagorasScale;
      const leftAngle = -pythagorasAngle;
      const rightAngle = Math.PI / 2 - pythagorasAngle;

      const topY = -size;
      const leftX = -size / 2 + (nextSize / 2) * Math.cos(leftAngle);
      const rightX = size / 2 - (nextSize / 2) * Math.cos(rightAngle);

      const nextIsSub = isSubBranch || (depth === maxIterations && highlightSimilarity);

      drawPythagorasTree(-size / 2, topY, nextSize, leftAngle, depth - 1, nextIsSub);
      drawPythagorasTree(size / 2, topY, nextSize, rightAngle, depth - 1, isSubBranch);

      ctx.restore();
    }

    // Sierpinski Triangle
    function drawSierpinski(ax, ay, bx, by, cx, cy, depth, isSub) {
      if (depth === 0) {
        ctx.beginPath();
        ctx.moveTo(ax, ay);
        ctx.lineTo(bx, by);
        ctx.lineTo(cx, cy);
        ctx.closePath();

        if (isSub && highlightSimilarity) {
          ctx.fillStyle = 'rgba(251, 191, 36, 0.9)';
          ctx.shadowColor = '#fbbf24';
          ctx.shadowBlur = 10;
        } else {
          ctx.fillStyle = palettes[colorTheme](0.6);
          ctx.shadowBlur = 0;
        }
        ctx.fill();
        return;
      }

      const abx = (ax + bx) / 2, aby = (ay + by) / 2;
      const bcx = (bx + cx) / 2, bcy = (by + cy) / 2;
      const cax = (cx + ax) / 2, cay = (cy + ay) / 2;

      drawSierpinski(ax, ay, abx, aby, cax, cay, depth - 1, isSub || (depth === maxIterations && highlightSimilarity));
      drawSierpinski(abx, aby, bx, by, bcx, bcy, depth - 1, isSub);
      drawSierpinski(cax, cay, bcx, bcy, cx, cy, depth - 1, isSub);
    }

    // Koch Snowflake Segment
    function drawKochLine(ax, ay, bx, by, depth, isSubSegment = false) {
      if (depth === 0) {
        ctx.beginPath();
        ctx.moveTo(ax, ay);
        ctx.lineTo(bx, by);
        
        if (isSubSegment && highlightSimilarity) {
          ctx.strokeStyle = '#fbbf24';
          ctx.lineWidth = 3;
          ctx.shadowColor = '#fbbf24';
          ctx.shadowBlur = 10;
        } else {
          ctx.strokeStyle = palettes[colorTheme](0.7);
          ctx.lineWidth = 1.5;
          ctx.shadowBlur = 0;
        }
        ctx.stroke();
        return;
      }

      const dx = (bx - ax) / 3, dy = (by - ay) / 3;
      const p1x = ax + dx, p1y = ay + dy;
      const p3x = bx - dx, p3y = by - dy;

      const angle = -Math.PI / 3;
      const p2x = p1x + dx * Math.cos(angle) - dy * Math.sin(angle);
      const p2y = p1y + dx * Math.sin(angle) + dy * Math.cos(angle);

      drawKochLine(ax, ay, p1x, p1y, depth - 1, isSubSegment);
      drawKochLine(p1x, p1y, p2x, p2y, depth - 1, isSubSegment || (depth === maxIterations && highlightSimilarity));
      drawKochLine(p2x, p2y, p3x, p3y, depth - 1, isSubSegment);
      drawKochLine(p3x, p3y, bx, by, depth - 1, isSubSegment);
    }

    function drawKochSnowflake(cx, cy, size, depth) {
      const h = size * Math.sqrt(3) / 2;
      const p1 = { x: cx, y: cy - (2/3) * h };
      const p2 = { x: cx - size / 2, y: cy + (1/3) * h };
      const p3 = { x: cx + size / 2, y: cy + (1/3) * h };

      drawKochLine(p1.x, p1.y, p3.x, p3.y, depth, false);
      drawKochLine(p3.x, p3.y, p2.x, p2.y, depth, false);
      drawKochLine(p2.x, p2.y, p1.x, p1.y, depth, false);
    }

    // Cesàro Snowflake
    function drawCesaroLine(ax, ay, bx, by, depth, isSubSegment = false) {
      if (depth === 0) {
        ctx.beginPath();
        ctx.moveTo(ax, ay);
        ctx.lineTo(bx, by);
        
        if (isSubSegment && highlightSimilarity) {
          ctx.strokeStyle = '#fbbf24';
          ctx.lineWidth = 3;
          ctx.shadowColor = '#fbbf24';
          ctx.shadowBlur = 10;
        } else {
          ctx.strokeStyle = palettes[colorTheme](0.7);
          ctx.lineWidth = 1.5;
          ctx.shadowBlur = 0;
        }
        ctx.stroke();
        return;
      }

      const dx = bx - ax, dy = by - ay;
      const len = Math.hypot(dx, dy);
      const alpha = Math.atan2(dy, dx);

      const subLen = len / (2 + 2 * Math.cos(cesaroAngleRad / 2));

      const p1x = ax + (dx - subLen * Math.cos(alpha)) / 2;
      const p1y = ay + (dy - subLen * Math.sin(alpha)) / 2;
      const p3x = bx - (dx - subLen * Math.cos(alpha)) / 2;
      const p3y = by - (dy - subLen * Math.sin(alpha)) / 2;

      const midX = (ax + bx) / 2, midY = (ay + by) / 2;
      const perpAlpha = alpha - Math.PI / 2;

      const p2x = midX + subLen * Math.sin(cesaroAngleRad / 2) * Math.cos(perpAlpha);
      const p2y = midY + subLen * Math.sin(cesaroAngleRad / 2) * Math.sin(perpAlpha);

      drawCesaroLine(ax, ay, p1x, p1y, depth - 1, isSubSegment);
      drawCesaroLine(p1x, p1y, p2x, p2y, depth - 1, isSubSegment || (depth === maxIterations && highlightSimilarity));
      drawCesaroLine(p2x, p2y, p3x, p3y, depth - 1, isSubSegment);
      drawCesaroLine(p3x, p3y, bx, by, depth - 1, isSubSegment);
    }

    function drawCesaroSnowflake(cx, cy, size, depth) {
      const half = size / 2;
      const p1 = { x: cx - half, y: cy - half };
      const p2 = { x: cx + half, y: cy - half };
      const p3 = { x: cx + half, y: cy + half };
      const p4 = { x: cx - half, y: cy + half };

      drawCesaroLine(p1.x, p1.y, p2.x, p2.y, depth, false);
      drawCesaroLine(p2.x, p2.y, p3.x, p3.y, depth, false);
      drawCesaroLine(p3.x, p3.y, p4.x, p4.y, depth, false);
      drawCesaroLine(p4.x, p4.y, p1.x, p1.y, depth, false);
    }

    // Mandelbrot and Julia Complex Fractals
    function renderComplexFractal(type) {
      const width = Math.floor(canvas.width / window.devicePixelRatio);
      const height = Math.floor(canvas.height / window.devicePixelRatio);
      const imgData = ctx.createImageData(width, height);
      const data = imgData.data;

      const scale = 3.0 / (camera.zoom * Math.min(width, height));
      const maxIter = maxIterations * 12;

      for (let px = 0; px < width; px += 2) {
        for (let py = 0; py < height; py += 2) {
          let zx = (px - camera.x) * scale;
          let zy = (py - camera.y) * scale;

          let cx = type === 'julia' ? juliaC.r : zx;
          let cy = type === 'julia' ? juliaC.i : zy;

          if (type === 'mandelbrot') {
            zx = 0; zy = 0;
          }

          let n = 0;
          while (zx * zx + zy * zy < 4 && n < maxIter) {
            let tmp = zx * zx - zy * zy + cx;
            zy = 2 * zx * zy + cy;
            zx = tmp;
            n++;
          }

          // Highlight similarity for sub-bulbs
          let isSimilarSpot = false;
          if (highlightSimilarity && type === 'mandelbrot') {
            let distToBulb = Math.hypot((px - camera.x) * scale - (-0.12), (py - camera.y) * scale - 0.74);
            if (distToBulb < 0.18) isSimilarSpot = true;
          }

          let r = 0, g = 0, b = 0;
          if (n < maxIter) {
            let t = n / maxIter;
            if (isSimilarSpot) {
              r = 251; g = 191; b = 36;
            } else if (colorTheme === 'neon') {
              r = Math.floor(Math.sin(t * 10) * 127 + 128);
              g = Math.floor(Math.sin(t * 10 + 2) * 127 + 128);
              b = Math.floor(Math.sin(t * 10 + 4) * 127 + 128);
            } else if (colorTheme === 'fire') {
              r = Math.floor(t * 255);
              g = Math.floor(t * t * 255);
              b = 30;
            } else {
              r = 20; g = Math.floor(t * 255); b = Math.floor(t * 200);
            }
          }

          for (let dx = 0; dx < 2 && px + dx < width; dx++) {
            for (let dy = 0; dy < 2 && py + dy < height; dy++) {
              let idx = ((py + dy) * width + (px + dx)) * 4;
              data[idx] = r;
              data[idx + 1] = g;
              data[idx + 2] = b;
              data[idx + 3] = 255;
            }
          }
        }
      }
      ctx.putImageData(imgData, 0, 0);
    }

    // Main Render Loop
    function render() {
      const width = canvas.width / window.devicePixelRatio;
      const height = canvas.height / window.devicePixelRatio;

      ctx.fillStyle = '#020617';
      ctx.fillRect(0, 0, width, height);

      if (currentFractal === 'mandelbrot' || currentFractal === 'julia') {
        renderComplexFractal(currentFractal);
        return;
      }

      ctx.save();
      ctx.translate(camera.x, camera.y);
      ctx.scale(camera.zoom, camera.zoom);

      if (currentFractal === 'pythagoras') {
        const baseSize = Math.min(width, height) * (window.innerWidth < 768 ? 0.14 : 0.18);
        drawPythagorasTree(0, 0, baseSize, 0, maxIterations, false);
      } else if (currentFractal === 'sierpinski') {
        const size = Math.min(width, height) * 0.55;
        const h = size * Math.sqrt(3) / 2;
        drawSierpinski(0, -h / 2, -size / 2, h / 2, size / 2, h / 2, maxIterations, false);
      } else if (currentFractal === 'koch') {
        const size = Math.min(width, height) * 0.5;
        drawKochSnowflake(0, 0, size, maxIterations);
      } else if (currentFractal === 'cesaro') {
        const size = Math.min(width, height) * 0.42;
        drawCesaroSnowflake(0, 0, size, maxIterations);
      }

      ctx.restore();
    }


    // Mobile Drawer Toggle Logic
    function toggleDrawer(open) {
      isDrawerOpen = open !== undefined ? open : !isDrawerOpen;
      if (isDrawerOpen) {
        controlsPanel.classList.remove('translate-y-full');
        controlsPanel.classList.add('translate-y-0');
        toggleBtnText.textContent = 'Ocultar Panel';
      } else {
        controlsPanel.classList.add('translate-y-full');
        controlsPanel.classList.remove('translate-y-0');
        toggleBtnText.textContent = 'Ajustar Parámetros';
      }
    }

    toggleControlsBtn.addEventListener('click', () => toggleDrawer());
    closeDrawerBtn.addEventListener('click', () => toggleDrawer(false));

    // Mouse Navigation (Pan & Wheel Zoom)
    canvas.addEventListener('mousedown', (e) => {
      camera.isDragging = true;
      camera.dragStart = { x: e.clientX - camera.x, y: e.clientY - camera.y };
    });

    window.addEventListener('mousemove', (e) => {
      if (camera.isDragging) {
        camera.x = e.clientX - camera.dragStart.x;
        camera.y = e.clientY - camera.dragStart.y;
        render();
      }
    });

    window.addEventListener('mouseup', () => { camera.isDragging = false; });

    canvas.addEventListener('wheel', (e) => {
      e.preventDefault();
      const zoomFactor = e.deltaY < 0 ? 1.15 : 0.85;
      const mouseX = e.clientX;
      const mouseY = e.clientY;

      camera.x = mouseX - (mouseX - camera.x) * zoomFactor;
      camera.y = mouseY - (mouseY - camera.y) * zoomFactor;
      camera.zoom *= zoomFactor;
      render();
    }, { passive: false });

    // Touch Gestures (Mobile Pan & Pinch-to-Zoom)
    canvas.addEventListener('touchstart', (e) => {
      if (e.touches.length === 1) {
        camera.isDragging = true;
        camera.dragStart = { x: e.touches[0].clientX - camera.x, y: e.touches[0].clientY - camera.y };
      } else if (e.touches.length === 2) {
        camera.isDragging = false;
        touchState.initialPinchDist = Math.hypot(
          e.touches[0].clientX - e.touches[1].clientX,
          e.touches[0].clientY - e.touches[1].clientY
        );
        touchState.initialZoom = camera.zoom;
      }
    }, { passive: true });

    canvas.addEventListener('touchmove', (e) => {
      if (e.touches.length === 1 && camera.isDragging) {
        camera.x = e.touches[0].clientX - camera.dragStart.x;
        camera.y = e.touches[0].clientY - camera.dragStart.y;
        render();
      } else if (e.touches.length === 2) {
        const dist = Math.hypot(
          e.touches[0].clientX - e.touches[1].clientX,
          e.touches[0].clientY - e.touches[1].clientY
        );
        if (touchState.initialPinchDist > 0) {
          const factor = dist / touchState.initialPinchDist;
          camera.zoom = touchState.initialZoom * factor;
          render();
        }
      }
    }, { passive: true });

    canvas.addEventListener('touchend', () => { camera.isDragging = false; });

    // Quick Action Zoom Buttons
    btnZoomIn.addEventListener('click', () => {
      camera.zoom *= 1.25;
      render();
    });

    btnZoomOut.addEventListener('click', () => {
      camera.zoom *= 0.8;
      render();
    });

    resetViewBtn.addEventListener('click', resetCamera);

    // Fractal Selection & Sliders Listener Setup
    fractalSelect.addEventListener('change', (e) => {
      currentFractal = e.target.value;
      
      pythagorasControls.classList.toggle('hidden', currentFractal !== 'pythagoras');
      cesaroControls.classList.toggle('hidden', currentFractal !== 'cesaro');
      complexControls.classList.toggle('hidden', currentFractal !== 'julia');

      if (currentFractal === 'pythagoras') {
        iterSlider.max = 10;
      } else if (currentFractal === 'julia' || currentFractal === 'mandelbrot') {
        iterSlider.max = 8;
      } else {
        iterSlider.max = 6;
      }

      iterSlider.value = Math.min(iterSlider.value, iterSlider.max);
      maxIterations = parseInt(iterSlider.value);
      iterVal.textContent = maxIterations;
      fractalDescription.textContent = descriptions[currentFractal];
      similarityDetailText.textContent = similarityTexts[currentFractal];

      resetCamera();
    });

    iterSlider.addEventListener('input', (e) => {
      maxIterations = parseInt(e.target.value);
      iterVal.textContent = maxIterations;
      render();
    });

    angleSlider.addEventListener('input', (e) => {
      const deg = parseInt(e.target.value);
      pythagorasAngle = (deg * Math.PI) / 180;
      angleVal.textContent = `${deg}°`;
      render();
    });

    scaleSlider.addEventListener('input', (e) => {
      pythagorasScale = parseFloat(e.target.value);
      scaleVal.textContent = pythagorasScale.toFixed(2);
      render();
    });

    cesaroAngleSlider.addEventListener('input', (e) => {
      const deg = parseInt(e.target.value);
      cesaroAngleRad = (deg * Math.PI) / 180;
      cesaroAngleVal.textContent = `${deg}°`;
      render();
    });

    cxSlider.addEventListener('input', (e) => {
      juliaC.r = parseFloat(e.target.value);
      cxVal.textContent = juliaC.r.toFixed(3);
      render();
    });

    cySlider.addEventListener('input', (e) => {
      juliaC.i = parseFloat(e.target.value);
      cyVal.textContent = juliaC.i.toFixed(3);
      render();
    });

    selfSimilarityToggle.addEventListener('change', (e) => {
      highlightSimilarity = e.target.checked;
      similarityBadge.style.opacity = highlightSimilarity ? '1' : '0.3';
      render();
    });

    // Color theme buttons
    document.querySelectorAll('.theme-btn').forEach(btn => {
      btn.addEventListener('click', (e) => {
        document.querySelectorAll('.theme-btn').forEach(b => {
          b.classList.remove('border-indigo-500', 'bg-indigo-900/40', 'text-indigo-300');
          b.classList.add('border-slate-700', 'bg-slate-800', 'text-slate-300');
        });
        const target = e.currentTarget;
        target.classList.remove('border-slate-700', 'bg-slate-800', 'text-slate-300');
        target.classList.add('border-indigo-500', 'bg-indigo-900/40', 'text-indigo-300');
        
        colorTheme = target.getAttribute('data-theme');
        render();
      });
    });

    // Init App
    window.addEventListener('resize', resizeCanvas);
    window.onload = () => {
      resizeCanvas();
      resetCamera();
    };
  </script>
</body>
</html>
