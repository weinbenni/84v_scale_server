# 08 – Betrieb, Deployment, Wartung

## 8.1 Image-Aufbau

Auslieferung als ein **Master-Image** für den eMMC des CM4:

- `/boot` – signierter Bootloader, Kernel, Device-Tree, dm-verity-Hashes.
- `/` – Read-only-Root (Debian + Basis-Pakete).
- `/opt/legal` – Read-only Subvolume, dm-verity. Enthält:
  `scale-bridge`, `weight-broker`, `legal-display`, `legal-audit` und
  signierte Konfiguration.
- `/opt/app` – Read-write, regulär per Paket aktualisierbar.
  Enthält Container-Images / Binaries des nicht-legalen Teils.
- `/var/lib/scaleserver` – Read-write auf NVMe (PostgreSQL, App-Daten).
- `/var/lib/scaleserver/legal` – Read-write auf NVMe **und** Mirror auf
  USB-SSD; enthält DSAR + Event-Log.

## 8.2 Inbetriebnahme

1. Hardware verkabeln (Rhewa, Display, Drucker, USV, IO).
2. Erstinbetriebnahme-Wizard (am Touch-Panel):
   - Sprache, Datum/Zeit, Netzwerk
   - Anbindung Rhewa: Schnittstelle, Baudrate, Telegrammtyp
   - Lese-Test mit Live-Anzeige + BCC-Statistik
   - Eingabe Stammkonfiguration (Firma, Adresse)
3. Eichung & Plombierung:
   - Verifikation Software-ID (gegen Eichbescheid)
   - Plombierung Hardware
   - Ersteintrag im Eventlog mit Plombennummer
4. Stammdaten-Import (CSV oder REST).
5. Schulung Waagmeister + Fahrer.

## 8.3 Update-Strategie

### Nicht-legaler Teil
- Online- oder Offline-Update über signierte `.deb`/Container-Images.
- Rollback durch A/B-Slots auf NVMe.
- Updates können automatisch ausgerollt werden, müssen aber den legalen
  Teil ungestört lassen (Ressourcenlimits, separater Restart).

### Legaler Teil
- Ausschließlich durch geschulten Servicetechniker.
- Workflow:
  1. Anlieferung neues `legal-bundle.tar.zst.sig`.
  2. Service-Anmeldung mit Berechtigung „Service".
  3. W&M-Schalter offen (Plombe gebrochen, im Vorfeld dokumentiert).
  4. `scalectl legal-update <bundle>` prüft Signatur und schreibt
     Versionswechsel ins Eventlog.
  5. Reboot → dm-verity validiert neuen Hash.
  6. W&M-Schalter zurück, Plombe neu, Eintrag im Eventlog.

## 8.4 Backup

- **DSAR + Eventlog**: kontinuierlich gespiegelt auf USB-SSD; täglicher
  signierter Snapshot auf Netzlaufwerk (SFTP) oder Cloud-Bucket.
- **Fachdatenbank**: nightly `pg_dump` + WAL-Archivierung. Aufbewahrung
  konfigurierbar (z. B. 90 Tage).
- **Druckspool**: 30 Tage rolling.
- Restore-Test: dokumentierter Vorgang, einmal pro Quartal.

## 8.5 Monitoring & Alarmierung

- Prometheus-Exporter (siehe Doc 04) → lokales Grafana.
- Alarmkanäle: E-Mail, optional Matrix/SMS-Gateway.
- Pflicht-Alarme:
  - Telegrammausfall > 10 s
  - BCC-Fehlerquote > 1 %
  - DSAR-Mirror inkonsistent
  - USV auf Akku
  - NVMe SMART kritisch
  - Plombenschalter offen länger als 10 min

## 8.6 Fernwartung

- Nur nicht-legaler Teil per Wireguard-VPN.
- Zugang über zertifikatsbasiertes mTLS, Audit-Trail in `audit_app`.
- Eichrelevante Aktionen explizit nicht remote möglich (Update,
  Plombierung).

## 8.7 Ausfallszenarien

| Szenario | Reaktion |
|----------|----------|
| Rhewa offline | Wägung gesperrt, Live-Wert ausgegraut, Eventlog-Eintrag, Alarm |
| BCC-Fehler vereinzelt | Telegramm verworfen, Zähler erhöht |
| BCC-Fehler dauerhaft | Wägung gesperrt, Alarm „Schnittstelle defekt" |
| NVMe defekt | Failover auf USB-SSD-Mirror für DSAR; App-Funktionen reduziert; Alarm |
| Stromausfall | USV puffert 10 min, sauberes Shutdown nach 5 min |
| Drucker offline | Beleg im PDF-Spool, Hinweis im UI, Nachdruck möglich |
| Display defekt | Web-UI auf Ersatz-Tablet/Laptop, Großanzeige bleibt aktiv |
| DSAR-Hashbruch | Sofort harte Sperre, Alarm, Service-Einsatz, Eichbehörde informieren |

## 8.8 Wartungsintervalle

- Sichtprüfung Plomben: monatlich.
- Backup-Restore-Test: vierteljährlich.
- USV-Akku-Test: halbjährlich.
- Nacheichung gemäß Eichgesetz (typ. alle 2 Jahre für Fahrzeugwaagen).

## 8.9 Dokumentationspflichten

- Konformitätserklärung Modul (sofern eichamtlich erfasst).
- Bedienungsanleitung mit Hinweis auf eichrechtlich relevante Funktionen.
- Service-Handbuch (intern), inkl. Update-/Plombenprozess.
- Schulungsnachweis Waagmeister.
