# Data Sources & Processing Guide

## Quick Reference for OreNexus Data Pipeline

---

## 📡 Satellite Data Sources

### 1. Copernicus (Sentinel-1 SAR)
**What it does**: Detects ground disturbance using radar (works day/night, through clouds)

- **Access**: [Copernicus Open Access Hub](https://scihub.copernicus.eu/) or [EO Browser](https://apps.sentinel-hub.com/eo-browser/)
- **Product Type**: GRD (Ground Range Detected)
- **Polarization**: VV + VH (dual-polarization)
- **Processing Level**: Radiometric terrain-corrected, decibel gamma0
- **Resolution**: 10m
- **Bands Used**:
  - `VV`: Vertical transmit, Vertical receive (good for ground roughness)
  - `VH`: Vertical transmit, Horizontal receive (sensitive to vegetation)

**File Naming Convention**:
```
2023-01-10-00:00_2023-01-10-23:59_Sentinel-1_AWS-IW-VVVH_VV_-_decibel_gamma0_-_radiometric_terrain_corrected.tiff
2023-01-10-00:00_2023-01-10-23:59_Sentinel-1_AWS-IW-VVVH_VH_-_decibel_gamma0_-_radiometric_terrain_corrected.tiff
```

**Data Location**:
```
data/Sentinel1-SAR/EO_Browser_images/
├── Korba_Coal_AOI1_Jan10/
├── Korba_Coal_AOI1_Jan29/
└── Korba_Coal_AOI1_Feb27/
```

---

### 2. Copernicus (Sentinel-2 Multispectral)
**What it does**: Monitors vegetation health, bare soil, and visual appearance (optical imagery)

- **Access**: [EO Browser](https://apps.sentinel-hub.com/eo-browser/)
- **Product Type**: L2A (Bottom-of-Atmosphere reflectance)
- **Resolution**: 10m (B04, B08), 20m (B11)
- **Bands Used**:
  - `B04` (Red): 665nm - vegetation absorption, soil detection
  - `B08` (NIR): 842nm - vegetation reflection, water bodies
  - `B11` (SWIR): 1610nm - bare soil, moisture content

**File Naming Convention**:
```
2023-01-10-00:00_2023-01-10-23:59_Sentinel-2_L2A_B04_(Raw).tiff
2023-01-10-00:00_2023-01-10-23:59_Sentinel-2_L2A_B08_(Raw).tiff
2023-01-10-00:00_2023-01-10-23:59_Sentinel-2_L2A_B11_(Raw).tiff
```

**Data Location**:
```
data/Sentinel2-Hyperspectral/EO_Browser_images/
├── Korba_Coal_AOI1_Jan10/
├── Korba_Coal_AOI1_Jan30/
└── Korba_Coal_AOI1_RGB/
```

**Important**: Sentinel-2 L2A data has a scaling factor of 10,000. Divide raw pixel values by 10,000 to get reflectance (0-1 range).

---

### 3. Digital Elevation Model (DEM)
**What it does**: Provides terrain height data for volume calculation and 3D visualization

- **Sources**:
  - **Copernicus DEM**: 30m resolution (preferred for recent data)
  - **SRTM**: 30m resolution (2000 baseline, widely available)
- **Access**: [EO Browser](https://apps.sentinel-hub.com/eo-browser/) or [USGS Earth Explorer](https://earthexplorer.usgs.gov/)
- **Format**: GeoTIFF (single-band, elevation in meters)

**Data Location**:
```
data/SRTM-DEM/EO_Browser_images/Korba_Coal_AOI1/
data/SRTM-DEM/Synthetic_Data/
```

---

### 4. AOI Boundaries (Google Earth)
**What it does**: Defines the legal mining area for compliance checking

- **Source**: Google Earth (manual digitization) or government lease documents
- **Format**: KML or GeoJSON
- **Use Case**: 
  - Overlay on satellite imagery
  - Calculate area of mining activity inside vs outside boundaries

**Data Location**:
```
data/GoogleEarth/
├── Korba_Coal_AOI_1.kml
└── Korba_Coal_AOI_1.geojson
```

---

## 🔬 Analysis Workflows

### Workflow 1: Sentinel-1 Ground Disturbance Detection

**Notebook**: `EDTA/playground_sentinel1.ipynb`

**Steps**:
1. Load VV and VH bands for two dates (baseline and analysis date)
2. Apply 5x5 median filter to reduce speckle noise
3. Calculate temporal difference: `vv_diff = vv_date2 - vv_date1`
4. Apply thresholds:
   - VV increase > 4-9 dB → Ground disturbance
   - VH decrease < -3 to -8 dB → Vegetation loss
5. Combine both conditions for high-confidence hotspots
6. (Optional) Apply object-based segmentation (SLIC) for cleaner results

**Key Variables**:
- `vv_db_filtered_jan10`, `vv_db_filtered_feb27`: Filtered backscatter images
- `vv_diff`: Change map (positive = increase in backscatter)
- `change_hotspots_s1`: Boolean mask of detected mining activity

**Outputs**:
- Change detection maps (red = mining activity)
- Area statistics (hectares)
- Pixel coordinates of hotspots

---

### Workflow 2: Sentinel-2 Vegetation & Bare Soil Monitoring

**Notebook**: `EDTA/playground_sentinel2.ipynb`

**Steps**:
1. Load B04 (Red), B08 (NIR), B11 (SWIR) for two dates
2. Scale pixel values: `band_value / 10000`
3. Calculate indices:
   - **NDVI** = (B08 - B04) / (B08 + B04)
   - **BSI (approx)** = (B11 + B04 - B08) / (B11 + B04 + B08)
4. Calculate temporal difference:
   - `ndvi_diff = ndvi_date2 - ndvi_date1`
   - `bsi_diff = bsi_date2 - bsi_date1`
5. Apply thresholds:
   - BSI increase > 0.2 → New mining/clearing
   - NDVI increase > 0.15 → Vegetation regrowth
6. Classify pixels into categories (mining, regrowth, stable)

**Key Variables**:
- `ndvi_jan10`, `ndvi_jan30`: NDVI maps
- `bsi_jan10`, `bsi_jan30`: Bare soil index maps
- `ndvi_diff`, `bsi_diff`: Change maps
- `classified_change`: Classified land cover change (1=mining, 2=regrowth)

**Outputs**:
- False-color composites (visual interpretation)
- NDVI/BSI change maps
- Mining hotspot overlays
- Area statistics

---

### Workflow 3: Cross-Sensor Fusion (SAR + Optical)

**Method**: Object-Based Change Detection

**Steps**:
1. Run Workflows 1 and 2 independently
2. Apply SLIC segmentation to create image objects (5000 segments)
3. Calculate per-object statistics:
   - Mean VV, VH for each object
   - Mean NDVI, BSI for each object
4. Classify objects where:
   - VV increase > threshold AND
   - VH decrease < threshold AND
   - BSI increase > threshold
5. Map the final high-confidence objects

**Advantages**:
- Reduces false positives from pixel-level noise
- Better spatial coherence
- More reliable for reporting

---

## 📊 Index Interpretation Guide

### NDVI (Normalized Difference Vegetation Index)
| Value Range | Interpretation |
|-------------|----------------|
| < 0 | Water, bare rock, roads |
| 0 - 0.2 | Bare soil, sand, urban areas, **mining sites** |
| 0.2 - 0.5 | Sparse vegetation, grassland |
| 0.5 - 0.7 | Moderate vegetation, crops |
| 0.7 - 1.0 | Dense vegetation, forests |

**For Mining Detection**: Look for areas with NDVI < 0.2 or significant NDVI decrease over time.

---

### BSI (Bare Soil Index)
| Value Range | Interpretation |
|-------------|----------------|
| < -0.5 | Dense vegetation |
| -0.5 - 0 | Mixed vegetation and soil |
| 0 - 0.5 | Exposed soil, **mining sites** |
| > 0.5 | Very bare soil, sand |

**For Mining Detection**: Look for BSI > 0 or BSI increase > 0.2 over time.

---

### VV Backscatter (Sentinel-1)
| Change (dB) | Interpretation |
|-------------|----------------|
| < -3 | Decreased roughness (flooding, smoothing) |
| -3 to +3 | Stable, no significant change |
| +3 to +6 | Moderate roughness increase (possible activity) |
| > +6 | Strong roughness increase (**mining activity, construction**) |

**For Mining Detection**: Look for VV increase > 4-9 dB.

---

### VH Backscatter (Sentinel-1)
| Change (dB) | Interpretation |
|-------------|----------------|
| < -5 | Significant vegetation loss (**clearing, mining**) |
| -5 to -2 | Moderate vegetation decrease |
| -2 to +2 | Stable vegetation |
| > +2 | Vegetation increase (regrowth) |

**For Mining Detection**: Look for VH decrease < -3 to -8 dB.

---

## 🛠️ Data Processing Tips

### Handling Outliers
- Set pixels with value 0 or 65535 to NaN before processing
- Use percentile-based contrast stretching for visualization (2nd - 98th percentile)

### Reducing Noise
- Apply median filter (5x5 or 7x7) to SAR data
- Use object-based segmentation instead of pixel-level classification
- Normalize change maps by subtracting scene-wide mean

### Choosing Thresholds
- Start with conservative thresholds (fewer false positives)
- Validate with visual inspection of known mining sites
- Adjust based on local conditions (vegetation type, soil type)

### Temporal Considerations
- Use image pairs from the same season to avoid phenological changes
- Minimum time gap: 2 weeks (for Sentinel-1, due to revisit time)
- Optimal time gap: 1-2 months (balance between change detection and data availability)

---

## 📦 Data Download Instructions

### For Sentinel-1 (EO Browser)
1. Go to [EO Browser](https://apps.sentinel-hub.com/eo-browser/)
2. Select **Sentinel-1 GRD IW** as data source
3. Draw your AOI or upload KML
4. Set date range
5. Select **VV dB Gamma0** and **VH dB Gamma0** layers
6. Click "Download" → Export as GeoTIFF
7. Repeat for baseline and analysis dates

### For Sentinel-2 (EO Browser)
1. Go to [EO Browser](https://apps.sentinel-hub.com/eo-browser/)
2. Select **Sentinel-2 L2A** as data source
3. Draw your AOI or upload KML
4. Set date range
5. Use **Analytical** mode and download **B04, B08, B11** individually
6. Export as GeoTIFF
7. Repeat for multiple dates

### For DEM (EO Browser)
1. Go to [EO Browser](https://apps.sentinel-hub.com/eo-browser/)
2. Select **Copernicus DEM** as data source
3. Draw your AOI
4. Download as GeoTIFF (single-band elevation)

---

## 🔍 Quality Control Checklist

Before running analysis:
- [ ] Check that all images cover the same spatial extent
- [ ] Verify that images are from the same season (avoid agricultural cycles)
- [ ] Ensure cloud cover < 10% for optical imagery
- [ ] Confirm that Sentinel-1 data is terrain-corrected
- [ ] Validate that Sentinel-2 data is L2A (atmospherically corrected)
- [ ] Check that DEM resolution matches imagery resolution
- [ ] Verify AOI boundary file loads correctly in GIS software

---

## 📚 Further Reading

- [Sentinel-1 SAR User Guide](https://sentinels.copernicus.eu/web/sentinel/user-guides/sentinel-1-sar)
- [Sentinel-2 MSI User Guide](https://sentinels.copernicus.eu/web/sentinel/user-guides/sentinel-2-msi)
- [Copernicus DEM Documentation](https://spacedata.copernicus.eu/collections/copernicus-digital-elevation-model)
- [SAR Handbook by NASA](https://servirglobal.net/Global/Articles/Article/2674/sar-handbook-comprehensive-methodologies-for-forest-monitoring-and-biomass-estimation)
