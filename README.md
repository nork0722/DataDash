# DataDash
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>DataDash — AI-Powered Data Analytics Dashboard</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700;800&family=JetBrains+Mono:wght@400;500;600&display=swap" rel="stylesheet">
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css">
<script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js"></script>
<style>
/* ===================== Design tokens =====================
   Brief-mandated palette: #2563EB primary / #1E293B secondary / #3B82F6 accent / #F8FAFC bg
   Signature element: live "pulse grid" — a quiet animated lattice of dots behind the hero
   that brighten in a slow wave, evoking a dataset being scanned cell by cell.
   Display type: Poppins (brief-mandated). Data/numeric type: JetBrains Mono for tabular precision.
=========================================================== */
:root{
  --primary:#2563EB;
  --secondary:#1E293B;
  --accent:#3B82F6;
  --bg:#F8FAFC;
  --surface:#FFFFFF;
  --border:#E2E8F0;
  --text:#0F172A;
  --text-dim:#64748B;
  --success:#16A34A;
  --warn:#D97706;
  --danger:#DC2626;
  --radius:12px;
  --shadow:0 1px 2px rgba(15,23,42,.04), 0 8px 24px -8px rgba(15,23,42,.08);
  --sidebar-w:240px;
}
[data-theme="dark"]{
  --bg:#0B1220;
  --surface:#111A2C;
  --border:#1E293B;
  --text:#E2E8F0;
  --text-dim:#94A3B8;
  --secondary:#F1F5F9;
  --shadow:0 1px 2px rgba(0,0,0,.3), 0 8px 24px -8px rgba(0,0,0,.5);
}
*{box-sizing:border-box; margin:0; padding:0;}
html{scroll-behavior:smooth;}
body{
  font-family:'Poppins',sans-serif;
  background:var(--bg);
  color:var(--text);
  transition:background .25s ease, color .25s ease;
  -webkit-font-smoothing:antialiased;
}
.mono{font-family:'JetBrains Mono',monospace;}
a{color:inherit; text-decoration:none;}
button{font-family:inherit; cursor:pointer; border:none;}
::selection{background:var(--accent); color:#fff;}
@media (prefers-reduced-motion: reduce){ *{animation-duration:.001ms !important; transition-duration:.001ms !important;} }

/* ===== Top nav (landing) ===== */
.topnav{
  position:fixed; top:0; left:0; right:0; z-index:50;
  display:flex; align-items:center; justify-content:space-between;
  padding:18px 6vw;
  backdrop-filter:blur(10px);
  background:color-mix(in srgb, var(--bg) 80%, transparent);
  border-bottom:1px solid var(--border);
}
.brand{display:flex; align-items:center; gap:10px; font-weight:700; font-size:1.15rem; letter-spacing:-.02em;}
.brand .dot-logo{
  width:30px; height:30px; border-radius:9px;
  background:linear-gradient(135deg,var(--primary),var(--accent));
  display:flex; align-items:center; justify-content:center; color:#fff; font-size:.85rem;
}
.nav-actions{display:flex; align-items:center; gap:14px;}
.theme-toggle{
  width:42px; height:42px; border-radius:50%;
  background:var(--surface); border:1px solid var(--border); color:var(--text);
  display:flex; align-items:center; justify-content:center; font-size:1rem;
  transition:transform .2s ease;
}
.theme-toggle:hover{transform:rotate(20deg);}
.btn{
  padding:11px 22px; border-radius:9px; font-weight:600; font-size:.92rem;
  transition:transform .15s ease, box-shadow .15s ease, opacity .15s ease;
  display:inline-flex; align-items:center; gap:8px;
}
.btn-primary{background:var(--primary); color:#fff; box-shadow:0 6px 18px -6px rgba(37,99,235,.55);}
.btn-primary:hover{transform:translateY(-2px); box-shadow:0 10px 24px -6px rgba(37,99,235,.6);}
.btn-ghost{background:transparent; color:var(--text); border:1px solid var(--border);}
.btn-ghost:hover{background:var(--surface);}

/* ===== Hero ===== */
.hero{
  position:relative; overflow:hidden;
  padding:170px 6vw 90px;
  display:grid; grid-template-columns:1.1fr .9fr; gap:50px; align-items:center;
}
.pulse-grid{
  position:absolute; inset:0; z-index:0; opacity:.55;
  background-image:radial-gradient(circle, color-mix(in srgb, var(--primary) 35%, transparent) 1.4px, transparent 1.4px);
  background-size:26px 26px;
  -webkit-mask-image:radial-gradient(ellipse 60% 70% at 75% 35%, black 10%, transparent 70%);
  mask-image:radial-gradient(ellipse 60% 70% at 75% 35%, black 10%, transparent 70%);
  animation:pulseShift 9s ease-in-out infinite alternate;
}
@keyframes pulseShift{ 0%{background-position:0 0; opacity:.35;} 100%{background-position:60px 40px; opacity:.65;} }
.hero-content{position:relative; z-index:1;}
.eyebrow{
  display:inline-flex; align-items:center; gap:8px;
  font-family:'JetBrains Mono',monospace; font-size:.75rem; letter-spacing:.08em; text-transform:uppercase;
  color:var(--primary); background:color-mix(in srgb, var(--primary) 10%, transparent);
  padding:6px 12px; border-radius:20px; margin-bottom:22px; border:1px solid color-mix(in srgb, var(--primary) 25%, transparent);
}
.eyebrow .blink{width:6px; height:6px; border-radius:50%; background:var(--primary); animation:blink 1.6s ease-in-out infinite;}
@keyframes blink{0%,100%{opacity:1;}50%{opacity:.25;}}
h1.hero-title{
  font-size:clamp(2.4rem, 4.2vw, 3.6rem); font-weight:800; letter-spacing:-.03em; line-height:1.08;
  color:var(--secondary); margin-bottom:20px;
}
h1.hero-title .grad{
  background:linear-gradient(90deg,var(--primary),var(--accent));
  -webkit-background-clip:text; background-clip:text; color:transparent;
}
.hero-sub{font-size:1.08rem; color:var(--text-dim); max-width:520px; line-height:1.65; margin-bottom:34px;}
.hero-cta{display:flex; gap:14px; flex-wrap:wrap;}

/* hero data card mock */
.hero-visual{position:relative; z-index:1;}
.mock-card{
  background:var(--surface); border:1px solid var(--border); border-radius:18px;
  box-shadow:var(--shadow); padding:22px; transform:rotate(-1.2deg);
}
.mock-card-header{display:flex; justify-content:space-between; align-items:center; margin-bottom:16px;}
.mock-dots{display:flex; gap:6px;}
.mock-dots span{width:9px; height:9px; border-radius:50%; background:var(--border);}
.mock-bars{display:flex; align-items:flex-end; gap:8px; height:130px; padding:0 4px;}
.mock-bars i{
  flex:1; background:linear-gradient(180deg,var(--accent),var(--primary)); border-radius:6px 6px 2px 2px;
  animation:rise 1.4s ease forwards; transform-origin:bottom; transform:scaleY(0);
}
@keyframes rise{to{transform:scaleY(1);}}
.mock-stat-row{display:flex; gap:10px; margin-top:18px;}
.mock-stat{flex:1; background:var(--bg); border:1px solid var(--border); border-radius:10px; padding:10px 12px;}
.mock-stat .v{font-family:'JetBrains Mono'; font-weight:600; font-size:1.05rem; color:var(--secondary);}
.mock-stat .l{font-size:.72rem; color:var(--text-dim); text-transform:uppercase; letter-spacing:.05em;}

/* ===== Features ===== */
.section{padding:80px 6vw;}
.section-head{max-width:560px; margin:0 auto 46px; text-align:center;}
.section-head h2{font-size:clamp(1.7rem,2.6vw,2.3rem); font-weight:700; letter-spacing:-.02em; color:var(--secondary); margin-bottom:12px;}
.section-head p{color:var(--text-dim); font-size:1rem;}
.feature-grid{display:grid; grid-template-columns:repeat(auto-fit,minmax(250px,1fr)); gap:20px; max-width:1180px; margin:0 auto;}
.feature-card{
  background:var(--surface); border:1px solid var(--border); border-radius:var(--radius);
  padding:26px; transition:transform .2s ease, box-shadow .2s ease;
}
.feature-card:hover{transform:translateY(-4px); box-shadow:var(--shadow);}
.feature-icon{
  width:44px; height:44px; border-radius:11px; display:flex; align-items:center; justify-content:center;
  background:color-mix(in srgb, var(--primary) 12%, transparent); color:var(--primary); font-size:1.1rem; margin-bottom:16px;
}
.feature-card h3{font-size:1.02rem; font-weight:600; margin-bottom:8px; color:var(--secondary);}
.feature-card p{font-size:.88rem; color:var(--text-dim); line-height:1.55;}

.cta-band{
  margin:0 6vw 90px; padding:50px 6vw; border-radius:22px; text-align:center;
  background:linear-gradient(120deg,var(--secondary),#0B1220);
  color:#fff;
}
[data-theme="dark"] .cta-band{background:linear-gradient(120deg,#1E293B,#020617);}
.cta-band h2{font-size:1.8rem; margin-bottom:10px; font-weight:700;}
.cta-band p{color:#CBD5E1; margin-bottom:24px;}

footer.landing-footer{text-align:center; padding:30px; color:var(--text-dim); font-size:.82rem; border-top:1px solid var(--border);}

/* ===== App shell (dashboard) ===== */
#app-shell{display:none; min-height:100vh;}
#app-shell.active{display:flex;}
.sidebar{
  width:var(--sidebar-w); flex-shrink:0; background:var(--surface); border-right:1px solid var(--border);
  height:100vh; position:sticky; top:0; display:flex; flex-direction:column; padding:20px 14px; z-index:40;
}
.sidebar .brand{padding:6px 8px 22px;}
.nav-group{font-family:'JetBrains Mono'; font-size:.68rem; text-transform:uppercase; letter-spacing:.08em; color:var(--text-dim); padding:14px 12px 6px;}
.nav-item{
  display:flex; align-items:center; gap:11px; padding:10px 12px; border-radius:9px; color:var(--text-dim);
  font-size:.88rem; font-weight:500; margin-bottom:2px; transition:background .15s, color .15s;
}
.nav-item i{width:16px; text-align:center;}
.nav-item:hover{background:var(--bg); color:var(--text);}
.nav-item.active{background:color-mix(in srgb, var(--primary) 12%, transparent); color:var(--primary);}
.sidebar-foot{margin-top:auto; padding:12px; border-top:1px solid var(--border); display:flex; align-items:center; justify-content:space-between;}

.main-area{flex:1; min-width:0; display:flex; flex-direction:column;}
.topbar{
  position:sticky; top:0; z-index:30; display:flex; align-items:center; justify-content:space-between;
  padding:16px 28px; border-bottom:1px solid var(--border); background:color-mix(in srgb, var(--bg) 85%, transparent);
  backdrop-filter:blur(8px);
}
.topbar h2{font-size:1.15rem; font-weight:600; color:var(--secondary);}
.topbar-actions{display:flex; gap:10px; align-items:center;}
.content{padding:26px 28px 60px;}
.view{display:none;}
.view.active{display:block; animation:fadeUp .35s ease;}
@keyframes fadeUp{from{opacity:0; transform:translateY(8px);} to{opacity:1; transform:translateY(0);}}

/* drop zone */
.dropzone{
  border:2px dashed var(--border); border-radius:18px; padding:60px 20px; text-align:center;
  background:var(--surface); transition:border-color .2s, background .2s; cursor:pointer;
}
.dropzone.drag{border-color:var(--primary); background:color-mix(in srgb, var(--primary) 6%, transparent);}
.dropzone i{font-size:2.2rem; color:var(--primary); margin-bottom:14px;}
.dropzone h3{color:var(--secondary); margin-bottom:6px; font-weight:600;}
.dropzone p{color:var(--text-dim); font-size:.88rem;}
.dropzone input{display:none;}

/* cards / grid */
.grid{display:grid; gap:18px;}
.grid.cols-4{grid-template-columns:repeat(4,1fr);}
.grid.cols-3{grid-template-columns:repeat(3,1fr);}
.grid.cols-2{grid-template-columns:repeat(2,1fr);}
@media (max-width:900px){.grid.cols-4,.grid.cols-3,.grid.cols-2{grid-template-columns:1fr 1fr;}}
@media (max-width:600px){.grid.cols-4,.grid.cols-3,.grid.cols-2{grid-template-columns:1fr;}}

.card{background:var(--surface); border:1px solid var(--border); border-radius:var(--radius); padding:20px; box-shadow:var(--shadow);}
.card h4{font-size:.92rem; font-weight:600; color:var(--secondary); margin-bottom:14px; display:flex; align-items:center; gap:8px;}
.stat-card .v{font-family:'JetBrains Mono'; font-size:1.7rem; font-weight:600; color:var(--secondary);}
.stat-card .l{font-size:.78rem; color:var(--text-dim); margin-top:4px;}
.stat-card .icon{width:38px; height:38px; border-radius:10px; display:flex; align-items:center; justify-content:center; margin-bottom:12px; font-size:1rem;}

table{width:100%; border-collapse:collapse; font-size:.84rem;}
thead th{
  text-align:left; padding:9px 12px; background:var(--bg); color:var(--text-dim); font-weight:600;
  font-family:'JetBrains Mono'; font-size:.72rem; text-transform:uppercase; letter-spacing:.04em;
  border-bottom:1px solid var(--border); white-space:nowrap; position:sticky; top:0;
}
tbody td{padding:9px 12px; border-bottom:1px solid var(--border); font-family:'JetBrains Mono'; font-size:.8rem; white-space:nowrap; color:var(--text);}
tbody tr:hover{background:var(--bg);}
.table-wrap{overflow:auto; max-height:460px; border:1px solid var(--border); border-radius:10px;}

.badge{display:inline-flex; align-items:center; gap:5px; padding:3px 9px; border-radius:20px; font-size:.72rem; font-weight:600; font-family:'JetBrains Mono';}
.badge.num{background:color-mix(in srgb, var(--primary) 12%, transparent); color:var(--primary);}
.badge.cat{background:color-mix(in srgb, var(--warn) 14%, transparent); color:var(--warn);}
.badge.bool{background:color-mix(in srgb, var(--success) 14%, transparent); color:var(--success);}
.badge.good{background:color-mix(in srgb, var(--success) 14%, transparent); color:var(--success);}
.badge.bad{background:color-mix(in srgb, var(--danger) 14%, transparent); color:var(--danger);}

.controls-row{display:flex; gap:12px; flex-wrap:wrap; align-items:center; margin-bottom:18px;}
select, input[type=text], input[type=search]{
  padding:9px 12px; border-radius:8px; border:1px solid var(--border); background:var(--surface); color:var(--text);
  font-family:inherit; font-size:.85rem; min-width:150px;
}
.chart-box{height:380px; position:relative;}

.qual-row{display:flex; align-items:center; justify-content:space-between; padding:10px 0; border-bottom:1px solid var(--border); font-size:.85rem;}
.qual-row:last-child{border-bottom:none;}
.bar-track{flex:1; height:6px; border-radius:4px; background:var(--bg); margin:0 14px; overflow:hidden;}
.bar-fill{height:100%; border-radius:4px; background:var(--primary);}

.insight-item{display:flex; gap:14px; padding:16px 0; border-bottom:1px solid var(--border);}
.insight-item:last-child{border-bottom:none;}
.insight-ico{width:36px; height:36px; flex-shrink:0; border-radius:9px; display:flex; align-items:center; justify-content:center; font-size:.95rem;}
.insight-item h5{font-size:.9rem; font-weight:600; color:var(--secondary); margin-bottom:4px;}
.insight-item p{font-size:.83rem; color:var(--text-dim); line-height:1.5;}

.toast-stack{position:fixed; bottom:22px; right:22px; z-index:200; display:flex; flex-direction:column; gap:10px;}
.toast{
  background:var(--secondary); color:#fff; padding:13px 18px; border-radius:10px; font-size:.85rem;
  box-shadow:var(--shadow); display:flex; align-items:center; gap:10px; min-width:240px;
  animation:toastIn .25s ease;
}
@keyframes toastIn{from{opacity:0; transform:translateX(20px);} to{opacity:1; transform:translateX(0);}}
.toast i{color:var(--accent);}

.loader-overlay{
  position:fixed; inset:0; background:rgba(15,23,42,.45); z-index:300; display:none;
  align-items:center; justify-content:center; backdrop-filter:blur(2px);
}
.loader-overlay.active{display:flex;}
.spinner{width:46px; height:46px; border-radius:50%; border:3px solid rgba(255,255,255,.25); border-top-color:#fff; animation:spin .8s linear infinite;}
@keyframes spin{to{transform:rotate(360deg);}}

.empty-state{text-align:center; padding:60px 20px; color:var(--text-dim);}
.empty-state i{font-size:2.4rem; color:var(--border); margin-bottom:14px;}

.fab-back{
  position:fixed; bottom:20px; left:20px; z-index:60; width:0; opacity:0;
}
.mobile-menu-btn{display:none;}
@media (max-width:880px){
  .hero{grid-template-columns:1fr; padding-top:140px;}
  .sidebar{position:fixed; left:-260px; transition:left .25s ease; box-shadow:var(--shadow);}
  .sidebar.open{left:0;}
  .mobile-menu-btn{display:flex; width:38px;height:38px;align-items:center;justify-content:center;border-radius:8px;background:var(--surface);border:1px solid var(--border);}
}
::-webkit-scrollbar{width:9px; height:9px;}
::-webkit-scrollbar-thumb{background:var(--border); border-radius:6px;}
</style>
</head>
<body>

<!-- ============ LANDING PAGE ============ -->
<div id="landing">
  <nav class="topnav">
    <div class="brand"><div class="dot-logo"><i class="fa-solid fa-chart-line"></i></div>DataDash</div>
    <div class="nav-actions">
      <button class="theme-toggle" id="themeToggleLanding" aria-label="Toggle dark mode"><i class="fa-solid fa-moon"></i></button>
      <button class="btn btn-primary" id="ctaTopLaunch"><i class="fa-solid fa-bolt"></i> Launch Dashboard</button>
    </div>
  </nav>

  <section class="hero">
    <div class="pulse-grid"></div>
    <div class="hero-content">
      <div class="eyebrow"><span class="blink"></span> No signup · No database · Runs in your browser</div>
      <h1 class="hero-title">Turn raw CSVs into <span class="grad">clear decisions.</span></h1>
      <p class="hero-sub">Upload a dataset and DataDash instantly profiles it — structure, data quality, statistics, correlations — then surfaces plain-English insights and lets you build interactive charts in seconds.</p>
      <div class="hero-cta">
        <button class="btn btn-primary" id="ctaHeroLaunch"><i class="fa-solid fa-upload"></i> Upload a CSV</button>
        <button class="btn btn-ghost" id="ctaHeroDemo"><i class="fa-solid fa-play"></i> Try sample dataset</button>
      </div>
    </div>
    <div class="hero-visual">
      <div class="mock-card">
        <div class="mock-card-header">
          <div class="mock-dots"><span></span><span></span><span></span></div>
          <span class="mono" style="font-size:.72rem;color:var(--text-dim);">revenue_by_region.csv</span>
        </div>
        <div class="mock-bars">
          <i style="height:40%;animation-delay:.05s"></i><i style="height:70%;animation-delay:.1s"></i>
          <i style="height:55%;animation-delay:.15s"></i><i style="height:90%;animation-delay:.2s"></i>
          <i style="height:35%;animation-delay:.25s"></i><i style="height:65%;animation-delay:.3s"></i>
          <i style="height:80%;animation-delay:.35s"></i><i style="height:50%;animation-delay:.4s"></i>
        </div>
        <div class="mock-stat-row">
          <div class="mock-stat"><div class="v">12,480</div><div class="l">Rows</div></div>
          <div class="mock-stat"><div class="v">0.6%</div><div class="l">Missing</div></div>
          <div class="mock-stat"><div class="v">A−</div><div class="l">Quality</div></div>
        </div>
      </div>
    </div>
  </section>

  <section class="section" id="features">
    <div class="section-head">
      <h2>Everything an analyst opens ten tabs for</h2>
      <p>Profiling, quality checks, statistics, visualization, and AI narration — one dashboard.</p>
    </div>
    <div class="feature-grid">
      <div class="feature-card"><div class="feature-icon"><i class="fa-solid fa-file-csv"></i></div><h3>Drag-and-drop CSV upload</h3><p>Drop a file or browse — validated instantly, processed entirely in memory, nothing leaves your browser.</p></div>
      <div class="feature-card"><div class="feature-icon"><i class="fa-solid fa-table"></i></div><h3>Dataset overview</h3><p>Rows, columns, dtypes, memory footprint, and a live preview the moment your file loads.</p></div>
      <div class="feature-card"><div class="feature-icon"><i class="fa-solid fa-shield-halved"></i></div><h3>Data quality scan</h3><p>Missing values, duplicates, empty columns, and a column-by-column health breakdown.</p></div>
      <div class="feature-card"><div class="feature-icon"><i class="fa-solid fa-square-root-variable"></i></div><h3>Statistical summary</h3><p>Mean, median, mode, std dev, variance, quartiles — auto-computed for every numeric column.</p></div>
      <div class="feature-card"><div class="feature-icon"><i class="fa-solid fa-chart-pie"></i></div><h3>Interactive visualizations</h3><p>Bar, line, pie, histogram, scatter, box, and correlation heatmap — pick your axes and theme.</p></div>
      <div class="feature-card"><div class="feature-icon"><i class="fa-so
