# Remote Config via GitHub — Implementatieplan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** De bookmarklet haalt config op via een profielpicker die JSON-bestanden uit de `storage/` map van de GitHub-repo laadt, zodat de config niet verloren gaat bij een Citrix-sessiereset.

**Architecture:** Bij opstarten controleert de bookmarklet `localStorage`. Is die leeg (nieuwe Citrix-sessie), dan haalt hij de profiellijst op via de GitHub Contents API en toont een dropdown. Na selectie laadt hij de config van de raw GitHub URL en slaat hem op in `localStorage` als sessiecache. Bij een GitHub-fout valt hij terug op `localStorage` of de config-editor met standaard voorbeeldconfig.

**Tech Stack:** Vanilla JavaScript ES5 (bookmarklet), GitHub Contents API (ongeauthenticeerd, publieke repo)

---

## Betrokken bestanden

| Bestand | Actie | Verantwoordelijkheid |
|---|---|---|
| `storage/rick.json` | Aanmaken | Voorbeeldprofiel voor Rick |
| `subtaskcreator/Create Subtasks Jira.js` | Aanpassen | Profielpicker + aangepaste opstartlogica |

---

### Task 1: Maak storage/rick.json aan

**Files:**
- Create: `storage/rick.json`

- [ ] **Stap 1: Maak de map en het bestand aan**

Maak `storage/rick.json` aan met:

```json
{
  "teams": {
    "Team A": {
      "Story": [
        "- Technische analyse",
        "- Implementatie",
        "- Code review",
        "- QA / testen"
      ],
      "Defect": [
        "- Reproduceren",
        "- Oorzaakanalyse",
        "- Fix implementeren",
        "- Regressietest"
      ]
    }
  }
}
```

- [ ] **Stap 2: Commit**

```bash
git add storage/rick.json
git commit -m "feat: add storage folder with rick profile"
```

---

### Task 2: Voeg GITHUB_API constante toe

**Files:**
- Modify: `subtaskcreator/Create Subtasks Jira.js`

De bookmarklet is één minified regel. De constante komt direct na `STORAGE_KEY`.

- [ ] **Stap 1: Voeg constante toe**

Zoek in `subtaskcreator/Create Subtasks Jira.js`:

```
var STORAGE_KEY='__jst_config';
```

Vervang door:

```
var STORAGE_KEY='__jst_config';var GITHUB_API='https://api.github.com/repos/byte-growth/jira-javascript-util/contents/storage';
```

- [ ] **Stap 2: Commit**

```bash
git add "subtaskcreator/Create Subtasks Jira.js"
git commit -m "feat: add GITHUB_API constant"
```

---

### Task 3: Voeg openProfilePicker toe en pas opstartlogica aan

**Files:**
- Modify: `subtaskcreator/Create Subtasks Jira.js`

De `openProfilePicker` functie:
1. Toont "Profielen laden…" in de overlay
2. Haalt profiellijst op van GitHub Contents API
3. Toont dropdown met profielnamen (bestandsnamen zonder `.json`)
4. Bij "Laden": haalt config op via `download_url`, slaat op via `saveConfig()`, opent `openMain()`
5. Bij GitHub-fout: fallback naar `getConfig()` (localStorage) of `openEditor()`

De opstartlogica wijzigt van `if(!cfg){openEditor();}else{openMain();}` naar `if(cfg){openMain();}else{openProfilePicker();}`.

- [ ] **Stap 1: Voeg functie in en vervang opstartlogica**

Zoek aan het einde van `subtaskcreator/Create Subtasks Jira.js`:

```
if(!cfg){openEditor();}else{openMain();}})();
```

Vervang door:

