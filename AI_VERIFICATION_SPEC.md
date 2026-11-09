# Hyatt Commercial — OM Aerial & Location Intelligence Engine
**Full Nuts-to-Bolts Technical Specification & Independent AI Audit Document**

---

## 1. Copy-Paste Technical Specification (For Another AI to Verify)

Copy everything inside the block below and paste it into any AI (ChatGPT, Claude, Gemini, Grok) along with `app.js` if you want an independent verification of what you built:

```markdown
# INDEPENDENT AI ARCHITECTURAL AUDIT REQUEST
Please review and verify the architecture, mathematical algorithms, spatial GIS pipelines, 3D rendering engine, and CRE copywriting guardrails of the "Hyatt Commercial — Automated Offering Memorandum (OM) Aerial & Location Intelligence Engine" described below.

## I. SYSTEM OVERVIEW & CORE PURPOSE
We built a zero-build, pure client-side Commercial Real Estate (CRE) Location Intelligence & Automated Offering Memorandum (OM) Web Application (`index.html`, `styles.css`, `app.js` — ~5,110 lines of vanilla ES2022 JavaScript).
Given ANY commercial street address, the system:
1. Matches and autocompletes addresses as the user types using an **Okapi BM25 + Levenshtein/Prefix Fuzzy Matching Engine** combined with a **Zero-Cost-First Multi-Engine Rooftop Geocoding Pipeline**.
2. Pre-calculates spatial geometry behind a 4-stage animated Pre-Flight Curtain before revealing the maps (preventing UI stutter or half-rendered states).
3. Isolates the **exact subject building footprint** in **Google Earth Photorealistic 3D Tiles (CesiumJS)** with a zero-drift locked rooftop pointer pin and custom `+ / − / 🎯` zoom and 60 FPS orbit controls.
4. Renders a separate **2D/2.5D Hybrid Vector Road & Access Map (MapLibre GL JS)** displaying surveyed building roof boundaries, road-snapped turn-in access badges (`Primary Driveway`, `Frontage Sidewalk`, `Protected Traffic Signal`), MDOT AADT traffic counts, and turn-by-turn OSRM driving routes to nearby anchors.
5. Generates an **Institutional White & Forest-Emerald Offering Memorandum (OM) Brochure** dynamically tailored by buyer profile (General, Senior/Retiree, Young Founder, Medical/Clinical) governed by strict Senior CRE Broker copywriting guardrails (`cre-broker-om-copywriting` skill), with live 3D aerial screenshot capture and **True Vector-Text PDF Export (`jsPDF`)**.

---

## II. NUTS-TO-BOLTS SUBSYSTEM ARCHITECTURE

### 1. Address Search, Okapi BM25 Ranking & Zero-Cost-First Rooftop Geocoding
* **Clean Centered Landing UX (`body.landing-mode` -> `body.calculating-mode`):**
  - Starts with a minimal centered search bar (`#glass-topbar`) and zero pre-populated dropdown suggestions when the input is empty (`q.length === 0`).
  - Once a valid address is selected/submitted, the search bar smoothly transitions to the top command bar via CSS cubic-bezier transitions while `#preflight-calc-card` displays a 4-step progress bar before opening the map stage.
* **Okapi BM25 + Fuzzy Prefix / Levenshtein Matcher (`rankCandidatesWithBM25`):**
  - Tokenizes query and address documents with street-suffix synonym normalization (`rd <-> road`, `dr <-> drive`, `blvd <-> boulevard`, `pkwy <-> parkway`, `hwy <-> highway`, `ctr <-> center`, etc.).
  - Computes corpus Inverse Document Frequency (IDF):
    IDF(t) = ln(1 + (N - df(t) + 0.5) / (df(t) + 0.5))
  - Computes Okapi BM25 document score (k1 = 1.5, b = 0.75) across exact token hits, prefix matches, and 1-edit-distance Levenshtein typo tolerances, plus house-number exact/prefix boosts (+3.4 / +1.8).
* **Zero-Cost-First Rooftop Consensus Pipeline (`resolveExactAddressZeroCostFirst`):**
  - **Tier 0 (`localStorage` Persistent Cache `hyatt_exact_geocode_cache_v2` & `AUTOCOMPLETE_MEMORY_CACHE`):** Repeat lookups resolve in `0ms` at `$0.00` cost.
  - **Tier 1 (Verified Local Commercial Corpus):** High-confidence BM25 matches resolve immediately to surveyed building rooftop centroids (`$0.00` cost).
  - **Tier 2 (100% Free Parallel GIS Consensus — Photon OSM Building Nodes + ArcGIS `PointAddress` Parcels):** Queries `photon.komoot.io/api` (OpenStreetMap `housenumber` building nodes) and `geocode.arcgis.com` (`Addr_type === "PointAddress"` county tax-assessor parcel centroids) in parallel (`$0.00` cost, no API key required).
  - **Tier 3 (Official Google Maps Geocoding & Places API v1 Fallback):** Uses `maps.googleapis.com/maps/api/geocode/json` and `places.googleapis.com/v1/places:autocomplete` only when a Google `placeId` is explicitly clicked or when Free Tiers 1–2 lack an exact `PointAddress` match.
  - **Invalid Address Guardrail:** If all tiers return null, displays inline error (`⚠️ Couldn't find "..." — please type a new address`) without opening broken maps.

---

