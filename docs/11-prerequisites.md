# 11 – Prerequisites

Voraussetzungen, die **vor** Beginn der Implementierung bzw. einzelner
Features geklärt, beschafft oder dokumentiert sein müssen. Status:
✅ vorhanden / 🟡 teilweise / ❌ offen / 📌 zu klären.

## 11.1 Eichrechtliche Voraussetzungen

| # | Prerequisite | Warum nötig | Status |
|---|--------------|-------------|--------|
| L1 | Eichbescheid / Baumusterprüfbescheinigung des Rhewa 84 V (Kopie) | Definiert zulässige Telegramme, Wägebereich, Teilung | 📌 vom Kunden anfordern |
| L2 | Klassifizierung des Moduls (Mitanzeige vs. Kontrollanzeige) | Bestimmt, ob eigene Baumusterprüfung nötig | ❌ offen, M0 |
| L3 | Frühgespräch mit Eichbehörde / benannter Stelle (PTB, Landeseichdir.) | Frühe Risikominimierung | ❌ offen, M0 |
| L4 | Aufbewahrungsfristen DSAR & Wägescheine festgelegt | Datenmodell, Backup-Konzept | 📌 mit Kunde + Steuerberater |
| L5 | Plombierungs-Konzept dokumentiert (Stelle, Material, Verantwortlicher) | Hardware-Bauform, Service-Prozess | ❌ offen |
| L6 | Signaturschlüssel (Ed25519) für DSAR + Update-Bundles | Kein Code ohne Schlüsselzeremonie | ❌ offen, vor M2 |
| L7 | Software-Identifikations-Schema (Name, Version, Hash, ID) | WELMEC 7.2 P3 | ❌ offen, vor M2 |

## 11.2 Hardware-Voraussetzungen

| # | Prerequisite | Notiz | Status |
|---|--------------|-------|--------|
| H1 | Raspberry Pi CM4 4 GB / 32 GB eMMC + Industrie-Carrier | Lieferzeit prüfen, BOM einfrieren | ❌ Bemusterung in M1 |
| H2 | Touch-Panel(s) – Innenraum 15.6", Außensäule 12" | Hersteller, Schutzklasse, Lesbarkeit Sonne | ❌ offen |
| H3 | Drucker (A4 + Thermo-Beleg) inkl. Treiber | CUPS-Kompatibilität verifizieren | ❌ offen |
| H4 | Galvanisch isolierter RS232/RS485-Wandler zum Rhewa | EMV, Blitzschutz | ❌ offen |
| H5 | DC-USV (LiFePO4) mit DI-Powerfail-Signal | Sauberes Shutdown | ❌ offen |
| H6 | NVMe (Industrial, hohe TBW) + zweiter Datenträger USB-SSD | DSAR-Mirror | ❌ offen |
| H7 | TPM 2.0 auf Carrier oder steckbares Modul | Schlüsselablage | ❌ offen |
| H8 | RFID-/Barcode-Reader (Modell festlegen) | Treiber, Kommunikation | ❌ offen |
| H9 | Feldgeräte: Ampel, Schranke, Lichtschranke, ggf. Kamera | Pflicht/optional je Standort | 📌 standortabhängig |
| H10 | Schaltschrank IP54 mit plombierbarem Innenfach | Mechanik, Beschriftung | ❌ offen |

## 11.3 Information & Daten vom Kunden

| # | Prerequisite | Notiz | Status |
|---|--------------|-------|--------|
| D1 | Bestand Rhewa 84 V (Seriennummer, Firmware, Eichbescheid-Stand) | Telegramm-Variante ableiten | 📌 |
| D2 | Tatsächlich am Rhewa geschaltete Telegramm-Variante | Parser-Auswahl | 📌 Sniffer in M1 |
| D3 | Liste aller Schnittstellen / ERP-Systeme | Konnektor-Aufwand | 📌 |
| D4 | Stammdaten-Initialbestand (CSV: Fahrzeuge, Kunden, Materialien) | Migration, Importer | 📌 |
| D5 | Druckvorlagen-Anforderungen (Logo, Pflichtfelder, Layout) | Template-Hash | 📌 |
| D6 | Rollenmodell &amp; Benutzerliste, Auth-Anbindung (lokal/OIDC) | Sicherheitskonzept | 📌 |
| D7 | Sprachen (DE/EN/PL/CZ/...) und Schichtmodelle | i18n, Reports | 📌 |
| D8 | Unternehmensdaten (Firmenname, Adresse, USt-ID, Logo) | Wägescheinkopf, E-Rechnung | 📌 |

## 11.4 Software-Plattform / OS

| # | Prerequisite | Notiz | Status |
|---|--------------|-------|--------|
| S1 | Auswahl Basis-OS (Raspberry Pi OS Lite vs. Yocto-Custom) | Updatequalität vs. Reproduzierbarkeit | ❌ Entscheidung in M1 |
| S2 | Kernel mit dm-verity, overlayfs, AppArmor/SELinux | Schutz legaler Bereich | ❌ verifizieren |
| S3 | systemd ≥ 254 (cgroups v2, Hardening-Direktiven) | Service-Isolation | ❌ verifizieren |
| S4 | Secure Boot Strategie für CM4 (signed bootloader) | Manipulationsschutz | ❌ offen |
| S5 | Container-Runtime Podman + rootless Setup | nicht-legaler Teil | ❌ offen |
| S6 | PostgreSQL 16 + Backup-Tool (pg_basebackup, pgBackRest) | Fachdaten | ❌ offen |
| S7 | TLS-CA-Konzept (interne CA, Zertifikatsverteilung) | Web-UI, REST | ❌ offen |
| S8 | NTP/PPS-Quelle (z. B. GPS-Modul oder verlässlicher LAN-NTP) | Zeitstempel DSAR | ❌ offen |
| S9 | Wireguard-Server für Fernwartung | Service-Zugriff | ❌ offen |

