# radioland-logos

Fan-submitted and manually curated station logos for the RadioLand app
(w2njl.com / RadioLand iOS + Android).

## Layout

```
logos/
  FM/<CALLSIGN>_<FREQUENCY>.jpg
  AM/<CALLSIGN>_<FREQUENCY>.jpg
  WX/<CALLSIGN>_<FREQUENCY>.jpg
```

Frequencies are the FCC-authorized frequency (102.9 not 102.90). Callsigns
include any `-LP` / `-FM` suffix exactly as stored in the RadioLand DB
(`callsign` column of `fcc_database` for FM and `us_am_database` for AM).

## Serving

GitHub Pages serves this repo at
`https://w2njl.github.io/radioland-logos/<path>`.

Entries are registered in `radiolanddb.station_logos` with `is_radiodns=TRUE`,
so the API's resolver (`resolve_station_logo()` in `myscript.py`) short-
circuits straight to the Pages URL instead of falling back to the FCC
publicfiles placeholder.

## Adding a logo

1. Drop the image at `logos/<BAND>/<CALLSIGN>_<FREQ>.jpg`
2. Commit + push to `main`
3. Wait ~1 min for Pages to redeploy
4. Insert the DB row:

```sql
INSERT INTO station_logos (callsign, frequency, broadcast_band, country, logo_url, is_radiodns)
VALUES ('<CALLSIGN>', <FREQ>, '<BAND>', 'US',
        'https://w2njl.github.io/radioland-logos/logos/<BAND>/<CALLSIGN>_<FREQ>.jpg',
        TRUE)
ON CONFLICT (callsign, frequency, broadcast_band) DO UPDATE
  SET logo_url = EXCLUDED.logo_url, is_radiodns = TRUE, updated_at = CURRENT_TIMESTAMP;
```

## Image preparation

The app renders logos in a square thumbnail with BoxFit.cover — any wide or
tall source will get its sides/top cropped. Pad the image to **1:1 aspect
ratio** (white background, centered) before committing. One-liner:

```python
from PIL import Image
img = Image.open(src); w,h = img.size; s = max(w,h)
canvas = Image.new("RGB", (s,s), (255,255,255))
canvas.paste(img, ((s-w)//2, (s-h)//2))
canvas.save(dst, quality=92, optimize=True)
```
