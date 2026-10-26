# AFP Messdaten-Generator

Statische Seite zum Erzeugen von PolyWorks|DataLoop-AFP-Messdaten im Format **PW-Measurements 1.2**. Pro Teil entsteht eine CSV, der Batch wird als ZIP geladen. Es werden nur Control-Istwerte geschrieben, keine Geometrie (kein XYZ/IJK).

Die Seite ist eine einzelne Datei (`index.html`, CSS und JavaScript eingebettet). Sie läuft offline per Doppelklick und auf statischem Hosting. Ein Build oder npm ist nicht nötig.

## Nutzung

1. `index.html` im Browser öffnen oder die GitHub-Pages-Adresse unten aufrufen.
2. **Lauf:** Piece-Template-UID, Piece-Name-Basis, Startindex, Jahr, ISO-Kalenderwoche, Arbeitstage (1–7), Anzahl Teile, ZIP-Dateiname.
3. **Properties:** Name plus Wertepool. Kommagetrennte Einträge werden pro Teil zufällig gewählt. Ein einzelner Wert ohne Komma bleibt fest. Leerer Pool bleibt leer.
4. **Controls:** Objekt, Control, Nominal, Offset, StdDev, Trend Start/Ende, Modus, optional „Nur >0“. Zeilen lassen sich ergänzen, aus einer bestehenden PW-Measurements-CSV anhängen oder per Auswahl löschen.
5. **Rauschen für alle / Auswahl:** Offset, StdDev, Trend und Modus auf alle Controls oder nur die angehakten Zeilen schreiben.
6. **SPC-Vorschau:** Ist-Verlauf über die Teile, mit Linien für Nominal, Mittel (Nominal+Offset) und Trend. Offset und Trend als Entwurf verschieben und mit „Übernehmen“ in die Auswahl bzw. das aktuelle Control schreiben. „Neu würfeln“ zieht das zufällige Rauschen neu.
7. **Erweitert:** Wenn eine Property beim Teil einen bestimmten Wert hat, kommt ein zusätzlicher Offset auf ein Control. Vorbelegt ist `Device = Crysta Apex 1` → `circle 1 / Diameter` +0,05. Die Zeile löschen, wenn dieser Versatz nicht gewünscht ist.
8. **Config speichern / laden** schreibt die Eingaben als JSON (`messdaten-config.json`).
9. **CSVs erzeugen (ZIP).** In Chrome und Edge öffnet sich der Speichern-Dialog, sonst ein normaler Download.

Die Teile liegen zeitlich auf den ersten N Tagen der gewählten ISO-Kalenderwoche (Montag 00:00 UTC, gleichmäßiger Abstand, kleiner Jitter). N ist das Feld Arbeitstage. Die Zeitstempel sind UTC, ohne Zeitzone im Feld `Date`.

Formel der Ist-Werte:

`Nominal + Offset + Rauschen(Modus, StdDev) + linearer Trend + passende Regel-Offsets`

Modi: **Gauß** (normalverteilt), **Sinus** (eine Periode über alle Teile, Amplitude = StdDev), **Zickzack** (abwechselnd +StdDev und −StdDev), **Ausreißer** (meist Gauß, etwa 5 % der Teile mit 4–6× StdDev). „Nur >0“ hebt nichtpositive Werte auf einen kleinen positiven Zufallswert (0,001 bis 0,05). Die SPC-Vorschau zeigt die Regeln nicht und klemmt „Nur >0“ stabil mit +0,01.

## GitHub Pages

Quelle ist das **Wurzelverzeichnis** des Branches **`main`** (Ordner `/`, nicht `/docs`). `index.html` liegt auf `main`.

https://deanosaurus89.github.io/AFP-sample-piece-generator-add-noise/

Einmalig muss jemand mit Admin-Recht Pages einschalten: **Settings → Pages → Build and deployment → Source: Deploy from a branch → Branch `main`, Folder `/ (root)` → Save.** Danach veröffentlicht jeder Push auf `main` die Seite. Die Pages-API ließ sich mit dem vorhandenen Token nicht schalten (`pages: write` fehlt, HTTP 403).

## CSV-Schema

Kodierung UTF-8, Trenner Komma, Zeilenende CRLF, Dezimalpunkt. Dateiname `<Basis><Index vierstellig>.csv`, mit der Vorbelegung also `piece_0001.csv`, `piece_0002.csv`, …

```
TYPE,PW-Measurements
VERSION,1.2
PIECE TEMPLATE DATALOOP UID,<uid>
[UNTRANSLATABLE PROPERTIES]
Piece name,piece_0001
Date,<YYYY-MM-DD HH:MM:SS>
[Properties]
<Property-Name>,<Wert>
[Controls]
Object Name,Control Name,Control Description,Measured Value,Note
<Objekt>,<Control>,,<Ist-Wert>,
```

`Control Description` und `Note` bleiben leer. Der Abschnitt `[UNTRANSLATABLE PROPERTIES]` heißt im Format genau so. Enthält ein Feld Komma, Anführungszeichen oder Zeilenumbruch, wird es nach CSV-Regeln gequotet.

Der CSV-Import liest ab `[Controls]` die Spalten Object Name, Control Name und Measured Value (als neues Nominal) und hängt die Zeilen an.

Die Config-JSON verwendet `uid`, `piece_base`, `start_index`, `year`, `week`, `num_days`, `num_parts`, `zip_name`, `properties` (Paare), `controls` (Objekt, Control, Nominal, Offset, StdDev, Trend Start, Trend Ende, `"True"`/`"False"`, Modus) und `rules`. Ältere Configs ohne Modus und ohne `rules` laden weiterhin; fehlende `rules` lassen die aktuellen Regeln stehen.