```
function openProfilePicker(){var box=document.createElement('div');box.style='background:#fff;border-radius:10px;padding:24px;width:400px;max-width:95vw;box-shadow:0 8px 40px rgba(0,0,0,0.22)';box.innerHTML='<div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:16px"><h2 style="font-size:16px;font-weight:600;color:#172B4D;margin:0">Kies jouw profiel</h2><button id="__jst_ppx" style="background:none;border:none;font-size:22px;cursor:pointer;color:#6B778C;line-height:1;padding:0">\xd7</button></div><div id="__jst_pp_cnt"><p style="font-size:13px;color:#6B778C">Profielen laden…</p></div>';ov.innerHTML='';ov.appendChild(box);document.getElementById('__jst_ppx').onclick=function(){ov.remove();};fetch(GITHUB_API,{headers:{'Accept':'application/vnd.github.v3+json'}}).then(function(r){return r.json();}).then(function(files){if(!Array.isArray(files))throw new Error('Onverwacht antwoord van GitHub');var profiles=files.filter(function(f){return/\.json$/.test(f.name);});if(!profiles.length)throw new Error('Geen profielen gevonden in storage/');var opts=profiles.map(function(f){return'<option value="'+f.download_url+'">'+f.name.replace(/\.json$/,'')+'</option>';}).join('');document.getElementById('__jst_pp_cnt').innerHTML='<label style="font-size:12px;color:#6B778C;display:block;margin-bottom:4px">Profiel</label><select id="__jst_pp_sel" style="width:100%;padding:7px 8px;border:1px solid #DFE1E6;border-radius:6px;font-size:13px;color:#172B4D;background:#fff;margin-bottom:12px">'+opts+'</select><div id="__jst_pp_err" style="font-size:12px;color:#FF5630;min-height:16px;margin-bottom:8px"></div><div style="display:flex;gap:8px"><button id="__jst_pp_ok" style="flex:1;padding:9px;background:#0052CC;color:#fff;border:none;border-radius:6px;font-size:14px;font-weight:600;cursor:pointer">Laden</button><button id="__jst_pp_ca" style="padding:9px 16px;background:#F4F5F7;color:#172B4D;border:none;border-radius:6px;font-size:14px;cursor:pointer">Annuleren</button></div>';document.getElementById('__jst_pp_ca').onclick=function(){ov.remove();};document.getElementById('__jst_pp_ok').onclick=function(){var url=document.getElementById('__jst_pp_sel').value;var btn=document.getElementById('__jst_pp_ok');btn.disabled=true;btn.textContent='Laden…';fetch(url).then(function(r){return r.json();}).then(function(data){if(!data.teams)throw new Error('Ontbrekende "teams" sleutel');saveConfig(data);cfg=data;openMain();}).catch(function(e){document.getElementById('__jst_pp_err').textContent='Laden mislukt: '+e.message;btn.disabled=false;btn.textContent='Laden';});};}).catch(function(){var fb=getConfig();if(fb){cfg=fb;openMain();}else{openEditor();}});}if(cfg){openMain();}else{openProfilePicker();}})();
```

- [ ] **Stap 2: Commit**

```bash
git add "subtaskcreator/Create Subtasks Jira.js"
git commit -m "feat: profile picker on startup, fallback to localStorage or editor"
```

---

### Task 4: Push en verifieer GitHub API

- [ ] **Stap 1: Push naar GitHub**

```bash
git push origin main
```

- [ ] **Stap 2: Verifieer GitHub Contents API in de browser**

Open:
```
https://api.github.com/repos/byte-growth/jira-javascript-util/contents/storage
```

Verwacht: HTTP 200 met een JSON-array die minimaal dit object bevat:

```json
[
  {
    "name": "rick.json",
    "download_url": "https://raw.githubusercontent.com/byte-growth/jira-javascript-util/main/storage/rick.json"
  }
]
```

Als je een 404 ziet, is de push niet geslaagd of heet de map anders.  
Als je een 403 ziet, is de rate limit bereikt (60 req/uur voor ongeauthenticeerde calls) — wacht een minuut en probeer opnieuw.

- [ ] **Stap 3: Verifieer raw config URL**

Open:
```
https://raw.githubusercontent.com/byte-growth/jira-javascript-util/main/storage/rick.json
```

Verwacht: de JSON-inhoud van `rick.json` (teams, Story, Defect etc.).

---

### Task 5: Handmatig testen in browser

- [ ] **Stap 1: Maak bladwijzer aan**

Kopieer de volledige inhoud van `subtaskcreator/Create Subtasks Jira.js`. Maak een nieuwe browser-bladwijzer aan en plak de inhoud als URL (begint al met `javascript:`).

- [ ] **Stap 2: Test profielpicker — localStorage leeg (nieuwe sessie)**

Open een Jira-issuepagina. Verwijder eerst eventuele bestaande config:

```javascript
// Plak dit in de browser-console op de Jira-pagina:
localStorage.removeItem('__jst_config');
```

Klik de bladwijzer. Verwacht:
- Overlay verschijnt met "Profielen laden…"
- Na ~1 seconde: dropdown met de optie "rick"
- Selecteer "rick", klik "Laden"
- Hoofdscherm verschijnt met de teams en issue-typen uit `rick.json`

- [ ] **Stap 3: Test sessiecache — zelfde sessie, tweede klik**

Sluit de overlay (klik "Annuleren"). Klik de bladwijzer opnieuw. Verwacht:
- Overlay opent direct het hoofdscherm, géén profielpicker
- De config is nog aanwezig vanuit de vorige keer (localStorage sessiecache)

- [ ] **Stap 4: Test fallback — GitHub niet bereikbaar**

Zet in browser DevTools → Network tab → "Offline". Verwijder config uit localStorage:

```javascript
localStorage.removeItem('__jst_config');
```

Klik bladwijzer. Verwacht:
- Profielpicker verschijnt kort (of de laad-indicator)
- Na de mislukte fetch: config-editor opent met de standaard voorbeeldconfig (identiek aan het originele gedrag vóór deze wijziging)

Zet netwerk terug naar "Online" na de test.
