# 🪨 OreNexus: Mining Smarter — Mapping Impact, Value, and Compliance Gaps

**OreNexus** is a software tool that leverages satellite imagery (EO/SAR) and AI to automatically detect, monitor, and analyze open crust mining activities. It provides a comprehensive solution for **government authorities**, **environmental agencies**, and **mining corporations** to track compliance, assess environmental impact, and gain economic insights.

> **Current Status**: Proof of Concept | Actively developing core analysis pipelines and web interface

---

## 📝 Problem Statement

Traditional monitoring of open crust mining is often manual and inefficient. Tracking the exact extent of mining operations in real-time is challenging, making it difficult to detect when activities extend beyond legally authorized boundaries (defined by Shapefiles or KML files).

This lack of oversight can lead to:
- Unchecked **illegal mining**
- Severe **environmental degradation**
- **Revenue loss** for governing bodies

Hence, there is a need for an **automated system** to:
- Delineate mining areas
- Identify illegal operations
- Calculate extraction volumes
- Visualize the data for effective decision-making

---

## 🎯 What We've Built So Far

### ✅ Data Processing Pipelines
- **Sentinel-1 SAR Analysis**: Ground disturbance detection using VV/VH polarization temporal differencing
- **Sentinel-2 Optical Analysis**: Vegetation monitoring (NDVI) and bare soil detection (BSI)
- **Object-Based Change Detection**: SLIC segmentation for reducing false positives
- **Hotspot Mapping**: Combined SAR + optical change detection for high-confidence mining activity identification

### ✅ Interactive Web Frontend
- **2D Mapping Interface**: Leaflet.js-based interactive map with AOI upload, layer control, and temporal analysis
- **Modular Architecture**: Clean separation of concerns (CSS modules, JS modules)
- **Mining Site Management**: Panel for tracking multiple sites and their status
- **3D Terrain Viewer**: In development for volumetric visualization

### ✅ Report Generation System
- **Automated PDF Reports**: Environmental and economic impact assessments
- **Synthetic Data Generator**: Testing infrastructure for multi-site analysis
- **Visualization Dashboards**: Charts, time-series plots, and compliance scorecards

---

## 📊 Data Sources

We pull data from multiple satellite platforms:

| Source | Data Type | Purpose | Resolution |
|--------|-----------|---------|------------|
| **Copernicus (Sentinel-1)** | SAR (VV/VH polarization) | Ground disturbance detection | 10m |
| **Copernicus (Sentinel-2)** | Multispectral (RGB + NIR + SWIR) | Vegetation health, bare soil monitoring | 10-20m |
| **Copernicus/SRTM** | Digital Elevation Model | Terrain analysis, volume calculation | 30m |
| **Google Earth** | KML/GeoJSON | AOI boundary definitions | Vector |

**Current Test Area**: Korba Coal Mining Region, Chhattisgarh (Jan-Feb 2023 imagery)

---

## 🔬 Analysis Methods (Implemented)

### Sentinel-1 SAR
- Speckle noise reduction (median filtering)
- VV backscatter increase detection → Ground exposure/mining activity
- VH backscatter decrease detection → Vegetation loss
- Scene-wide normalization to remove atmospheric effects
- Dual-threshold hotspot mapping (VV+VH combined)

### Sentinel-2 Optical
- NDVI calculation → Vegetation health monitoring
- BSI approximation → Bare soil/mining exposure
- BAI (Burn Area Index) → Disturbed land identification
- Temporal change detection → Mining expansion tracking
- Classification: Mining/clearing vs vegetation regrowth

### Cross-Sensor Fusion
- Object-based analysis (SLIC segmentation)
- Multi-sensor agreement for high-confidence detection
- Per-object statistics (mean VV, VH, NDVI, BSI)

---

## 🗂️ Project Structure

```
OreNexus/
├── app/                           # Web frontend (Leaflet + vanilla JS)
│   ├── index.html                # Main 2D mapping interface
│   ├── 3d_view.html             # 3D terrain viewer (WIP)
│   ├── css/                     # Modular stylesheets
│   └── js/                      # Modular JavaScript (config, core, modules, utils)
│
├── data/                          # Satellite imagery and AOI files
│   ├── GoogleEarth/              # KML/GeoJSON boundaries
│   ├── Sentinel1-SAR/            # Radar imagery (VV/VH polarizations)
│   ├── Sentinel2-Hyperspectral/  # Optical bands (B04, B08, B11, etc.)
│   └── SRTM-DEM/                 # Digital Elevation Models
│
├── EDTA/                          # Exploratory Data & Temporal Analysis (Jupyter notebooks)
│   ├── playground_sentinel1.ipynb   # SAR change detection pipeline
│   ├── playground_sentinel2.ipynb   # Optical index calculation & classification
│   ├── playground_dem.ipynb         # Elevation analysis (WIP)
│   └── playground_polished.ipynb    # Refined analysis workflows
│
├── report_generation/             # PDF report generators
│   ├── main.py                   # Matplotlib-based report generation
│   ├── economic_report.html      # Economic metrics template
│   └── environmental_report.html # Environmental impact template
│
├── visualizations_demo/           # Demo dashboards and synthetic data
│   ├── synthetic_dataset.py      # Generates test mining site data
│   └── visualizations.py         # Chart generation examples
│
├── microscripts/                  # Utility scripts
│   ├── kml_to_gearth.py         # KML/GeoJSON conversion
│   └── process_korba.py         # AOI-specific preprocessing
│
└── documents/                     # Project documentation and notes
```

