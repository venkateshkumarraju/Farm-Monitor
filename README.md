# Farm Monitor

Farm Monitor is a browser-based GIS workspace that helps farmers inspect fields, compare satellite imagery, and turn vegetation and moisture indices into practical field-visit and irrigation decisions.

## Features

- Upload a zipped shapefile (`.shp`, `.shx`, `.dbf`, and `.prj`) or GeoJSON boundary.
- Draw a field polygon directly on the map and zoom to its boundary.
- Use WGS 84 (`EPSG:4326`) field coordinates and satellite, topographic, or street base maps.
- Search open Sentinel-2 L2A imagery through the Element84 Earth Search STAC API.
- Automatically retrieve scenes and build an NDVI/NDMI season trend from the 10 newest available acquisitions.
- Check for new imagery every 10 days while the app is open.
- Visualize true-color imagery, NDVI, NDMI, NDWI, SAVI, BSI, NDRE, and EVI.
- Upload georeferenced drone or Cartosat-2 GeoTIFF files, inspect band count/metadata, and adjust band mappings.
- Save the field, monitoring date, and season history in browser `localStorage`.
- Read plain-language recommendations and the research/implementation roadmap in the side panel.

> **Important:** Satellite and drone indices support field decisions; they do not replace soil laboratory tests, agronomic advice, or ground verification. NDRE and bare-soil indicators are proxies, not direct soil-fertility measurements. Cartosat-2 imagery is processed only when the user supplies a lawfully obtained GeoTIFF; the app does not download Cartosat-2 products automatically.

## Requirements

- Node.js **20.19.0 or newer** (Node.js 22 LTS recommended)
- npm **10.0.0 or newer**
- A modern browser with internet access for map tiles, STAC searches, and Sentinel-2 Cloud Optimized GeoTIFF reads

There are no Python dependencies. `requirements.txt` records the runtime requirements, while exact JavaScript dependency versions are locked in `package-lock.json`.

## Automatic setup

### Windows PowerShell

```powershell
.\setup.ps1
```

### macOS / Linux / Git Bash

```sh
chmod +x ./setup.sh
./setup.sh
```

### Cross-platform alternative

```sh
node scripts/setup.mjs
```

The setup script verifies the Node.js version, runs `npm ci`, executes lint and unit tests, builds the production app, and audits production dependencies.

## Manual installation

```sh
npm ci
npm run dev
```

Open the local URL printed by Vite. To create and preview a production build:

```sh
npm run build
npm run preview
```

The deployable static site is generated in `dist/`.

## Verification commands

```sh
npm run lint          # ESLint
npm test              # Unit tests for geometry, updates, and band detection
npm run build         # TypeScript and Vite production build
npm run check         # All three checks above
npm audit --omit=dev  # Production dependency security audit
```

## GitHub setup

The repository should include source files, `package.json`, `package-lock.json`, setup scripts, tests, and documentation. Do not commit `node_modules/`, `dist/`, local environment files, or recordings.

```sh
git init
git add .
git commit -m "Initial Farm Monitor release"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git
git push -u origin main
```

A GitHub Actions workflow in `.github/workflows/ci.yml` runs the same lint, tests, and build on every push and pull request.

## Project structure

```text
src/
  components/       Leaflet map and reusable UI controls
  hooks/            Monitoring-state persistence
  lib/              STAC, geometry, indices, decisions, and drone imagery
  types/            Shared TypeScript models
  App.tsx           Main GIS workspace
scripts/setup.mjs   Clean-install and verification automation
tests/              Node unit tests (run through tsx)
.github/workflows/  Continuous integration
```

## Data and service notes

- **Sentinel-2:** Element84 Earth Search v1, `sentinel-2-l2a` collection, public AWS COGs.
- **Base maps:** Esri World Imagery, OpenStreetMap, and OpenTopoMap; their respective attribution is shown on the map.
- **Fonts:** Manrope and Space Grotesk through Google Fonts.
- **Storage:** Field state is local to the current browser. Clearing browser data removes it; there is no account sync or server database.
- **Scheduling:** The 10-day refresh runs when the application is open and an update is due. It is not a server-side background scheduler.
- **Availability:** External imagery services can be unavailable or rate-limited. Use the manual imagery check to retry after a failure.

## Optimization and safety behavior

- Heavy GIS libraries are split into separate production chunks.
- Only required UI components and dependencies are included.
- Satellite bands for one index are read concurrently; NDVI and NDMI share their NIR raster within each scene.
- Superseded STAC searches are aborted, and failed automatic searches do not retry continuously.
- Field changes cancel trend-state updates and clear incompatible cached layers.
- Production rasters are windowed and resampled to bounded grids before analysis.
