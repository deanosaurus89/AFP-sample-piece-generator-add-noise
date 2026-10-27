# AFP Measurement Generator

Static page that builds PolyWorks|DataLoop AFP measurements in the **PW-Measurements 1.2** format. Each part becomes a CSV, and the batch downloads as a ZIP. Only control measured values are written, no geometry (no XYZ/IJK).

The page is a single file (`index.html`, CSS and JavaScript embedded). It runs offline by double-click and on static hosting. No build or npm.

## Use

1. Open `index.html` in a browser, or the GitHub Pages address below. The first visit opens a short tour (Skip or Done remembers that). **Tour** in the header starts it again.
2. **Import CSV** (compact bar at the top): a PolyWorks AFP CSV loads the piece-template UID, the property rows (name plus example value), and the controls. Nominal comes from Measured Value; offset 0, std. dev. 0.1, trend 0, mode Gaussian. UTF-8, UTF-16 LE, and UTF-16 BE (with a BOM, or without one for Latin text). The import replaces the current properties and controls. Add more rows afterwards. A real UTF-16 LE export is at `examples/oede-milkyway-combinator.csv`.
3. **Run** (pinned while scrolling; use Pin to release it): piece base, year, ISO week, workdays (1–7), part count, ZIP file name. The UID field is filled from `PIECE TEMPLATE DATALOOP UID` on CSV import. Trailing digits in the piece base are the first piece number and the zero-padding width: `Piece_000001` with 100 parts is `Piece_000001` … `Piece_000100`. If the number needs more digits than that width, it grows. With no trailing digits, names run `piece_1` … `piece_100`. The CSV file name and the `Piece name` field are that same string. Older configs may still contain `start_index`; it is ignored.
4. **Properties:** name plus a value pool. Comma-separated entries are picked at random per part. A single value with no comma stays fixed. An empty pool stays empty.
5. **Controls:** object, control, nominal, offset, std. dev., trend start/end, mode, optional Only >0. Add rows or delete the selection. The eye opens the SPC preview for that control only. In the same card, **Noise for all / selection** writes offset, std. dev., trend, mode, and Only >0 onto every control or only the checked rows. **Randomize** fills offset, std. dev., trend start, trend end, and mode at random. Only >0 is left unchanged. Each number stays inside the min/max next to the buttons. Defaults: offset −0.5 to 0.5, std. dev. 0.05 to 0.5, trend −0.3 to 0.3.
6. **SPC preview:** opened from a control’s eye. Measured values across the part count, with lines for nominal, mean (nominal+offset), and trend. Reroll draws the random noise again. The chart uses the row values; there is no second set of offset/trend fields.
7. **Config** (collapsed): saves every input as JSON (`messdaten-config.json`) and loads it again, from a file or pasted JSON. Includes `noise_ranges` when you save from this version. Older configs without a mode still load. A saved `rules` list is ignored; property-to-control offsets are not applied. Missing `noise_ranges` leave the current ranges.
8. **CSV preview** (collapsed): **Preview first CSV** builds the first piece and shows it in the text area. It does not open a save dialog and does not create a ZIP. **Copy first CSV** copies that text.
9. **Generate CSVs (ZIP).** Chrome and Edge open a save dialog; other browsers download the file. That still writes the full batch and also refreshes the first-CSV preview.

Parts are placed across the first N days of the chosen ISO week (Monday 00:00 UTC, even spacing, small jitter). N is the Days field. Timestamps are UTC, with no time zone in `Date`.

Measured-value formula:

`nominal + offset + noise(mode, std. dev.) + linear trend`

Modes: **Gaussian** (normal), **Sine** (one period across all parts, amplitude = std. dev.), **Zigzag** (alternating +std. dev. and −std. dev.), **Outliers** (mostly Gaussian, about 5% of parts at 4–6× std. dev.). Only >0 lifts non-positive values to a small positive random value (0.001 to 0.05). The SPC preview clamps Only >0 with a stable +0.01.

## GitHub Pages

The source is the **root** of branch **`main`** (folder `/`, not `/docs`). `index.html` lives on `main`.

https://deanosaurus89.github.io/AFP-sample-piece-generator-add-noise/

Someone with admin rights turns Pages on once: **Settings → Pages → Build and deployment → Source: Deploy from a branch → Branch `main`, Folder `/ (root)` → Save.** After that, every push to `main` publishes the page. The Pages API could not be switched with the available token (`pages: write` is missing, HTTP 403).

## CSV schema

Written CSVs: UTF-8, comma separator, CRLF line endings, decimal point. The file name is the piece name plus `.csv`. With the default base that is `piece_1.csv`, `piece_2.csv`, and so on. A base of `Piece_000001` keeps six digits until the counter outgrows them. Import reads those files and also UTF-16 LE/BE, as PolyWorks writes them.

```
TYPE,PW-Measurements
VERSION,1.2
PIECE TEMPLATE DATALOOP UID,<uid>
[UNTRANSLATABLE PROPERTIES]
Piece name,piece_1
Date,<YYYY-MM-DD HH:MM:SS>
[Properties]
<property name>,<value>
[Controls]
Object Name,Control Name,Control Description,Measured Value,Note
<object>,<control>,,<measured value>,
```

`Control Description` and `Note` stay empty. The section is named `[UNTRANSLATABLE PROPERTIES]` exactly. A field that contains a comma, quote, or line break is quoted per CSV rules.

CSV import takes `PIECE TEMPLATE DATALOOP UID`, every row under `[Properties]` (name and value; Piece name and Date from `[UNTRANSLATABLE PROPERTIES]` stay out), and under `[Controls]` the object name, control name, and Measured Value as the nominal. The page’s properties and controls are replaced. Offset, std. dev., trend, and mode start at 0 / 0.1 / 0 / Gaussian.

Config JSON uses `uid`, `piece_base`, `year`, `week`, `num_days`, `num_parts`, `zip_name`, `properties` (pairs), `controls` (object, control, nominal, offset, std. dev., trend start, trend end, `"True"`/`"False"`, mode), and `noise_ranges` (`offset`, `std`, `trend_start`, `trend_end`, each a min/max pair). A saved `start_index` or `rules` list from an older file is ignored; the counter is the trailing digits of `piece_base`, and property-to-control offsets are not applied.
