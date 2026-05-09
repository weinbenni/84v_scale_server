# 02 – Eichrechtliche Anforderungen

## 2.1 Geltungsbereich

Fahrzeugwaagen für den geschäftlichen Verkehr unterliegen in Deutschland
dem **Mess- und Eichgesetz (MessEG)** und der **Mess- und Eichverordnung
(MessEV)**, EU-weit der **Measuring Instruments Directive (MID, 2014/32/EU,
Anhang V – NAWI)** sowie der OIML R 76. Für Software gilt **WELMEC 7.2**.

Da der Wert vom bereits zugelassenen **Rhewa 84 Vario** stammt, ist das
hier geplante System ein **peripheres Gerät** im Sinne von WELMEC 7.2.
Maßgeblich sind insbesondere die Anforderungsklassen:

- **Klasse L** (Long-term storage of measurement data) – relevant, weil
  Wägescheindaten dauerhaft gespeichert und wiederverwendet werden.
- **Klasse T** (Transmission of measurement data via network) – relevant,
  da Messwerte über Netz an Anzeige/Drucker/ERP gehen.
- **Klasse D** (Download of legally relevant software).
- ggf. Klasse S (Software separation).

## 2.2 Software-Trennung (WELMEC 7.2 Kap. 5)

Strenge Aufteilung in **legalen** und **nicht-legalen** Softwareteil:

| Bereich | Legal (eichgesichert) | Nicht-legal |
|---------|----------------------|-------------|
| Telegramm-Empfang vom Rhewa | ✔ | – |
| Checksum / BCC-Prüfung | ✔ | – |
| Anzeige Bruttogewicht + Status | ✔ | – |
| Eichpflichtiges Logbuch (DSAR) | ✔ | – |
| Wägescheindruck (gewichtsführende Felder) | ✔ | – |
| Stammdaten (Fahrzeug, Material, Kunde) | – | ✔ |
| Disposition / Auftrag | – | ✔ |
| Web-UI Backoffice | – | ✔ |
| ERP-Schnittstelle | – (Werte nur unverändert weiterreichen) | ✔ (Steuerdaten) |
| Auswertung, Reports, Export | – | ✔ |

Trennung wird durchgesetzt durch:

- Eigene Prozesse (`scale-bridge`, `weight-broker`, `legal-display`) in
  einem getrennten Linux-User mit eigenen Schreibrechten.
- Read-only signiertes Image-Subvolume (`/opt/legal`) mit den eichrechtlich
  geprüften Binaries und der genehmigten Konfiguration.
- Kommunikation legal ↔ nicht-legal nur über eine **definierte, protokolliert
  schmalbandige Schnittstelle** (Unix-Domain-Socket + Protobuf), die
  Messwerte ausschließlich liefert, nie entgegennimmt.

## 2.3 Identifikation der legalen Software

Anforderung WELMEC 7.2 P3:

- Fest hinterlegte **Software-Identifikation** (Name + Version + Hash)
  für jeden legalen Bestandteil.
- Aufruf über UI-Funktion „Info / Eichdaten" und über CLI
  `scalectl legal-info`.
- Anzeige enthält:
  - Bezeichnung, Versionsnummer, Build-Datum
  - SHA-256-Hash des signierten Pakets
  - Public Key Fingerprint des Signaturschlüssels
  - Zulassungsnummer / Baumusterprüfbescheinigung (sofern vorhanden)
  - Identifikation des angeschlossenen Rhewa (Seriennr., FW-Version
    aus Telegramm sofern verfügbar)

## 2.4 Eichpflichtiges Logbuch (DSAR)

DSAR = *Device-Specific Audit Record*. Anforderung Klasse L.

- Append-only, integer indexiert, nicht löschbar während der
  Aufbewahrungsfrist (mind. 3 Monate, Empfehlung 12 Monate).
