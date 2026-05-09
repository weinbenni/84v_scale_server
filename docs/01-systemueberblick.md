# 01 – Systemüberblick

## 1.1 Zielsetzung

Erweiterung einer bestehenden Fahrzeugwaage mit Rhewa 84 Vario um ein
modernes, vernetztes Anzeige-, Wiege- und Auswertesystem. Das Modul ersetzt
**nicht** die eichamtliche Anzeige des Rhewa 84 V, sondern ergänzt sie als
peripheres Gerät.

Kernziele:

1. **Eichfähige Mitanzeige** des aktuellen Bruttogewichts an einem zweiten
   Ort (Halle, Fahrerhaus-Außenanzeige, Waagmeisterbüro).
2. **Wägescheinerstellung** mit Identifikation (Fahrzeug, Material, Kunde),
   Drucken und Archivieren.
3. **Erste/Zweite Wägung** (Vor-/Nachwägung) inkl. Tara-Verwaltung pro
   Fahrzeug.
4. **Datenrückgrat** für ERP-/Disponenten-Anbindung (REST, CSV, optional
   OPC-UA / MQTT).
5. **Selbstbedienung** für Fahrer per RFID/Barcode + Touch.

## 1.2 Komponentenübersicht

```
                            ┌──────────────────────────┐
                            │   Rhewa 84 Vario (legal) │
                            │   Wägeterminal,          │
                            │   Anzeige, Eichplombe    │
                            └──────────┬───────────────┘
                                       │ RS232/RS485 (eichgesicherter
                                       │ Telegrammausgang, kontinuierlich
                                       │ + Anforderung)
                                       ▼
┌────────────────────────────────────────────────────────────────────┐
│  84V Scale Server  (Raspberry Pi CM4 auf Industrie-Carrier)        │
│                                                                    │
│  ┌──────────────┐  ┌────────────────┐  ┌────────────────────────┐  │
│  │ scale-bridge │→ │ weight-broker  │→ │ legal-display (Qt/Web) │  │
│  │ (legal Teil) │  │ (legal Teil)   │  │ + DSAR-Logbuch         │  │
│  └──────────────┘  └───────┬────────┘  └────────────────────────┘  │
│                            │ signierte Messwert-Events             │
│                            ▼                                       │
│  ┌────────────────────────────────────────────────────────────┐    │
│  │  Wiege-Applikation  (nicht-legaler Teil)                   │    │
│  │  - Wägescheine, Stammdaten, Disposition                    │    │
│  │  - Web-UI (Waagmeister + Selbstbedienung)                  │    │
│  │  - REST/MQTT/OPC-UA, Druck-Service, Backup                 │    │
│  └────────────────────────────────────────────────────────────┘    │
└──────────┬─────────────────────┬────────────────────┬──────────────┘
           │                     │                    │
           ▼                     ▼                    ▼
   Touch-Display           Selbstbedien-          ERP /
   Waagmeister             säule (Außen)          Disposition
                           RFID/Barcode/Drucker
```

## 1.3 Rolle des Rhewa 84 Vario

Das Rhewa 84 Vario ist und bleibt das **eichamtlich zugelassene
Wägeterminal**:

- A/D-Wandlung der Wägezellensignale
- Justierung, Linearisierung, Tarierung
- gesetzlich gesicherte Anzeige am Originalgerät
- Eichplombe / W&M-Schalter
- Ausgabe geprüfter Telegramme über serielle Schnittstelle

Der 84V Scale Server konsumiert dessen Telegramme **read-only** und besitzt
keinerlei Möglichkeit, Justierung, Eichparameter oder Kalibrierung des
Rhewa zu beeinflussen.

## 1.4 Logischer Datenfluss

1. Rhewa 84 V sendet zyklisch (z. B. 10 Hz) Bruttogewicht inkl.
   Stillstandsbit, Nullbit, Vorzeichen, Einheit, Status.
2. `scale-bridge` parst das Telegramm, prüft Checksum/BCC, normalisiert auf
   ein internes signiertes Datenformat (Protobuf + Ed25519-Signatur).
3. `weight-broker` verteilt Messwert-Events an Subscriber:
   - `legal-display` → Anzeige + DSAR-Logbuch bei Wägescheinerzeugung
   - `weighing-app` → fachliche Verarbeitung (nicht-legal)
4. Bei einer Wägung erzeugt `legal-display` einen *eichpflichtigen
   Datensatz* (Zeit, Wert, Stillstand, fortlaufende Nummer, Hash) und
   speichert ihn im DSAR-konformen Logbuch.
5. Die Wiege-Applikation referenziert nur die ID des eichpflichtigen
   Datensatzes; sie verändert Gewichtswerte niemals.

## 1.5 Standorte und Netze

- Waagenhaus (CM4-Server, Touch-Panel, Drucker)
- Außensäule mit Selbstbedienterminal (zweites Panel + RFID/Drucker)
- Bürorechner Waagmeister/Disposition (Browser)
- ggf. ERP-Server (Anbindung über VLAN / VPN)

## 1.6 Nicht-Ziele

- Kein eigener gesetzlicher Wägewert ohne Rhewa 84 V.
- Keine Manipulation oder Re-Justierung der Waage.
- Kein Ersatz für gesetzlich vorgeschriebene Eichungen / Nacheichungen.
