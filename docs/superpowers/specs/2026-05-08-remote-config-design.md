# Remote Config via GitHub — Ontwerp

**Datum:** 2026-05-08  
**Aanleiding:** Citrix-omgeving wist localStorage bij elke nieuwe sessie, waardoor de bookmarklet-config telkens verloren gaat.

---

## Probleem

De bookmarklet slaat config op in `localStorage` onder de sleutel `__jst_config`. In de Citrix-omgeving (roaming Windows-sessie op `\\OMNASP001\HOME$`) wordt `localStorage` gewist bij elke nieuwe inlog. Collega's hebben hetzelfde probleem.

## Oplossing

Config wordt per persoon opgeslagen als JSON-bestand in deze publieke GitHub-repo (`byte-growth/jira-javascript-util`). De bookmarklet haalt de config op via de raw GitHub URL bij elke opstart. Elke collega heeft zijn eigen bestand en zijn eigen bookmarklet-variant met die URL hardcoded.

---

## Mapstructuur

```
storage/
  rick.json
  <collega>.json
  ...
```

Bestandsformaat is ongewijzigd:

```json
{
  "teams": {
    "Team A": {
      "Story": [
        "- Technische analyse",
        "- Implementatie"
      ]
    }
  }
}
```

---

## Bookmarklet gedrag

### Opstarten

1. Bookmarklet bevat een hardcoded `CONFIG_URL` per persoon:  
   `https://raw.githubusercontent.com/byte-growth/jira-javascript-util/main/storage/<naam>.json`
2. Bij elke opstart wordt `fetch(CONFIG_URL)` aangeroepen.
3. Bij succes: response wordt geparsed en opgeslagen in `localStorage` als sessiecache.
4. Bij mislukking (netwerk, offline): fallback naar bestaande `localStorage`-waarde als die aanwezig is.
5. Als beide falen: config-editor wordt geopend (bestaand gedrag).

### Config aanpassen

Wijzigingen worden aangebracht door het eigen JSON-bestand in de repo te bewerken en te committen (via GitHub webinterface of git). De volgende sessie pikt de bookmarklet de nieuwe config automatisch op.

De bestaande config-editor (knop "Config" in de UI) blijft werken voor tijdelijke wijzigingen binnen de sessie. Deze wijzigingen worden alleen naar `localStorage` geschreven, niet naar GitHub.

### Fallback volgorde

```
fetch(CONFIG_URL) geslaagd  →  gebruik remote config  →  sla op in localStorage
fetch(CONFIG_URL) mislukt   →  gebruik localStorage indien aanwezig
beide leeg                  →  open config-editor
```

---

## Per-persoon bookmarklet

Elke collega heeft een eigen bladwijzer-variant met hun URL:

```
javascript:(function(){var CONFIG_URL='https://raw.githubusercontent.com/byte-growth/jira-javascript-util/main/storage/rick.json';...})()
```

De rest van de bookmarklet-code is identiek. Instellen: eenmalig de bladwijzer aanmaken met de eigen URL. Als bladwijzers ook worden gewist in Citrix, kan het `.js`-bestand op de P-schijf worden bewaard en eenmalig per sessie als bladwijzer worden toegevoegd.

---

## Wat verandert er niet

- Jira API-integratie
- UI en alle teksten
- Config-editor voor lokale/tijdelijke wijzigingen
- Gedrag na het aanmaken van subtaken

---

## Wat er geïmplementeerd moet worden

1. Map `storage/` aanmaken in de repo met een voorbeeldbestand (`rick.json`).
2. In de bookmarklet: `CONFIG_URL` constante toevoegen bovenaan de IIFE.
3. Opstartlogica aanpassen: eerst `fetch(CONFIG_URL)`, dan pas `getConfig()` (localStorage), dan editor.
4. Na succesvolle fetch: resultaat via `saveConfig()` in localStorage opslaan.
5. Foutafhandeling: fetch-fouten stille fallback, geen zichtbare foutmelding tenzij alles ontbreekt.