### 2. Dual-Engine Spatial Rendering (Google 3D Earth vs. Normal Vector Access Map)
* **Mode 1 — `1. 3D Buildings` (CesiumJS + Google Photorealistic 3D Tiles API):**
  - Streams `https://tile.googleapis.com/v1/3dtiles/root.json` into `#cesium-earth-container` with `preserveDrawingBuffer: true`, `maximumScreenSpaceError: 4`, and `dynamicScreenSpaceError: false` (preventing level-of-detail downsampling blur during camera rotation).
  - **Zero-Stall, Zero-Drift Locked Rooftop Pivot (`cesiumCachedPivotCartesian`):**
    - Samples the 3D mesh surface elevation (`cesiumViewer.scene.sampleHeight`) **once** when stationary (`refreshCesiumLockedHeightSample`) and caches the exact `Cesium.Cartesian3` rooftop coordinate in `cesiumCachedPivotCartesian`.
    - Both the `subject-site-pointer` entity (`renderCesiumYellowSiteBorder`) and the 60 FPS orbit loop (`rotateCameraFrame` using `cesiumViewer.camera.lookAt(lockedPivot, new Cesium.HeadingPitchRange(...))`) share the exact same `cesiumCachedPivotCartesian`—resulting in **0 GPU stalls** during animation and **0.0px sideways pin drift**.
    - All yellow polygon border lines are stripped from Google 3D Earth so only the clean 3D building and pointer pin are shown.
    - Includes custom wheel zoom and floating `+ / − / 🎯` controls (`zoomCesiumCameraByFactor`) that dynamically scale `cesiumViewRange` (`35m` to `850m`) around the locked building.
* **Modes 2 & 3 — `2. Access & Traffic Map` & `3. Nearby Places Map` (MapLibre GL JS):**
  - Uses Google Ultra-HD Hybrid tiles (`mt0-mt3.google.com/vt/lyrs=y`) with crisp GeoJSON vector layers:
    - `subject-parcel` & `condo-buildings` (`#facc15` gold 3.5px outline + translucent fill).
    - `real-road-corridors` (`#f43f5e` highways, `#f59e0b` arterial roads).
    - `pedestrian-sidewalk` (`#34d399` emerald dashed frontage line).
    - `osrm-driving-route` (`#0ea5e9` core line + `#38bdf8` glow layer).

---

### 3. Real Road Geometry Snapping & Live Spatial Environment Pipeline (`fetchLiveSpatialEnvironment`)
* **Multi-Mirror Overpass API (`fetchOverpassMultiMirror`) + OSRM Nearest (`router.project-osrm.org/nearest/v1/driving/`):**
  - Queries building footprints (`way["building"]`), road ways (`way["highway"]`), traffic signal nodes (`node["highway"="traffic_signals"]`), and POIs around `[lat, lng]`.
  - **Subject Building Isolation:** Scores building footprints by minimum vertex/interior distance (`minDistanceToBuildingMiles` + ray-casting `isPointInsidePolygon`) and isolates the primary subject structure (`computePolygonAreaSqft` via shoelace formula in local Mercator meters).
  - **100% Real Road-Snapped Access Badges (Zero Fake Offsets):**
    - Projects the building centroid onto the nearest non-motorway street geometry (`realStreetCurbPt`, with OSRM `nearest` waypoint fallback).
    - Places `↱ Primary Driveway` at the curb-cut interpolation point (`0.25 * bldg + 0.75 * curb`), `🚶 Frontage Sidewalk` along the frontage road vector, and `🚦 Signalized Access` at the nearest real OpenStreetMap `highway=traffic_signals` node or street junction vertex.
  - **Nearby Anchors & Real Turn-by-Turn OSRM Drive Times:**
    - Categorizes real nearby Hospitals/Medical Centers, Grocery/Pharmacies, Wine/Dining Hotspots, and Highway Interchanges (using Free Overpass OSM first + Google Places `places:searchNearby` fallback), then computes real road mileage and drive minutes via `router.project-osrm.org/route/v1/driving/`.
    - Clicking any nearby place or access card triggers `focusOnNearbyPlace(poi)` or `focusOnAccessPoint(pt)`, flying the normal map with right-panel padding compensation and dropping an animated `.map-pointout-wrapper` callout pin directly over the target.

---

### 4. CRE Senior Broker OM Copywriting Engine & True Vector PDF Generator
* **Senior Broker Voice & Subtle Persona Tailoring (`buildPersonaPlainEnglishContent` + `cre-broker-om-copywriting` Skill):**
  - Dynamically rewrites the executive summary, 4 KPI metric boxes, and investment highlights for 4 buyer profiles (`general`, `senior`, `young`, `medical`) plus custom natural-language instructions.
  - **Fourth-Wall Regex Guardrail (`cleanAndFormatAiOutput`):** Strips any LLM preambles and blocks fourth-wall-breaking meta-commentary (never prints buyer age or phrases like *"Considering your 45-year-old target audience"*), while automatically bolding key metrics (`<strong>$24–$28/SF</strong>`, `<strong>31,520 AADT</strong>`, `<strong>5 minutes</strong>`).
* **Live Screenshot Capture (`captureAerialToDocument` / `getGoogle3DEarthSnapshotWithYellowBorder`):**
  - Captures live WebGL canvas snapshots (`toDataURL("image/jpeg", 0.95)`) from either Google 3D Earth (`cesiumViewer.canvas`) or the Normal Map (`map.getCanvas()`), automatically refreshing after 3D tiles settle (`1.2s` / `2.4s`) and whenever the user clicks **`📸 Update Photo`** or **`🔄 Switch Angle`**.
* **True Vector-Text PDF Export (`generateTrueVectorPdfDocument` + `drawRichParagraphPdf`):**
  - Uses `jsPDF` to draw crisp, selectable vector typography (`Helvetica` & `Times`), inline bold keyword runs (`<strong>` parser), emerald branding headers, KPI cards, the live 3D aerial photograph, building schedule table, and nearby anchor cards into a single-page Letter PDF.
```
