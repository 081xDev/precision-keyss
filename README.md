<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Precision Fix — Key Manager</title>
<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@300;400;500;700&family=Syne:wght@400;700;800&display=swap" rel="stylesheet">
<style>
  :root {
    --bg:      #04040a;
    --panel:   #09090f;
    --panel2:  #0e0e18;
    --border:  #1e1e30;
    --purple:  #a020f0;
    --purplel: #c060ff;
    --purpled: #6010a0;
    --text:    #e0e0f0;
    --muted:   #5a5a7a;
    --green:   #00e676;
    --red:     #ff3d5a;
    --yellow:  #ffd600;
    --cyan:    #00d4ff;
  }

  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: 'JetBrains Mono', monospace;
    min-height: 100vh;
    overflow-x: hidden;
  }

  /* Animated background grid */
  body::before {
    content: '';
    position: fixed;
    inset: 0;
    background-image:
      linear-gradient(rgba(160,32,240,0.03) 1px, transparent 1px),
      linear-gradient(90deg, rgba(160,32,240,0.03) 1px, transparent 1px);
    background-size: 40px 40px;
    pointer-events: none;
    z-index: 0;
  }

  /* ── LOGIN ── */
  #login-screen {
    position: fixed;
    inset: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 100;
    background: var(--bg);
  }

  .login-box {
    width: 420px;
    background: var(--panel);
    border: 1px solid var(--border);
    padding: 48px 40px 40px;
    position: relative;
    animation: fadeUp .4s ease;
  }

  .login-box::before {
    content: '';
    position: absolute;
    top: 0; left: 0; right: 0;
    height: 2px;
    background: linear-gradient(90deg, transparent, var(--purple), transparent);
  }

  .login-logo {
    font-family: 'Syne', sans-serif;
    font-weight: 800;
    font-size: 28px;
    color: var(--purplel);
    letter-spacing: -1px;
    margin-bottom: 4px;
  }

  .login-sub {
    font-size: 11px;
    color: var(--muted);
    margin-bottom: 36px;
    letter-spacing: 2px;
  }

  .field-label {
    font-size: 10px;
    letter-spacing: 2px;
    color: var(--muted);
    margin-bottom: 8px;
  }

  .field-input {
    width: 100%;
    background: var(--bg);
    border: 1px solid var(--border);
    color: var(--text);
    font-family: 'JetBrains Mono', monospace;
    font-size: 14px;
    padding: 12px 16px;
    outline: none;
    transition: border-color .2s;
    margin-bottom: 20px;
  }

  .field-input:focus { border-color: var(--purple); }

  .btn {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    padding: 12px 24px;
    font-family: 'JetBrains Mono', monospace;
    font-size: 13px;
    font-weight: 700;
    border: none;
    cursor: pointer;
    transition: all .2s;
    letter-spacing: 1px;
  }

  .btn-primary {
    background: var(--purple);
    color: #fff;
    width: 100%;
    justify-content: center;
    padding: 14px;
  }
  .btn-primary:hover { background: var(--purplel); }

  .btn-green  { background: rgba(0,230,118,.1); color: var(--green);  border: 1px solid rgba(0,230,118,.2); }
  .btn-red    { background: rgba(255,61,90,.1);  color: var(--red);   border: 1px solid rgba(255,61,90,.2); }
  .btn-cyan   { background: rgba(0,212,255,.1);  color: var(--cyan);  border: 1px solid rgba(0,212,255,.2); }
  .btn-yellow { background: rgba(255,214,0,.1);  color: var(--yellow);border: 1px solid rgba(255,214,0,.2); }
  .btn-green:hover  { background: rgba(0,230,118,.2); }
  .btn-red:hover    { background: rgba(255,61,90,.2); }
  .btn-cyan:hover   { background: rgba(0,212,255,.2); }
  .btn-yellow:hover { background: rgba(255,214,0,.2); }

  .login-err {
    color: var(--red);
    font-size: 11px;
    text-align: center;
    margin-top: -12px;
    margin-bottom: 16px;
    min-height: 16px;
  }

  /* ── APP ── */
  #app { display: none; position: relative; z-index: 1; }

  header {
    background: var(--panel);
    border-bottom: 1px solid var(--border);
    padding: 0 32px;
    height: 56px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    position: sticky;
    top: 0;
    z-index: 10;
  }

  .header-brand {
    font-family: 'Syne', sans-serif;
    font-weight: 800;
    font-size: 18px;
    color: var(--purplel);
    letter-spacing: -0.5px;
  }

  .header-brand span { color: var(--muted); font-weight: 400; font-size: 13px; margin-left: 10px; }

  .header-right { display: flex; align-items: center; gap: 16px; }

  .badge {
    font-size: 10px;
    letter-spacing: 1.5px;
    padding: 4px 10px;
    border: 1px solid;
  }
  .badge-green  { color: var(--green);  border-color: rgba(0,230,118,.3); }
  .badge-purple { color: var(--purplel); border-color: rgba(160,32,240,.3); }

  .logout-btn {
    background: none;
    border: 1px solid var(--border);
    color: var(--muted);
    font-family: 'JetBrains Mono', monospace;
    font-size: 11px;
    padding: 6px 14px;
    cursor: pointer;
    letter-spacing: 1px;
    transition: all .2s;
  }
  .logout-btn:hover { border-color: var(--red); color: var(--red); }

  .main { padding: 32px; max-width: 1400px; margin: 0 auto; }

  /* Stats */
  .stats-row {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 16px;
    margin-bottom: 32px;
  }

  .stat-card {
    background: var(--panel);
    border: 1px solid var(--border);
    padding: 20px 24px;
    position: relative;
    overflow: hidden;
  }

  .stat-card::after {
    content: '';
    position: absolute;
    bottom: 0; left: 0; right: 0;
    height: 1px;
  }
  .stat-card.s-purple::after { background: var(--purple); }
  .stat-card.s-green::after  { background: var(--green); }
  .stat-card.s-red::after    { background: var(--red); }
  .stat-card.s-yellow::after { background: var(--yellow); }

  .stat-label { font-size: 10px; letter-spacing: 2px; color: var(--muted); margin-bottom: 8px; }
  .stat-value { font-family: 'Syne', sans-serif; font-size: 32px; font-weight: 800; }
  .stat-value.c-purple { color: var(--purplel); }
  .stat-value.c-green  { color: var(--green); }
  .stat-value.c-red    { color: var(--red); }
  .stat-value.c-yellow { color: var(--yellow); }

  /* Generator */
  .section {
    background: var(--panel);
    border: 1px solid var(--border);
    margin-bottom: 24px;
  }

  .section-header {
    padding: 16px 24px;
    border-bottom: 1px solid var(--border);
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  .section-title {
    font-size: 11px;
    letter-spacing: 2px;
    color: var(--purplel);
    font-weight: 700;
  }

  .section-body { padding: 24px; }

  .gen-form {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr auto auto;
    gap: 12px;
    align-items: end;
  }

  .form-group { display: flex; flex-direction: column; gap: 6px; }

  select.field-input {
    margin-bottom: 0;
    cursor: pointer;
    appearance: none;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='8' viewBox='0 0 12 8'%3E%3Cpath d='M1 1l5 5 5-5' stroke='%235a5a7a' stroke-width='1.5' fill='none'/%3E%3C/svg%3E");
    background-repeat: no-repeat;
    background-position: right 14px center;
    padding-right: 36px;
  }

  input.field-input { margin-bottom: 0; }

  /* Table */
  .table-wrap { overflow-x: auto; }

  table {
    width: 100%;
    border-collapse: collapse;
    font-size: 12px;
  }

  thead tr {
    border-bottom: 1px solid var(--border);
  }

  th {
    text-align: left;
    font-size: 10px;
    letter-spacing: 2px;
    color: var(--muted);
    padding: 10px 16px;
    font-weight: 500;
  }

  tbody tr {
    border-bottom: 1px solid rgba(30,30,48,.5);
    transition: background .15s;
  }
  tbody tr:hover { background: var(--panel2); }

  td {
    padding: 14px 16px;
    vertical-align: middle;
  }

  .key-cell {
    font-family: 'JetBrains Mono', monospace;
    font-size: 12px;
    color: var(--cyan);
    letter-spacing: 1px;
    cursor: pointer;
    transition: color .2s;
  }
  .key-cell:hover { color: #fff; }

  .status-pill {
    display: inline-flex;
    align-items: center;
    gap: 6px;
    font-size: 10px;
    letter-spacing: 1px;
    padding: 3px 10px;
    border: 1px solid;
  }

  .pill-active   { color: var(--green);  border-color: rgba(0,230,118,.25);  background: rgba(0,230,118,.07); }
  .pill-expired  { color: var(--red);    border-color: rgba(255,61,90,.25);  background: rgba(255,61,90,.07); }
  .pill-revoked  { color: var(--muted);  border-color: rgba(90,90,122,.25);  background: rgba(90,90,122,.07); }

  .dot { width: 6px; height: 6px; border-radius: 50%; display: inline-block; }
  .dot-green  { background: var(--green); }
  .dot-red    { background: var(--red); }
  .dot-muted  { background: var(--muted); }

  .actions { display: flex; gap: 6px; }

  .icon-btn {
    background: none;
    border: 1px solid var(--border);
    color: var(--muted);
    font-size: 12px;
    padding: 5px 10px;
    cursor: pointer;
    transition: all .2s;
    font-family: 'JetBrains Mono', monospace;
  }
  .icon-btn:hover { color: var(--text); border-color: var(--purplel); }
  .icon-btn.danger:hover { color: var(--red); border-color: var(--red); }

  /* Filters */
  .filters {
    display: flex;
    gap: 12px;
    align-items: center;
    flex-wrap: wrap;
  }

  .filter-btn {
    font-family: 'JetBrains Mono', monospace;
    font-size: 10px;
    letter-spacing: 1px;
    padding: 6px 14px;
    border: 1px solid var(--border);
    background: none;
    color: var(--muted);
    cursor: pointer;
    transition: all .2s;
  }
  .filter-btn.active, .filter-btn:hover {
    border-color: var(--purple);
    color: var(--purplel);
    background: rgba(160,32,240,.08);
  }

  .search-input {
    background: var(--bg);
    border: 1px solid var(--border);
    color: var(--text);
    font-family: 'JetBrains Mono', monospace;
    font-size: 12px;
    padding: 7px 14px;
    outline: none;
    width: 260px;
    transition: border-color .2s;
  }
  .search-input:focus { border-color: var(--purple); }

  /* Toast */
  #toast {
    position: fixed;
    bottom: 32px;
    right: 32px;
    z-index: 999;
    display: flex;
    flex-direction: column;
    gap: 8px;
    pointer-events: none;
  }

  .toast-item {
    background: var(--panel2);
    border: 1px solid var(--border);
    border-left: 3px solid var(--purple);
    padding: 12px 20px;
    font-size: 12px;
    animation: slideIn .3s ease, fadeOut .3s ease 2.7s forwards;
    pointer-events: none;
    max-width: 340px;
  }
  .toast-item.success { border-left-color: var(--green); }
  .toast-item.error   { border-left-color: var(--red); }
  .toast-item.warning { border-left-color: var(--yellow); }

  /* Expiry bar */
  .exp-bar-wrap { display: flex; align-items: center; gap: 8px; }
  .exp-bar { width: 80px; height: 3px; background: var(--border); position: relative; }
  .exp-bar-fill { height: 100%; transition: width .3s; }

  /* Empty */
  .empty-state {
    text-align: center;
    padding: 60px 0;
    color: var(--muted);
  }
  .empty-state .icon { font-size: 48px; margin-bottom: 12px; opacity: .3; }
  .empty-state p { font-size: 12px; letter-spacing: 1px; }

  /* Modal */
  .modal-overlay {
    position: fixed;
    inset: 0;
    background: rgba(4,4,10,.85);
    z-index: 200;
    display: flex;
    align-items: center;
    justify-content: center;
    animation: fadeIn .2s ease;
  }
  .modal {
    background: var(--panel);
    border: 1px solid var(--border);
    padding: 36px;
    width: 480px;
    position: relative;
    animation: fadeUp .25s ease;
  }
  .modal::before {
    content: '';
    position: absolute;
    top: 0; left: 0; right: 0;
    height: 2px;
    background: var(--purple);
  }
  .modal-title {
    font-family: 'Syne', sans-serif;
    font-size: 18px;
    font-weight: 800;
    margin-bottom: 24px;
    color: var(--text);
  }
  .modal-key {
    background: var(--bg);
    border: 1px solid var(--border);
    padding: 16px;
    font-size: 16px;
    letter-spacing: 3px;
    color: var(--cyan);
    text-align: center;
    margin-bottom: 16px;
    cursor: pointer;
  }
  .modal-meta { color: var(--muted); font-size: 11px; line-height: 1.8; margin-bottom: 24px; }
  .modal-meta b { color: var(--text); }
  .modal-actions { display: flex; gap: 10px; justify-content: flex-end; }

  @keyframes fadeUp   { from { opacity:0; transform:translateY(16px); } to { opacity:1; transform:none; } }
  @keyframes fadeIn   { from { opacity:0; } to { opacity:1; } }
  @keyframes slideIn  { from { opacity:0; transform:translateX(20px); } to { opacity:1; transform:none; } }
  @keyframes fadeOut  { from { opacity:1; } to { opacity:0; } }

  /* Scrollbar */
  ::-webkit-scrollbar { width: 6px; height: 6px; }
  ::-webkit-scrollbar-track { background: var(--bg); }
  ::-webkit-scrollbar-thumb { background: var(--border); }
  ::-webkit-scrollbar-thumb:hover { background: var(--purpled); }

  .note {
    background: rgba(255,214,0,.06);
    border: 1px solid rgba(255,214,0,.15);
    padding: 12px 16px;
    font-size: 11px;
    color: var(--yellow);
    margin-bottom: 24px;
    letter-spacing: .5px;
    line-height: 1.6;
  }

  @media(max-width:900px) {
    .stats-row { grid-template-columns: repeat(2,1fr); }
    .gen-form { grid-template-columns: 1fr 1fr; }
  }
</style>
</head>
<body>

<!-- ══ LOGIN ══ -->
<div id="login-screen">
  <div class="login-box">
    <div class="login-logo">PRECISION FIX</div>
    <div class="login-sub">KEY MANAGER — ADMIN ACCESS</div>
    <div class="field-label">SENHA DE ADMINISTRADOR</div>
    <input type="password" class="field-input" id="pw-input" placeholder="••••••••••••"
           onkeydown="if(event.key==='Enter')doLogin()">
    <div class="login-err" id="login-err"></div>
    <button class="btn btn-primary" onclick="doLogin()">▶ ENTRAR</button>
  </div>
</div>

<!-- ══ APP ══ -->
<div id="app">
  <header>
    <div>
      <span class="header-brand">PRECISION FIX <span>KEY MANAGER</span></span>
    </div>
    <div class="header-right">
      <span class="badge badge-green">● ADMIN CONECTADO</span>
      <span class="badge badge-purple" id="hdr-date"></span>
      <button class="logout-btn" onclick="doLogout()">SAIR</button>
    </div>
  </header>

  <div class="main">

    <div class="note">
      ⚠ IMPORTANTE — As keys são armazenadas no seu navegador (IndexedDB). Para uso em produção, migre para um servidor com banco de dados real (SQLite / Postgres). Exporte regularmente com o botão "EXPORTAR JSON".
    </div>

    <!-- Stats -->
    <div class="stats-row">
      <div class="stat-card s-purple">
        <div class="stat-label">TOTAL DE KEYS</div>
        <div class="stat-value c-purple" id="st-total">0</div>
      </div>
      <div class="stat-card s-green">
        <div class="stat-label">ATIVAS</div>
        <div class="stat-value c-green" id="st-active">0</div>
      </div>
      <div class="stat-card s-red">
        <div class="stat-label">EXPIRADAS</div>
        <div class="stat-value c-red" id="st-expired">0</div>
      </div>
      <div class="stat-card s-yellow">
        <div class="stat-label">REVOGADAS</div>
        <div class="stat-value c-yellow" id="st-revoked">0</div>
      </div>
    </div>

    <!-- Generator -->
    <div class="section">
      <div class="section-header">
        <span class="section-title">GERAR NOVA KEY</span>
      </div>
      <div class="section-body">
        <div class="gen-form">
          <div class="form-group">
            <div class="field-label">LABEL / CLIENTE</div>
            <input type="text" class="field-input" id="gen-label" placeholder="ex: João Silva">
          </div>
          <div class="form-group">
            <div class="field-label">DURAÇÃO</div>
            <select class="field-input" id="gen-duration">
              <option value="1">1 dia</option>
              <option value="3">3 dias</option>
              <option value="7" selected>7 dias</option>
              <option value="14">14 dias</option>
              <option value="30">30 dias</option>
              <option value="60">60 dias</option>
              <option value="90">90 dias</option>
              <option value="180">180 dias</option>
              <option value="365">1 ano</option>
              <option value="0">Sem expiração</option>
            </select>
          </div>
          <div class="form-group">
            <div class="field-label">QUANTIDADE</div>
            <input type="number" class="field-input" id="gen-qty" value="1" min="1" max="50" style="width:100px">
          </div>
          <button class="btn btn-green" onclick="generateKeys()" style="white-space:nowrap">
            + GERAR KEY
          </button>
          <button class="btn btn-cyan" onclick="exportJSON()" style="white-space:nowrap">
            ↓ EXPORTAR JSON
          </button>
        </div>
      </div>
    </div>

    <!-- Table -->
    <div class="section">
      <div class="section-header">
        <span class="section-title">KEYS GERADAS</span>
        <div class="filters">
          <input type="text" class="search-input" id="search-input"
                 placeholder="Buscar key ou label..."
                 oninput="renderTable()">
          <button class="filter-btn active" onclick="setFilter('all',this)">TODAS</button>
          <button class="filter-btn" onclick="setFilter('active',this)">ATIVAS</button>
          <button class="filter-btn" onclick="setFilter('expired',this)">EXPIRADAS</button>
          <button class="filter-btn" onclick="setFilter('revoked',this)">REVOGADAS</button>
        </div>
      </div>
      <div class="section-body" style="padding:0">
        <div class="table-wrap">
          <table>
            <thead>
              <tr>
                <th>KEY</th>
                <th>LABEL</th>
                <th>STATUS</th>
                <th>CRIADA EM</th>
                <th>EXPIRA EM</th>
                <th>TEMPO RESTANTE</th>
                <th>AÇÕES</th>
              </tr>
            </thead>
            <tbody id="keys-tbody"></tbody>
          </table>
          <div id="empty-state" class="empty-state" style="display:none">
            <div class="icon">🔑</div>
            <p>NENHUMA KEY ENCONTRADA</p>
          </div>
        </div>
      </div>
    </div>

    <!-- API snippet -->
    <div class="section">
      <div class="section-header">
        <span class="section-title">COMO VALIDAR NO PRECISION FIX (PYTHON)</span>
      </div>
      <div class="section-body">
        <pre style="background:var(--bg);border:1px solid var(--border);padding:20px;font-size:12px;line-height:1.7;overflow-x:auto;color:var(--cyan)"><span style="color:var(--muted)"># Cole esta função no seu precision_fix.py</span>
<span style="color:var(--yellow)">import</span> json, urllib.request, datetime

<span style="color:var(--purplel)">def</span> <span style="color:var(--green)">validate_key_online</span>(user_key: str) -> dict:
    <span style="color:var(--muted)">"""
    Consulta o arquivo keys.json exportado do painel.
    Retorne True se a key existe, está ACTIVE e não expirou.
    """</span>
    <span style="color:var(--muted)"># Substitua pela URL onde você hospedou o keys.json</span>
    API_URL = <span style="color:var(--yellow)">"https://seusite.com/keys.json"</span>
    <span style="color:var(--yellow)">try</span>:
        req  = urllib.request.urlopen(API_URL, timeout=5)
        data = json.loads(req.read().decode())
        now  = datetime.datetime.utcnow().isoformat()
        <span style="color:var(--yellow)">for</span> k <span style="color:var(--yellow)">in</span> data[<span style="color:var(--yellow)">"keys"</span>]:
            <span style="color:var(--yellow)">if</span> k[<span style="color:var(--yellow)">"key"</span>] == user_key.upper().strip():
                <span style="color:var(--yellow)">if</span> k[<span style="color:var(--yellow)">"status"</span>] != <span style="color:var(--yellow)">"active"</span>:
                    <span style="color:var(--yellow)">return</span> {<span style="color:var(--yellow)">"valid"</span>: False, <span style="color:var(--yellow)">"reason"</span>: <span style="color:var(--yellow)">"revogada"</span>}
                <span style="color:var(--yellow)">if</span> k[<span style="color:var(--yellow)">"expires_at"</span>] <span style="color:var(--yellow)">and</span> k[<span style="color:var(--yellow)">"expires_at"</span>] < now:
                    <span style="color:var(--yellow)">return</span> {<span style="color:var(--yellow)">"valid"</span>: False, <span style="color:var(--yellow)">"reason"</span>: <span style="color:var(--yellow)">"expirada"</span>}
                <span style="color:var(--yellow)">return</span>  {<span style="color:var(--yellow)">"valid"</span>: True,  <span style="color:var(--yellow)">"label"</span>:  k[<span style="color:var(--yellow)">"label"</span>]}
        <span style="color:var(--yellow)">return</span> {<span style="color:var(--yellow)">"valid"</span>: False, <span style="color:var(--yellow)">"reason"</span>: <span style="color:var(--yellow)">"não encontrada"</span>}
    <span style="color:var(--yellow)">except</span> Exception <span style="color:var(--yellow)">as</span> e:
        <span style="color:var(--yellow)">return</span> {<span style="color:var(--yellow)">"valid"</span>: False, <span style="color:var(--yellow)">"reason"</span>: <span style="color:var(--yellow)">f"erro: {e}"</span>}</pre>
      </div>
    </div>

  </div>
</div>

<!-- Toast container -->
<div id="toast"></div>

<!-- Modal -->
<div id="modal-overlay" style="display:none"></div>

<script>
// ══ CONFIG ══
const ADMIN_PASSWORD = "artigos2024"; // Troque para uma senha forte!
const DB_NAME = "PrecisionKeyDB";
const DB_VERSION = 1;
const STORE = "keys";

let db = null;
let allKeys = [];
let currentFilter = "all";

// ══ INDEXEDDB ══
function openDB() {
  return new Promise((res, rej) => {
    const req = indexedDB.open(DB_NAME, DB_VERSION);
    req.onupgradeneeded = e => {
      const d = e.target.result;
      if (!d.objectStoreNames.contains(STORE)) {
        const store = d.createObjectStore(STORE, { keyPath: "id", autoIncrement: true });
        store.createIndex("key", "key", { unique: true });
        store.createIndex("status", "status");
      }
    };
    req.onsuccess = e => { db = e.target.result; res(db); };
    req.onerror   = e => rej(e);
  });
}

function dbGetAll() {
  return new Promise((res, rej) => {
    const tx   = db.transaction(STORE, "readonly");
    const req  = tx.objectStore(STORE).getAll();
    req.onsuccess = e => res(e.target.result);
    req.onerror   = e => rej(e);
  });
}

function dbPut(obj) {
  return new Promise((res, rej) => {
    const tx  = db.transaction(STORE, "readwrite");
    const req = tx.objectStore(STORE).put(obj);
    req.onsuccess = e => res(e.target.result);
    req.onerror   = e => rej(e);
  });
}

function dbDelete(id) {
  return new Promise((res, rej) => {
    const tx  = db.transaction(STORE, "readwrite");
    const req = tx.objectStore(STORE).delete(id);
    req.onsuccess = () => res();
    req.onerror   = e => rej(e);
  });
}

// ══ AUTH ══
function doLogin() {
  const pw  = document.getElementById("pw-input").value;
  const err = document.getElementById("login-err");
  if (pw === ADMIN_PASSWORD) {
    sessionStorage.setItem("admin_auth", "1");
    document.getElementById("login-screen").style.display = "none";
    document.getElementById("app").style.display = "block";
    initApp();
  } else {
    err.textContent = "✗ Senha incorreta.";
    document.getElementById("pw-input").value = "";
    setTimeout(() => err.textContent = "", 3000);
  }
}

function doLogout() {
  sessionStorage.removeItem("admin_auth");
  location.reload();
}

// ══ INIT ══
async function initApp() {
  await openDB();
  document.getElementById("hdr-date").textContent = new Date().toLocaleDateString("pt-BR", {
    day:"2-digit", month:"short", year:"numeric"
  }).toUpperCase();
  await refresh();
  setInterval(refresh, 30000); // auto-refresh a cada 30s
}

async function refresh() {
  allKeys = await dbGetAll();
  updateStats();
  renderTable();
}

function updateStats() {
  const now = new Date().toISOString();
  const active  = allKeys.filter(k => k.status === "active" && (!k.expires_at || k.expires_at > now)).length;
  const expired = allKeys.filter(k => k.status === "active" && k.expires_at && k.expires_at <= now).length;
  const revoked = allKeys.filter(k => k.status === "revoked").length;
  document.getElementById("st-total").textContent   = allKeys.length;
  document.getElementById("st-active").textContent  = active;
  document.getElementById("st-expired").textContent = expired;
  document.getElementById("st-revoked").textContent = revoked;
}

// ══ GENERATE ══
function makeKey() {
  const chars = "ABCDEFGHJKLMNPQRSTUVWXYZ23456789";
  const seg = () => Array.from({length:5}, () => chars[Math.floor(Math.random()*chars.length)]).join("");
  return `${seg()}-${seg()}-${seg()}-${seg()}`;
}

async function generateKeys() {
  const label    = document.getElementById("gen-label").value.trim() || "—";
  const days     = parseInt(document.getElementById("gen-duration").value);
  const qty      = Math.min(50, parseInt(document.getElementById("gen-qty").value) || 1);
  const now      = new Date();
  const created  = now.toISOString();
  const expires  = days === 0 ? null : new Date(now.getTime() + days * 86400000).toISOString();

  const generated = [];
  for (let i = 0; i < qty; i++) {
    const key = makeKey();
    const rec = { key, label: qty > 1 ? `${label} #${i+1}` : label,
                  status: "active", created_at: created, expires_at: expires,
                  duration_days: days };
    await dbPut(rec);
    generated.push(key);
  }

  await refresh();

  if (qty === 1) {
    showModal(generated[0], { label: qty > 1 ? `${label} #1` : label,
      created_at: created, expires_at: expires, duration_days: days });
  } else {
    toast(`${qty} keys geradas com sucesso!`, "success");
  }
}

// ══ TABLE ══
function setFilter(f, el) {
  currentFilter = f;
  document.querySelectorAll(".filter-btn").forEach(b => b.classList.remove("active"));
  el.classList.add("active");
  renderTable();
}

function getKeyStatus(k) {
  if (k.status === "revoked") return "revoked";
  if (k.expires_at && new Date(k.expires_at) <= new Date()) return "expired";
  return "active";
}

function timeRemaining(expires_at) {
  if (!expires_at) return { text: "∞ SEM EXPIRAÇÃO", pct: 100, color: "#a020f0" };
  const now  = Date.now();
  const exp  = new Date(expires_at).getTime();
  const diff = exp - now;
  if (diff <= 0) return { text: "EXPIRADA", pct: 0, color: "#ff3d5a" };
  const d = Math.floor(diff / 86400000);
  const h = Math.floor((diff % 86400000) / 3600000);
  const m = Math.floor((diff % 3600000) / 60000);
  let text = d > 0 ? `${d}d ${h}h` : h > 0 ? `${h}h ${m}m` : `${m}m`;
  // color
  const color = diff < 86400000 ? "#ff3d5a" : diff < 86400000*3 ? "#ffd600" : "#00e676";
  return { text, pct: 100, color };
}

function fmtDate(iso) {
  if (!iso) return "—";
  return new Date(iso).toLocaleDateString("pt-BR", {
    day:"2-digit", month:"2-digit", year:"numeric", hour:"2-digit", minute:"2-digit"
  });
}

function renderTable() {
  const search = document.getElementById("search-input").value.toLowerCase();
  const tbody  = document.getElementById("keys-tbody");
  const empty  = document.getElementById("empty-state");

  let filtered = allKeys.filter(k => {
    const status = getKeyStatus(k);
    if (currentFilter !== "all" && status !== currentFilter) return false;
    if (search && !k.key.toLowerCase().includes(search) && !k.label.toLowerCase().includes(search)) return false;
    return true;
  }).sort((a,b) => new Date(b.created_at) - new Date(a.created_at));

  if (filtered.length === 0) {
    tbody.innerHTML = "";
    empty.style.display = "block";
    return;
  }
  empty.style.display = "none";

  tbody.innerHTML = filtered.map(k => {
    const status = getKeyStatus(k);
    const tr     = timeRemaining(k.expires_at);
    const pillCls = status === "active" ? "pill-active" : status === "expired" ? "pill-expired" : "pill-revoked";
    const dotCls  = status === "active" ? "dot-green" : status === "expired" ? "dot-red" : "dot-muted";
    const statusLabel = status === "active" ? "ATIVA" : status === "expired" ? "EXPIRADA" : "REVOGADA";

    return `
    <tr>
      <td><span class="key-cell" onclick="copyKey('${k.key}')" title="Clique para copiar">${k.key}</span></td>
      <td style="color:var(--text);font-size:12px">${escHtml(k.label)}</td>
      <td><span class="status-pill ${pillCls}"><span class="dot ${dotCls}"></span>${statusLabel}</span></td>
      <td style="color:var(--muted);font-size:11px">${fmtDate(k.created_at)}</td>
      <td style="color:var(--muted);font-size:11px">${fmtDate(k.expires_at)}</td>
      <td>
        <div class="exp-bar-wrap">
          <div class="exp-bar"><div class="exp-bar-fill" style="width:${tr.pct}%;background:${tr.color}"></div></div>
          <span style="font-size:10px;color:${tr.color}">${tr.text}</span>
        </div>
      </td>
      <td>
        <div class="actions">
          <button class="icon-btn" onclick="copyKey('${k.key}')" title="Copiar">📋</button>
          ${status !== 'revoked' ? `<button class="icon-btn danger" onclick="revokeKey(${k.id})" title="Revogar">🚫 REVOGAR</button>` : `<button class="icon-btn" onclick="reactivateKey(${k.id})" title="Reativar">✓ REATIVAR</button>`}
          <button class="icon-btn danger" onclick="deleteKey(${k.id})" title="Deletar">✕</button>
        </div>
      </td>
    </tr>`;
  }).join("");
}

// ══ ACTIONS ══
async function revokeKey(id) {
  const k = allKeys.find(x => x.id === id);
  if (!k) return;
  k.status = "revoked";
  await dbPut(k);
  await refresh();
  toast("Key revogada.", "warning");
}

async function reactivateKey(id) {
  const k = allKeys.find(x => x.id === id);
  if (!k) return;
  k.status = "active";
  await dbPut(k);
  await refresh();
  toast("Key reativada.", "success");
}

async function deleteKey(id) {
  if (!confirm("Deletar esta key permanentemente?")) return;
  await dbDelete(id);
  await refresh();
  toast("Key deletada.", "error");
}

function copyKey(key) {
  navigator.clipboard.writeText(key).then(() => toast(`Copiada: ${key}`, "success"));
}

// ══ EXPORT ══
function exportJSON() {
  const now  = new Date().toISOString();
  const data = {
    exported_at: now,
    total: allKeys.length,
    keys: allKeys.map(k => ({
      key:         k.key,
      label:       k.label,
      status:      getKeyStatus(k),
      created_at:  k.created_at,
      expires_at:  k.expires_at,
      duration_days: k.duration_days,
    }))
  };
  const blob = new Blob([JSON.stringify(data, null, 2)], { type: "application/json" });
  const url  = URL.createObjectURL(blob);
  const a    = document.createElement("a");
  a.href     = url;
  a.download = `precision_keys_${now.slice(0,10)}.json`;
  a.click();
  URL.revokeObjectURL(url);
  toast("JSON exportado com sucesso!", "success");
}

// ══ MODAL ══
function showModal(key, meta) {
  const exp = meta.expires_at ? fmtDate(meta.expires_at) : "Sem expiração";
  const dur = meta.duration_days === 0 ? "Vitalícia" : `${meta.duration_days} dias`;
  const overlay = document.getElementById("modal-overlay");
  overlay.style.display = "flex";
  overlay.innerHTML = `
    <div class="modal">
      <div class="modal-title">🔑 KEY GERADA</div>
      <div class="modal-key" onclick="copyKey('${key}')" title="Clique para copiar">${key}</div>
      <div class="modal-meta">
        <b>Label:</b> ${escHtml(meta.label)}<br>
        <b>Duração:</b> ${dur}<br>
        <b>Criada em:</b> ${fmtDate(meta.created_at)}<br>
        <b>Expira em:</b> ${exp}<br><br>
        Clique na key para copiar.
      </div>
      <div class="modal-actions">
        <button class="btn btn-cyan" onclick="copyKey('${key}');closeModal()">📋 COPIAR KEY</button>
        <button class="btn btn-primary" onclick="closeModal()">FECHAR</button>
      </div>
    </div>`;
}

function closeModal() {
  document.getElementById("modal-overlay").style.display = "none";
}

// ══ TOAST ══
function toast(msg, type = "info") {
  const container = document.getElementById("toast");
  const el = document.createElement("div");
  el.className = `toast-item ${type}`;
  el.textContent = msg;
  container.appendChild(el);
  setTimeout(() => el.remove(), 3100);
}

// ══ UTILS ══
function escHtml(s) {
  return String(s).replace(/&/g,"&amp;").replace(/</g,"&lt;").replace(/>/g,"&gt;");
}

// ══ START ══
if (sessionStorage.getItem("admin_auth") === "1") {
  document.getElementById("login-screen").style.display = "none";
  document.getElementById("app").style.display = "block";
  openDB().then(() => initApp());
}

document.getElementById("pw-input")?.addEventListener("keydown", e => {
  if (e.key === "Enter") doLogin();
});
</script>
</body>
</html>
