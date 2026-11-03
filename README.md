# 🛰️ Satellite Over Me

A single-file web app that shows **real satellite imagery and live weather over your exact location** — no API keys, no build step, no backend, no tracking.

Open `index.html`. Allow location. That's it.

```
git clone https://github.com/TanimowoObaloluwaDavid/satellite-over-me
cd satellite-over-me
start index.html     # or just double-click it
```

Works from `file://`. Nothing is uploaded anywhere; the only outbound traffic is map tiles and weather data.

---

## The four tabs

| Tab | What it answers | Freshness |
|---|---|---|
| **Sharp** | What does my street look like from orbit, and **when was this photo taken?** | months – years |
| **Live** | What is happening over my head right now? | **10 minutes** |
| **Daily** | Yesterday's full-country pass | ~1 day |
| **Weather** | Temperature, wind, rain, cloud — plus radar | ~10 minutes |

### Sharp
Loads the highest-detail imagery available (Esri, Bing, or Google), applies an **unsharp mask** to the tile layer, and — the useful bit — reads **Esri's imagery index** to tell you the actual capture date of the pixels you're looking at:

> This photo was taken **2025-04-12** (1.6 yrs ago) · 0.46 m/px · Vantor

Press **Check sharpness here** and it measures, rather than guesses. It downloads the tile at your coordinates at zooms 16–19 from each provider, computes mean gradient magnitude (edge energy) in a canvas, and compares consecutive zooms:

- real detail at z+1 → energy stays roughly flat
- a silently **upscaled** tile at z+1 → energy collapses to ~25%

That ratio is how it distinguishes genuine 0.1 m imagery from a blurred stretched copy, and why the "native detail" readout is trustworthy.

### Live
**EUMETSAT's public GeoServer** (`view.eumetsat.int/geoserver/wms`) serves geostationary imagery with CORS open and no authentication:

| Layer ID | Satellite | Cadence | Resolution |
|---|---|---|---|
| `mtg_fd:rgb_geocolour` | MTG-I1 (0°) | **10 min** | ~1 km |
| `msg_fes:rgb_convection` | Meteosat-11 (0°) | 15 min | 3 km |
| `msg_fes:rgb_microphysics` | Meteosat-11 (0°) | 15 min | 3 km |
| `msg_fes:rgb_fog` | Meteosat-11 (0°) | 15 min | 3 km |
| `copernicus:daily_sentinel3ab_olci_l1_rgb_fulres` | Sentinel-3 | daily | 300 m |
| `eps:m03_rgb_natural_fog` | Metop-C AVHRR | daily | 1 km |

Requested as **256 px tiles**, because EUMETSAT renders each request on demand: 256 px takes 2–8 s and succeeds, 1024 px times out and returns HTTP 500/502. Timestamps are pushed back one full cadence step so the app never asks for a scan that hasn't finished processing.

If their server is overloaded, the app says so instead of failing silently — and **Diagnose** prints the real HTTP status, content type, response size and any `ServiceException` text.

### Daily
NASA GIBS WMTS, cached and instant:

- `VIIRS_SNPP_CorrectedReflectance_TrueColor` — 250 m true colour, one full pass per day
- `MODIS_Aqua_Cloud_Fraction_Day` — daily cloud composite

Zoom is capped at each product's real pixel limit rather than upscaling forever.

### Weather
Open-Meteo current conditions + 10-hour strip, unit toggle, and a RainViewer radar overlay. No key required.

---

## Honest limitations

**There is no real-time, high-resolution satellite view of your house.** That is physics, not software:

- Sharp imagery comes from low-orbit satellites that revisit a given point daily at best, and the pictures are archived, not streamed. Free copies are typically 6–24 months old.
- Genuinely live imagery comes from geostationary weather satellites 36,000 km up: a new frame every 10 minutes at ~1–2 km per pixel. You can watch cloud streets over your city; you cannot see your street.
- Sharpening does not invent detail. It restores edges in pixels that already exist.

Other caveats:

- **Google tiles are an unofficial source.** They are the sharpest widely available imagery, but scraping `mt1.google.com` is not licensed for production use. Esri is the default, intentionally. If you fork this for anything real, drop the Google option and register for a proper provider.
- **Night-time true colour goes black** — it's reflected sunlight. Use Convection RGB after dark.
- **Clouds win in humid climates.** In the Niger Delta or the Gulf Coast, most optical passes are white. Radar (Sentinel-1) is the all-weather answer; this app does not include it yet.
- EUMETSAT's keyless endpoint is fine for personal use. For a product, register at `api.eumetsat.int` and use a token.

---

## Data sources

| Provider | Endpoint | Used for |
|---|---|---|
| Esri World Imagery | `server.arcgisonline.com/.../World_Imagery/MapServer` | base imagery + **capture-date index** |
| Bing Maps tiles | `t0.tiles.virtualearth.net` | alternative base |
| Google tiles | `mt1.google.com/vt` | sharpest base (unofficial) |
| EUMETSAT EUMETView | `view.eumetsat.int/geoserver/wms` | live geostationary + daily polar |
| NASA GIBS | `gibs.earthdata.nasa.gov/wmts/epsg3857/best` | daily VIIRS / MODIS |
| Open-Meteo | `api.open-meteo.com` | weather, no key |
| RainViewer | `api.rainviewer.com` | radar |
| ipapi.co | `ipapi.co` | coarse fallback when geolocation is denied |

Imagery © Esri, Maxar, Earthstar Geographics · © EUMETSAT · NASA EOSDIS GIBS · © OpenStreetMap contributors · weather by Open-Meteo (CC-BY 4.0) · radar by RainViewer.

## Privacy

Geolocation is requested by your browser and never transmitted by this app — it only positions the map locally. Requests go directly from your browser to the tile providers, so those providers see your IP and viewport, exactly as on any map website.

## License

MIT — see [LICENSE](LICENSE).
