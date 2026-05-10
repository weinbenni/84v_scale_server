# 10 – Feature-Backlog der Wiege-App

Detailliertes Backlog für den **nicht-legalen Applikationsteil**
(`scaleserver-*`). Für jedes Epic werden Stories, Akzeptanzkriterien
(AK), Abhängigkeiten und MoSCoW-Priorität festgehalten.

Konventionen:
- **M** = Must (MVP), **S** = Should, **C** = Could, **W** = Won't (jetzt nicht).
- *Stage*: bezieht sich auf Roadmap-Meilensteine (M3 = Fachapp-MVP,
  M4 = Selbstbed./Integration, M5 = Pilot, post-M6 = später).
- IDs: `EP-x` Epic, `US-x.y` Story.

---

## EP-1 · Wägekern (Wägung Erfassen / Drucken)
*Stage: M3 · Priorität: M*

### US-1.1 Live-Wert-Anzeige im App-Header
- Web-UI zeigt im Header den eichpflichtigen Wert (read-only Spiegel
  des `weight-broker`-Streams).
- AK: Wert + Status &lt; 500 ms hinter Telegramm; bei Stream-Verlust
  klare Sperranzeige; keine eigene Wertinterpretation.
- Abhängigkeiten: <code>weight-broker</code>-Subscriber-API (legal).

### US-1.2 Einzelwägung
- Operator wählt Fahrzeug, Material, Auftrag → Klick „Erfassen".
- AK: nur bei Stillstand; DSAR-ID wird zurückgeliefert; Beleg-Datensatz
  in <code>weighings</code>; Wert nicht clientseitig editierbar.
- AK Negativ: kein DSAR-Aufruf bei fehlender Auswahl Pflichtfelder.

### US-1.3 Doppelwägung (1./2. Wägung)
- 1. Wägung erzeugt offene Lieferung; 2. Wägung schließt sie.
- AK: konfigurierbar voll→leer oder leer→voll;
  Verfallsdatum für offene 1. Wägungen (Default 7 Tage).
- AK: Storno offener Wägungen mit Grund + Audit.

### US-1.4 Tara-Übernahme
- Tara aus Stamm (Festtara), aus offener 1. Wägung oder aus aktuellem
  Bruttowert (Live-Tara).
- AK: Quellen klar gekennzeichnet (Symbol + Tooltip);
  Festtara-Gültigkeit wird beim Auswählen geprüft.

### US-1.5 Wägescheindruck
- Druck auf vorkonfiguriertem Drucker (A4 oder Thermo).
- AK: Pflichtfelder vollständig; gewichtsführende Felder aus
  signiertem Template; PDF-Spool fällt nie verloren (Replay nach
  Druckerausfall).
- AK: Belegnummer monoton wachsend, Lücken auditierbar.

### US-1.6 Beleg-Storno
- 4-Augen-Prinzip durch zweiten autorisierten Benutzer.
- AK: Originalbeleg bleibt unverändert; neuer Beleg „Storno zu Nr. X"
  mit Grund; DSAR-Eventeintrag; Reports markieren Storno separat.

---

## EP-2 · Stammdaten-Verwaltung
*Stage: M3 · Priorität: M*

### US-2.1 Fahrzeuge
- CRUD: Kennzeichen, RFID-UIDs (n:1), Festtara mit Gültigkeit,
  Halter, Notizen, Sperrkennzeichen.
- AK: Kennzeichen unique; Festtara-Änderungen versioniert;
  Sperre verhindert Wägescheinerzeugung.

### US-2.2 Materialien
- CRUD: Bezeichnung, Einheit, Standardpreis, Steuerklasse, GTIN.
- AK: Pflichtfelder; Inaktivierung statt Löschen.

### US-2.3 Kunden / Lieferanten
- CRUD inkl. Adresse, Zahlungsbedingungen, Bonitätssperre,
  Standard-Material.
- AK: USt-ID-Validierung (Format), Dublettenprüfung über Name+PLZ.

### US-2.4 Aufträge
- CRUD inkl. Auftragsnummer, Soll-Menge, Zeitraum, Material, Kunde.
- AK: Restmenge automatisch nach jeder Wägung;
  Status (offen/aktiv/abgeschlossen/abgebrochen).

### US-2.5 Massenimport (CSV)
- Importer für Fahrzeuge, Materialien, Kunden, Aufträge.
- AK: Vorschau, Fehlerprotokoll, Atomic Commit (alles oder nichts).