## 11.5 Entwicklungsumgebung

Dieser Stand bezieht sich auf die aktuelle Container-Umgebung:

| Werkzeug | Anforderung | Status hier |
|----------|-------------|-------------|
| git | Quellcodeverwaltung | ✅ vorhanden |
| Python 3.11+ | API-Backend (FastAPI), Skripte | ✅ 3.11.15 |
| Node 20+ | Frontend-Build | ✅ Node 22 |
| Rust 1.75+ | Legaler Kern, Tools | ✅ vorhanden (cargo) |
| GCC / CMake | C/C++-Komponenten | ✅ vorhanden |
| Docker | Build &amp; Test-Umgebung | ✅ vorhanden |
| psql Client | DB-Tooling | ✅ vorhanden |
| Podman | Runtime auf Zielsystem | ❌ noch zu installieren |
| sqlite3 CLI | DSAR-Inspektion | ❌ noch zu installieren |
| Qt 6 / qmake6 | Legal-Display | ❌ noch zu installieren |
| protoc + protobuf-libs | IPC-Schemas | ❌ noch zu installieren |
| OpenSSL / libsodium | Signaturen, TLS | ❌ verifizieren |
| Cross-Compile Toolchain ARM64 | Zielsystem CM4 | ❌ noch zu installieren |

Empfehlung: Reproduzierbare Dev-Container bereitstellen (`devcontainer.json`
oder `Containerfile`) mit allen oben markierten Tools, damit jeder
Entwickler/Servicetechniker bit-identisch baut.

## 11.6 Build- &amp; Release-Pipeline

| # | Prerequisite | Notiz | Status |
|---|--------------|-------|--------|
| B1 | CI-System (GitHub Actions, GitLab CI o. ä.) | Tests, Bundles | ❌ offen |
| B2 | Artefakt-Repository (für `legal-bundle.tar.zst.sig`, App-Container) | Auslieferung | ❌ offen |
| B3 | Reproducible-Build-Setup (fixe Toolchain, gepinnte Abhängigkeiten) | WELMEC-Audit | ❌ offen |
| B4 | Code-Signing-HSM oder TPM-basierte Schlüsselablage | Schlüssel nie auf Build-Host | ❌ offen |
| B5 | Versionierungsschema (SemVer + Build-Hash) | UI-Anzeige Eichdaten | ❌ offen |
| B6 | Hardware-in-the-Loop-Prüfstand (Rhewa + CM4 + Display) | Vor jedem Legal-Release | ❌ offen, M2 |

## 11.7 Test &amp; Qualitätssicherung

| # | Prerequisite | Notiz | Status |
|---|--------------|-------|--------|
| Q1 | Telegramm-Simulator (`rhewa-sim`) lauffähig | Unit/Integrationstests | ❌ in M2 |
| Q2 | Test-Datenbasis (Fixtures: Fahrzeuge, Aufträge, Telegramme) | Repro-Tests | ❌ |
| Q3 | UI-Smoke-Tests (Playwright) für Web-UI | Regression Backoffice | ❌ |
| Q4 | Lasttest-Skripte (Telegramm-Strom, parallele Wägungen) | Performance | ❌ |
| Q5 | Statische Analyse (clippy, mypy, eslint) + Coverage | Qualität | ❌ |
| Q6 | Eichrechtliche Testfälle (BCC-Fehler, Stillstand, Zeitlücken) | Konformität | ❌, vor M5 |

## 11.8 Betrieb &amp; Organisation

| # | Prerequisite | Status |
|---|--------------|--------|
| O1 | Servicekonzept (1st/2nd Level, Reaktionszeiten) | ❌ |
| O2 | Schulungskonzept Waagmeister, Servicetechniker | ❌ |
| O3 | Notfallhandbuch (DSAR-Hashbruch, Stromausfall, Datenverlust) | ❌ |
| O4 | Datenschutzerklärung (RFID-Daten, Kennzeichen-OCR, Kameras) | ❌ |
| O5 | Versicherung / Haftung (Eichrechtsverstoß, Datenverlust) | ❌ |
| O6 | EDI-/Schnittstellenverträge mit Kunden / Disposition | 📌 |

## 11.9 Zusammenfassung – Gating für die Meilensteine

- **M0 abschließbar**, sobald L1, L2, L3, D1, D2 vorliegen.
- **M1 (HW-Prototyp)** braucht zusätzlich H1, H4, H10, S1.
- **M2 (Legaler Kern)** erfordert L6, L7, S2, S4, B1, B3, B4, Q1.
- **M3 (Fachapplikation)** braucht D3–D8, S5, S6, S7.
- **M4 (Selbstbedienung &amp; Integration)** braucht H8, H9, D3.
- **M5 (Eichung &amp; Pilot)** setzt L1–L7 vollständig voraus + Q6.

Diese Tabelle ist die Master-Checkliste; Status wird in jedem
Wochen-Review aktualisiert.
