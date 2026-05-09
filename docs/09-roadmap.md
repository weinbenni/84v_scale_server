# 09 – Roadmap, Risiken, offene Fragen

## 9.1 Meilensteine

### M0 – Konzept & Eichrechts-Klärung (4–6 Wochen)
- Abstimmung mit Eichbehörde / benannter Stelle (Klassifizierung,
  Baumuster ja/nein).
- Verifikation Telegramm-Variante des konkreten Rhewa 84 V.
- Festlegung: „Mitanzeige eichamtlich" vs. „Kontrollanzeige".
- Hardware-Stückliste final + Bemusterung.

### M1 – HW-Prototyp (4 Wochen)
- CM4-Carrier ausgewählt, mechanischer Aufbau im Schaltschrank.
- Verbindung Rhewa ↔ Server stabil; Telegramm-Sniffer läuft.
- Touch-Panel + Drucker betriebsbereit.

### M2 – Legaler Software-Kern (8 Wochen)
- `scale-bridge`, `weight-broker`, `legal-display`, `legal-audit`.
- DSAR + Eventlog, Hash-Kette, Mirror.
- dm-verity-Image, Signatur-Pipeline, Boot-Verifikation.
- HiL-Tests gegen `rhewa-sim` und Realgerät.

### M3 – Fachapplikation MVP (10 Wochen)
- Stammdaten, Wägung 1./2., Wägescheindruck.
- Web-UI Waagmeister, Druckvorlagen, lokale DB.
- Backup, Monitoring, Update-Pipeline.

### M4 – Selbstbedienung & Integration (8 Wochen)
- Außensäule (RFID, Drucker, Schranke, Ampel).
- ERP-Anbindung (REST/CSV), MQTT, OPC-UA optional.
- Mehrsprachigkeit, Auswertungen.

### M5 – Eichung & Pilot (4–8 Wochen + behördliche Laufzeit)
- Pilot bei einem Anwender, Begleitung durch Eichbehörde.
- Zertifizierungsdokumente, Schulungsmaterial, Service-Handbuch.

### M6 – Serienfreigabe
- Stückliste eingefroren, Lieferantenauswahl, Lager.
- Roll-out-Plan, Support-Konzept.

## 9.2 Aufwandsindikation (grob)

| Bereich | Personentage |
|---------|-------------:|
| Eichrechts-/Behördenklärung | 20 |
| Hardware-Engineering | 40 |
| Legaler Softwarekern | 80 |
| Fachapplikation | 120 |
| Selbstbedienung & IO | 50 |
| Integration ERP / MQTT / OPC-UA | 40 |
| Test, HiL, Pilot | 60 |
| Doku, Schulung, Zertifizierung | 30 |
| **Summe** | **≈ 440 PT** |

## 9.3 Risiken

| # | Risiko | Wirkung | Gegenmaßnahme |
|---|--------|---------|---------------|
| R1 | Eichbehörde stuft Modul als baumusterprüfpflichtig ein | Termin- und Kostenrisiko | Frühzeitige Abstimmung in M0; Pfad „Kontrollanzeige" als Plan B |
| R2 | Telegramm-Variante des Rhewa abweichend dokumentiert | Implementierungsverzug | Sniffer-Test in M1, Parser-Plugin-Architektur |
| R3 | CM4-Verfügbarkeit | Lieferengpässe | Alternative IPC vorbereiten, Schnittstellen identisch |
| R4 | EMV-Probleme im Schaltschrank | Telegrammfehler, BCC-Fehler | Galvanische Trennung, geschirmte Leitungen, frühe EMV-Messung |
| R5 | DSAR-Hashbruch durch Bug | Eichrechtsverletzung | Strenge Tests, Read-only-Mounts, Reviews, Reproduzierbare Builds |
| R6 | Update-Prozess legal Teil zu komplex für Servicetechniker | Fehlbedienung | Wizard + klare Doku + Offline-Bundles + Schulung |
| R7 | NVMe-Verschleiß durch Logging | Datenverlust | Industrie-NVMe, Logrotation, separater Mirror, SMART-Monitoring |

## 9.4 Offene Fragen

- Welche konkrete Telegramm-Variante des Rhewa 84 V ist beim Kunden
  geschaltet? (Eichbescheid einsehen.)
- Soll das Modul als „eichamtliche Mitanzeige" oder reine
  „Kontrollanzeige" in den Verkehr gebracht werden?
- Sind Achswägungen gefordert? Falls ja: ist der Eichbescheid des
  Rhewa dafür zugelassen?
- Welche ERP-Systeme sind anzubinden? (Bestimmt Konnektor-Aufwand.)
- Ist Fahrer-Selbstbedienung zwingend Bestandteil oder optionales
  Ausbaustadium?
- Welche Aufbewahrungsfrist für DSAR und Wägescheine ist betrieblich
  und gesetzlich gefordert (steuer-/handelsrechtlich i. d. R. 10 Jahre)?
- Bevorzugte Sprache der UI-Bibliothek (React/Svelte/Vue)?
- Make-or-Buy bei Touch-Panels und Außensäule?

## 9.5 Folgepotenzial

- Ankopplung weiterer Waagen (Bandwaagen, Kontrollwaagen) mit gleichem
  Architekturmuster.
- KI-gestützte Plausibilitätsprüfung (z. B. Gewicht vs. Material vs.
  historische Daten) als Anti-Betrugs-Modul – streng nicht-legal.
- Mobile App für Disponenten / Fahrer.
- Multi-Standort-Mandantenfähigkeit (zentrales Backoffice, dezentrale
  84V-Server).