### US-2.6 Stammdaten-Audit-Trail
- Wer hat wann was geändert.
- AK: lückenlos in <code>audit_app</code>; nicht editierbar.

---

## EP-3 · Benutzer, Rollen, Auth
*Stage: M3 · Priorität: M*

### US-3.1 Lokale Benutzerverwaltung
- Benutzer mit Rollen: Waagmeister, Disponent, Service, Admin,
  Auditor (read-only).
- AK: Passwort-Policy (min 12 Zeichen, MFA optional), Rollen
  rechtebasiert (RBAC).

### US-3.2 PIN/RFID-Login am Touch
- Schneller Login per PIN (4–8 Stellen) oder RFID-Karte.
- AK: PIN-Hash (Argon2); Sperrzeit nach 5 Fehlversuchen;
  RFID-UID hashed gespeichert.

### US-3.3 OIDC-Anbindung (optional)
- SSO über Keycloak/Azure AD.
- AK: Rollen-Mapping konfigurierbar; lokaler Notfall-Admin bleibt
  bestehen.

### US-3.4 Session- und Audit-Log
- Anmeldungen, kritische Aktionen, fehlgeschlagene Logins.
- AK: vom <code>audit_app</code> getrennt vom DSAR; mind. 1 Jahr.

---

## EP-4 · Selbstbedienung Außensäule
*Stage: M4 · Priorität: M*

### US-4.1 Identifikation Fahrer/Fahrzeug
- RFID-Auflage oder PIN/QR.
- AK: Erkennung &lt; 1 s; bei Fehlversuch lokalisierter Hinweis.

### US-4.2 Geführter Wäge-Wizard
- Schritte: Identifikation → Auftrag wählen → Stillstand → Beleg drucken.
- AK: Touch-tauglich (≥ 80 px); Sprache aus Fahrer-Profil;
  Timeout (Default 60 s) → Reset.

### US-4.3 Schranken-/Ampelsteuerung
- Ampel grün + Schranke öffnen nach erfolgreichem Druck.
- AK: Sperre, falls Lichtschranke noch belegt;
  Notaus-DI hat Vorrang.

### US-4.4 Beweisfoto (optional)
- Snapshot Kennzeichen + Brücke beim Wägezeitpunkt.
- AK: Bild gespeichert + mit Wägung verknüpft;
  Aufbewahrung gemäß DSGVO konfigurierbar.

### US-4.5 Mehrsprachigkeit
- DE/EN/PL/CZ + leicht erweiterbar.
- AK: alle Texte aus Übersetzungsdateien; Sprachwechsel ohne Restart.

---

## EP-5 · ERP- &amp; Datenintegration
*Stage: M4 · Priorität: M (REST/CSV), S (OPC-UA/MQTT)*

### US-5.1 REST-API Wägescheine (lesend)
- ERP holt Wägescheine seit X.
- AK: JSON-Schema versioniert; Pagination; OAuth/Token-Auth;
  Antwort enthält DSAR-ID + Hash.

### US-5.2 REST-API Stammdaten (schreibend)
- ERP pusht Aufträge / Stammdaten in die Wiege-App.
- AK: idempotent über External-ID; Validierung; Konflikt-Strategie.

### US-5.3 CSV-Watchfolder Export
- Periodischer Export der Belege.
- AK: konfigurierbar (Pfad, Intervall, Format), Erfolgsquittung.

### US-5.4 MQTT-Publisher Live-Werte
- Live-Wert + Wägung-Events nach MQTT.
- AK: TLS, Retain konfigurierbar; QoS 1; eichrechtlicher Hinweis.

### US-5.5 OPC-UA-Server (Live-Werte read-only)
- Leitsystem holt Live-Wert + Status.
- AK: Companion Spec (z. B. PA-DIM) prüfen; nur lesend; DSAR-Bezug.

### US-5.6 E-Rechnungs-Anhang (XRechnung/ZUGFeRD)
- Wägescheine als Beleg an Rechnung anhängen.
- AK: ZUGFeRD-PDF/A-3 valide; Schemaprüfung im CI.

---

## EP-6 · Disposition &amp; Workflow
*Stage: M4 · Priorität: S*

### US-6.1 Auftragsplanung
- Drag-Drop Liste offener Aufträge nach Tag/Schicht.
- AK: Konfliktwarnung (z. B. Material nicht verfügbar).

