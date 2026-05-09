# 06 – Anbindung Rhewa 84 Vario

## 6.1 Ausgangssituation

Rhewa 84 Vario unterstützt mehrere serielle Telegrammtypen. Die exakte
Verfügbarkeit hängt von Bestellcode und Eichbescheid ab. Die folgende
Liste ist ein **Planungsstand** und wird vor Implementierung mit dem
beim Kunden vorhandenen Gerät und dem zugehörigen Datenblatt /
Bedienungsanleitung verifiziert.

Erwartete Telegramm-Modi (laut Rhewa-Familie 82/83/84):
- Kontinuierliche Ausgabe (z. B. „Standard-Output", 10 Hz)
- Auf Anforderung („Print"-Telegramm bei Stillstand)
- Anzeige-Mitlauf für Fern-/Großanzeige
- Optional: Modbus-RTU/TCP, sofern entsprechendes Modul gesteckt ist

## 6.2 Schnittstellenparameter

Voreinstellung im Plan (anzupassen je nach Konfiguration):

- Schnittstelle: RS232 (3-Draht) oder RS485 (2-Draht / 4-Draht)
- Baudrate: 9600 (Standard), konfigurierbar bis 115200
- 8N1, kein Handshake
- Galvanische Trennung am 84V-Server (siehe Hardware Doc 03)

## 6.3 Telegramm-Parsing

`scale-bridge` ist auf das zu verwendende Telegramm konfigurierbar
(Parser-Plugin). Geplante Parser:

1. **Rhewa-Standard kontinuierlich** (ASCII, ETX/Checksumme)
2. **Rhewa-Print** (Anforderungs-Telegramm mit Stillstandsbit)
3. **Generic Continuous (Toledo-kompatibel)** als Fallback
4. **Modbus-Holding-Register** für die Modbus-Variante

Pro Telegramm extrahiert der Parser:

- Bruttowert (Integer + Skalierung) inkl. Vorzeichen
- Einheit (kg/t)
- Status: Stillstand, Null, Brutto/Netto, Über-/Unterlast
- ggf. Tarawert (wenn vom Rhewa geliefert)
- Sequenz-/Print-Nummer (wenn vom Rhewa vergeben)
- Prüfsumme (BCC/CRC)

## 6.4 Validierung

Nach dem Parsen prüft `scale-bridge`:

1. Prüfsumme korrekt → sonst verwerfen + Fehlerzähler erhöhen.
2. Plausibilität: Wert innerhalb des deklarierten Wägebereichs.
3. Zeitlicher Abstand seit letztem Telegramm < Timeout (z. B. 500 ms).
4. Stabiler Telegramm-Typ; Wechsel des Telegramms ohne
   Konfigurationsänderung → Fehlerereignis.

Erst nach erfolgreicher Validierung wird ein signiertes Messwert-Frame
an den `weight-broker` weitergegeben.

## 6.5 Zeitstempelung

- Empfang im Userspace mit `CLOCK_MONOTONIC` + Mapping auf UTC.
- Latenzbudget zwischen Telegramm-Empfang und DSAR-Eintrag: < 200 ms.
- Bei Sprüngen der Systemzeit (NTP, RTC-Sync) → Eintrag im Eventlog;
  laufende Wägungen werden nicht abgebrochen, neue verwenden den neuen
  Zeitstempel.

## 6.6 Anforderungs-/Print-Modus

Für eichpflichtige Wägescheinerzeugung:

1. UI fordert „Wert übernehmen" an.
2. `scale-bridge` sendet (falls verfügbar) Anforderungstelegramm an
   Rhewa und wartet auf Print-Telegramm mit Stillstand.
3. Der erhaltene Wert wird sofort signiert und in DSAR geschrieben.
4. Wägeschein referenziert die DSAR-ID.

Falls der Rhewa keinen expliziten Print-Modus hat, wird der erste
Stillstands-Wert nach Anforderung aus dem kontinuierlichen Stream
übernommen, mit gleichem DSAR-Verfahren.

## 6.7 Diagnose / Simulator

Für Entwicklung und HiL-Test:

- `rhewa-sim` Werkzeug, das auf `/dev/ttyUSBx` oder TCP einen Rhewa
  emuliert (kontinuierliches Telegramm + Print auf Anfrage).
- Konfigurierbar: Wertverlauf (Rampe, Treppe, Rauschen), Stillstand,
  Über-/Unterlast, gestörte Telegramme (BCC-Fehler, halb gesendete
  Frames, Baudratenwechsel).
- Wird für Unit- und Integrationstests verwendet, ist nicht Bestandteil
  des produktiven Auslieferungs-Bundles.

## 6.8 Offene Punkte / Verifikationsbedarf

- Tatsächlich geschaltete Telegramm-Variante des Kundengeräts (im Eich-
  bescheid des Rhewa hinterlegt).
- Genaue ASCII-Frame-Struktur (Start/Stop-Zeichen, Feldlängen).
- Verfügbarkeit eines Print-Telegramms vs. nur kontinuierliche Ausgabe.
- Modbus-Adresskarte des optionalen Modbus-Moduls.
- Eichrechtliche Vorgabe, ob das Print-Telegramm für die DSAR-Ablage
  ausreicht oder ein zusätzliches Display-Mitlauftelegramm gefordert ist.
