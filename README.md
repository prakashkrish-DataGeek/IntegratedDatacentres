# 🌐 EcoGrid AI — Datacentres Siting Intelligence Platform

> **Multi-vector spatial risk scoring for hyper-scale compute infrastructure, combining satellite geospatial data, power grid telemetry, hydrological modelling, seismic hazard classification, and land surface temperature analytics to produce a defensible Composite Risk Index for data centre siting decisions.**

[![Python](https://img.shields.io/badge/Python-3.11-blue?style=flat&logo=python)](https://python.org)
[![GEE](https://img.shields.io/badge/Google%20Earth%20Engine-4285F4?style=flat&logo=google&logoColor=white)](https://earthengine.google.com)
[![Folium](https://img.shields.io/badge/Folium-0.14-green?style=flat)](https://python-visualization.github.io/folium/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Live Platform](https://img.shields.io/badge/Live%20Platform-View%20Online-orange)](https://prakashkrish-datageek.github.io/Ecogrid-AI/)

---

## 🗺 Live Platform

**[→ Interactive Infrastructure Intelligence Dashboard](https://prakashkrish-datageek.github.io/IntegratedDatacentres/)**

Analysing **14.9 GW of proposed AI data centre capacity** across 11 clusters in the US, China, and India — six constraint dimensions, one defensible risk matrix.

---

## Overview

The AI infrastructure buildout is proceeding at extraordinary speed. Hyperscalers are committing tens of billions of dollars to compute campuses. Governments are offering land, grid connections, and tax incentives to attract investment. The press releases share the same architecture: a large number, a renewable energy percentage, and a net-zero target dated conveniently far into the future.

What is largely absent from these announcements is a rigorous, multi-vector accounting of the physical constraints that will determine whether these facilities can actually operate — sustainably, at scale, and without creating liabilities that appear later on the balance sheet rather than in the planning document.

EcoGrid AI is built to provide that accounting.

The platform combines five independently weighted spatial risk vectors — drawn from satellite earth observation, national grid telemetry, hydrogeological survey data, seismic zonation standards, and climate reanalysis — to produce a **Composite Risk Index (CRI)** that supports infrastructure siting decisions from pre-feasibility through to regulatory permitting.

This is not an academic exercise. It is a practitioner-grade tool, built by someone who has spent 25 years working inside asset-heavy infrastructure organisations where the difference between the press release and operational reality is measured in billions.

---

## The Core Problem: What the Market Is Not Pricing

Three physical constraints are systematically underpriced in current AI infrastructure investment decisions.

**1. Groundwater depletion at the siting location.**
A hyperscale data centre operating conventional evaporative cooling consumes between 1.5 and 5 litres of water per kilowatt-hour of IT load. In Bengaluru — one of the highest-growth compute corridors in Asia — the Central Ground Water Board classifies large portions of the aquifer as over-exploited, with annual depletion running at 1.8 to 4.5 metres per year. The regulatory and social licence risk of large-scale water extraction in this environment is not in the site selection models of most operators.

**2. Grid interconnection lead times and their carbon accounting implications.**
In Northern Virginia — the world's highest-density compute corridor — queued grid interconnection requests exceed confirmed absorption capacity by a ratio estimated at over 10:1. Lead times for new dedicated connections are running five to seven years. The emissions that occur while a facility operates on the marginal grid mix during that period are Scope 2 emissions. Most sustainability disclosures treat them as a footnote.

**3. The gap between Scope 2 commitments and Scope 2 reality.**
The majority of hyperscaler net-zero commitments rest on Renewable Energy Certificates or Power Purchase Agreements that are temporally and locationally decoupled from actual consumption. A certificate for wind energy generated at midnight in one grid region cannot accurately claim to offset compute running at noon in another. Hourly, location-matched carbon accounting — the standard Google has adopted for its 24/7 Carbon-Free Energy commitment — reveals a materially different picture from annual averages.

EcoGrid AI quantifies all three of these constraints, at site level, using publicly available satellite and ground-truth data.

---

## Methodology

### Composite Risk Index (CRI)

The CRI is a weighted sum of five independent spatial vectors, with weights derived from an Analytical Hierarchy Process (AHP) calibrated to the relative materiality of each constraint for long-life critical infrastructure.

```
CRI = (0.30 × Hydro_score) + (0.25 × Grid_score) + (0.20 × Env_score)
      + (0.15 × Seismic_score) + (0.10 × LST_score)

Range: 0–100
  0–45   → Acceptable
  46–65  → Elevated Risk
  66–100 → High Risk (substantive design mitigation required)
```

### Vector Definitions

| Vector | Weight | Description | Primary Data Source |
|--------|--------|-------------|---------------------|
| Hydrological Stress | 0.30 | Groundwater depletion rate, LULC impervious fraction, recharge deficit, proximity to WRIS-registered waterbodies | CGWB · India-WRIS · ESRI LULC 10m |
| Grid Reliability | 0.25 | Substation proximity, interconnection queue latency, regional renewable mix, T&D loss | CEA Grid Atlas · Regional Load Dispatch Centres |
| Environmental Volatility | 0.20 | SAR backscatter z-score (subsidence proxy), 10-year flood event count, urban NDVI | Sentinel-1 GRD · Sentinel-2 SR |
| Seismic Hazard | 0.15 | BIS IS-1893:2016 zone classification, Peak Ground Acceleration | Bureau of Indian Standards |
| Land Surface Temperature | 0.10 | Urban LST peak (summer), Urban Heat Island delta-T, annual precipitation | Landsat-9 ST_B10 · ERA5 Reanalysis |

### Individual Vector Score Formulas

```python
# Hydrological Stress
water_score = min(100, int((gw_depletion_m_yr * 18) + (lulc_impervious_pct * 0.4)))

# Grid Reliability
grid_score  = min(100, int((substation_km * 5) + (50 - renewable_mix_pct)))

# Environmental Volatility
env_score   = min(100, int((sar_backscatter_zscore * 55) + (flood_events_10yr * 8)))

# Seismic Hazard (BIS IS-1893:2016 zone mapping)
# Zone II → 20  |  Zone III → 50  |  Zone IV → 75  |  Zone V → 95

# Land Surface Temperature
lst_score   = min(100, int((lst_urban_celsius - 30) * 3.5))
```

### Solar Resource Assessment (Opportunity Layer)

The solar assessment runs as an independent opportunity layer, separate from the risk CRI, to quantify on-site renewable generation potential as a partial mitigation for grid interconnection constraints.

```python
# System efficiency: 75% (shading + inverter + cable + degradation losses)
solar_daily_mwh = (roof_sqm * 0.2 * avg_ghi_kwh_m2_day * 0.75) / 1000
solar_mw_offset = solar_daily_mwh / 24  # Continuous equivalent offset
```

### SAR Backscatter Note

Google Earth Engine does not support phase-based InSAR processing. For millimetre-precision vertical displacement measurement, phase-based InSAR requires:

- **Copernicus EGMS** (European Ground Motion Service): https://egms.land.copernicus.eu/
- **NASA ARIA**: https://aria.jpl.nasa.gov/
- **SNAP + StaMPS/MintPy** with Sentinel-1 SLC data

This project uses **SAR backscatter temporal variance as a free, fast proxy** appropriate for hotspot screening and pre-feasibility assessment.

---

## Data Sources

| Dataset | Source | Resolution | GEE Collection / Reference | CRI Vector |
|---------|--------|------------|---------------------------|------------|
| Sentinel-1 GRD | ESA Copernicus | 20m | `COPERNICUS/S1_GRD` | Environmental |
| Sentinel-2 SR | ESA Copernicus | 10m | `COPERNICUS/S2_SR_HARMONIZED` | NDVI / LULC |
| ESRI LULC 2023 | Impact Observatory | 10m | `projects/sat-io/open-datasets/landcover/ESRI_Global-LULC_10m_TS` | Hydrology |
| NASADEM | NASA JPL | 30m | `NASA/NASADEM_HGT/001` | Terrain / Seismic |
| Landsat-9 C2 L2 | USGS / NASA | 30m | `LANDSAT/LC09/C02/T1_L2` | LST / UHI |
| ERA5 Daily Reanalysis | ECMWF / Copernicus | ~28km | `ECMWF/ERA5_LAND/DAILY_AGGR` | Climate / Solar GHI |
| CHIRPS Daily | UCSB CHG | ~5.5km | `UCSB-CHG/CHIRPS/DAILY` | Precipitation |
| CGWB Well Log Database | Central Ground Water Board | Point | indiawris.gov.in/wris | Hydrology |
| CEA Substation Atlas | Central Electricity Authority | Vector | cea.nic.in | Grid |
| BIS IS-1893:2016 | Bureau of Indian Standards | Zone Map | Seismic Zone Classification | Seismic |

---

## Repository Structure

```
ecogrid-ai/
├── Ecogrid-AI-v2.py                  ← Main dashboard generator (v2 — five-vector CRI)
├── Ecogrid-AI-old.py                 ← Original prototype (three-vector, archived)
├── Ecogrid-AI-index.html             ← Generated dashboard (run Ecogrid-AI-v2.py to regenerate)
├── index.html                        ← GitHub Pages live platform (multi-corridor, six-dimension)
│
├── articles/
│   ├── Part1_Sovereign_Debt.md       ← Part 1: Sovereign Debt & The Finite Earth
│   ├── Part2_Energy_Hunger.md        ← Part 2: Where AI's Energy Hunger Meets the Planet's Last Boundaries
│   └── Part3_Gigawatt_Blind_Spot.md  ← Part 3: The Gigawatt Blind Spot (Scope 2 accountability)
│
├── gee_scripts/                      ← Google Earth Engine JavaScript scripts
│   ├── 01_sentinel1_sar_variance.js  ← S1 backscatter temporal z-score computation
│   ├── 02_lulc_impervious_fraction.js← ESRI LULC impervious surface extraction
│   ├── 03_lst_uhi_landsat9.js        ← Landsat-9 LST + Urban Heat Island delta
│   └── 04_era5_ghi_solar.js          ← ERA5 Global Horizontal Irradiance extraction
│
├── requirements.txt                  ← Python dependencies
└── README.md
```

---

## Quick Start

### Requirements

```bash
pip install -r requirements.txt
```

**requirements.txt**
```
folium>=0.14.0
numpy>=1.24.0
```

### Generate the Single-Site Dashboard

```bash
# Clone the repository
git clone https://github.com/prakashkrish-DataGeek/ecogrid-ai.git
cd ecogrid-ai

# Run the dashboard generator
python Ecogrid-AI-v2.py

# Output: Ecogrid-AI-index.html (open in any browser)
```

### Extend to Additional Corridors

The `CORRIDORS` configuration dictionary at the top of `Ecogrid-AI-v2.py` is designed for extension. Add a new entry with the corridor's coordinates, seismic zone, CGWB aquifer category, and CEA grid region:

```python
CORRIDORS = {
    "bengaluru_east": {
        "label":           "Bengaluru-East Compute Sector (Phase-1)",
        "lat":             12.9716,
        "lon":             77.5946,
        "seismic_zone":    "II",
        "cgwb_category":   "Over-exploited",
        "cea_regional_grid": "Southern Regional Grid",
    },
    "hyderabad_west": {
        "label":           "Hyderabad-West Compute Corridor",
        "lat":             17.3850,
        "lon":             78.4867,
        "seismic_zone":    "II",
        "cgwb_category":   "Critical",
        "cea_regional_grid": "Southern Regional Grid",
    },
    # Add further corridors here
}
```

### GEE Scripts

The GEE scripts in `gee_scripts/` are designed to run directly in [code.earthengine.google.com](https://code.earthengine.google.com):

1. Open any `.js` file from `gee_scripts/`
2. Copy and paste into a new GEE Script
3. Click **Run** — results appear in the map panel and console
4. Use the **Tasks** tab to export rasters to Google Drive

---

## Key Findings: Bengaluru-East Compute Sector

The current analysis is focused on the Bengaluru-East compute corridor — the highest-growth AI infrastructure zone in South Asia. The findings from the five-vector CRI are summarised below.

| Finding | Value | Implication |
|---------|-------|-------------|
| Groundwater depletion rate | 1.8–4.5 m/year | CGWB: Over-exploited. Open-loop cooling non-viable without regulatory risk |
| SW recharge deficit | 180–420 mm/year | High impervious cover (65–92%) suppresses natural recharge systemically |
| Grid interconnection queue | 18–48 months | On-site solar DG is the primary bridge strategy, not a secondary option |
| T&D loss (state average) | 9–18% | Material Scope 2 multiplier on purchased grid energy |
| Peak urban LST | 32–46°C | +4.5 to +11°C UHI delta adds 8–14% to cooling energy load vs rural siting |
| 10-year flood events | 1–7 occurrences | Critical infrastructure elevation and SAR monitoring mandatory |
| Solar yield potential | 4.9–6.4 kWh/m²/day GHI | 25,000–120,000 m² viable rooftop area supports 2–15 MW continuous offset |
| Estimated solar payback | 4.5–8.2 years | Economics support owned generation, not just PPA purchasing |

---

## The Scope 2 Accountability Framework

EcoGrid AI is underpinned by a specific position on carbon accounting for AI infrastructure. It is worth stating explicitly.

Most hyperscaler Scope 2 commitments rest on annual Renewable Energy Certificates or Power Purchase Agreements that are temporally and locationally decoupled from actual grid consumption. A certificate for renewable energy generated in one region and one hour cannot accurately represent the carbon intensity of compute running in a different region and a different hour.

**Hourly, location-matched Scope 2 accounting** — the standard that Google has adopted for its 24/7 Carbon-Free Energy commitment — reveals a materially different picture from annual averages, particularly during the multi-year window between facility commissioning and dedicated renewable interconnect completion.

EcoGrid AI's CRI grid vector explicitly penalises long interconnection queue latency because that latency has a direct carbon consequence that does not appear in annual Scope 2 reporting under current industry conventions. The solar opportunity assessment is structured as a mitigation quantification precisely because on-site generation is the only mechanism that eliminates the temporal decoupling problem entirely.

The three-part article series linked below develops this argument in full.

---

## Article Series

This platform is the analytical backbone of a three-part series on natural capital, physical infrastructure constraints, and the sustainability accountability gap in AI infrastructure investment.

| Part | Title | Link |
|------|-------|------|
| Part 1 | Sovereign Debt & The Finite Earth — when nature's balance sheet finally corrects | [→ Read](https://prakashkrish-datageek.github.io/Soveriegn-Debt-and-Nature/) |
| Part 2 | Where AI's Energy Hunger Meets the Planet's Last Boundaries — what 14.9 GW of proposed capacity actually looks like | [→ Read](https://prakashkrish-datageek.github.io/Ecogrid-AI/) |
| Part 3 | The Gigawatt Blind Spot — why the AI infrastructure race is borrowing against a carbon budget it has not read | [→ Read on LinkedIn](https://linkedin.com/in/prakashkrishnamachari) |

---

## Relationship to Bengaluru Groundwater Stress Project

EcoGrid AI's hydrological stress vector builds directly on the methodology developed in the companion project:

**[bengaluru-groundwater-stress](https://github.com/prakashkrish-DataGeek/bengaluru-groundwater-stress)** — a multi-sensor satellite analysis combining Sentinel-1 SAR, Sentinel-2 NDVI, Landsat-9 LST, and NASADEM to map recharge zones, subsidence hotspots, and urban water stress across Bengaluru.

The GWPZ (Groundwater Potential Zone) formula from that project:

```
GWPZ = (0.30 × LULC_score) + (0.25 × Slope_score)
      + (0.25 × NDVI_score) + (0.20 × Lineament_score)
```

...directly informs EcoGrid AI's hydrological vector construction, with the LULC impervious fraction and NDVI greenery index carried forward as input parameters. Where the groundwater project characterises the aquifer system, EcoGrid AI translates that characterisation into an infrastructure siting risk score.

---

## Limitations and Production Extensions

The current implementation simulates spatial data extraction via seeded random distributions that are calibrated to realistic parameter ranges for the Bengaluru corridor. This is a deliberate design choice for a publicly deployable prototype — it avoids API key dependencies while preserving the full analytical and visualisation framework.

**To convert to live data extraction**, replace the `fetch_spatial_intelligence()` function body with authenticated Google Earth Engine API calls:

```python
import ee
ee.Initialize(project='your-gee-project-id')

# Example: extract LULC impervious fraction at target location
lulc = ee.ImageCollection("projects/sat-io/open-datasets/landcover/ESRI_Global-LULC_10m_TS") \
         .filterDate('2023-01-01', '2023-12-31').mosaic()
point = ee.Geometry.Point([lon, lat])
lulc_val = lulc.sample(point, 10).first().get('b1').getInfo()
```

**Planned extensions for v3:**

- Multi-corridor batch analysis with comparative ranking dashboard
- Integration of CGWB's Dynamic Ground Water Resources Assessment (DGWRA) district-level data
- Scope 2 carbon intensity layer drawing from the Indian Grid real-time merit order dispatch data (POSOCO)
- Hourly carbon intensity matching overlay aligned with the 24/7 CFE accounting standard
- Water consumption modelling by cooling technology type (evaporative, air-cooled, liquid immersion)

---

## Mitigation Framework

For sites where the CRI exceeds 45, EcoGrid AI generates a six-category mitigation roadmap aligned to each risk vector. The categories and their regulatory grounding are:

| Category | Key Intervention | Regulatory Reference |
|----------|-----------------|---------------------|
| Hydrology | Closed-loop / immersion cooling; artificial recharge injection pits | CGWB groundwater extraction norms; KSPCB NOC |
| Grid | On-site solar DG with single-axis tracking; AI workload scheduling | CEA Grid Code; Karnataka Solar Policy |
| Environmental | Sentinel-1 automated backscatter monitoring; flood plinth elevation | MoEF&CC EIA Notification 2006 |
| Seismic | IS-1893:2016 compliant structural design; seismic base isolation for critical plant | BIS IS-1893; IS-456; IS-1888 |
| Thermal | High-albedo roof coatings; CFD-optimised data hall airflow; water-side economisers | ASHRAE 90.4; Green Building Council India |
| Governance | Environmental Impact Assessment; BESCOM dedicated feeder; BBMP/KSPCB coordination | EIA Notification 2006; Electricity Act 2003 |

---

## Author

**Prakash Krishnamachari**

Senior Data & AI Executive — 25 years across Shell, Maersk, and TotalEnergies.
Currently building at the intersection of earth observation, AI, and infrastructure intelligence.

- GitHub: [@prakashkrish-DataGeek](https://github.com/prakashkrish-DataGeek)
- LinkedIn: [linkedin.com/in/prakashkrishnamachari](https://linkedin.com/in/prakashkrishnamachari)
- Email: prakash.krishnamachari@gmail.com
- Live Platform: [prakashkrish-datageek.github.io/Ecogrid-AI](https://prakashkrish-datageek.github.io/Ecogrid-AI/)

---

## License

MIT License — see [LICENSE](LICENSE) for full terms.

Satellite imagery and derivative data products are subject to the terms of use of their respective agencies (ESA Copernicus, NASA, USGS, ECMWF). EcoGrid AI does not redistribute raw satellite data.

---

*Built with open satellite data, open-source Python, and 25 years of watching the gap between infrastructure announcements and operational reality.*