### US-6.2 Tour-/Fahrplan-Verknüpfung
- Mehrere Wägungen zu einer Tour.
- AK: Tour-Status, Soll-/Ist-Vergleich, Druck Tour-Übersicht.

### US-6.3 Kunden-Whitelist/Blacklist
- Sperre einzelner Fahrzeuge/Kunden.
- AK: Sperre wirkt sofort an Säule und Waagmeister-UI.

### US-6.4 Zeitfenster-Steuerung
- Annahme nur in definierten Slots (Verkehrslenkung).
- AK: Ausnahmen pro Auftrag möglich.

---

## EP-7 · Auswertungen, Reports, Dashboards
*Stage: M4 · Priorität: S*

### US-7.1 Standard-Reports
- Tages-/Schicht-/Monatsreport, Material-Bilanz, Fahrzeug-Statistik.
- AK: PDF + Excel; Filter (Datum, Kunde, Material).

### US-7.2 Dashboard
- KPIs: Anzahl Wägungen, Tonnage, Auslastung, offene 1. Wägungen,
  Druckerstatus.
- AK: Live, in &lt; 2 s aktualisiert.

### US-7.3 Eich-Audit-Bericht
- Liste aller DSAR-Einträge mit Hash-Prüfung; Export (signiertes ZIP).
- AK: nur Auditor-Rolle, Zugriff in Audit-Log.

### US-7.4 Custom-Report-Builder
- Auswahl Felder + Filter + Speicherung als Vorlage.
- *Priorität C*; nicht MVP.

---

## EP-8 · Drucker- &amp; Belegmanagement
*Stage: M3 · Priorität: M*

### US-8.1 Druckerprofile
- Mehrere Drucker (A4, Thermo, Etiketten); Wahl pro Belegtyp.
- AK: CUPS-/ESC-POS-/ZPL-Treiber unterstützt; Statusabfrage.

### US-8.2 PDF-Archiv
- Jeder Beleg zusätzlich als PDF/A-3.
- AK: Aufbewahrungsrichtlinie konfigurierbar; Suche nach Belegnummer.

### US-8.3 Druckwiederholung
- Replay nach Papier-/Druckerstörung.
- AK: kein doppelter DSAR-Eintrag; Beleg trägt „REPRINT".

### US-8.4 E-Mail-/Cloud-Versand
- Beleg an Kunde per E-Mail oder in Kunden-SFTP.
- AK: Versandprotokoll, Wiederholbarkeit, Zustellbestätigung.

---

## EP-9 · IO &amp; Feldgeräte
*Stage: M4 · Priorität: M*

### US-9.1 GPIO-Konfiguration im UI
- DI/DO benennen, Logik (NO/NC), Pulsdauer einstellen.
- AK: Test-Knopf (DO toggeln) mit Audit-Eintrag.

### US-9.2 Verkehrssteuerung (Ampel/Schranke)
- Steuerung über Workflow-Engine (Wägung → Ampel grün → Schranke).
- AK: konfigurierbare Sequenzen; Notaus-DI hat absolute Priorität.

### US-9.3 Kennzeichen-OCR
- Kamera-Bild via OCR vorschlagen; Operator bestätigt.
- AK: niemals automatisch ohne Bestätigung; Trefferquote im Log.

### US-9.4 Lichtschranken-Plausibilität
- Wägung nur, wenn Lichtschranke „Brücke belegt" meldet.
- AK: Verstoß protokolliert, Wägung gesperrt mit Klartextfehler.

---

## EP-10 · Konfiguration &amp; Service
*Stage: M3 · Priorität: M*

### US-10.1 Konfigurations-UI (App)
- Schnittstellen, Drucker, IO, Mailserver, ERP-Endpoints.
- AK: Rollback nach n Sekunden ohne Bestätigung;
  Versionierung der Config.

### US-10.2 Diagnose-Seite
- Live-Telegramm-Sniffer, Druckerstatus, USV, NVMe-Health, BCC-Quote.
- AK: keine Schreibrechte am legalen Teil.

### US-10.3 Update-Verwaltung (App-Teil)
- Upload signierter App-Pakete; A/B-Slot Anzeige; Rollback-Knopf.
- AK: Update-Log; legalen Teil nicht berührbar.

### US-10.4 Backup &amp; Restore
- Manuelles Backup + Restore-Wizard.
- AK: Restore-Test dokumentierbar; Hash der Backup-Datei.

---

## EP-11 · Sicherheit &amp; Compliance
*Stage: M3 · Priorität: M*

