# OreNexus - Technical Documentation

## Overview

OreNexus is an automated mining activity monitoring system that combines satellite imagery analysis, geospatial data processing, and AI/ML techniques to detect, monitor, and report on open-crust mining operations. The system focuses on compliance monitoring, environmental impact assessment, and volumetric analysis of mining sites.

---

## Project Status: **Proof of Concept Phase**

### ✅ Completed Components

#### 1. **Data Infrastructure**
- **Satellite Data Sources**:
  - **Copernicus/EO Browser**: Sentinel-1 (SAR) and Sentinel-2 (Multispectral) imagery
  - **Google Earth**: AOI (Area of Interest) boundary data in KML/GeoJSON format
  - **SRTM/Copernicus DEM**: Digital Elevation Models for terrain analysis

- **Test Data Coverage**:
  - Primary AOI: Korba Coal Mining Area, Chhattisgarh
  - Temporal Range: January - February 2023
  - Data Types: VV/VH polarization (SAR), RGB + NIR bands (Optical), 30m DEM

#### 2. **Frontend Web Application** (`app/`)
- **Framework**: Vanilla HTML/CSS/JavaScript with modular architecture
- **Mapping**: Leaflet.js-based interactive 2D mapping interface
- **Key Features**:
  - AOI upload and boundary visualization
  - Date/time picker for temporal analysis
  - Satellite source selection
  - Mining site management panel
  - Layer control for different data overlays
  - Analysis tools integration
  - 2D/3D view toggle (3D in development)

- **Structure**:
  ```
  app/
  ├── index.html              # Main 2D interface
  ├── 3d_view.html           # 3D terrain viewer (WIP)
  ├── css/                   # Modular stylesheets
  │   ├── base.css
  │   ├── components.css
  │   ├── layout.css
  │   └── themes.css
  └── js/                    # Modular JavaScript
      ├── config.js
      ├── main.js
      ├── core/
      ├── data/
      ├── modules/
      └── utils/
  ```

#### 3. **Exploratory Data Analysis** (`EDTA/`)

Two Jupyter notebooks demonstrate satellite data processing pipelines:

##### **Sentinel-1 SAR Analysis** (`playground_sentinel1.ipynb`)
- **Purpose**: Detect ground disturbance and vegetation changes using radar backscatter
- **Methods Implemented**:
  - Speckle noise reduction (median filtering)
  - VV/VH backscatter temporal differencing
  - Radar Vegetation Index (RVI) calculation
  - Ground disturbance detection via VV increase
  - Vegetation loss detection via VH decrease
  - Object-based change detection using SLIC segmentation
  - High-confidence hotspot mapping (combined VV+VH thresholds)

- **Key Findings**:
  - Successfully detected mining activity changes between Jan 10 - Feb 27, 2023
  - Object-based analysis reduces false positives compared to pixel-based methods
  - Dual-polarization analysis (VV+VH) provides more reliable change detection

##### **Sentinel-2 Optical Analysis** (`playground_sentinel2.ipynb`)
- **Purpose**: Monitor vegetation health and bare soil exposure
- **Methods Implemented**:
  - Band loading and preprocessing (B04-Red, B08-NIR, B11-SWIR)
  - NDVI (Normalized Difference Vegetation Index) calculation
  - BSI (Bare Soil Index) approximation
  - BAI (Burn Area Index) for disturbed land
  - Temporal change detection (Jan 10 - Jan 30, 2023)
  - False-color composite generation
  - Mining/clearing vs vegetation regrowth classification

- **Key Findings**:
  - BSI increase > 0.2 strongly correlates with new mining activity
  - NDVI decrease confirms vegetation loss in active mining zones
  - Combined NDVI+BSI thresholds effectively classify land cover changes

#### 4. **Report Generation System** (`report_generation/`)

- **Current Implementation**: PDF report generator using matplotlib
- **Report Types**:
  - Economic reports (revenue, costs, production metrics)
  - Environmental impact reports (ecological degradation, compliance)
- **Features**:
  - Multi-page PDF generation with charts and statistics
  - Time-series visualizations
  - Compliance flag summaries
  - Spatial metrics (area, volume, depth)

#### 5. **Demo Visualizations** (`visualizations_demo/`)

- **Synthetic Dataset Generator**: Creates realistic mining site data for testing
- **Data Types**:
  - Entity master data (mine locations, ownership, status)
  - Temporal data (monthly extraction volumes, workforce)
  - Elevation profiles (depth measurements)
  - District statistics
  - Compliance flags

