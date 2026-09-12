# US Corn & Soybean — In-Season Area Forecast

Static portal: weekly, calibrated corn and soybean planted-equivalent acreage for the US
Corn Belt by state and county, with a 120 m class map and a USDA June *Acreage* benchmark.
Runs entirely on GitHub Pages — no server, no object store, no runtime dependency on the
processing cluster. The US twin of the UK winter-wheat portal (same viewer shell, same
publish flow), built from what the US product has today: classification and area.
Yield and production panels are reserved for the US yield model now in training; the
field-level layer follows the Delineate-Anything segmentation of the belt.

## What is here

| path | what | ships |
|---|---|---|
| `index.html` | viewer (MapLibre GL + PMTiles) | once |
| `lib/` | vendored maplibre-gl 5.6.0 + pmtiles (no CDN) | once |
| `admin/admin_state.geojson`, `admin/admin_county.geojson` | Census cartographic boundaries, simplified | once |
| `tiles/class_<week>.pmtiles` | 120 m corn / soybean class map, PNG tiles z5–z10 | per week (~30 MB) |
| `manifest.json` | weeks, belt KPIs, state + county statistics, USDA comparison, method text | per week |

Numbers are **planted-equivalent acres**: calibrated expected area (Σ P(class) × pixel
area on the 120 m belt mosaic), Olofsson error-adjusted with the classifier's own error
matrix at this lead time (95% interval), then converted to survey planted acres with the
CDL→NASS factor. Counties sum to states; states sum to the belt.

## Budget

The 30 m display COGs of the source product are 0.3–3.8 GB per layer per week; even a
z12 class pyramid would be ~0.4 GB/week against GitHub Pages' 1 GB site cap and 100 MB
per-file limit. The 120 m mosaic — the very raster the acreage is computed from — tiles to
~30 MB/week at z10, so a full season fits. Use `--pixel-weeks latest:N` in the pack
builder if the season runs long.

## Deploying

1. Create a repo and push this directory to the default branch.
2. **Settings → Pages → Source: Deploy from a branch**, pick the branch and `/ (root)`.
3. Rename `CNAME.example` to `CNAME` with your subdomain (e.g. `corn.rayx.co.uk`), add the
   DNS `CNAME` record to `<user>.github.io`, then tick **Enforce HTTPS**.

`.nojekyll` is present so Pages serves the files verbatim.

## Regenerating

```bash
sbatch jobs/us_portal_pack.sbatch            # builds into /gws/ssde/j25a/nceo_isp/us_portal_pack
# or by hand (SLURM, not the login node):
python scripts/build_us_portal_pack.py --out /gws/ssde/j25a/nceo_isp/us_portal_pack --weeks auto
```

Publishing to the public site is `scripts/publish_us_portal_snapshot.sh` (one orphan
snapshot commit, same pattern as the UK portal).

## Caveats carried from the source product

- Weeks classified during Planetary Computer outages are missing (2026-07-20) or, when
  tiles are missing, carry `n_tiles_partial` / `degraded` counts — shown under the map.
- The class map is 120 m and shows corn and soybean only; "other" is transparent.
- USDA comparison uses the June *Acreage* planted acres for states inside the model domain;
  states only partly covered by the tile set are included as covered.
