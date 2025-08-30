(function () {
  if (!window.Game || !Game.ready) { alert("Cookie Clicker not ready yet. Wait for the UI to load, then paste again."); return; }

  const MOD_ID = "godzi-helper";
  const STORAGE_KEY = "godziHelperCfgV2";

  const defaults = {
    sellMode: "all",         // "all" or "count"
    sellCount: 10,
    rebuy: true,
    rebuyDelayMs: 200,
    selected: {},            // { "Cursor": true, ... }
    showFloatingButton: true,

    // pause controls
    pauseHotkeys: true,      // P pause, O step
    autoPauseBeforeBurst: false,
    autoResumeAfterBurst: true
  };

  const loadCfg = () => {
    try {
      const raw = localStorage.getItem(STORAGE_KEY);
      return raw ? { ...defaults, ...JSON.parse(raw) } : { ...defaults };
    } catch { return { ...defaults }; }
  };
  const saveCfg = (cfg) => localStorage.setItem(STORAGE_KEY, JSON.stringify(cfg));
  let cfg = loadCfg();

  // seed building selection once
  if (Object.keys(cfg.selected).length === 0) {
    Game.ObjectsById.forEach(o => { if (o.name !== "Cursor") cfg.selected[o.name] = false; });
    saveCfg(cfg);
  }

  // ===== Pause framework (wrap Game.Logic) =====
  let paused = false;
  const originalLogic = Game.Logic;
  Game.Logic = function logicWrapper() {
    if (paused) return;
    return originalLogic.call(Game);
  };
  function setPaused(on) {
    paused = !!on;
    if (paused) Game.Notify("Paused", "Game logic is halted. Use Step or unpause to resume.", [16,5]);
  }
  function togglePause() {
    setPaused(!paused);
    const pBtn = document.getElementById("godziHelperPauseBtn");
    if (pBtn) pBtn.textContent = paused ? "Unpause" : "Pause";
  }
  function stepOneTick() {
    const wasPaused = paused;
    paused = true;
    originalLogic.call(Game);
    if (Game.Draw) Game.Draw();
    paused = wasPaused;
  }

  // ===== Godzamok Burst =====
  async function doBurst() {
    const chosen = Object.keys(cfg.selected).filter(n => cfg.selected[n]);
    if (!chosen.length) { Game.Notify("Godzamok Helper","No buildings selected.",[16,5]); return; }

    const snapshot = {};
    const sells = {};
    let total = 0;

    chosen.forEach(n => {
      const obj = Game.Objects[n];
      snapshot[n] = obj.amount;
      const nSell = (cfg.sellMode === "all") ? obj.amount : Math.min(cfg.sellCount|0, obj.amount|0);
      sells[n] = Math.max(0, nSell|0);
      total += sells[n];
    });

    if (!total) { Game.Notify("Godzamok Helper","Nothing to sell.",[16,5]); return; }

    if (cfg.autoPauseBeforeBurst) setPaused(true);

    // SELL
    chosen.forEach(n => { const k = sells[n]; if (k>0) Game.Objects[n].sell(k); });

    // REBUY
    if (cfg.rebuy) {
      const delay = Math.max(0, cfg.rebuyDelayMs|0);
      await new Promise(r => setTimeout(r, delay));
      chosen.forEach(n => {
        const obj = Game.Objects[n];
        const need = Math.max(0, (snapshot[n]|0) - (obj.amount|0));
        if (need>0) obj.buy(need);
      });
    }

    if (cfg.autoPauseBeforeBurst && cfg.autoResumeAfterBurst) setPaused(false);

    Game.Notify("Godzamok Burst",
      `Sold ${total} building${total===1?"":"s"}${cfg.rebuy?" and repurchased.":"."}`, [16,5]);
  }

  // ===== Floating buttons =====
  function addFloatingButtons() {
    ["godziHelperBurstBtn","godziHelperPauseBtn","godziHelperStepBtn","godziHelperConfigBtn"].forEach(id=>{
      const el=document.getElementById(id); if (el) el.remove();
    });
    if (!cfg.showFloatingButton) return;

    const makeBtn = (id, label, onclick, bottomPx) => {
      const b = document.createElement("button");
      b.id = id;
      b.textContent = label;
      Object.assign(b.style, {
        position:"fixed", right:"12px", bottom:bottomPx, zIndex:99999,
        padding:"8px 12px", fontFamily:"inherit", fontSize:"14px",
        borderRadius:"8px", border:"1px solid #444", background:"#deb887",
        cursor:"pointer", boxShadow:"0 2px 6px rgba(0,0,0,0.3)"
      });
      b.onmouseenter = ()=> b.style.filter="brightness(1.05)";
      b.onmouseleave = ()=> b.style.filter="";
      b.onclick = onclick;
      document.body.appendChild(b);
      return b;
    };

    makeBtn("godziHelperBurstBtn","Godzamok Burst", doBurst, "12px");
    makeBtn("godziHelperPauseBtn", paused?"Unpause":"Pause", togglePause, "52px");
    makeBtn("godziHelperStepBtn","Step", stepOneTick, "92px");
    makeBtn("godziHelperConfigBtn","Config", openConfig, "132px");
  }

  // ===== Hotkeys =====
  function onKey(e){
    if (!cfg.pauseHotkeys) return;
    const k = e.key?.toLowerCase?.();
    if (k==='p') togglePause();
    else if (k==='o') stepOneTick();
  }

  // ===== Tabbed Config Modal =====
  function ensurePromptScrollCSS() {
    const id="godziConfigScrollCSS";
    if (document.getElementById(id)) return;
    const s=document.createElement("style");
    s.id=id;
    s.textContent=`
      .godzi-tabs { display:flex; gap:8px; margin-bottom:8px; }
      .godzi-tabbtn { padding:4px 10px; border:1px solid #444; border-radius:6px; cursor:pointer; background:#caa67a; }
      .godzi-tabbtn.active { filter:brightness(1.08); }
      .godzi-tabpanel { max-height:60vh; overflow:auto; border:1px solid #444; border-radius:6px; padding:8px; background:rgba(0,0,0,0.15); }
      .godzi-grid { line-height:1.8; display:flex; flex-wrap:wrap; gap:4px 14px; }
      .godzi-grid label { display:inline-block; width:220px; }
    `;
    document.head.appendChild(s);
  }

  function buildGeneralHTML() {
    return `
      <div class="listing"><b>General</b></div>
      <div class="listing">
        <label><input type="radio" name="godzi-sellmode" value="all"> Sell <b>all</b> selected</label>
      </div>
      <div class="listing">
        <label><input type="radio" name="godzi-sellmode" value="count"> Sell <b>N per building</b></label>
        &nbsp; N: <input id="godzi-sellcount" type="number" min="1" style="width:70px">
      </div>
      <div class="listing">
        <label><input id="godzi-rebuy" type="checkbox"> Rebuy to original amounts</label>
        &nbsp; Delay (ms): <input id="godzi-delay" type="number" min="0" style="width:80px">
      </div>
      <div class="listing"><b>Pause controls</b></div>
      <div class="listing">
        <a class="option" onclick="window.${MOD_ID}.togglePause()">Toggle Pause</a>
        <a class="option" onclick="window.${MOD_ID}.step()">Step 1 tick</a>
      </div>
      <div class="listing">
        <label><input id="godzi-autopause" type="checkbox"> Auto-pause before burst</label>
        &nbsp;&nbsp;
        <label><input id="godzi-autoresume" type="checkbox"> Auto-resume after burst</label>
      </div>
      <div class="listing">
        <label><input id="godzi-hotkeys" type="checkbox"> Enable hotkeys (P pause, O step)</label>
      </div>
      <div class="listing">
        <label><input id="godzi-floating" type="checkbox"> Show floating buttons</label>
      </div>
    `;
  }

  function buildBuildingsHTML() {
    const boxes = Game.ObjectsById.map(o => `
      <label><input type="checkbox" data-bld="${o.name}"> ${o.name}</label>
    `).join("");
    return `
      <div class="listing"><b>Buildings to affect</b></div>
      <div class="listing godzi-grid">${boxes}</div>
      <div class="listing"><a class="option" onclick="window.${MOD_ID}.burst()">Run Godzamok Burst now</a></div>
    `;
  }

  function openConfig() {
    ensurePromptScrollCSS();
    const html = `
      <div class="listing"><b>Godzamok Helper — Settings</b></div>
      <div class="godzi-tabs">
        <button class="godzi-tabbtn active" id="godzi-tab-general">General</button>
        <button class="godzi-tabbtn" id="godzi-tab-buildings">Buildings</button>
      </div>
      <div class="godzi-tabpanel">
        <div id="godzi-panel-general">${buildGeneralHTML()}</div>
        <div id="godzi-panel-buildings" style="display:none;">${buildBuildingsHTML()}</div>
      </div>
    `;
    Game.Prompt(html, ['Close']);

    // Tab switching
    const btnGen = document.getElementById("godzi-tab-general");
    const btnBld = document.getElementById("godzi-tab-buildings");
    const panGen = document.getElementById("godzi-panel-general");
    const panBld = document.getElementById("godzi-panel-buildings");
    btnGen.onclick = () => { btnGen.classList.add("active"); btnBld.classList.remove("active"); panGen.style.display=""; panBld.style.display="none"; };
    btnBld.onclick = () => { btnBld.classList.add("active"); btnGen.classList.remove("active"); panBld.style.display=""; panGen.style.display="none"; };

    // Populate current values
    try {
      const raw = localStorage.getItem(STORAGE_KEY);
      const cur = raw ? JSON.parse(raw) : { ...defaults };
      // radios
      document.querySelectorAll('input[name="godzi-sellmode"]').forEach(r=>{
        r.checked = (r.value === (cur.sellMode==='count'?'count':'all'));
      });
      // fields
      const $ = (id)=>document.getElementById(id);
      if ($('godzi-sellcount')) $('godzi-sellcount').value = cur.sellCount ?? 10;
      if ($('godzi-rebuy')) $('godzi-rebuy').checked = !!cur.rebuy;
      if ($('godzi-delay')) $('godzi-delay').value = cur.rebuyDelayMs ?? 200;
      if ($('godzi-autopause')) $('godzi-autopause').checked = !!cur.autoPauseBeforeBurst;
      if ($('godzi-autoresume')) $('godzi-autoresume').checked = cur.autoResumeAfterBurst !== false;
      if ($('godzi-hotkeys')) $('godzi-hotkeys').checked = cur.pauseHotkeys !== false;
      if ($('godzi-floating')) $('godzi-floating').checked = cur.showFloatingButton !== false;

      // buildings
      if (cur.selected) {
        Object.entries(cur.selected).forEach(([name,on])=>{
          const box = document.querySelector(`input[data-bld="${name}"]`);
          if (box) box.checked = !!on;
        });
      }
    } catch {}

    // Wire change handlers → save immediately via public API
    // radios
    document.querySelectorAll('input[name="godzi-sellmode"]').forEach(r=>{
      r.addEventListener('change', ()=> window[MOD_ID].setSellMode(r.value));
    });
    // fields
    const bind = (id, fn) => { const el=document.getElementById(id); if (el) el.addEventListener('change', ()=>fn(el.value, el.checked)); };
    bind('godzi-sellcount', v => window[MOD_ID].setSellCount(v));
    bind('godzi-delay', v => window[MOD_ID].setDelay(v));
    bind('godzi-rebuy', (_,c) => window[MOD_ID].toggleRebuy(c));
    bind('godzi-autopause', (_,c)=> window[MOD_ID].setAutoPause(c));
    bind('godzi-autoresume',(_,c)=> window[MOD_ID].setAutoResume(c));
    bind('godzi-hotkeys',   (_,c)=> window[MOD_ID].toggleHotkeys(c));
    bind('godzi-floating',  (_,c)=> window[MOD_ID].toggleFloating(c));
    // buildings
    document.querySelectorAll('input[data-bld]').forEach(box=>{
      box.addEventListener('change', ()=> window[MOD_ID].toggleBuilding(box.getAttribute('data-bld'), box.checked));
    });
  }

  // ===== Public API for menu hooks / config =====
  window[MOD_ID] = {
    burst: doBurst,
    togglePause, step: stepOneTick,
    setSellMode: (m)=>{ cfg.sellMode = (m==='count'?'count':'all'); saveCfg(cfg); },
    setSellCount: (v)=>{ cfg.sellCount = Math.max(1, (v|0)||1); saveCfg(cfg); },
    toggleRebuy: (on)=>{ cfg.rebuy = !!on; saveCfg(cfg); },
    setDelay: (v)=>{ cfg.rebuyDelayMs = Math.max(0, (v|0)||0); saveCfg(cfg); },
    toggleFloating: (on)=>{ cfg.showFloatingButton = !!on; saveCfg(cfg); addFloatingButtons(); },
    toggleBuilding: (name,on)=>{ cfg.selected[name] = !!on; saveCfg(cfg); },
    setAutoPause: (on)=>{ cfg.autoPauseBeforeBurst = !!on; saveCfg(cfg); },
    setAutoResume:(on)=>{ cfg.autoResumeAfterBurst = !!on; saveCfg(cfg); },
    toggleHotkeys:(on)=>{ cfg.pauseHotkeys = !!on; saveCfg(cfg); }
  };

  // ===== Register as a mod so it also appears in Options → Mods =====
  Game.registerMod("Godzamok Helper", {
    init: function () {
      Game.customOptionsMenu.push(()=>`<div class="listing"><b>Godzamok Helper</b> — Use the floating <i>Config</i> button to adjust settings.</div>`);
      addFloatingButtons();
      window.addEventListener('keydown', onKey, true);
      Game.Notify("Godzamok Helper", "Loaded. Configure via the bottom-right Config button.", [16,5]);
    },
    save: function(){ return JSON.stringify(cfg); },
    load: function(str){
      try { const loaded = JSON.parse(str); cfg = { ...defaults, ...loaded }; saveCfg(cfg); addFloatingButtons(); } catch(e){}
    }
  });

  if (Game.onMenu === 'prefs') Game.UpdateMenu();
})();