- **Visualization Examples**:
  - Bar charts, line plots, heatmaps
  - Geospatial overlays
  - Compliance scorecards

---

## Data Processing Pipeline

### Current Workflow

```
1. Data Acquisition
   ├── Copernicus Open Access Hub → Sentinel-1/2 imagery
   ├── EO Browser → Processed TIFF exports
   └── Google Earth → AOI boundaries (KML/GeoJSON)

2. Preprocessing (Jupyter Notebooks)
   ├── Sentinel-1: Speckle filtering, dB conversion
   ├── Sentinel-2: Band scaling, index calculation
   └── DEM: Elevation data extraction

3. Analysis
   ├── Change Detection: Temporal differencing
   ├── Segmentation: SLIC object-based analysis
   ├── Classification: Threshold-based mining detection
   └── Volumetrics: DEM-based volume estimation (planned)

4. Visualization
   ├── Frontend: Leaflet maps with GeoJSON overlays
   └── Reports: Matplotlib-based PDF generation

5. Output
   ├── Change hotspot maps
   ├── Statistical summaries
   └── Automated PDF reports
```

---

## Technology Stack

### Data Processing & Analysis
- **Python 3.x**
- **Core Libraries**:
  - `rasterio` - Geospatial raster I/O
  - `geopandas` - Vector geospatial operations
  - `numpy` - Numerical computing
  - `scipy` - Signal processing (median filtering)
  - `matplotlib` - Plotting and visualization
  - `scikit-image` - Image segmentation (SLIC)
  - `pandas` - Data manipulation

### Frontend
- **Leaflet.js** - Interactive mapping
- **Leaflet.draw** - Drawing tools
- **Vanilla JavaScript** (ES6+) - Modular application architecture
- **CSS3** - Responsive styling

### Report Generation
- **matplotlib** - Chart generation
- **seaborn** - Statistical visualizations
- **markdown2** + **weasyprint** - Markdown to PDF conversion (in development)

---

## Data Sources & Formats

### Input Data

| Data Type | Source | Format | Resolution | Use Case |
|-----------|--------|--------|------------|----------|
| SAR Imagery | Sentinel-1 (Copernicus) | GeoTIFF | 10m | Ground disturbance detection |
| Optical Imagery | Sentinel-2 (Copernicus) | GeoTIFF | 10-20m | Vegetation monitoring, RGB visualization |
| Digital Elevation Model | SRTM/Copernicus | GeoTIFF | 30m | Terrain analysis, volume calculation |
| AOI Boundaries | Google Earth | KML/GeoJSON | Vector | Legal mining area definition |

### Output Data

| Output Type | Format | Description |
|-------------|--------|-------------|
| Change Detection Maps | PNG/GeoTIFF | Hotspot overlays showing mining activity |
| Statistical Reports | PDF | Compliance and impact assessment documents |
| Visualization Dashboards | HTML | Interactive web-based analysis tools |
| Metrics | CSV/JSON | Quantitative area, volume, and compliance data |

---

## Key Algorithms & Methods

### 1. **Sentinel-1 Change Detection**
- **VV Backscatter Increase**: Indicates new ground exposure (mining activity)
  - Threshold: +4 to +9 dB increase
- **VH Backscatter Decrease**: Indicates vegetation loss
  - Threshold: -3 to -8 dB decrease
- **Normalization**: Scene-wide mean subtraction to remove atmospheric effects
- **Despeckle**: 5x5 or 7x7 median filter to reduce SAR noise

### 2. **Sentinel-2 Index Calculation**
- **NDVI** = (NIR - Red) / (NIR + Red)
  - Healthy vegetation: 0.6 - 0.9
  - Bare soil/mining: -0.1 - 0.2
- **BSI (approx)** = (SWIR + Red - NIR) / (SWIR + Red + NIR)
  - High values indicate exposed soil
- **Temporal Thresholding**: BSI increase > 0.2 flags new mining activity

### 3. **Object-Based Change Detection**
- **SLIC Segmentation**: Groups pixels into meaningful objects
- **Per-Object Statistics**: Mean VV, VH, NDVI, BSI per segment
- **Combined Criteria**: Both SAR and optical changes must agree for high confidence

### 4. **Volume Estimation** (Planned)
- **Method**: Simpson's Rule or DEM differencing
- **Inputs**: Pre/post-mining DEM data
- **Output**: Cubic meters of extracted material

---

## Current Limitations & Future Work

