<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>CashFlow Tracker</title>
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="CashFlow">
<meta name="mobile-web-app-capable" content="yes">
<meta name="theme-color" content="#0c0c0e">
<link rel="icon" type="image/svg+xml" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 64 64'%3E%3Crect width='64' height='64' rx='14' fill='%230c0c0e'/%3E%3Ctext x='32' y='44' font-size='36' text-anchor='middle' fill='%2322d3a0'%3E%24%3C/text%3E%3C/svg%3E">
<link rel="apple-touch-icon" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 180 180'%3E%3Crect width='180' height='180' rx='40' fill='%230c0c0e'/%3E%3Ctext x='90' y='125' font-size='100' text-anchor='middle' fill='%2322d3a0'%3E%24%3C/text%3E%3C/svg%3E">
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@tabler/icons-webfont@3.19.0/dist/tabler-icons.min.css">
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2/dist/umd/supabase.min.js"></script>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800;900&display=swap');

  :root {
    --bg:     #0c0c0e;
    --bg2:    #13131a;
    --bg3:    #1c1c26;
    --bg4:    #242430;
    --border: #2a2a38;
    --txt:    #f0f0f8;
    --txt2:   #9090b0;
    --txt3:   #4a4a60;
    --pos:    #22d3a0;
    --neg:    #f43f5e;
    --accent: #7c3aed;
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: var(--bg);
    color: var(--txt);
    font-family: 'Inter', system-ui, sans-serif;
    min-height: 100vh;
  }

  .app { max-width: 960px; margin: 0 auto; padding: 2rem 1.25rem 5rem; }

  .top-bar {
    display: flex; align-items: center; justify-content: space-between;
    margin-bottom: 2rem; flex-wrap: wrap; gap: 12px;
  }
  .logo { font-size: 20px; font-weight: 900; letter-spacing: -.04em; color: var(--txt); }
  .logo span { color: var(--pos); }

  .nav {
    display: flex; background: var(--bg2); border: 1px solid var(--border);
    border-radius: 14px; overflow: hidden; margin-bottom: 2rem;
  }
  .nb {
    flex: 1; padding: 13px 8px; font-size: 13px; font-weight: 700;
    text-align: center; cursor: pointer; border: none; background: transparent;
    color: var(--txt3); border-right: 1px solid var(--border);
    transition: all .2s; letter-spacing: .01em; font-family: inherit;
    display: flex; align-items: center; justify-content: center; gap: 6px;
  }
  .nb:last-child { border-right: none; }
  .nb.on { background: var(--bg3); color: var(--txt); box-shadow: inset 0 -2px 0 var(--pos); }
  .nb:hover:not(.on) { background: var(--bg3); color: var(--txt2); }

  .pg { display: none; }
  .pg.on { display: block; }

  .sec-hdr {
    display: flex; align-items: center; justify-content: space-between;
    margin-bottom: 1.5rem; flex-wrap: wrap; gap: 12px;
  }
  .sec-title { font-size: 20px; font-weight: 800; letter-spacing: -.03em; }
  .sec-sub { font-size: 12px; color: var(--txt3); margin-top: 3px; font-weight: 500; }

  .yr-switch { display: flex; align-items: center; gap: 8px; }
  .yr-current {
    padding: 7px 14px; font-size: 14px; font-weight: 800; border-radius: 8px;
    background: var(--bg3); color: var(--txt); border: 1px solid var(--border);
    letter-spacing: -.01em;
  }
  .yr-edit-btn {
    width: 32px; height: 32px; border-radius: 8px; border: 1px solid var(--border);
    background: transparent; color: var(--txt3); cursor: pointer; font-family: inherit;
    display: flex; align-items: center; justify-content: center; transition: all .15s; font-size: 14px;
  }
  .yr-edit-btn:hover { background: var(--bg3); color: var(--txt); }
  .yr-input {
    width: 90px; padding: 7px 10px; font-size: 14px; font-weight: 800; text-align: center;
  }

  .pie-wrap { display: flex; gap: 28px; align-items: center; flex-wrap: wrap; }
  .pie-legend { flex: 1; min-width: 200px; display: flex; flex-direction: column; gap: 11px; }
  .pie-leg-item { display: flex; justify-content: space-between; align-items: center; font-size: 13px; font-weight: 600; color: var(--txt2); gap: 10px; }
  .pie-leg-name { display: flex; align-items: center; gap: 8px; }
  .pie-leg-val { color: var(--txt); font-weight: 700; white-space: nowrap; }

  .kpi-row { display: grid; grid-template-columns: repeat(auto-fit, minmax(150px, 1fr)); gap: 12px; margin-bottom: 2rem; }
  .kpi {
    background: var(--bg2); border: 1px solid var(--border); border-radius: 14px;
    padding: 18px 20px; position: relative; overflow: hidden;
  }
  .kpi::after {
    content: ''; position: absolute; bottom: 0; left: 0; right: 0; height: 2px;
    background: var(--kpi-color, var(--accent));
  }
  .kpi-label { font-size: 11px; font-weight: 700; letter-spacing: .09em; text-transform: uppercase; color: var(--txt3); margin-bottom: 10px; }
  .kpi-value { font-size: 24px; font-weight: 900; letter-spacing: -.04em; }

  .cards-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(220px, 1fr)); gap: 14px; }

  .cat-card {
    background: var(--bg2); border: 1px solid var(--border); border-radius: 16px;
    padding: 1.25rem; display: flex; flex-direction: column; gap: 12px;
  }
  .cat-head { display: flex; align-items: center; gap: 12px; }
  .cat-icon {
    width: 42px; height: 42px; border-radius: 12px;
    display: flex; align-items: center; justify-content: center; flex-shrink: 0;
  }
  .cat-name { font-size: 14px; font-weight: 700; color: var(--txt); }
  .cat-count { font-size: 11px; color: var(--txt3); margin-top: 2px; font-weight: 500; }
  .cat-amount { font-size: 28px; font-weight: 900; letter-spacing: -.04em; }
  .bar-bg { height: 3px; background: var(--bg3); border-radius: 2px; overflow: hidden; }
  .bar-fill { height: 3px; border-radius: 2px; }

  .cat-box-head {
    display: flex; align-items: baseline; justify-content: space-between; gap: 10px;
    padding-bottom: 12px; border-bottom: 1px solid var(--border);
  }
  .cat-box-head .cat-name { font-size: 15px; }
  .cat-box-head .cat-amount { font-size: 18px; }
  .cat-box-head-left { display: flex; align-items: center; gap: 8px; }
  .box-expand-btn {
    width: 24px; height: 24px; border-radius: 7px; border: 1px solid var(--border);
    background: transparent; color: var(--txt3); cursor: pointer; font-family: inherit;
    display: flex; align-items: center; justify-content: center; font-size: 12px;
    transition: all .15s; flex-shrink: 0;
  }
  .box-expand-btn:hover { background: var(--bg3); color: var(--txt); }
  .tx-empty { font-size: 12px; color: var(--txt3); padding: 4px 0; }

  .tx-list {
    display: flex; flex-direction: column; max-height: 224px; overflow: hidden;
    transition: max-height .25s ease;
  }
  .tx-list.expanded {
    max-height: 420px; overflow-y: auto; padding-right: 4px;
  }
  .tx-list.expanded::-webkit-scrollbar { width: 6px; }
  .tx-list.expanded::-webkit-scrollbar-track { background: transparent; }
  .tx-list.expanded::-webkit-scrollbar-thumb { background: var(--border); border-radius: 3px; }
  .tx { display: flex; justify-content: space-between; align-items: center; padding: 8px 0; border-bottom: 1px solid var(--border); }
  .tx:last-child { border-bottom: none; }
  .tx-desc { font-size: 13px; font-weight: 500; color: var(--txt2); }
  .tx-mo { font-size: 11px; color: var(--txt3); margin-top: 2px; }
  .tx-amt { font-size: 13px; font-weight: 700; }

  .pos { color: var(--pos); }
  .neg { color: var(--neg); }

  .add-btn {
    padding: 8px 16px; font-size: 13px; font-weight: 700; border-radius: 9px;
    border: 1px solid var(--border); background: var(--bg3); color: var(--txt);
    cursor: pointer; font-family: inherit; display: flex; align-items: center; gap: 6px;
    transition: background .15s;
  }
  .add-btn:hover { background: var(--bg4); }

  .add-box {
    background: var(--bg2); border: 1px solid var(--border); border-radius: 16px;
    padding: 1.5rem; margin-top: 1.5rem;
  }
  .new-cat-box {
    background: var(--bg3); border: 1px dashed var(--border); border-radius: 12px;
    padding: 1rem; margin: 4px 0 14px;
  }
  .cat-chip-list { display: none; }
  .custom-select { position: relative; flex: 1; }
  .custom-select-trigger {
    width: 100%; display: flex; align-items: center; justify-content: space-between; gap: 8px;
    padding: 11px 14px; border-radius: 10px; border: 1px solid var(--border); background: var(--bg3);
    color: var(--txt); font-family: inherit; font-size: 14px; font-weight: 600; cursor: pointer; transition: border-color .15s;
  }
  .custom-select-trigger:hover { border-color: var(--txt3); }
  .custom-select-trigger i { color: var(--txt3); font-size: 13px; }
  .custom-select-menu {
    position: absolute; top: calc(100% + 6px); left: 0; right: 0; z-index: 40;
    background: var(--bg3); border: 1px solid var(--border); border-radius: 10px;
    padding: 6px; max-height: 280px; overflow-y: auto; box-shadow: 0 16px 40px rgba(0,0,0,.55);
  }
  .custom-select-opt {
    display: flex; align-items: center; justify-content: space-between; gap: 8px;
    padding: 9px 10px; border-radius: 8px; font-size: 14px; font-weight: 600; color: var(--txt2);
  }
  .custom-select-opt:hover { background: var(--bg4); }
  .custom-select-opt.sel { color: var(--txt); }
  .custom-select-opt-name { flex: 1; cursor: pointer; }
  .cat-chip .chip-del {
    width: 16px; height: 16px; border-radius: 50%; border: none; background: transparent;
    color: var(--txt3); cursor: pointer; display: flex; align-items: center; justify-content: center;
    font-size: 13px; line-height: 1; font-weight: 700; opacity: .45; transition: all .15s; padding: 0;
  }
  .cat-chip .chip-del:hover { opacity: 1; background: var(--neg); color: #fff; }
  .add-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(150px, 1fr)); gap: 12px; margin-bottom: 14px; }
  .field-label { font-size: 11px; font-weight: 700; letter-spacing: .07em; text-transform: uppercase; color: var(--txt3); margin-bottom: 6px; }

  input, select {
    width: 100%; padding: 9px 12px; border-radius: 8px; border: 1px solid var(--border);
    background: var(--bg3); color: var(--txt); font-family: inherit; font-size: 13px;
    font-weight: 500; transition: border-color .15s;
  }
  input:focus, select:focus { outline: none; border-color: var(--accent); }
  select option { background: var(--bg3); }

  .confirm-btn {
    padding: 10px 22px; font-size: 13px; font-weight: 700; border-radius: 9px;
    border: none; background: var(--pos); color: #000; cursor: pointer;
    font-family: inherit; transition: opacity .15s;
  }
  .confirm-btn:hover { opacity: .85; }

  .chart-box {
    background: var(--bg2); border: 1px solid var(--border); border-radius: 16px;
    padding: 1.5rem;
  }
  .chart-legend { display: flex; gap: 18px; margin-bottom: 1.25rem; flex-wrap: wrap; }
  .leg-item { display: flex; align-items: center; gap: 7px; font-size: 12px; font-weight: 600; color: var(--txt2); }

  .fort-split { display: grid; grid-template-columns: repeat(auto-fit, minmax(150px, 1fr)); gap: 14px; margin-bottom: 1.5rem; }
  .fort-card {
    background: var(--bg2); border: 1px solid var(--border); border-radius: 14px; padding: 1.25rem;
  }
  .fort-card-label { font-size: 11px; font-weight: 700; letter-spacing: .09em; text-transform: uppercase; color: var(--txt3); margin-bottom: 10px; }
  .fort-card-value { font-size: 26px; font-weight: 900; letter-spacing: -.04em; }

  .fort-rows { background: var(--bg2); border: 1px solid var(--border); border-radius: 16px; padding: 1.5rem; margin-bottom: 1.5rem; }
  .fort-row { display: flex; justify-content: space-between; align-items: center; padding: 13px 0; border-bottom: 1px solid var(--border); }
  .fort-row:last-child { border-bottom: none; }
  .fort-row-label { font-size: 14px; font-weight: 500; color: var(--txt2); display: flex; align-items: center; gap: 10px; }
  .fort-row-val { font-size: 16px; font-weight: 800; letter-spacing: -.02em; }
  .dot { display: inline-block; width: 9px; height: 9px; border-radius: 50%; flex-shrink: 0; }

  .fort-total {
    display: flex; justify-content: space-between; align-items: center;
    margin-top: 1.25rem; padding-top: 1.25rem; border-top: 1px solid var(--border);
  }
  .fort-total-label { font-size: 16px; font-weight: 800; color: var(--txt); }
  .fort-total-val { font-size: 32px; font-weight: 900; letter-spacing: -.04em; }

  .tbl-section { margin-bottom: 16px; border: 1px solid var(--border); border-radius: 16px; overflow: hidden; }
  .tbl-cat-hdr {
    display: flex; align-items: center; gap: 12px; padding: 14px 18px;
    background: var(--bg2); cursor: pointer; user-select: none;
    border-bottom: 1px solid var(--border); transition: background .15s;
  }
  .tbl-cat-hdr:hover { background: var(--bg3); }
  .tbl-cat-lbl { font-size: 14px; font-weight: 700; color: var(--txt); flex: 1; }
  .tbl-cat-info { font-size: 12px; color: var(--txt3); font-weight: 500; }
  .tbl-cat-arrow { font-size: 14px; color: var(--txt3); transition: transform .2s; }
  .tbl-cat-arrow.open { transform: rotate(90deg); }
  .clear-cat-btn {
    padding: 4px 10px; font-size: 11px; font-weight: 700; border-radius: 6px;
    border: 1px solid var(--neg); background: transparent; color: var(--neg);
    cursor: pointer; font-family: inherit; white-space: nowrap;
    transition: all .15s; margin-left: 8px; flex-shrink: 0;
  }
  .clear-cat-btn:hover { background: var(--neg); color: #fff; }
  .tbl-body { display: none; background: var(--bg); }
  .tbl-body.open { display: block; }

  .tbl-header {
    display: grid; grid-template-columns: 2fr 1fr 1fr 1.2fr 36px;
    padding: 8px 18px; background: var(--bg2); border-bottom: 1px solid var(--border);
  }
  .tbl-hcell { font-size: 10px; font-weight: 700; letter-spacing: .1em; text-transform: uppercase; color: var(--txt3); }

  .tbl-row {
    display: grid; grid-template-columns: 2fr 1fr 1fr 1.2fr 36px;
    padding: 6px 18px; border-bottom: 1px solid var(--border); align-items: center;
    transition: background .1s;
  }
  .tbl-row:last-child { border-bottom: none; }
  .tbl-row:hover { background: var(--bg2); }

  .cell-inp {
    width: 100%; padding: 5px 8px; border-radius: 6px; border: 1px solid transparent;
    background: transparent; color: var(--txt); font-family: inherit; font-size: 13px;
    font-weight: 600; transition: all .15s;
  }
  .cell-inp:hover { border-color: var(--border); background: var(--bg3); }
  .cell-inp:focus { border-color: var(--accent); background: var(--bg3); outline: none; }

  .cell-sel {
    width: 100%; padding: 5px 6px; border-radius: 6px; border: 1px solid transparent;
    background: transparent; color: var(--txt2); font-family: inherit; font-size: 12px;
    font-weight: 500; transition: all .15s; cursor: pointer;
  }
  .cell-sel:hover { border-color: var(--border); background: var(--bg3); }
  .cell-sel:focus { border-color: var(--accent); background: var(--bg3); outline: none; }
  .cell-sel option { background: #1c1c26; }

  .del-btn {
    background: none; border: none; cursor: pointer; color: var(--neg);
    font-size: 15px; padding: 4px; opacity: .35; transition: opacity .15s;
    display: flex; align-items: center; justify-content: center;
  }
  .del-btn:hover { opacity: 1; }

  #colorPanel {
    position: fixed; top: 0; right: -380px; width: 360px; height: 100vh;
    background: var(--bg2); border-left: 1px solid var(--border);
    z-index: 9999; transition: right .28s ease; overflow-y: auto;
    padding: 1.75rem 1.5rem 3rem; box-shadow: -12px 0 50px rgba(0,0,0,.7);
  }
  #colorPanel.open { right: 0; }
  #overlay { display: none; position: fixed; inset: 0; background: rgba(0,0,0,.55); z-index: 9998; backdrop-filter: blur(3px); }
  #overlay.on { display: block; }

  .cp-hdr { display: flex; justify-content: space-between; align-items: center; margin-bottom: 1.75rem; }
  .cp-title { font-size: 16px; font-weight: 800; color: var(--txt); }
  .cp-close { background: none; border: none; cursor: pointer; color: var(--txt2); font-size: 20px; line-height: 1; }
  .cp-sec { margin-bottom: 1.75rem; }
  .cp-sec-title { font-size: 10px; font-weight: 700; letter-spacing: .1em; text-transform: uppercase; color: var(--txt3); margin-bottom: .85rem; padding-bottom: .5rem; border-bottom: 1px solid var(--border); }
  .cp-row { display: flex; align-items: center; justify-content: space-between; margin-bottom: .8rem; }
  .cp-lbl { font-size: 13px; font-weight: 500; color: var(--txt2); }
  .cp-swatch {
    width: 32px; height: 32px; border-radius: 8px; border: 1px solid var(--border);
    position: relative; overflow: hidden; cursor: pointer; flex-shrink: 0;
  }
  .cp-swatch input[type=color] { position: absolute; inset: -6px; width: calc(100% + 12px); height: calc(100% + 12px); opacity: 0; cursor: pointer; }
  .cp-swatch-preview { position: absolute; inset: 0; border-radius: 7px; pointer-events: none; }
  .cp-reset {
    width: 100%; padding: 10px; font-size: 13px; font-weight: 700; border-radius: 9px;
    border: 1px solid var(--border); background: var(--bg3); color: var(--txt2);
    cursor: pointer; font-family: inherit; transition: background .15s;
  }
  .cp-reset:hover { background: var(--bg4); }

  .fab {
    position: fixed; bottom: 1.75rem; right: 1.75rem; width: 48px; height: 48px;
    border-radius: 50%; border: 1px solid var(--border); background: var(--bg3);
    color: var(--txt); cursor: pointer; display: flex; align-items: center; justify-content: center;
    box-shadow: 0 6px 30px rgba(0,0,0,.6); transition: transform .15s, background .15s;
    font-size: 18px; z-index: 100;
  }
  .fab:hover { transform: scale(1.1); background: var(--bg4); }

  .save-btn {
    padding: 9px 18px; font-size: 13px; font-weight: 700; border-radius: 10px;
    border: 1px solid var(--pos); background: transparent; color: var(--pos);
    cursor: pointer; font-family: inherit; display: flex; align-items: center; gap: 7px;
    transition: all .2s; letter-spacing: .01em;
  }
  .save-btn:hover { background: var(--pos); color: #000; }
  .save-btn.saved { background: var(--pos); color: #000; }

  .toast {
    position: fixed; bottom: 2rem; left: 50%; transform: translateX(-50%) translateY(20px);
    background: var(--pos); color: #000; font-size: 13px; font-weight: 700;
    padding: 10px 22px; border-radius: 30px; opacity: 0; pointer-events: none;
    transition: all .3s; z-index: 9999; white-space: nowrap;
  }
  .toast.show { opacity: 1; transform: translateX(-50%) translateY(0); }
</style>
</head>
<body>

<div id="overlay" onclick="closePanel()"></div>

<div id="colorPanel">
  <div class="cp-hdr">
    <span class="cp-title">🎨 Couleurs</span>
    <button class="cp-close" onclick="closePanel()"><i class="ti ti-x"></i></button>
  </div>
  <div class="cp-sec">
    <div class="cp-sec-title">Positif / Négatif</div>
    <div class="cp-row"><span class="cp-lbl">Gains &amp; positif</span><label class="cp-swatch" id="sw-pos"><div class="cp-swatch-preview"></div><input type="color" oninput="setColor('pos',this.value)"></label></div>
    <div class="cp-row"><span class="cp-lbl">Pertes &amp; négatif</span><label class="cp-swatch" id="sw-neg"><div class="cp-swatch-preview"></div><input type="color" oninput="setColor('neg',this.value)"></label></div>
  </div>
  <div class="cp-sec">
    <div class="cp-sec-title">Courbes PNL</div>
    <div class="cp-row"><span class="cp-lbl">PNL cumulé</span><label class="cp-swatch" id="sw-c0"><div class="cp-swatch-preview"></div><input type="color" oninput="setChart(0,this.value)"></label></div>
    <div class="cp-row"><span class="cp-lbl">Trading</span><label class="cp-swatch" id="sw-c2"><div class="cp-swatch-preview"></div><input type="color" oninput="setChart(2,this.value)"></label></div>
  </div>
  <div class="cp-sec">
    <div class="cp-sec-title">Catégories</div>
    <div id="catSwatches"></div>
  </div>
  <button class="cp-reset" onclick="resetColors()"><i class="ti ti-refresh" style="margin-right:6px"></i>Réinitialiser</button>
</div>

<button class="fab" onclick="openPanel()"><i class="ti ti-palette"></i></button>

<div id="loginScreen" style="display:flex;min-height:100vh;align-items:center;justify-content:center;flex-direction:column;gap:20px;padding:20px;">
  <div class="logo" style="font-size:32px">cash<span>flow</span></div>

  <div id="emailStep" style="display:flex;flex-direction:column;gap:10px;width:100%;max-width:320px">
    <p style="color:var(--txt3);font-size:14px;text-align:center;margin:0 0 4px">Entre ton adresse email pour continuer.</p>
    <input id="loginEmail" type="email" placeholder="ton@email.com" autocomplete="email"
      style="background:var(--bg2);border:1px solid var(--border);border-radius:10px;padding:12px 14px;color:var(--txt);font-size:14px;font-family:inherit;width:100%">
    <button class="save-btn" onclick="goToCodeStep()" style="justify-content:center;padding:12px;font-size:14px">
      Continuer
    </button>
    <p id="emailErr" style="display:none;color:var(--neg);font-size:12px;text-align:center;margin:0"></p>
  </div>

  <div id="codeStep" style="display:none;flex-direction:column;gap:10px;width:100%;max-width:320px">
    <p style="color:var(--txt3);font-size:14px;text-align:center;margin:0 0 4px">Entre ton code de connexion.</p>
    <input id="loginCode" type="password" placeholder="Ton code" autocomplete="current-password"
      style="background:var(--bg2);border:1px solid var(--border);border-radius:10px;padding:12px 14px;color:var(--txt);font-size:14px;font-family:inherit;width:100%">
    <button class="save-btn" onclick="login()" style="justify-content:center;padding:12px;font-size:14px">
      <i class="ti ti-check"></i>&nbsp;Se connecter
    </button>
    <button onclick="backToEmailStep()" style="background:none;border:none;color:var(--txt3);font-size:12px;cursor:pointer;font-family:inherit">← Changer d'adresse email</button>
    <p style="color:var(--txt3);font-size:11px;text-align:center;margin:0">La première fois, ce code crée ton compte. Note-le bien — il te sera redemandé à chaque connexion.</p>
    <p id="loginErr" style="display:none;color:var(--neg);font-size:12px;text-align:center;margin:0"></p>
  </div>
</div>

<div class="app" id="appRoot" style="display:none">


  <div class="top-bar">
    <div class="logo">cash<span>flow</span></div>
    <div style="display:flex;align-items:center;gap:12px">
      <div class="yr-switch" id="yrSwitch"></div>
      <button class="save-btn" id="saveBtn" onclick="signOut()"><i class="ti ti-logout"></i>Déconnexion</button>
    </div>
  </div>
  <div class="toast" id="toast">✓ Enregistré</div>

  <nav class="nav">
    <button class="nb on" onclick="goTab(0,this)"><i class="ti ti-layout-cards"></i>Catégories</button>
    <button class="nb"    onclick="goTab(1,this)"><i class="ti ti-chart-line"></i>PNL</button>
    <button class="nb"    onclick="goTab(2,this)"><i class="ti ti-safe"></i>Fortune</button>
    <button class="nb"    onclick="goTab(3,this)"><i class="ti ti-table-options"></i>Données</button>
    <button class="nb"    onclick="goTab(4,this)"><i class="ti ti-plus"></i>Ajouter</button>
  </nav>

  <div class="pg on" id="tab0">
    <div class="sec-hdr">
      <div><div class="sec-title">Par catégorie</div><div class="sec-sub">Totaux & dernières transactions</div></div>
    </div>
    <div class="cards-grid" id="kpi0" style="margin-bottom:1.5rem"></div>
  </div>

  <div class="pg" id="tab1">
    <div class="sec-hdr">
      <div><div class="sec-title">Courbe PNL</div><div class="sec-sub">Profit &amp; loss mensuel et cumulé</div></div>
    </div>
    <div class="kpi-row" id="kpi1"></div>
    <div class="chart-box" style="margin-bottom:1.5rem">
      <div class="chart-legend" id="pnlLegend"></div>
      <div style="position:relative;height:320px">
        <canvas id="pnlCanvas"></canvas>
      </div>
    </div>

    <div class="chart-box" style="margin-bottom:1.5rem">
      <div style="font-size:12px;font-weight:700;letter-spacing:.07em;text-transform:uppercase;color:var(--txt3);margin-bottom:1rem">Évolution de la fortune</div>
      <div style="position:relative;height:280px">
        <canvas id="fortAreaCanvas"></canvas>
      </div>
    </div>

    <div class="chart-box">
      <div style="font-size:12px;font-weight:700;letter-spacing:.07em;text-transform:uppercase;color:var(--txt3);margin-bottom:1rem">Cumul mois par mois</div>
      <div style="position:relative;height:220px">
        <canvas id="fortCanvas"></canvas>
      </div>
    </div>
  </div>

  <div class="pg" id="tab2">
    <div class="sec-hdr">
      <div><div class="sec-title">Fortune nette</div></div>
    </div>
    <div class="fort-split" id="fortSplit"></div>

    <div class="chart-box" style="margin-bottom:1.5rem">
      <div style="font-size:12px;font-weight:700;letter-spacing:.07em;text-transform:uppercase;color:var(--txt3);margin-bottom:1rem">Répartition des revenus</div>
      <div class="pie-wrap">
        <div style="position:relative;height:240px;width:240px;flex-shrink:0">
          <canvas id="pieCanvas"></canvas>
        </div>
        <div class="pie-legend" id="pieLegend"></div>
      </div>
    </div>

    <div class="fort-rows" id="fortRows"></div>
  </div>

  <div class="pg" id="tab3">
    <div class="sec-hdr">
      <div><div class="sec-title">Toutes les données</div><div class="sec-sub">Clique sur un champ pour le modifier directement</div></div>
    </div>
    <div id="dataTable"></div>
  </div>

  <div class="pg" id="tab4">
    <div class="sec-hdr">
      <div><div class="sec-title">Ajouter une transaction</div><div class="sec-sub">Enregistre un revenu, un trade, une dépense ou tout autre mouvement</div></div>
    </div>
    <div class="add-box">
      <div class="add-grid">
        <div>
          <div class="field-label">Catégorie</div>
          <div style="display:flex;gap:8px">
            <div class="custom-select" id="aCatWrap">
              <button type="button" class="custom-select-trigger" onclick="toggleCatDropdown()">
                <span id="aCatLabel">Revenus</span>
                <i class="ti ti-chevron-down"></i>
              </button>
              <div class="custom-select-menu" id="aCatMenu" style="display:none"></div>
            </div>
            <input type="hidden" id="aCat" value="revenus">
            <button type="button" class="yr-edit-btn" onclick="toggleNewCat()" title="Ajouter une nouvelle catégorie"><i class="ti ti-plus"></i></button>
          </div>
        </div>
        <div>
          <div class="field-label">Description</div>
          <input id="aDesc" type="text" placeholder="ex : Salaire">
        </div>
        <div>
          <div class="field-label">Mois</div>
          <select id="aMon">
            <option value="0">Janvier</option><option value="1">Février</option><option value="2">Mars</option>
            <option value="3">Avril</option><option value="4">Mai</option><option value="5">Juin</option>
            <option value="6">Juillet</option><option value="7">Août</option><option value="8">Septembre</option>
            <option value="9">Octobre</option><option value="10">Novembre</option><option value="11">Décembre</option>
          </select>
        </div>
        <div>
          <div class="field-label">Montant (€)</div>
          <input id="aAmt" type="number" placeholder="0">
        </div>
      </div>

      <div class="new-cat-box" id="newCatBox" style="display:none">
        <div class="add-grid" style="margin-bottom:12px">
          <div>
            <div class="field-label">Nom de la nouvelle catégorie</div>
            <input id="ncName" type="text" placeholder="ex : Dividendes">
          </div>
        </div>
        <button class="confirm-btn" type="button" onclick="createCategory()">Créer la catégorie</button>
      </div>

      <button class="confirm-btn" onclick="doAdd()">Confirmer</button>
    </div>
  </div>

</div>

<script>
const MO = ['Jan','Fév','Mar','Avr','Mai','Jun','Jul','Aoû','Sep','Oct','Nov','Déc'];
const MO_F = ['Janvier','Février','Mars','Avril','Mai','Juin','Juillet','Août','Septembre','Octobre','Novembre','Décembre'];

const CATS = {
  revenus:        { label:'Revenus',        icon:'ti-trending-up',   bg:'#0d2e22', ic:'#22d3a0', bar:'#22d3a0', sign: 1 },
  trading:        { label:'Trading',        icon:'ti-chart-candle',  bg:'#0d1e35', ic:'#3b9eff', bar:'#3b9eff', sign: 1 },
  investissement: { label:'Investissement', icon:'ti-building-bank', bg:'#2a1c09', ic:'#f59e0b', bar:'#f59e0b', sign: 1 },
  epargne:        { label:'Épargne',        icon:'ti-piggy-bank',    bg:'#1a1535', ic:'#a78bfa', bar:'#a78bfa', sign: 1 },
  depot:          { label:'Dépôt compte',   icon:'ti-wallet',        bg:'#1a1a24', ic:'#888780', bar:'#888780', sign: 1 },
  argent:         { label:'Argent liquide', icon:'ti-cash',          bg:'#0d2e22', ic:'#2dd4bf', bar:'#2dd4bf', sign: 1 },
  depenses:       { label:'Dépenses',       icon:'ti-receipt',       bg:'#2d0f0f', ic:'#f43f5e', bar:'#f43f5e', sign:-1 },
};

const DB = {};

const DFLTS = {
  pos:'#22d3a0', neg:'#f43f5e',
  charts:['#22d3a0','#888780','#3b9eff'],
  cats:{
    revenus:{bg:'#0d2e22',ic:'#22d3a0',bar:'#22d3a0'},
    trading:{bg:'#0d1e35',ic:'#3b9eff',bar:'#3b9eff'},
    investissement:{bg:'#2a1c09',ic:'#f59e0b',bar:'#f59e0b'},
    epargne:{bg:'#1a1535',ic:'#a78bfa',bar:'#a78bfa'},
    depot:{bg:'#1a1a24',ic:'#888780',bar:'#888780'},
    argent:{bg:'#0d2e22',ic:'#2dd4bf',bar:'#2dd4bf'},
    depenses:{bg:'#2d0f0f',ic:'#f43f5e',bar:'#f43f5e'},
  }
};
let CLR = JSON.parse(JSON.stringify(DFLTS));

function ha(hex, a) {
  const r = parseInt(hex.slice(1,3),16), g = parseInt(hex.slice(3,5),16), b = parseInt(hex.slice(5,7),16);
  return 'rgba('+r+','+g+','+b+','+a+')';
}
function applyCSSColors() {
  document.documentElement.style.setProperty('--pos', CLR.pos);
  document.documentElement.style.setProperty('--neg', CLR.neg);
}
function setColor(k, v) { CLR[k] = v; applyCSSColors(); syncSwatches(); renderAll(); }
function setChart(i, v) { CLR.charts[i] = v; syncSwatches(); renderAll(); }
function setCatColor(cat, prop, v) { CLR.cats[cat][prop] = v; CATS[cat][prop] = v; syncSwatches(); renderAll(); }
function resetColors() {
  const customCats = {};
  Object.keys(CATS).forEach(k => { if (!DFLTS.cats[k]) customCats[k] = JSON.parse(JSON.stringify(CLR.cats[k])); });
  CLR = JSON.parse(JSON.stringify(DFLTS));
  Object.assign(CLR.cats, customCats);
  Object.keys(CATS).forEach(k => {
    if (CLR.cats[k]) { CATS[k].bg = CLR.cats[k].bg; CATS[k].ic = CLR.cats[k].ic; CATS[k].bar = CLR.cats[k].bar; }
  });
  applyCSSColors(); buildCatSwatches(); syncSwatches(); renderAll();
}
function setSwatch(id, val) {
  const el = document.getElementById(id);
  if (!el) return;
  el.querySelector('.cp-swatch-preview').style.background = val;
  el.querySelector('input').value = val;
}
function syncSwatches() {
  setSwatch('sw-pos', CLR.pos);
  setSwatch('sw-neg', CLR.neg);
  CLR.charts.forEach((c,i) => setSwatch('sw-c'+i, c));
  Object.keys(CLR.cats).forEach(cat => {
    ['bg','ic','bar'].forEach(p => setSwatch('sw-'+cat+'-'+p, CLR.cats[cat][p]));
  });
}
function buildCatSwatches() {
  let html = '';
  Object.keys(CATS).forEach(cat => {
    const c = CATS[cat];
    html += '<div style="margin-bottom:1rem">'
      + '<div style="font-size:12px;font-weight:700;color:var(--txt2);margin-bottom:.5rem;display:flex;align-items:center;gap:8px">'
      + '<i class="ti '+c.icon+'" style="color:'+c.ic+'"></i>'+c.label+'</div>'
      + '<div style="display:flex;gap:10px">'
      + '<div><div class="cp-lbl" style="font-size:10px;text-align:center;margin-bottom:4px">Fond</div>'
      + '<label class="cp-swatch" id="sw-'+cat+'-bg"><div class="cp-swatch-preview" style="background:'+c.bg+'"></div><input type="color" value="'+c.bg+'" oninput="setCatColor(\''+cat+'\',\'bg\',this.value)"></label></div>'
      + '<div><div class="cp-lbl" style="font-size:10px;text-align:center;margin-bottom:4px">Icône</div>'
      + '<label class="cp-swatch" id="sw-'+cat+'-ic"><div class="cp-swatch-preview" style="background:'+c.ic+'"></div><input type="color" value="'+c.ic+'" oninput="setCatColor(\''+cat+'\',\'ic\',this.value)"></label></div>'
      + '<div><div class="cp-lbl" style="font-size:10px;text-align:center;margin-bottom:4px">Barre</div>'
      + '<label class="cp-swatch" id="sw-'+cat+'-bar"><div class="cp-swatch-preview" style="background:'+c.bar+'"></div><input type="color" value="'+c.bar+'" oninput="setCatColor(\''+cat+'\',\'bar\',this.value)"></label></div>'
      + '</div></div>';
  });
  document.getElementById('catSwatches').innerHTML = html;
}
function openPanel()  { document.getElementById('colorPanel').classList.add('open'); document.getElementById('overlay').classList.add('on'); }
function closePanel() { document.getElementById('colorPanel').classList.remove('open'); document.getElementById('overlay').classList.remove('on'); }

let yr = 2026, pChart = null, fChart = null, fAreaChart = null, pieChart = null;
const fmt = n => new Intl.NumberFormat('fr-FR',{style:'currency',currency:'EUR',maximumFractionDigits:0}).format(n);
const fmtS = n => (n >= 0 ? '+' : '') + fmt(n);
const rows = () => DB[yr] || [];

const months = () => {
  const now = new Date();
  const currentYear = now.getFullYear();
  if (yr === currentYear) return now.getMonth() + 1;
  if (yr > currentYear) return 1;
  return 12;
};

const msum = (cat, m) => rows().filter(e => e.cat===cat && e.month===m).reduce((s,e) => s+e.amt, 0);
const catTotal = cat => rows().filter(e => e.cat===cat).reduce((s,e) => s+e.amt, 0);

const currentMonthIdx = () => months() - 1;

function calcKPI() {
  let rev=0, exp=0, trd=0;
  Object.keys(CATS).forEach(k => {
    const t = catTotal(k);
    if (k==='depenses') exp += t;
    else if (k==='trading') trd += t;
    else rev += t;
  });
  return { rev, exp, trd, net: rev+trd-exp };
}

function calcMonthKPI(m) {
  let rev=0, exp=0, trd=0;
  Object.keys(CATS).forEach(k => {
    const t = msum(k, m);
    if (k==='depenses') exp += t;
    else if (k==='trading') trd += t;
    else rev += t;
  });
  return { rev, exp, trd, net: rev+trd-exp };
}

function renderKPI(id, items) {
  const el = document.getElementById(id);
  if (!el) return;
  el.innerHTML = items.map(({l,v,c}) =>
    '<div class="kpi" style="--kpi-color:'+c+'">'
    +'<div class="kpi-label">'+l+'</div>'
    +'<div class="kpi-value" style="color:'+c+'">'+v+'</div></div>'
  ).join('');
}

function renderAll() {
  const {rev,exp,trd,net} = calcKPI();
  const monthKPI = calcMonthKPI(currentMonthIdx());
  const budget = Math.max(monthKPI.net, 0) * 0.10;
  const kpisPNL = [
    {l:'Gains totaux',  v:fmt(rev+trd), c:CLR.pos},
    {l:'Trading PNL',   v:fmt(trd),     c:CLR.charts[2]},
    {l:'Dépenses',      v:fmt(exp),     c:CLR.neg},
    {l:'Budget mensuel (10%)', v:fmt(budget), c:CLR.pos},
  ];
  renderKPI('kpi1', kpisPNL);
  renderTopBoxes();
  renderDataTable();
  renderPNL();
  renderFortune();
}

function catBoxHTML(label, total, colorHex, entries) {
  const sign = total < 0 ? '-' : '+';
  const txHtml = entries.length
    ? entries.map(e => {
        const amtColor = e.cat === 'trading' ? CLR.charts[2]
          : (CATS[e.cat] && CATS[e.cat].sign < 0) ? CLR.neg
          : CLR.pos;
        const showNeg = (CATS[e.cat] && CATS[e.cat].sign < 0) || e.amt < 0;
        return '<div class="tx">'
          +'<div><div class="tx-desc">'+escHtml(e.desc)+'</div><div class="tx-mo">'+MO[e.month]+'</div></div>'
          +'<div class="tx-amt" style="color:'+amtColor+'">'+(showNeg?'-':'+')+fmt(Math.abs(e.amt))+'</div>'
          +'</div>';
      }).join('')
    : '<div class="tx-empty">Aucune transaction</div>';
  return '<div class="cat-card">'
    +'<div class="cat-box-head">'
    +'<div class="cat-box-head-left"><div class="cat-name">'+label+'</div>'
    +'<button type="button" class="box-expand-btn" onclick="toggleBoxExpand(this)" title="Voir toutes les transactions"><i class="ti ti-chevron-down"></i></button></div>'
    +'<div class="cat-amount" style="color:'+colorHex+'">'+sign+fmt(Math.abs(total))+'</div>'
    +'</div>'
    +'<div class="tx-list">'+txHtml+'</div>'
    +'</div>';
}

function toggleBoxExpand(btn) {
  const card = btn.closest('.cat-card');
  if (!card) return;
  const list = card.querySelector('.tx-list');
  if (!list) return;
  const expanded = list.classList.toggle('expanded');
  const icon = btn.querySelector('i');
  if (icon) icon.className = expanded ? 'ti ti-chevron-up' : 'ti ti-chevron-down';
  btn.title = expanded ? 'Réduire' : 'Voir toutes les transactions';
}

function renderTopBoxes() {
  const el = document.getElementById('kpi0');
  if (!el) return;
  const {rev,exp,trd} = calcKPI();
  const gainEntries  = rows().filter(e => CATS[e.cat] && CATS[e.cat].sign > 0).slice().reverse();
  const expEntries   = rows().filter(e => e.cat === 'depenses').slice().reverse();
  const tradeEntries = rows().filter(e => e.cat === 'trading').slice().reverse();
  el.innerHTML =
      catBoxHTML('Gains totaux', rev+trd, CLR.pos, gainEntries)
    + catBoxHTML('Dépenses', -exp, CLR.neg, expEntries)
    + catBoxHTML('Trading PNL', trd, CLR.charts[2], tradeEntries);
}

function renderPNL() {
  const allEntries = rows();
  const maxMonth = allEntries.length > 0 ? Math.max(...allEntries.map(e => e.month)) + 1 : months();
  const m = Math.max(months(), maxMonth);
  const labels = MO.slice(0, m);
  let cum = 0;
  const cumD = Array.from({length:m}, (_,i) => {
    let n = 0; Object.keys(CATS).forEach(k => { n += CATS[k].sign * msum(k,i); }); cum += n; return Math.round(cum);
  });
  let cumTrd = 0;
  const trdD = Array.from({length:m}, (_,i) => { cumTrd += msum('trading',i); return Math.round(cumTrd); });
  let cumExp = 0;
  const expD = Array.from({length:m}, (_,i) => { cumExp += msum('depenses',i); return Math.round(cumExp); });
  const gc = 'rgba(255,255,255,.05)', tc = 'rgba(255,255,255,.3)';
  if (pChart) { pChart.destroy(); pChart = null; }
  pChart = new Chart(document.getElementById('pnlCanvas'), {
    type:'line',
    data:{ labels, datasets:[
      {label:'PNL cumulé', data:cumD, borderColor:CLR.charts[0], backgroundColor:ha(CLR.charts[0],.1), fill:true, tension:.4, pointRadius:4, pointBackgroundColor:CLR.charts[0], borderWidth:2.5},
      {label:'Trading', data:trdD, borderColor:CLR.charts[2], fill:false, tension:.4, pointRadius:4, pointBackgroundColor:CLR.charts[2], borderWidth:2, pointStyle:'triangle'},
      {label:'Dépenses', data:expD, borderColor:CLR.neg, fill:false, tension:.4, pointRadius:4, pointBackgroundColor:CLR.neg, borderWidth:2, pointStyle:'rectRot'},
    ]},
    options:{
      responsive:true, maintainAspectRatio:false,
      interaction:{mode:'index',intersect:false},
      plugins:{
        legend:{display:false},