---

## 🚀 Roadmap & Future Plans

### Phase 1: Backend & Automation (Next 3 months)
- [ ] Backend API development (FastAPI for data processing, Node.js for frontend API)
- [ ] Automated satellite data fetching (Copernicus API / AWS Open Data)
- [ ] PostgreSQL + PostGIS database integration
- [ ] DEM-based volume calculation (Simpson's Rule)
- [ ] Complete 3D terrain visualization with depth overlays

### Phase 2: ML Model Training (3-6 months)
- [ ] Collect and label training data (mining vs non-mining regions)
- [ ] Train semantic segmentation model (U-Net / DeepLabV3+)
- [ ] Deploy mining area detection model
- [ ] Implement illegal boundary detection (AOI intersection)
- [ ] Model performance benchmarking

### Phase 3: Specialized Reporting (6-9 months)
- [ ] **Government Reports**: Compliance-focused, legal violations, revenue impact
- [ ] **Environmental Reports**: Ecological damage, water/forest proximity, biodiversity impact
- [ ] **Economic Reports**: Production estimates, cost-benefit analysis, ore quality
- [ ] **Corporate Reports**: Operational efficiency, KPIs, comparison metrics
- [ ] Automated template-based PDF generation

### Phase 4: Enterprise Features (9-12 months)
- [ ] Multi-AOI batch processing
- [ ] Side-by-side mine comparison dashboard
- [ ] Predictive analytics (future impact forecasting)
- [ ] Real-time alert system for unauthorized activity
- [ ] Public transparency dashboard for stakeholders

---

## 💻 Tech Stack

### Current Implementation
- **Frontend**: Leaflet.js, Vanilla JavaScript (ES6+), CSS3
- **Data Processing**: Python 3.x (rasterio, geopandas, numpy, scipy, matplotlib, scikit-image)
- **Report Generation**: Matplotlib, Seaborn, Pandas
- **Notebooks**: Jupyter Lab

### Planned Tech Stack (Production)
- **Frontend**: React (TypeScript), Mapbox GL JS, Chart.js / ECharts
- **Backend API**: FastAPI (Python) for data processing, Node.js + Express (TypeScript) for web API
- **Database**: PostgreSQL + PostGIS, Redis for caching
- **ML Framework**: PyTorch / TensorFlow for model training and inference
- **DevOps**: Docker, GitHub Actions, AWS/GCP deployment, Cloudflare CDN

---

## 📚 Documentation

For detailed technical documentation, see [DOCUMENTATION.md](./DOCUMENTATION.md)

**Key Documentation Sections**:
- Data processing pipeline walkthrough
- Algorithm explanations (SAR change detection, NDVI/BSI calculation)
- Jupyter notebook usage examples
- Current limitations and workarounds
- API design (planned)

---

## 🔧 Quick Start

### Prerequisites
```bash
pip install rasterio geopandas numpy scipy matplotlib scikit-image pandas seaborn jupyter
```

### Run Analysis Notebooks
```bash
cd EDTA/
jupyter lab
# Open playground_sentinel1.ipynb or playground_sentinel2.ipynb
```

### Launch Web Interface
```bash
cd app/
python -m http.server 8000
# Open http://localhost:8000 in your browser
```

---

## 📈 Key Achievements

- ✅ Successful SAR-based ground disturbance detection (Jan-Feb 2023, Korba Coal AOI)
- ✅ NDVI + BSI temporal change detection for vegetation loss monitoring
- ✅ Object-based segmentation to reduce pixel-level noise
- ✅ Modular web interface with interactive mapping
- ✅ Automated PDF report generation for environmental/economic metrics
- ✅ Synthetic dataset generator for testing multi-site analysis

---

## 🤝 Contributing

This is an active research and development project. Contributions, suggestions, and feedback are welcome!

---

## 📞 Contact

For questions, collaboration inquiries, or demo requests, please reach out via the project repository.

---

**OreNexus** empowers authorities and enterprises to **mine smarter**, **monitor transparently**, and **manage sustainably** — bridging the gap between compliance, environmental protection, and economic growth.
