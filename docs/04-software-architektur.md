# 04 – Software-Architektur

## 4.1 Betriebssystem

- **Raspberry Pi OS Lite (64-Bit, Debian Bookworm)** als Basis.
  Alternativ: **Yocto-basiertes Custom-Image** für strengere
  Reproduzierbarkeit (empfohlen, sobald die Architektur stabil ist).
- Read-only-Root via overlayfs; veränderliche Bereiche
  (`/var/lib/scaleserver`, `/var/log/scaleserver`) auf NVMe.
- Systemd als Init.
- Zeitsynchronisation: lokaler chrony, Quelle = GPS-Modul (PPS) oder
  internes NTP. Manipulation an der Zeit löst DSAR-Ereignis aus.

## 4.2 Prozesslandschaft

Aufteilung in unabhängige Systemd-Services. Jeder Service hat eigenen
Linux-User und eigene Dateirechte.

### Legaler Teil (`/opt/legal`, dm-verity, signiert)

| Service | Verantwortung | User |
|---------|---------------|------|
| `scale-bridge.service` | Serielle Anbindung Rhewa, Telegramm-Parsing, BCC-Prüfung | `legal-bridge` |
| `weight-broker.service` | Verteilen signierter Messwert-Events via Unix-Socket | `legal-broker` |
| `legal-display.service` | Vollbild-Wertanzeige (Qt/QML oder Chromium-Kiosk auf statischem Bundle), DSAR-Logbuch, Wägeschein-Druck (gewichtsführende Felder) | `legal-display` |
| `legal-audit.service` | Schreibt DSAR + Ereignislogbuch, signiert, spiegelt auf zweiten Datenträger | `legal-audit` |

### Nicht-legaler Teil (`/opt/app`, normales Update)

| Service | Verantwortung | User |
|---------|---------------|------|
| `scaleserver-api.service` | REST/JSON + WebSocket, Auth, Stammdaten | `scaleapp` |
| `scaleserver-web.service` | Web-UI (statisches Frontend, served durch API oder nginx) | `scaleapp` |
| `scaleserver-print.service` | Druckaufträge (nicht-gewichtsführend), Layoutverwaltung | `scaleapp` |
| `scaleserver-integration.service` | ERP-Sync, MQTT-Bridge, OPC-UA-Server | `scaleint` |
| `scaleserver-io.service` | GPIO/DI/DO, Ampel, Schranke, Kamera-Trigger | `scaleio` |
| `scaleserver-rfid.service` | RFID-/Barcode-Reader, Selbstbedienlogik | `scaleio` |
| `scaleserver-backup.service` | Datenbank- und Logbuch-Backups | `scalebkp` |

## 4.3 Inter-Prozess-Kommunikation

- **Legal → nicht-legal:** ausschließlich über `weight-broker` per
  Unix-Domain-Socket. Datenformat = signiertes Protobuf-Frame
  (Messwert + Status + Sequenznummer + Ed25519-Signatur).
  Subscriber können den Wert lesen, aber nicht zurückschreiben.
- **Innerhalb legal:** lokaler ZeroMQ-Pub/Sub *oder* Unix-Sockets.
- **Innerhalb nicht-legal:** REST + interner Message-Bus (NATS oder
  Redis Streams) für Events wie „Wägung abgeschlossen", „Schranke
  geöffnet", „Druck fertig".

## 4.4 Datenmodell

### Eichrechtlich (SQLite, append-only, zweiter Mirror)
- `dsar_entries` – fortlaufend, hash-verkettet, signiert.
- `event_log` – Boot, Konfig-Änderung, Plomben-/Schalterstatus.

### Fachlich (PostgreSQL auf NVMe; alternativ SQLite für Single-Node)
- `vehicles` (Kennzeichen, Tara-Werte mit Gültigkeit, RFID-UIDs)
- `customers`, `suppliers`
- `materials` (Artikel, Einheit, Standardpreise)
- `orders` (Auftrag, Disposition)
- `weighings` (1./2. Wägung, Brutto/Netto/Tara, DSAR-Referenz)
- `tickets` (Wägeschein, PDF-Pfad, Druckstatus, Storno)
- `users` (Waagmeister, Fahrer, Rollen, Auth)
- `audit_app` (Bedienprotokoll der Applikation, kein Ersatz für DSAR)

