# 03 – Hardware

## 3.1 Anforderungen

- 24/7-Betrieb in einem Waagenhaus (Industrieumgebung, EMV).
- Temperaturbereich –20 °C bis +55 °C (beheiztes Gehäuse für Außensäule).
- Lange Verfügbarkeit ( ≥ 7 Jahre Ersatzteile).
- Galvanische Trennung zur Wägeelektronik des Rhewa.
- USV/Akkupuffer für sauberes Herunterfahren bei Netzausfall.
- Plombierbares Gehäuse für die metrologisch relevanten Komponenten
  (CM4 + Speicher + Schnittstellenkarte zum Rhewa).

## 3.2 Rechnerplattform

**Empfohlen:** Raspberry Pi Compute Module 4

- CM4 mit 4 GB RAM, 32 GB eMMC (Lite-Variante NICHT verwenden – wir
  brauchen den eMMC, damit das System ohne Wechseldatenträger bootet
  und versiegelt werden kann).
- WLAN/BT deaktivierbar (für plombierten Betrieb empfohlen).
- Industrie-Carrier-Board, z. B. Compulab, Waveshare CM4 Industrial,
  oder kundenspezifisches Carrier mit:
  - 2 × isolierte RS232/RS485 (für Rhewa, optional zweite Waage / Display)
  - 2 × CAN (zukunftsoffen, z. B. Schrankenanbindung)
  - 4 × galvanisch getrennte Digital-Eingänge (Lichtschranken,
    Stillstandskontakt, Notaus-Quittung)
  - 4 × Relais-/SSR-Ausgänge (Ampeln, Schranken, Hupe)
  - 2 × USB 3.0, 2 × USB 2.0
  - 2 × Gigabit-Ethernet (eines für ERP-VLAN, eines für Feldgeräte)
  - HDMI + DSI (Touch-Display)
  - RTC mit Stützbatterie
  - TPM 2.0 (für Boot-Schlüssel und Signaturschlüssel)
  - 24 V DC Industrieversorgung mit Verpolschutz

**Alternative:** Industrie-PC mit x86 (z. B. Intel Atom x6000) – falls
WELMEC-Konformität an einen bereits zertifizierten OS-Stack gekoppelt
werden soll.

## 3.3 Speicher

- eMMC (CM4) für OS und legalen Softwarestand (read-only via dm-verity).
- M.2-NVMe (über PCIe des Carriers) für veränderliche Daten:
  Datenbank, Logs, Backups. Mindestens 256 GB, Industrie-Qualität (SLC
  oder pSLC, hohe TBW).
- Zweiter Datenträger (USB-SSD oder zweite M.2) als gespiegeltes
  Append-Log für DSAR und Ereignislogbuch.

## 3.4 Anzeige- und Bedienelemente

### Hauptbedienplatz Waagmeister
- 15.6" oder 21.5" kapazitives Touch-Display, IP65-Front, 1000 cd/m² für
  Tageslicht, HDMI + USB-Touch.
- Industrie-Tastatur optional.
- Etikettendrucker (z. B. Zebra ZD421) und A4-Laserdrucker für
  Wägescheine.

### Selbstbediensäule (Außen)
- 12" Outdoor-Touch-Panel hinter Glas, hinterleuchtete Tastfolie als
  Backup.
- RFID-Leser (HF/13.56 MHz oder UHF, je nach Flotte).
- Barcode-/QR-Scanner (industriell, fest verbaut).
- Thermo-Belegdrucker (Kioskdrucker mit Cutter).
- Sprechstelle (Intercom) optional, IP-basiert.
- Heizung + Lüfter, beheizte Scheibe.

### Großanzeige (optional)
- LED-Außenanzeige (Ampel-/Gewichtsanzeige) über RS485 oder Ethernet.
  Wert wird vom legalen Display-Service direkt bedient.

## 3.5 Anbindung Rhewa 84 Vario

- Primär: serieller Datenausgang (RS232 oder RS485 – je nach Bestellcode
  des Rhewa, beide üblich).
- Galvanische Trennung zwingend (Optokoppler bzw. ADM2587 o. ä.).
- Geschirmtes Kabel, einseitig auf Schirm aufgelegt am Rhewa.
- Konfiguration im Rhewa: kontinuierliche Ausgabe + Anforderungs-Modus,
  damit sowohl Live-Anzeige als auch eichpflichtige Einzeltelegramme
  möglich sind.
- Optional: zweite Schnittstelle (Ethernet/Modbus-TCP, falls Rhewa-Modul
  vorhanden) als Redundanz/Diagnose.

## 3.6 Peripherie & Feldgeräte

| Gerät | Anbindung | Zweck |
|-------|-----------|-------|
| Ampel rot/grün | Relais-Ausgang | Ein-/Ausfahrt |
| Schranke | Relais + Endschalter (DI) | Zufahrtssteuerung |
| Lichtschranke Achserkennung | DI | Plausibilität / Aufzeichnung |
| Kamera (Kennzeichen) | IP/PoE | optionale OCR, Beweisbild |
| Wechselbrückenleser RFID | USB/RS485 | Fahrzeug-ID Selbstbedienung |
| Etikettendrucker | USB/Ethernet | Wägeschein |
| Großanzeige | RS485/Ethernet | Wertdarstellung |

## 3.7 USV und Stromversorgung

- 24 V DC redundant (zwei Netzteile, Diodenkopplung).
- DC-USV mit LiFePO4-Akku: hält System 10 min, signalisiert
  „PowerFail" über DI → kontrolliertes Shutdown.
- Überspannungs- und Blitzschutz an allen außenliegenden Leitungen
  (RS485 zur Säule, LED-Anzeige, RFID).

## 3.8 Schaltschrank / Mechanik

- Schaltschrank IP54, 19" oder Wandgehäuse, mit:
  - Plombierbarem Innenfach für CM4-Carrier + Anschluss zum Rhewa
    (metrologische „Sealable Area")
  - Frei zugänglichem Bereich für nicht-legale Komponenten (NVMe-Wechsel,
    Drucker-Schnittstellen)
- Beschriftung mit Eich-/Konformitätszeichen, Hersteller, Typ, Seriennr.,
  Software-ID-Anzeige.

## 3.9 Sicherheits-/Plombenkonzept

- Hardware-W&M-Schalter (Jumper auf Carrier) → GPIO-Eingang am CM4.
  „Closed" = Eichmodus, „Open" = Servicemodus mit Logbuch-Eintrag.
- Plombiertes Gehäuse über dem CM4 + Rhewa-Anschlusskarte.
- Drahtplombe oder Klebeplombe mit Seriennummer, dokumentiert im
  Ereignislogbuch.

## 3.10 Stückliste (Beispiel, indikativ)

1. Raspberry Pi CM4 4 GB / 32 GB eMMC
2. Industrie-Carrier mit oben genannten Schnittstellen
3. M.2 NVMe 256 GB Industrial
4. USB-SSD 256 GB für DSAR-Mirror
5. 15.6" Touch-Panel (Innenraum)
6. 12" Outdoor-Touch-Panel (Säule)
7. RFID-Leser, Barcodescanner, Thermo-Belegdrucker
8. LED-Großanzeige (optional)
9. RS485-Isolator + Überspannungsschutz
10. 24 V Netzteil + DC-USV mit LiFePO4-Akku
11. Schaltschrank IP54 + Plombiermaterial
