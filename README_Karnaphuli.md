# Karnaphuli River Estuary — Water Quality Assessment Using Remote Sensing and Machine Learning

## Overview

Industrial discharge, ship-breaking operations, and urban runoff have made the Karnaphuli River estuary — Bangladesh's most commercially significant coastal waterway — one of the most environmentally stressed estuarine systems in South Asia. Yet its pollution has rarely been mapped continuously in space using scalable, reproducible methods.

This project develops and evaluates an integrated monitoring framework that combines **in-situ laboratory water sampling**, **Sentinel-2A satellite reflectance**, and **machine learning regression modelling** to assess physicochemical and heavy metal pollution across the estuary. It demonstrates that freely available multispectral satellite data, when combined with ground-truth measurements and proper statistical modelling, can predict 40–58% of the spatial variability in key water quality parameters — making it a viable complement to costly conventional monitoring.

---

## Study Area

The **lower Karnaphuli River estuary**, Chattogram, Bangladesh — covering approximately 22 km from Shah Amanat Bridge to the edge of Patenga Beach. Sampling locations span between **22.325°N, 91.836°E** and **22.234°N, 91.789°E**, within the river–sea transition zone characterized by tidal mixing, ship-breaking activities, and dense industrial development along the Sitakunda coastal belt.

---

## Research Objectives

1. Quantify the spatial variation of physicochemical parameters (pH, EC, TDS, TSS, turbidity) and 14 heavy metals (Al, Fe, As, Pb, Be, Cd, Cr, Co, Hg, Cu, Mn, Ni, Zn, V) across the estuary
2. Establish statistical relationships between Sentinel-2A spectral reflectance and laboratory-measured water quality indicators
3. Develop and validate predictive machine learning regression models for key parameters
4. Map contamination hotspots using GIS-based Kriging interpolation and regression-based raster prediction

---

## Methodology

### 1. Field Sampling
- 50 water samples collected from ~1 ft surface depth, March 15, 2023
- Georeferenced using GNSS; approximately 300 m average inter-sample spacing
- Laboratory analysis: pH meter, DO meter, turbidity meter, Hanna Instrument (EC/TDS), 47 mm filter membranes (TSS, USEPA Method 160.2), ICP for 14 heavy metals
- Samples diluted 10× in HNO₃ prior to metal analysis to account for estuarine salinity

### 2. Satellite Image Acquisition and Processing
- **Platform:** Sentinel-2A, Level-2A (Bottom of Atmosphere, atmospherically corrected)
- **Acquisition date:** March 12–17, 2023 (closest available to sampling date), ≤20% cloud cover
- **Image retrieval:** Google Earth Engine (GEE) — JavaScript API (see `code/GEE_image_acquisition.js`)
- **Reflectance extraction:** SNAP (Sentinel Application Platform) at sampling point coordinates

### 3. Machine Learning Modelling (Python — Google Colab)
All modelling steps performed in Python using scikit-learn 1.6.1, NumPy 2.0.2, and Pandas 2.2.2.

**Preprocessing:**
- Z-score standardization (μ = 0, σ = 1)
- Correlation-based pruning (|r| > 0.90 → remove lower-correlated predictor)
- Variance Inflation Factor (VIF) thresholding (VIF > 5 → remove predictor)

**Models evaluated:**

| Model | Result |
|---|---|
| Multiple Linear Regression (MLR) with 10-fold CV | ✅ Best overall performance |
| MLR with Leave-One-Out CV (LOOCV) | ✅ Used for small-sample stability |
| Principal Component Regression (PCR) | ✅ Comparable to MLR after preprocessing |
| Ridge Regression | ❌ Did not improve on MLR |
| Lasso Regression | ❌ Did not improve on MLR |
| Support Vector Regression (SVR) | ❌ Did not improve on MLR |

**Evaluation metrics:** R², RMSE, MAE; residual normality checked via QQ plots.

### 4. Spatial Mapping (ArcMap 10.8)
- **Kriging interpolation** (Ordinary Kriging, Spherical semivariogram) from 50 observed field values → continuous raster surface
- **Raster calculation** using MLR/PCR regression equations applied to Sentinel-2A reflectance rasters → spatially predicted parameter distributions

---

## Key Results

### Regression Model Performance (R² values)

| Parameter | R² | Significant Bands |
|---|---|---|
| Total Suspended Solids (TSS) | **0.58** | B6, B7, B8, B9, B11, B12 |
| Turbidity | 0.50 | B6, B7, B8, B9, B11 |
| Cobalt (Co) | 0.51 | B6, B7, B8, B9 |
| EC | 0.47 | B1, B7, B8 |
| TDS | 0.47 | B1, B7, B8 |
| Beryllium (Be) | 0.49 | B6, B8, B9 |
| Aluminum (Al) | 0.46 | B6, B8, B9 |
| Iron (Fe) | 0.45 | B6, B8, B9 |
| Manganese (Mn) | 0.45 | B6, B8, B9 |
| Nickel (Ni) | 0.45 | B6, B7, B8, B9 |
| pH | 0.43 | B1, B6, B7, B8, B11 |

> Parameters below R² = 0.40 (dissolved metals: Cd, Pb, As, Hg, Zn, Cu) showed no statistically significant satellite-based predictions and were excluded from mapping.

**Sentinel-2A Bands 6–9** (red-edge at 740 nm and 783 nm, NIR at 842 nm, water vapour at 945 nm) emerged as the most consistently significant spectral predictors, driven by particle backscattering mechanisms in this turbid, industrial estuary (mean turbidity: 41 NTU; mean TSS: 0.466 g/L).

### Observed Exceedances vs. Standards (DoE Bangladesh, 1997)

