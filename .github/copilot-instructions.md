You are helping me develop an interactive UHF repeater propagation analysis tool as a single-file HTML application.

## Current State
A working single-file HTML app (`uhf_propagation_map.html`) with:
- Leaflet.js map, default center 30°26′7″N 98°0′45″W (Texas Hill Country) — location is user-editable
- **Terrain-adjusted coverage polygons** (not flat circles) computed from real SRTM elevation data
  - 36 bearings × 8 elevation samples per bearing via Open-Elevation API (POST), with automatic
    fallback to Open-Meteo elevation API
  - Per-bearing LOS limits calculated with K=4/3 effective-Earth curvature correction
  - Four tiers: Excellent ~15 mi (green), Good ~25 mi (cyan), Marginal ~40 mi (amber), LOS Max ~55 mi (red)
- **Map loading overlay** — blurred dark overlay with spinner shown while terrain is being fetched;
  dismissed on completion; re-shown on every recalculation trigger
- Layer toggle: Topo (OpenTopoMap), OSM, Satellite (ESRI)
- Dark industrial UI theme using "Share Tech Mono" and "Barlow Condensed" fonts
- **Draggable repeater marker** — dragging it updates lat/lon inputs and triggers terrain recompute
- Footer status bar (site coords and power update live)

## Sidebar Panels (in order)
1. **// Repeater Location** — editable lat/lon decimal inputs + Site Elev. (ft) input (blank = auto-fetch);
   drag-to-update or ↻ APPLY LOCATION button; resets elevation placeholder on location change
2. **// Repeater Parameters** — editable inputs: TX Power (W), Frequency (MHz), Ant. Gain (dBd),
   Cable Loss (dB), Rx Sensitivity (dBm), Ant. Height (ft); live EIRP readout; ↻ RECALCULATE button
3. **// Link Budget** — EIRP, mobile ant., body/cable loss, Rx sensitivity, system margin + progress bar;
   all values update live on Recalculate
4. **// Coverage Rings** — static legend swatches for the four tiers
5. **// HAAT Factors** — Avg. Terrain, HAAT estimate, Radio Horizon, K-factor
6. **// Terrain Analysis** — source, resolution, Min/Avg/Max LOS (mi), Status indicator
7. Layer toggle note box

## Tech Stack
- Single HTML file (no build step)
- Leaflet.js 1.9.4 via CDN (cdnjs.cloudflare.com)
- Google Fonts via CDN
- Vanilla JS only — no frameworks
- CSS variables for theming (dark palette: --bg, --surface, --border, --accent, --accent2, --accent3)

## Design Language
- Dark industrial/tactical aesthetic
- CSS vars: --bg:#0a0d0f, --surface:#111518, --accent:#00e5ff (cyan), --accent2:#ff6b00, --accent3:#39ff14
- Monospace readouts for all RF values (`Share Tech Mono`)
- Input fields: dark background, 1px border, monospace, right-aligned; `.param-input` class
- Apply/action buttons: `.apply-btn` class — full-width, cyan border, 11px mono, letter-spacing
- No rounded cards, no gradients on UI surfaces — flat/sharp borders only

## Key JS Architecture
- `LAT` / `LON` — mutable globals updated by drag or Apply Location
- `circleLayers[]` — tracks all polygon/circle map layers; cleared and repopulated on every recompute
- `labelGroup` — Leaflet LayerGroup for compass rose distance labels; redrawn via `drawLabels()`
- `repMarker` — draggable Leaflet marker; rebuilt via `buildRepeaterMarker()`
- `computeTerrainCoverage()` — async; shows overlay, fetches elevations, computes LOS polygons, hides overlay
- `showOverlay(msg)` / `hideOverlay()` / `setOverlayMsg(text)` — overlay state helpers
- `calcLinkBudget()` — reads inputs, computes EIRP/margin, updates all link budget DOM elements
- `updateHeaderFooter()` — syncs header coords/power and footer site/power spans

## RF Formulas in Use
- Free-space path loss: FSPL(dB) = 20·log10(d_km) + 20·log10(f_MHz) + 32.44
- Radio horizon: d_km = 4.12·√(h_m)  (effective Earth, K=4/3)
- EIRP: EIRP(dBm) = P_tx(dBm) + G_ant(dBi) - L_cable(dB)
- Earth bulge: h(m) = d·(D−d) / (2·Re_eff) where Re_eff = 8495 km
- Fresnel zone r1: r1(m) = 17.3·√(d1·d2 / (f_GHz · d_total))  (not yet implemented)

## Elevation API Strategy
1. Primary: `POST https://api.open-elevation.com/api/v1/lookup` — batch all points in one request,
   25 s timeout
2. Fallback: `GET https://api.open-meteo.com/v1/elevation` — batched in chunks of ≤100 points,
   15 s timeout per chunk
- Site elevation can be manually overridden via the Site Elev. (ft) input; blank = auto-fetch
- On location change (drag or Apply), elevation input placeholder resets to "AUTO"

## File Structure (single file sections)
1. `<head>` — fonts, Leaflet CSS, inline `<style>` (includes `.param-input`, `.apply-btn`, spinner keyframes)
2. `<body>` — header bar, main flex (map + sidebar), footer bar, map loading overlay div
3. Inline `<script>` — map init, layer toggles, terrain analysis, marker setup, editable params, event handlers

## Planned Enhancements (not yet implemented)
1. **Fresnel zone calculator** — click a map point, draw line from repeater, show 1st Fresnel zone
   radius at midpoint for 440 MHz in a popup
2. **Terrain profile chart** — SVG elevation profile between repeater and a clicked point; mark LOS obstructions
3. **Asymmetric coverage overlay** — bearing/range override table in sidebar to reshape polygons
4. **CTCSS/DCS tone display** — tone/code, offset direction, output frequency, copy-to-clipboard
5. **Export KMZ** — JSZip-based in-browser KMZ download of coverage polygons + repeater placemark

## Constraints
- Must remain a single HTML file
- No npm, no build tooling
- All dependencies via CDN only (cdnjs.cloudflare.com preferred)
- Keep the existing dark theme and font choices — do not introduce new visual languages
- Mobile-responsive layout is a stretch goal, not primary

When making changes, preserve all existing functionality unless explicitly told to replace it.
Prefix all new sidebar panels with the `// PANEL_NAME` style title convention already in use.