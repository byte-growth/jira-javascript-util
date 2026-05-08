# Remote Config via GitHub — Ontwerp

**Datum:** 2026-05-08  
**Aanleiding:** Citrix-omgeving wist localStorage bij elke nieuwe sessie, waardoor de bookmarklet-config telkens verloren gaat.

---

## Probleem

De bookmarklet slaat config op in `localStorage` onder de sleutel `__jst_config`. In de Citrix-omgeving (roaming Windows-sessie op `\\OMNASP001\HOME$`) wordt `localStorage` gewist bij elke nieuwe inlog. Collega's hebben hetzelfde probleem.

## Oplossing

Config wordt per persoon opgeslagen als JSON-bestand in de `storage/` map van deze publieke GitHub-repo (`byte-growth/jira-javascript-util`). De bookmarklet is voor iedereen identiek — geen naam hardcoded. Bij opstarten haalt de bookmarklet de lijst van beschikbare profielen op via de GitHub Contents API en toont een profielpicker. Na het kiezen wordt de config opgehaald en opgeslagen in `localStorage` als sessiecache.

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

### Opstarten — profielpicker

Wanneer `localStorage` leeg is (nieuwe Citrix-sessie):

1. Roep de GitHub Contents API aan:  
   `https://api.github.com/repos/byte-growth/jira-javascript-util/contents/storage`
2. Filter de response op `*.json` bestanden en haal namen en `download_url` op.
3. Toon een **profielpicker**: dropdown met alle namen (zonder `.json`) + "Laden" knop.
4. Gebruiker kiest naam → fetch de bijbehorende `download_url` → parse config → sla op in `localStorage` (sleutel `__jst_config` en `__jst_profile`).
5. Open het hoofdscherm.

Wanneer `localStorage` al een config heeft (zelfde sessie, bookmarklet opnieuw gestart):

1. Sla de profielpicker over.
2. Open direct het hoofdscherm met de config uit `localStorage`.

### Fallback volgorde

```
localStorage heeft config   →  gebruik localStorage  (zelfde sessie)
localStorage leeg           →  probeer GitHub URL
  GitHub geslaagd           →  gebruik remote config, sla op in localStorage
  GitHub mislukt            →  gebruik localStorage indien aanwezig
  beide leeg/mislukt        →  open config-editor met standaard voorbeeldconfig
```

De config-editor met standaard voorbeeldconfig is het bestaande gedrag: een bewerkbaar JSON-veld met een complete voorbeeldstructuur, zonder dat er al een profiel bestaat. Dit gedrag blijft ongewijzigd.

### Config aanpassen

Wijzigingen worden aangebracht door het eigen JSON-bestand in de repo te bewerken en te committen (via GitHub webinterface of git). De volgende Citrix-sessie (nieuwe `localStorage`) pikt de bookmarklet de nieuwe config automatisch op via de profielpicker.

De bestaande config-editor (knop "Config" in de UI) blijft werken voor tijdelijke wijzigingen binnen de sessie. Deze wijzigingen worden alleen naar `localStorage` geschreven, niet naar GitHub.

---

## Profielpicker UI

Een nieuw scherm (zelfde overlay-stijl als de bestaande editor):

- Titel: "Kies jouw profiel"
- Dropdown met namen van alle gevonden `.json` bestanden in `storage/`
- Knop "Laden"
- Knop "Annuleren" (sluit de overlay)
- Als de GitHub API niet bereikbaar is: toon melding "Kon profielen niet ophalen" en ga verder met lokale config of editor

---

## Wat verandert er niet

- Jira API-integratie
- UI en alle teksten van het hoofdscherm
- Config-editor voor lokale/tijdelijke wijzigingen
- Gedrag na het aanmaken van subtaken
- Eén identieke bookmarklet voor iedereen

---

## Wat er geïmplementeerd moet worden

1. Map `storage/` aanmaken in de repo met een voorbeeldbestand (`rick.json`).
2. Opstartlogica aanpassen: check localStorage → zo niet, haal profiellijst op via GitHub Contents API.
3. Profielpicker UI bouwen (nieuw scherm in de bestaande overlay-stijl).
4. Na keuze: fetch config van `download_url`, sla op via `saveConfig()` in localStorage.
5. Foutafhandeling: GitHub API-fout → stille fallback naar localStorage of editor, geen harde foutmelding.
6. Bestaande config-editor en hoofdscherm ongewijzigd laten.
