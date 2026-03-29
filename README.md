## Notebooks

### 1. `MaskingWorkflow.ipynb` — Climate Region Masking

Filters NARCliM climate model data by shapefile region boundaries (Margaret River wine subregions) and performs kriging interpolation within each region.

**What it does:**
- Loads a SWWA wine-region shapefile, reprojects to WGS84, and strips Z coordinates
- Assigns official subregion names via a hardcoded index map
- Loads a NARCliM 4km rotated-pole NetCDF grid
- Creates boolean region masks using a dual buffer strategy — a fixed buffer applied to every region, plus an auto-buffer fallback for regions that still capture zero grid cells
- Converts Kelvin → °C at plotting/statistics time using `is_kelvin()` / `to_celsius()` helpers
- Interactive widgets for buffer tuning, time series exploration, and spatial mapping
- Kriging interpolation per region: fine-grid upscaling, gap-fill, and centroid surface
- Saves masks as NetCDF and NumPy, and exports regional statistics and time series to CSV

**Key dependencies:** `geopandas`, `xarray`, `shapely`, `pykrige`, `scipy`, `ipywidgets`, `matplotlib`

---

### 2. `point_vs_neighbourhood_extraction_new_.ipynb` — Point vs Neighbourhood Extraction

Extracts climate data from NARCliM2.0 rotated WRF grids at multiple sites using two methods: single nearest grid cell, and spatially averaged neighbourhood.

**What it does:**
- Supports multiple sites simultaneously via a `SITES` list (edit in the configuration cell)
- Finds the nearest grid cell using the **Haversine formula** for accurate great-circle distance on the rotated grid
- Defines the neighbourhood by a **radius in kilometres**, not grid indices
- Creates a land mask from Natural Earth 50m coastlines to exclude ocean cells from neighbourhood averages
- Applies cosine-latitude area weighting to account for varying grid cell size
- Adaptive radius — automatically expands if fewer than `MIN_LAND_CELLS` land cells are found
- Outputs a summary table, per-site time series plots, detailed statistics, and a combined CSV export

**Key dependencies:** `xarray`, `numpy`, `pandas`, `shapely`, `cartopy`, `matplotlib`

---

### 3. `new05_basic_climate_analysis.ipynb` — Basic Climate Analysis (Western Australia)

A complete, memory-efficient workflow for DJF climatology, climate change signal, and anomaly time series analysis over Western Australia using NARCliM2.0 data streamed via OPeNDAP/THREDDS.

**What it does:**
- Streams data from the NCI THREDDS server — only DJF months are loaded, reducing download volume by ~75%
- Computes 30-year DJF reference climatology (1980–2009) and future climatology (2070–2099)
- Calculates the climate change signal (future minus reference)
- Generates a WA-averaged DJF anomaly time series from 1951–2100 with 10-year smoothing
- Computes all-season (DJF/MAM/JJA/SON) climatology maps over the reference period
- Regrids outputs from the native rotated WRF grid to a regular 0.05° lat/lon grid using **xESMF bilinear regridding** for GIS export
- Exports results as NetCDF files and a Perth DJF time series

**Key dependencies:** `xarray`, `xesmf`, `numpy`, `pandas`, `cartopy`, `matplotlib`

---

## Requirements

```bash
pip install numpy pandas xarray geopandas shapely cartopy matplotlib pykrige scipy ipywidgets xesmf
```

> **Note:** `pykrige` is used for kriging in `MaskingWorkflow.ipynb` with automatic fallback to `scipy.interpolate.RBFInterpolator` if not available. `xesmf` is required only for `new05_basic_climate_analysis.ipynb`.

A shared utility module `climate_utils.py` is required by all notebooks — ensure it is in the same directory. It provides the `kelvin_to_celsius()` helper used across the workflow.

---

## Data

All notebooks connect to **NARCliM2.0** data hosted on the NCI THREDDS server via OPeNDAP:

```
https://dapds00.nci.org.au/thredds/dodsC/zz63/NARCliM2-0/output-CMIP6/DD
```

`MaskingWorkflow.ipynb` reads local NetCDF files. Update the `NETCDF_FILES` paths in the configuration cell to point to your local data.

---

## Configuration

Each notebook has a single **Configuration cell** at the top. To adapt a notebook to a different dataset, variable, region, or time period — edit only that cell. No other cells need to be changed.

---

## Output

| Notebook | Output |
|---|---|
| `MaskingWorkflow.ipynb` | `region_masks/` — NetCDF masks, NumPy masks, statistics CSV, per-region time series CSV |
| `point_vs_neighbourhood_extraction_new_.ipynb` | `output/` — combined CSV with point and neighbourhood values for all sites |
| `new05_basic_climate_analysis.ipynb` | `output/` — regridded NetCDF climatologies, climate change signal, Perth DJF time series |
