# AFP Measurement Generator

Static page that builds PolyWorks|DataLoop AFP measurements in the **PW-Measurements 1.2** format. Each part becomes a CSV, and the batch downloads as a ZIP. Only control measured values are written, no geometry (no XYZ/IJK).

The page is a single file (`index.html`, CSS and JavaScript embedded). It runs offline by double-click and on static hosting. No build or npm.

## Use

1. Open `index.html` in a browser, or the GitHub Pages address below.
2. **Import CSV** (compact bar at the top): a PolyWorks AFP CSV loads the piece-template UID, the property rows (name plus example value), and the controls. Nominal comes from Measured Value; offset 0, std. dev. 0.1, trend 0, mode Gaussian. UTF-8, UTF-16 LE, and UTF-16 BE (with a BOM, or without one for Latin text). The import replaces the current properties and controls. Add more rows afterwards. A real UTF-16 LE export is at `examples/oede-milkyway-combinator.csv`.
3. **Run** (pinned while scrolling; use Pin to release it): piece-name base, start index, year, ISO week, workdays (1–7), part count, ZIP file name. The UID field is filled from `PIECE TEMPLATE DATALOOP UID` on CSV import.
4. **Properties:** name plus a value pool. Comma-separated entries are picked at random per part. A single value with no comma stays fixed. An empty pool stays empty.
5. **Controls:** object, control, nominal, offset, std. dev., trend start/end, mode, optional Only >0. Add rows or delete the selection. The eye opens the SPC preview for that control only.
6. **Noise for all / selection:** write offset, std. dev., trend, mode, and Only >0 onto every control or only the checked rows.
7. **Randomize:** fill offset, std. dev., trend start, trend end, and mode at random for all controls or the selection. Only >0 is left unchanged. Each number stays inside the min/max next to the buttons. Defaults: offset −0.5 to 0.5, std. dev. 0.05 to 0.5, trend −0.3 to 0.3.
8. **SPC preview:** opened from a control’s eye. Measured values across the part count, with lines for nominal, mean (nominal+offset), and trend. Reroll draws the random noise again. The chart uses the row values; there is no second set of offset/trend fields.
9. **Advanced:** when a property on a part has a given value, an extra offset is added to a control. Preset: `Device = Crysta Apex 1` → `circle 1 / Diameter` +0.05. Delete the row if you do not want that shift.
10. **Config** (collapsed): saves every input as JSON (`messdaten-config.json`) and loads it again, from a file or pasted JSON. Includes `noise_ranges` when you save from this version. Older configs without a mode or without `rules` still load; missing `rules` leave the current rules, missing `noise_ranges` leave the current ranges.
11. **Generate CSVs (ZIP).** Chrome and Edge open a save dialog; other browsers download the file.

Parts are placed across the first N days of the chosen ISO week (Monday 00:00 UTC, even spacing, small jitter). N is the Days field. Timestamps are UTC, with no time zone in `Date`.

Measured-value formula:

`nominal + offset + noise(mode, std. dev.) + linear trend + matching rule offsets`

Modes: **Gaussian** (normal), **Sine** (one period across all parts, amplitude = std. dev.), **Zigzag** (alternating +std. dev. and −std. dev.), **Outliers** (mostly Gaussian, about 5% of parts at 4–6× std. dev.). Only >0 lifts non-positive values to a small positive random value (0.001 to 0.05). The SPC preview does not apply the rules and clamps Only >0 with a stable +0.01.

## GitHub Pages

The source is the **root** of branch **`main`** (folder `/`, not `/docs`). `index.html` lives on `main`.

https://deanosaurus89.github.io/AFP-sample-piece-generator-add-noise/

Someone with admin rights turns Pages on once: **Settings → Pages → Build and deployment → Source: Deploy from a branch → Branch `main`, Folder `/ (root)` → Save.** After that, every push to `main` publishes the page. The Pages API could not be switched with the available token (`pages: write` is missing, HTTP 403).

## CSV schema

Written CSVs: UTF-8, comma separator, CRLF line endings, decimal point. File name `<base><index padded to 4>.csv`, so with the default base `piece_0001.csv`, `piece_0002.csv`, and so on. Import reads those files and also UTF-16 LE/BE, as PolyWorks writes them.

```
TYPE,PW-Measurements
VERSION,1.2
PIECE TEMPLATE DATALOOP UID,<uid>
[UNTRANSLATABLE PROPERTIES]
Piece name,piece_0001
Date,<YYYY-MM-DD HH:MM:SS>
[Properties]
<property name>,<value>
[Controls]
Object Name,Control Name,Control Description,Measured Value,Note
<object>,<control>,,<measured value>,
```

`Control Description` and `Note` stay empty. The section is named `[UNTRANSLATABLE PROPERTIES]` exactly. A field that contains a comma, quote, or line break is quoted per CSV rules.

CSV import takes `PIECE TEMPLATE DATALOOP UID`, every row under `[Properties]` (name and value; Piece name and Date from `[UNTRANSLATABLE PROPERTIES]` stay out), and under `[Controls]` the object name, control name, and Measured Value as the nominal. The page’s properties and controls are replaced. Offset, std. dev., trend, and mode start at 0 / 0.1 / 0 / Gaussian.

Config JSON uses `uid`, `piece_base`, `start_index`, `year`, `week`, `num_days`, `num_parts`, `zip_name`, `properties` (pairs), `controls` (object, control, nominal, offset, std. dev., trend start, trend end, `"True"`/`"False"`, mode), `rules`, and `noise_ranges` (`offset`, `std`, `trend_start`, `trend_end`, each a min/max pair).
