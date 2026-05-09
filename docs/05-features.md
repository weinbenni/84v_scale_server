# 05 – Features der Wiegesoftware

## 5.1 Überblick nach Rollen

| Rolle | Hauptaufgaben |
|-------|---------------|
| Waagmeister | Wägungen erfassen, Wägescheine drucken, Stammdaten pflegen, Korrekturen |
| Disponent | Aufträge anlegen, Lieferungen planen, Auswertungen |
| Fahrer | Selbstbedienung an der Säule, RFID-Identifikation |
| Service | Konfiguration, Updates (nicht-legal), Diagnose |
| Eichbeamter / Auditor | Read-only Zugriff auf DSAR + Ereignislog |

## 5.2 Kern-Wiegefunktionen

### 5.2.1 Live-Anzeige
- Großgewichts-Anzeige (Brutto), Einheit, Vorzeichen.
- Status-Icons: Stillstand, Null, Brutto/Netto, Über-/Unterlast,
  Verbindung zum Rhewa.
- Heartbeat-Indikator (Telegramm-Alter ≤ 1 s, sonst rot).
- DSAR-fähig: jeder dargestellte Wert hat zugehörige Sequenznummer.

### 5.2.2 Einzel-/Doppelwägung
- **Einzelwägung:** Brutto = Bruttowert, Tara aus Stammdaten oder
  Festtara-Eingabe → Netto.
- **Doppelwägung (1. + 2. Wägung):**
  - 1. Wägung: Fahrzeug voll → Brutto erfassen, Wägung „offen".
  - 2. Wägung: Fahrzeug leer → Tara erfassen, Netto = Brutto − Tara.
  - Reihenfolge konfigurierbar (auch leer→voll möglich).
- **Stillstandskontrolle:** Werterfassung nur bei `motion=0`-Bit; sonst
  Hinweis und kein DSAR-Eintrag.

### 5.2.3 Wägeschein
- Felder: Belegnummer (eichpflichtig), Datum/Uhrzeit, Brutto, Tara,
  Netto, Einheit, Material, Auftrag, Kunde, Lieferant, Fahrzeug-Kz,
  Fahrer, Bemerkungen, Eichdaten (DSAR-Nr., Software-ID, Rhewa-ID).
- PDF-Layout konfigurierbar – aber gewichtsführende Felder kommen aus
  signiertem Template-Block, dessen Hash im Ereignislogbuch steht.
- Ausgabe: A4-Drucker, Thermobon, PDF-Archiv, optional E-Mail.
- Storno: nur durch berechtigte Rolle, mit Grund und neuem DSAR-Ereignis;
  ursprünglicher Schein bleibt unverändert.

### 5.2.4 Tara-Verwaltung
- **Festtara** pro Fahrzeug mit Gültigkeitsdatum (z. B. 30 Tage).
- **Erstwägungstara** mit Verfallsdatum (offene 1. Wägung räumt sich
  automatisch nach n Tagen → Eintrag im Logbuch).
- **Live-Tara** (manuell mit aktuellem Bruttowert übernehmen).

### 5.2.5 Achs-/Mehrteilwägung (optional)
- Achserfassung über Lichtschranken-DI oder manuell.
- Summenbildung mehrerer Teilwägungen → Gesamtgewicht.
  Achswägungen sind eichrechtlich heikel: Implementierung nur, wenn
  Rhewa und Eichbescheid das vorsehen.

## 5.3 Stammdaten

- Fahrzeuge: Kennzeichen, RFID-UIDs, Festtara, Fahrer, Eigentümer.
- Materialien: Bezeichnung, Einheit, Standardpreis, Steuerklasse.
- Kunden / Lieferanten: Adresse, Belegfilter, Bonitätsstatus.
- Aufträge / Verträge: Kontingente, Zeiträume, Preis pro Material.
- Benutzer: Rollen, Schichten, PIN, RFID, optional OIDC-Bindung.

Stammdaten-Import per CSV und über REST.

## 5.4 Selbstbedienung (Außensäule)