| Parameter | Observed Average | Standard (inland industrial) | Status |
|---|---|---|---|
| EC | 21.4 mS/cm | 12 mS/cm | ⚠️ Exceeds |
| TDS | 10.8 g/L | 1.7 g/L | ⚠️ Exceeds |
| TSS | 0.47 g/L | 0.15 g/L | ⚠️ Exceeds |
| Turbidity | 41.0 NTU | 10 NTU | ⚠️ Exceeds |
| Aluminum (Al) | 12.8 mg/L | 0.2 mg/L (drinking) | ⚠️ Exceeds |
| Iron (Fe) | 15.5 mg/L | 2 mg/L | ⚠️ Exceeds |
| Copper (Cu) | 2.56 mg/L | 0.5 mg/L | ⚠️ Exceeds |
| Mercury (Hg) | 0.032 mg/L | 0.01 mg/L | ⚠️ Exceeds |
| Lead (Pb) | 0.135 mg/L | 0.1 mg/L | ⚠️ Exceeds |
| pH | 7.9 | 6.5–8.5 | ✅ Within range |
| COD | 53.15 mg/L | 200 mg/L | ✅ Within range |

---

## Repository Structure

```
Karnaphuli-water-quality-RS-ML/
│
├── README.md
│
├── data/
│   └── water_samples/
│       └── Karnaphuli_field_measurements.csv     # 50-point lab results + coordinates
│
├── code/
│   ├── GEE_image_acquisition.txt                  # Google Earth Engine image retrieval
│   └── ML_regression_modelling.ipynb             # Python: preprocessing, MLR, PCR, QQ plots
│
├── outputs/
│   ├── maps/
│   │   ├── physicochemical/
│   │   │   ├── EC.jpg                            # Interpolation + raster map pair
│   │   │   ├── TDS.jpg
│   │   │   ├── TSS.jpg
│   │   │   ├── pH.jpg
│   │   │   ├── Turbidity.jpg
│   │   │   └── rasters/
│   │   │       └── pH_ras.tif                    # Regression-predicted raster
│   │   └── heavy_metals/
│   │       ├── Al.jpg
│   │       ├── Be.jpg
│   │       ├── Co.jpg
│   │       ├── Fe.jpg
│   │       ├── Mn.jpg
│   │       ├── Ni.jpg
│   │       └── rasters/
│   │           ├── AL_Int.tif                    # Kriging interpolation raster
│   │           ├── Al_ras.tif                    # Regression-predicted raster
│   │           ├── Co_int.tif
│   │           ├── Fe_int.tif
│   │           ├── Fe_ras.tif
│   │           ├── Mn_int.tif
│   │           ├── Mn_ras.tif
│   │           ├── Ni.tif
│   │           └── Ni_ras.tif
│   └── qq_plots/
│       ├── Al.jpeg
│       ├── Be.jpeg
│       ├── Co.jpeg
│       ├── COD.jpeg
│       ├── Cr.jpeg
│       ├── EC.jpeg
│       ├── Fe.jpeg
│       ├── Mn.jpeg
│       ├── Ni.jpeg
│       ├── pH.jpeg
│       ├── TDS.jpeg
│       ├── TSS.jpeg
│       └── Turbidity.jpeg
│
└── docs/
    └── Manuscript_revised_Final.docx             # Full research paper (Accepted for publication)
```

---

## How to Reproduce

### Prerequisites

```bash
pip install numpy==2.0.2 pandas==2.2.2 scikit-learn==1.6.1 matplotlib seaborn
```

Satellite image retrieval requires a free [Google Earth Engine](https://earthengine.google.com/) account.
Spectral extraction at sampling points was performed in [SNAP](https://step.esa.int/main/toolboxes/snap/) (free, ESA).
Spatial mapping was performed in ArcMap 10.8 using the Spatial Analyst toolbox.

### Steps

1. **Satellite image** — run `code/GEE_image_acquisition.js` in the GEE Code Editor to retrieve the March 2023 Sentinel-2A Level-2A image; extract reflectance values at sampling coordinates using SNAP
2. **Modelling** — open `code/ML_regression_modelling.ipynb` in Google Colab; load `data/water_samples/Karnaphuli_field_measurements.csv` and the extracted reflectance values; run all cells to reproduce preprocessing, regression models, cross-validation, and QQ plots
3. **Mapping** — load the regression equations into ArcMap raster calculator against the Sentinel-2 band rasters; run Kriging interpolation from the 50 field points for comparison

---

## Tools and Libraries

| Tool / Library | Purpose |
|---|---|
| Google Earth Engine (GEE) | Sentinel-2A image acquisition |
| SNAP (ESA) | Spectral reflectance extraction at sample points |
| Python — NumPy 2.0.2 | Data preprocessing |
| Python — Pandas 2.2.2 | Data handling and standardization |
| Python — Scikit-learn 1.6.1 | MLR, PCR, Ridge, Lasso, SVR, cross-validation |
| Google Colab | Cloud-based Python environment |
| ArcMap 10.8 | Kriging interpolation, raster calculation mapping |

---

## Citation

> Tasnim, S. S., **Tarana, M. Y.**, Moniruzzaman, M., Afrin, S., & Shreya, S. S. (2026). *Evaluating Heavy Metal and Physicochemical Pollution in the Karnaphuli River Estuary through Remote Sensing and Machine Learning Approaches.* Manuscript under review.

---

## Funding

This research was supported by the **CTRGC Grant: CTRG-22-SHI.S-17**, awarded by North South University, Bangladesh, for the quantitative assessment of heavy metals, toxic gases, and eutrophication in the vicinity of the Sitakunda chemical explosion site and the Karnaphuli River estuary.

---

## License

Shared for academic and research purposes. Please contact the corresponding author before reproducing or adapting any part of this work.
