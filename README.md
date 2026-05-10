# 84V Scale Server

Plattform für ein eichfähiges Zusatz-Anzeigegerät und Wiegesoftware-Server für
Fahrzeugwaagen mit Rhewa 84 Vario Wägeterminal.

Dieses Repository enthält in der jetzigen Phase ausschließlich Planungs- und
Architekturdokumente. Die eigentliche Implementierung folgt erst, wenn die
hier beschriebene Architektur, das eichrechtliche Konzept und die
Hardware-Stückliste freigegeben sind.

## Dokumente

| Datei | Inhalt |
|-------|--------|
| [docs/01-systemueberblick.md](docs/01-systemueberblick.md) | Gesamtarchitektur, Komponenten, Datenflüsse |
| [docs/02-eichrecht.md](docs/02-eichrecht.md) | WELMEC 7.2, MID, Software-Trennung, Logbuch |
| [docs/03-hardware.md](docs/03-hardware.md) | CM4-Carrier, Schnittstellen, Peripherie, Schutzart |
| [docs/04-software-architektur.md](docs/04-software-architektur.md) | Services, Persistenz, APIs |
| [docs/05-features.md](docs/05-features.md) | Fachliche Funktionen der Wiegesoftware |
| [docs/06-rhewa-protokoll.md](docs/06-rhewa-protokoll.md) | Anbindung Rhewa 84 Vario, Telegramme |
| [docs/07-visualisierung.md](docs/07-visualisierung.md) | UI/UX, Bildschirme, Bedienung |
| [docs/08-betrieb-deployment.md](docs/08-betrieb-deployment.md) | Inbetriebnahme, Update, Backup, Monitoring |
| [docs/09-roadmap.md](docs/09-roadmap.md) | Meilensteine, Risiken, offene Fragen |
| [docs/10-feature-backlog.md](docs/10-feature-backlog.md) | App-Backlog: Epics, Stories, Akzeptanzkriterien, MoSCoW |
| [docs/11-prerequisites.md](docs/11-prerequisites.md) | Prerequisites-Checkliste (Eichrecht, HW, OS, Daten, Build, Test) |

## Kurzfassung

- **Messwert-Quelle:** Rhewa 84 Vario (eichamtlich zugelassenes Wägeterminal)
- **Embedded-Server:** Raspberry Pi Compute Module 4 auf Industrie-Carrier
- **Rolle des Servers:** *peripheres Anzeige- und Datenverarbeitungsgerät*
  nach WELMEC 7.2 Klasse L bzw. M – die metrologische Verantwortung verbleibt
  beim Rhewa 84 V.
- **Software-Trennung:** Legaler Teil (Telegramm-Empfang, Anzeige,
  Wägescheindruck, Logbuch) ist signiert, geprüft und versioniert getrennt
  vom nicht-legalen Teil (Disposition, Auswertung, Web-UI, Schnittstellen).
- **Visualisierung:** Lokales Touch-Display (HDMI/DSI) plus Web-UI für
  Waagmeister-Arbeitsplatz, Fahrer-Selbstbedienung und Auswertung.

## Projektwebsite

Eine anschauliche Single-Page-Site mit Architektur, Mockups und Roadmap
liegt unter [`site/index.html`](site/index.html). Die Seite ist
selbstgenügsam (HTML + CSS + minimal JS, keine Build-Tools) und kann
direkt im Browser geöffnet oder über jeden statischen Webserver
ausgeliefert werden:

```bash
python3 -m http.server --directory site 8080
# → http://localhost:8080
```

## Branch

Entwicklung erfolgt auf `claude/plan-weighing-display-module-hCZWb`.