Beziehung: `weighings.dsar_id → dsar_entries.id`. Gewichtswerte werden
in `weighings` redundant gespeichert, gelten aber **nur** als gültig,
wenn die DSAR-Signatur verifiziert.

## 4.5 Schnittstellen

### Eingehend
- Serielle Telegramme vom Rhewa 84 V (siehe Doc 06)
- HTTP/REST + WebSocket vom Web-UI
- MQTT (Kameraauslöser, Sensoren)
- Modbus-TCP (optional weitere Feldgeräte)

### Ausgehend
- REST/JSON für ERP (Belege exportieren)
- CSV/XML-Export (manuelle Abrechnung)
- OPC-UA-Server für Leitsystem (read-only Live-Wert, nur kontextuell mit
  DSAR-ID, kein eichrechtlicher Ersatz)
- MQTT-Publisher für Live-Daten (Dashboards)
- Druckerbefehle (CUPS, ESC/POS, ZPL)

## 4.6 Stack-Empfehlung

- **Sprache legal:** C++17 oder Rust für `scale-bridge` + `weight-broker`
  (deterministisch, kleine Abhängigkeiten, leicht prüfbar).
  Anzeige `legal-display` als Qt/QML (LGPL beachten) – eigenständig
  startbar ohne nicht-legalen Teil.
- **Sprache Applikation:** Python (FastAPI) oder Go für
  `scaleserver-api` und Integrationen. Web-Frontend: TypeScript +
  React/Svelte, gebaut zu statischen Assets.
- **DB:** PostgreSQL 16 (Container oder nativ) für Fachdaten, SQLite mit
  WAL für DSAR + Event-Log.
- **Auth:** lokaler User-Store + optional OIDC (Keycloak) für
  Bürorechner.
- **Containerisierung:** nicht-legale Services in Podman-Containern;
  legale Services laufen direkt auf dem Host (für klare Prüfbarkeit).

## 4.7 Konfiguration

- Legal: `/opt/legal/etc/legal.toml` – signiertes Bundle, Aktualisierung
  nur über Update-Pipeline.
- Applikation: `/etc/scaleserver/app.yaml` – über UI editierbar,
  versioniert.
- Geheimnisse: in TPM gespeichert, abgerufen via tpm2-tss.

## 4.8 Logging und Telemetrie

- Strukturiertes Logging (JSON) in journald.
- Lokale Aggregation mit Promtail → Loki (optional, Container).
- Prometheus-Exporter pro Service (Metriken: Telegramm-Rate, BCC-Fehler,
  Stillstand-Latenz, Druckerstatus, USV-Stand).
- Grafana-Dashboard im nicht-legalen UI eingebunden.

## 4.9 Sicherheit

- Firewall (nftables) Default-Deny.
- Reverse-Proxy mit TLS (nginx oder Caddy), eigene CA für interne
  Zertifikate.
- Rolle-basiertes Auth-Modell: Waagmeister, Disponent, Fahrer,
  Servicetechniker, Eichbeamter (read-only Audit).
- Rate-Limit + Audit-Logging an der API.
- Hardware-Token (TPM) für Signaturschlüssel des DSAR.
- Regelmäßige automatische Sicherheitsupdates **nur** für nicht-legalen
  Teil; legaler Teil nur über kontrollierten, signierten Update-Prozess.

## 4.10 Test- und Build-Pipeline

- CI baut zwei Artefakte: `legal-bundle.tar.zst.sig` und
  `app-bundle.tar.zst`.
- Unit-Tests + Integrationstests gegen einen Rhewa-Telegramm-Simulator
  (siehe Doc 06).
- Hardware-in-the-Loop-Test mit echtem Rhewa 84 V auf Prüfstand vor
  jedem Release des legalen Teils.
- Reproduzierbare Builds (Bit-genau) für legalen Teil.
