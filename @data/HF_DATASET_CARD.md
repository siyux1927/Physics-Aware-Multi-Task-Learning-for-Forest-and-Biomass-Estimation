---
license: cc-by-4.0
task_categories:
- image-segmentation
- image-to-image
tags:
- SAR
- remote-sensing
- forest
- biomass
- segmentation
- regression
- sentinel-1
pretty_name: I-MAESTRO SAR Forest Patches
size_categories:
- 100M<n<1B
---

# I-MAESTRO SAR Forest Patches

## Overview

This dataset contains processed Sentinel-1 SAR imagery and forest structural attributes derived from the I-MAESTRO synthetic forest inventory, covering three European sites. It was prepared for multi-task learning experiments on joint forest genus segmentation, canopy height regression, and aboveground biomass regression.

## Source

**Forest inventory:** I-MAESTRO synthetic dataset — tree-level inventory data for three European forest landscapes (Bauges/France, Milicz/Poland, Sneznik/Slovenia), covering ~100,000 ha with 42M+ trees from 51 species at 25m grid resolution.

**SAR imagery:** Sentinel-1 GRD (Ground Range Detected) dual-polarization scenes acquired during the 2019 growing season, downloaded from the Alaska Satellite Facility (ASF). Processed with ESA SNAP: radiometric calibration, terrain correction (SRTM DEM), reprojection to local CRS, and resampling to 25m.

**Biomass targets** were estimated from tree-level data using species-specific allometric equations. Canopy height uses the 95th percentile height per cell. Dominant genus was determined by tree count per cell.

## Dataset Structure

```
training-patches/
  patches_{train|val|test}.npy          # (N, 5, 64, 64) float32
  labels_dominant_genus_{split}.npy     # (N,) int — dominant genus code
  sites_{split}.npy                     # (N,) str — site name

rasters/
  {bauges|milicz|sneznik}/
    {site}_vh.tif                       # VH backscatter (dB)
    {site}_vv.tif                       # VV backscatter (dB)
    {site}_biomass_t_ha_smooth.tif      # Biomass (t/ha)
    {site}_height95_smooth.tif          # Canopy height 95th pct (m)
    {site}_dom_genus_smooth.tif         # Dominant genus (integer code)

auxiliary/
  genus_map.json                        # genus name → integer code
  wood_density_allometric.json          # allometric parameters per species
  split-blocks/
    {train|val|test}_{site}_blocks.geojson  # spatial block boundaries
```

## Patch Format

Each `.npy` patch array has shape `(N, 5, 64, 64)` with channels in fixed order:

| Channel | Variable | Unit |
|---------|----------|------|
| 0 | VH backscatter | dB |
| 1 | VV backscatter | dB |
| 2 | Biomass (smoothed) | t/ha |
| 3 | Dominant genus (smoothed) | integer code |
| 4 | Canopy height 95th pct (smoothed) | m |

Channels 0-1 are model inputs; channels 2-4 are prediction targets. Invalid pixels are `NaN`.

**Splits:** Train 724 patches / Val 145 / Test 109. Patches were extracted with a 50% sliding window overlap within spatially blocked regions (4×4 patch blocks, ~3.2×3.2 km each) to prevent spatial autocorrelation between splits.

## Usage

```python
import numpy as np

patches = np.load("training-patches/patches_train.npy")   # (724, 5, 64, 64)
labels  = np.load("training-patches/labels_dominant_genus_train.npy")
sites   = np.load("training-patches/sites_train.npy")

# SAR inputs
X = patches[:, :2, :, :]   # VH, VV

# Regression targets
biomass = patches[:, 2, :, :]
genus   = patches[:, 3, :, :]
height  = patches[:, 4, :, :]
```

Genus integer codes are defined in `auxiliary/genus_map.json`. The six dominant genera used in training are: Abies (1), Fagus (11), Fraxinus (13), Picea (19), Pinus (20), Quercus (25).

## Sites

| Site | Country | CRS | Forest area | Dominant genera |
|------|---------|-----|-------------|-----------------|
| Bauges | France | EPSG:2154 | ~45,000 ha | Fagus, Abies, Pinus |
| Milicz | Poland | EPSG:2180 | ~20,000 ha | Pinus, Quercus |
| Sneznik | Slovenia | EPSG:3912 | ~5,300 ha | Fagus, Abies |

## License

Dataset released under CC-BY 4.0. SAR data sourced from Copernicus Sentinel-1 (ESA, open access). I-MAESTRO inventory data used for research purposes.