- Pro Wägung mind.:
  - laufende Nummer
  - Zeitstempel (UTC, monoton, gegen NTP-Sprünge geschützt)
  - Bruttowert + Einheit + Stillstandsflag + Vorzeichen
  - Quelle (Rhewa-Seriennr.)
  - Operator-ID / Fahrzeug-ID (sofern erfasst)
  - Hash der vorigen Eintrags (Hash-Chain)
  - Signatur über den Datensatz (Ed25519)
- Speicherung redundant: lokales SQLite mit WAL **und** rotierende
  Append-Only-Logdatei auf zweitem Datenträger.
- Anzeige & Druck immer mit eindeutiger DSAR-Nummer.

## 2.5 Schutz vor Veränderung

- Boot mit *secure boot* / signiertem Bootloader (CM4 EEPROM,
  `rpi-eeprom-config` + signed boot).
- `/opt/legal` als **dm-verity** Read-only Volume, Hash-Tree im Bootimage
  signiert.
- Konfiguration des legalen Teils in einem signierten Bundle, Manipulation
  führt beim Start zum Sperren der Wägefunktion und Eintrag im
  Ereignislogbuch.
- W&M-Schalter (siehe Hardware): solange „verriegelt", können
  metrologisch relevante Parameter nicht geändert werden – Änderungen sind
  nur mit gebrochener Plombe und Eintrag im Ereignislogbuch möglich.

## 2.6 Ereignis-Logbuch (Event Log)

Getrennt vom DSAR, ebenfalls append-only:

- Boot/Shutdown
- Versionswechsel legaler Software
- Konfigurationsänderungen (Schnittstellenparameter, Druckvorlagen-Hash
  der gewichtsführenden Bereiche)
- Brüche der Verbindung zum Rhewa
- W&M-Schalter Statuswechsel
- gescheiterte Telegramm-Plausibilität (CRC, Wertebereich, Stillstand)

Ereignis-Logbuch fasst min. die letzten 1000 Einträge bzw. die letzten
2 Jahre, je nachdem, was länger ist.

## 2.7 Datenübertragung (Klasse T)

- Messwerte zwischen `scale-bridge` und Anzeige/Drucker werden mit
  Ed25519 signiert und mit *fresh-nonce* versehen → kein Replay.
- Druck eines Wägescheins ist nur erlaubt, wenn die Signatur der
  Messwerte gültig ist und der Stillstand zum Zeitpunkt der Erfassung
  bestätigt war.
- Über Netz (REST/MQTT) ausgegebene Werte werden zusätzlich mit DSAR-ID
  versehen, sodass jeder externe Empfänger den eichrechtlichen Eintrag
  zurückverfolgen kann.

## 2.8 Software-Updates (Klasse D)

- Updates des legalen Teils ausschließlich als signiertes Bundle.
- Vor Aktivierung: Hash-Verifikation, Schreiben eines Ereignisses, Anzeige
  Versionswechsel.
- Updates des nicht-legalen Teils dürfen den legalen Teil nicht
  beeinträchtigen → eigene Paket-Pipeline, eigener Restart.
- „Field Update" nur durch geschulte Personen, dokumentiert im
  Konformitätsbewertungsverfahren.

## 2.9 Konformitätsbewertung

Empfohlen wird der Pfad:

1. Risikobewertung & Zuordnung WELMEC-Klassen (intern).
2. Baumusterprüfung des kombinierten Systems (Rhewa 84 V + Modul) durch
   benannte Stelle (z. B. PTB, Eichdirektion), sofern eine eigene
   eichamtliche Mitanzeige beworben werden soll.
3. Andernfalls: Betrieb des Moduls nur als „Kontrollanzeige" – die
   eichrechtliche Anzeige bleibt das Rhewa 84 V, das Modul muss dann
   nicht baumustergeprüft werden, **aber** die Wägescheine dürfen den
   Wert nur unverändert vom Rhewa übernehmen und müssen darauf verweisen.

> Hinweis: Die endgültige Klassifizierung ist mit der zuständigen
> Eichbehörde / benannten Stelle abzustimmen. Dieses Dokument ersetzt
> keine eichrechtliche Bewertung.