### Current Limitations
1. **Manual Data Download**: Satellite imagery must be manually exported from EO Browser
2. **No Automated Segmentation**: ML model for mining detection is not yet trained
3. **Limited AOI Coverage**: Only Korba Coal area has been analyzed
4. **Threshold-Based Detection**: Simple rule-based classification, not ML-powered
5. **No Real-Time Processing**: Analysis requires notebook execution
6. **3D Visualization Incomplete**: 3D terrain viewer is in development

### Planned Enhancements

#### Phase 1: Automation & Integration (Next 3 months)
- [ ] Backend API development (FastAPI/Node.js)
- [ ] Automated satellite data fetching (Copernicus/AWS S3)
- [ ] Database integration (PostgreSQL + PostGIS)
- [ ] DEM-based volume calculation implementation
- [ ] Complete 3D terrain visualization

#### Phase 2: ML Model Development (3-6 months)
- [ ] Training data collection and labeling
- [ ] Semantic segmentation model (U-Net/DeepLab)
- [ ] Mining area detection model deployment
- [ ] Illegal activity boundary detection
- [ ] Model performance evaluation

#### Phase 3: Report Specialization (6-9 months)
- [ ] **Government Reports**: Compliance-focused, legal boundary violations
- [ ] **Environmental Reports**: Ecological impact, proximity to water bodies/forests
- [ ] **Economic Reports**: Production estimates, cost-benefit analysis
- [ ] **Corporate Reports**: Operational efficiency, quality metrics
- [ ] Automated PDF generation from templates

#### Phase 4: Enterprise Features (9-12 months)
- [ ] Multi-AOI batch processing
- [ ] Side-by-side mine comparison dashboard
- [ ] Predictive analytics (future impact forecasting)
- [ ] Alert system for unauthorized activity
- [ ] Public-facing transparency dashboard

---

## Usage Examples

### Running Sentinel-1 Analysis

```python
# In playground_sentinel1.ipynb

# 1. Load data for two dates
vv_jan10, meta = read_s1_band(vv_path_jan10)
vv_feb27, _ = read_s1_band(vv_path_feb27)

# 2. Apply speckle filter
vv_jan10_filt = median_filter(vv_jan10, size=5)
vv_feb27_filt = median_filter(vv_feb27, size=5)

# 3. Calculate change
vv_diff = vv_feb27_filt - vv_jan10_filt

# 4. Detect hotspots
change_hotspots = vv_diff > 4.0  # 4dB increase threshold
```

### Running Sentinel-2 Analysis

```python
# In playground_sentinel2.ipynb

# 1. Calculate NDVI
ndvi = (b8 - b4) / (b8 + b4 + 1e-9)

# 2. Calculate BSI
bsi = (b11 + b4 - b8) / (b11 + b4 + b8 + 1e-9)

# 3. Detect new mining
mining_mask = bsi_diff > 0.2  # BSI increase threshold
```

### Launching Web Interface

```bash
# Serve the frontend locally
cd app/
python -m http.server 8000
# Open http://localhost:8000 in browser
```

---

## File Structure Summary

```
OreNexus/
├── app/                    # Web frontend (interactive mapping)
├── data/                   # Satellite imagery and AOI files
│   ├── GoogleEarth/       # KML/GeoJSON boundaries
│   ├── Sentinel1-SAR/     # Radar imagery (VV/VH)
│   ├── Sentinel2-Hyperspectral/  # Optical bands (RGB + NIR + SWIR)
│   └── SRTM-DEM/          # Elevation data
├── EDTA/                  # Jupyter notebooks for analysis
│   ├── playground_sentinel1.ipynb
│   ├── playground_sentinel2.ipynb
│   └── playground_dem.ipynb
├── report_generation/     # PDF report generators
├── visualizations_demo/   # Demo dashboards and synthetic data
├── microscripts/          # Utility scripts (KML conversion, etc.)
└── documents/             # Project documentation and notes
```

---

## Contributing & Development

### Prerequisites
```bash
pip install rasterio geopandas numpy scipy matplotlib scikit-image pandas seaborn
```

### Development Workflow
1. Add new analysis methods to EDTA notebooks
2. Export core logic to Python modules
3. Integrate with backend API (future)
4. Update frontend to consume new data
5. Test with different AOIs and date ranges

---

## References & Resources

- **Sentinel-1 SAR Data**: https://scihub.copernicus.eu/
- **Sentinel-2 Optical Data**: https://apps.sentinel-hub.com/eo-browser/
- **SRTM DEM**: https://earthexplorer.usgs.gov/
- **Leaflet.js Documentation**: https://leafletjs.com/
- **Rasterio Documentation**: https://rasterio.readthedocs.io/

---

## Contact & Support

For questions, issues, or collaboration inquiries, refer to the project README.