### US-11.1 TLS überall
- Alle HTTP-Endpunkte nur TLS; interne CA.
- AK: HSTS, mTLS für ERP-Anbindung optional.

### US-11.2 Rate-Limit + Brute-Force-Schutz
- An Login + REST-API.
- AK: Konfigurierbar; Alarmierung bei Schwellwertüberschreitung.

### US-11.3 DSGVO-Funktionen
- Auskunfts- &amp; Löschanträge bezogen auf Personendaten (Fahrer, Operator).
- AK: DSAR-Daten bleiben unangetastet (gesetzliche Aufbewahrung);
  personenbezogene Felder werden pseudonymisiert.

### US-11.4 Geheimnis-Verwaltung
- API-Tokens, Passwörter über TPM/Vault.
- AK: nicht im Klartext auf Disk; Rotation dokumentiert.

---

## EP-12 · Beobachtbarkeit
*Stage: M3 · Priorität: S*

### US-12.1 Strukturiertes Logging
- JSON-Logs mit Korrelation-ID.
- AK: Logrotation; sensible Felder maskiert.

### US-12.2 Metriken-Endpoint
- Prometheus-Exporter pro Service.
- AK: Standard-Metriken (Telegrammrate, BCC-Fehler, Stillstand-Latenz,
  Wägungen/h, DB-Latenz).

### US-12.3 Lokales Dashboard (Grafana)
- Vorgefertigte Dashboards (Betrieb, Eich-Health).
- AK: Read-only für nicht-Admins.

### US-12.4 Alarmrouting
- E-Mail / Matrix / SMS-Gateway; Akzeptanz im UI.
- AK: ack/snooze; Eskalationsstufen.

---

## EP-13 · Hochverfügbarkeit &amp; Notlauf
*Stage: M3 · Priorität: M*

### US-13.1 Offline-Notbetrieb
- Wägung möglich auch ohne ERP/Internet/zentrale DB-Replik.
- AK: minimaler Schein druckbar; Nachpflege im Backoffice.

### US-13.2 Zweit-Display Failover
- Bei Display-Ausfall übernimmt zweites Panel die Anzeige.
- AK: kein doppelter DSAR-Eintrag.

### US-13.3 Drucker-Fallback
- Bei Hauptdrucker offline → Fallback (Etikett oder zweiter A4).
- AK: Hinweis im UI; Originaldruck nachholbar.

---

## EP-14 · Mobile / Remote (post-M6)
*Stage: post-M6 · Priorität: C*

### US-14.1 Disponenten-App
- Mobile Übersicht offene Aufträge, Live-KPIs, Sperren.

### US-14.2 Fahrer-App
- Eigene Wägescheine als PDF; Status der nächsten Tour.

### US-14.3 Multi-Standort-Backoffice
- Zentrale Sicht über mehrere 84V-Server (Mandantenfähigkeit).

---

## Prioritätsmatrix (Übersicht)

| Epic | M3 | M4 | M5 | post-M6 |
|------|----|----|----|---------|
| EP-1 Wägekern | M | – | – | – |
| EP-2 Stammdaten | M | – | – | – |
| EP-3 Auth | M | – | – | – |
| EP-4 Selbstbedienung | – | M | – | – |
| EP-5 ERP/Integration | C (REST) | M | – | C (E-Rechnung) |
| EP-6 Disposition | – | S | – | – |
| EP-7 Reports | – | S | – | C (Builder) |
| EP-8 Drucker | M | – | – | – |
| EP-9 IO/Feld | – | M | – | – |
| EP-10 Service | M | – | – | – |
| EP-11 Sicherheit | M | – | – | – |
| EP-12 Observability | S | – | – | – |
| EP-13 HA/Notlauf | M | – | – | – |
| EP-14 Mobile | – | – | – | C |

## Definition of Done (App-Features)

- Code mit Unit- und Integrationstests; Coverage-Schwellwert eingehalten.
- Statische Analyse fehlerfrei (mypy/clippy/eslint).
- Akzeptanzkriterien automatisiert geprüft, soweit möglich (Playwright).
- Doku ergänzt (User-Doku DE, Service-Doku DE).
- i18n-Strings extrahiert; mind. DE komplett, EN als Fallback.
- Audit-Log-Eintrag, falls Aktion sicherheitsrelevant.
- Keine Regression in HiL-Tests des legalen Teils.
- Code-Review durch zweite Person; Merge nur grün durch CI.