- Identifikation per RFID, alternativ PIN oder QR-Code.
- Geführter Dialog:
  1. Fahrzeug erkannt → Anzeige Kennzeichen, Auftrag.
  2. Auftrag wählen / Material bestätigen.
  3. Hinweis „Bitte Stillstand abwarten".
  4. Bei Stillstand → Wägung erfassen → Beleg drucken.
- Kamera-Snapshot (Kennzeichen + Fahrzeug auf Brücke) wird mit
  Wägung referenziert (nicht-legal, Beweisfoto).
- Schranken- und Ampelsteuerung im Dialogfluss.
- Mehrsprachig (DE/EN/PL/CZ je nach Standort), Sprachwahl pro Fahrer im
  Stammsatz.

## 5.5 Disposition / Backoffice

- Auftragsliste, Filter nach Datum, Kunde, Material, Status.
- Wägescheinliste mit Suche, Drilldown auf DSAR.
- Bearbeitungs-Workflow für Korrekturen (z. B. falsches Material): nur
  nicht-gewichtsführende Felder änderbar; Änderungen versioniert.
- Bestandsverbuchung optional, oder Übergabe an ERP.

## 5.6 Auswertungen

- Tages-/Schicht-/Monatsberichte, Excel/PDF-Export.
- Material-Bilanzen (Eingang/Ausgang/Bestand).
- Fahrzeug-Statistiken (Anzahl Wägungen, Auslastung).
- Eich-Audit-Bericht: Liste aller DSAR-Einträge, Hash-Kette
  verifizieren, Export für Eichbehörde.

## 5.7 Schnittstellen / Integrationen

- ERP-Anbindung (SAP/Datev/Sage/Diamant) per REST oder CSV-Watchfolder.
- DATEV-Belegbild-Export.
- E-Rechnung (XRechnung/ZUGFeRD) optional.
- Telematik (Verknüpfung Wägung → Tour) per MQTT oder REST.
- Schranken-/Tor-Anbindung über IO-Service.

## 5.8 Bedienfunktionen Waagmeister

- „Quick-Wiegung": Hotkey, Stammdaten-Picker, sofort Druck.
- Vorlagen für wiederkehrende Aufträge.
- Nachbearbeitung mit Audit-Trail.
- Hinweis-/Sperrfunktion (z. B. „Fahrzeug XY: Zufahrt verboten").
- Notbetrieb: Wenn Datenbank nicht erreichbar, kann eichrechtlich
  trotzdem ein Beleg mit DSAR + minimaler Identifikation gedruckt
  werden; spätere Nachpflege im Backoffice.

## 5.9 Service- und Diagnosefunktionen

- Live-Telegramm-Sniffer (read-only) mit BCC-Statistik.
- Selbsttest-Seite: Verbindung Rhewa, NVMe-Health, USV-Stand,
  Druckerstatus, Schranken-IO.
- Version & Hash der legalen Software einsehbar.
- Fernwartung über VPN (Wireguard), nicht-legaler Teil only;
  legaler Teil per definitionem nur lokal updatebar.

## 5.10 Sicherheits- und Eich-Features

- DSAR-Browser (Read-only) mit Hash-Verifikation und Export.
- Ereignislogbuch-Browser.
- Anzeige W&M-Schalter-Status, Plombenseriennummer (manuell
  eingetragen + im Eventlog dokumentiert).
- Rollenbasierte Berechtigungen, 4-Augen-Prinzip für Storno und
  Stammdaten-Korrekturen mit Geldbetragseinfluss.

## 5.11 Hochverfügbarkeit / Notlauf

- Lokale Datenhaltung; ohne ERP/Internet voll arbeitsfähig.
- Bei Druckerausfall: Beleg in PDF-Spool, manuell nachdruckbar.
- Bei Ausfall des Hauptdisplays: zweites Anzeigegerät übernimmt
  (gleicher `weight-broker`-Stream).
- Automatischer DB-Backup auf zweite Platte und optional auf NAS/SFTP.
