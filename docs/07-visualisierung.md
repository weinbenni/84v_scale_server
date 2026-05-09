# 07 – Visualisierung und Bedienkonzept

## 7.1 UI-Schichten

| Schicht | Technik | Zweck |
|---------|---------|-------|
| **Legal-Display** | Qt/QML, Vollbild auf Touch-Panel | Eichrechtliche Wertanzeige + Wägeschein-gewichtsführende Felder |
| **Waagmeister-UI** | Web-App (TS + React/Svelte) im Chromium-Kiosk | Vollständige fachliche Bedienung |
| **Selbstbedien-UI** | Web-App mit eigenem Skin, große Buttons | Fahrer an der Außensäule |
| **Backoffice-UI** | identische Web-App über Browser im LAN | Disposition, Auswertung |

Wichtig: Das **Legal-Display** läuft als eigenständiger Prozess und
überlagert die Web-UI nicht. Der eichpflichtige Wert ist immer und
ohne Bedienschritt sichtbar.

## 7.2 Bildschirm-Layout Waagmeister-Touch (15.6")

```
┌────────────────────────────────────────────────────────────────────┐
│  LEGAL-BAND (oberste 35 % der Höhe, Qt-Vollbild)                   │
│                                                                    │
│              -10 240 kg            ●STILL  ●NULL  ●BRUTTO          │
│              ────────────                                         │
│         Rhewa SN 12345 / SW 1.4    DSAR-Vorschau: #00012345        │
└────────────────────────────────────────────────────────────────────┘
┌──────────────────┬──────────────────┬─────────────────────────────┐
│  Fahrzeug         │  Material        │  Auftrag                    │
│  [BB-AB 1234 ▼]  │  [Splitt 0/16 ▼] │  Auftrag #4711, 12 t        │
├──────────────────┴──────────────────┴─────────────────────────────┤
│  Tara: 7 320 kg (Fest, gültig 30 d)        Netto: 2 920 kg         │
│  [ 1. Wägung ]  [ 2. Wägung ]  [ Einzel ]  [ Tara setzen ]         │
├────────────────────────────────────────────────────────────────────┤
│  Letzte Wägescheine                                                │
│  Zeit       Kfz        Material    Brutto  Tara   Netto  DSAR      │
│  10:05     BB-AB 1234  Splitt     10240   7320    2920   #12344    │
│  09:58     B-XY 9999   Sand       18620   8500   10120   #12343    │
├────────────────────────────────────────────────────────────────────┤
│  Status: Rhewa OK | USV 100 % | Drucker OK | ERP synced 10:01      │
└────────────────────────────────────────────────────────────────────┘
```

Designprinzipien:

- Eichpflichtiger Wert oben, monotype, mind. 80 px hoch.
- Statussymbole (Stillstand/Null/Brutto) sind Pflichtbestandteil und
  dürfen nicht durch andere UI verdeckt werden.
- Aktionen, die einen DSAR-Eintrag erzeugen, sind eindeutig markiert
  („Eichpflichtig erfassen") und werden erst freigegeben, wenn
  Stillstand vorliegt.

## 7.3 Selbstbedien-Bildschirm (Außensäule, 12")

Sequenzieller Wizard mit großen Schaltflächen ( ≥ 80 × 80 px ),
hoher Kontrast, sonnentauglich.

1. Begrüßung & Aufforderung „RFID auflegen oder PIN eingeben".
2. Bestätigung Fahrzeug + Fahrer (mit Foto/Avatar).
3. Auftrag wählen (oder „kein Auftrag" → frei).
4. Hinweis „Bitte Stillstand abwarten" + Live-Wert im Großformat.
5. Beleg drucken / Schranke öffnen / Abfahrt.

Optional: Sprachdialog (TTS) für Hörhilfe.

## 7.4 Web-UI Backoffice (Desktop)

- Linke Navigation: Dashboard, Wägungen, Aufträge, Fahrzeuge,
  Materialien, Kunden, Auswertungen, Eichdaten, System.
- **Dashboard:** Live-Wert (read-only-Spiegel des Legal-Display),
  KPIs des Tages, offene 1. Wägungen, Druckerstatus.
- **Wägungen:** Tabelle mit Filter, Detail-Drilldown auf DSAR-Eintrag.
- **Eichdaten:** Versionsanzeige, DSAR-Browser, Hash-Verifikation,
  Export für Eichbehörde.
- **System:** Diagnose, Logs, Updates (nur nicht-legaler Teil),
  Plombenstatus.

## 7.5 Visualisierungs-Prinzipien (eichrechtlich)

- Live-Wert kommt **nur** aus signiertem Stream (`weight-broker`).
- Wert + Status werden zusammen gerendert; Status nicht ausblendbar.
- Bei Verbindungsverlust zum Rhewa: Wertanzeige wird ausgegraut, klare
  Fehlermeldung „Kein gültiges Telegramm seit X s", und keine
  Wägescheinerstellung möglich.
- Schriftart Live-Wert: nicht-proportional, fest, ohne dynamisches
  Größenwachstum, das den Wert verfälschen könnte (z. B. bei
  Negativwerten).
- Einheit nie weglassen, immer mit Vorzeichen.
- Sprache eichrechtliche Anzeige: deutsch, fachlich konform mit
  Eichbescheid.

## 7.6 Großanzeige (LED, optional)

- Eigener Treiber im legalen Bereich (`legal-display-led`), bedient die
  externe Anzeige direkt aus dem `weight-broker`-Stream.
- Anzeigemodus konfigurierbar: Brutto, Netto (nach Tara), Ampel-Status.
- Bei Stillstand wird der eingefrorene Wert hervorgehoben (z. B. blinken
  beim Übergang).

## 7.7 Druckbild Wägeschein

- A4-Vorlage mit Kopf (Firmenlogo, Adresse), Mitte (Kunde/Lieferung),
  Mess-Block (gewichtsführend, fester Bereich), Fuß (Eichdaten,
  Software-ID, DSAR-Nr., Hash).
- Mess-Block stammt aus signiertem Template; das nicht-legale
  Druckmodul fügt nur drumherum nicht-gewichtsführende Inhalte ein.
- Thermobon-Variante kompakter, gleiche Pflichtfelder.

## 7.8 Barrierefreiheit / Mehrsprachigkeit

- WCAG 2.1 AA für Web-UI.
- Tastaturbedienbar (Waagmeister kann ohne Maus arbeiten).
- Mehrsprachig (DE/EN/PL/CZ) für Web- und Selbstbedien-UI; eichrechtliche
  Anzeige in deutscher Standardsprache wie zugelassen.

## 7.9 Bedienkonzept Hot Keys (Waagmeister)

| Taste | Funktion |
|-------|----------|
| F1 | 1. Wägung erfassen |
| F2 | 2. Wägung erfassen |
| F3 | Einzelwägung |
| F4 | Tara aus Stamm |
| F5 | Wägeschein drucken |
| F6 | Stamm-Picker Fahrzeug |
| F7 | Stamm-Picker Material |
| F8 | Storno (mit 4-Augen) |
| F9 | Letzte Wägung anzeigen |
| F12 | Eichdaten / Info |

## 7.10 Style Guide

- Neutrale Farbpalette, hoher Kontrast (Mindestkontrast 7:1 für
  Werttext).
- Statusfarben: grün (OK), gelb (Warnung), rot (Sperre), blau
  (Information).
- Typo: Inter / Roboto Mono für Werte.
- Komponentbibliothek einheitlich (z. B. Radix UI / Mantine), damit
  Touch und Desktop konsistent bleiben.
