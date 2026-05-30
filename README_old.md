# Spatial Analysis of Paris Housing Prices — Endogenous Regionalization of the Real Estate Market

This repository archives a research project supervised by **Prof. Antonin Bergeaud (HEC Paris, Associate Professor of Economics, 2025 Best Young French Economist Award)**.

It analyzes the spatial heterogeneity of housing prices in Paris using the public French real estate transaction database (DVF+) and challenges the economic relevance of the 20 administrative *arrondissements* as units of analysis.

## 📖 Overview

Parisian housing prices exhibit extreme heterogeneity not only across the 20 administrative arrondissements but **within** each of them. The 15th arrondissement alone has the population of Bordeaux; treating it as a single market ignores the micro-local dynamics that drive real estate valuation.

Building on the DVF+ public dataset, this project:

1. Quantifies the gap between inter- and intra-district price dispersion in Paris
2. Descends to finer spatial resolutions (cadastral sections, then parcel-level coordinates)
3. Applies **spatially-constrained agglomerative clustering** (Ward with k-NN contiguity graph) to generate endogenous "economic zones" that may cross administrative boundaries
4. Quantifies explanatory power via OLS and ANOVA, and compares the endogenous partition to the administrative one at equal degrees of freedom

## 🔬 Main Findings

- **Micro-local heterogeneity is substantial.** Moving from 20 arrondissements to 1,025 cadastral sections reduces intra-unit price dispersion by 13.3% and increases geographic explanatory power (adjusted R²) by roughly 40%.
- **Administrative boundaries are suboptimal.** At equal cardinality (20 zones), endogenous Ward clusters reach adjusted R² = 27.9% vs. 21.7% for the official arrondissements. Market-based zones cross administrative borders, particularly in eastern Paris (10e–11e–12e continuity).
- **The central core behaves as one market.** The 1st, 2nd, and 3rd arrondissements merge into a single endogenous cluster, whereas the 15th and 16th split — reflecting genuine micro-local market structure rather than administrative delineation.
- **Structural vs. spatial effects.** A Spearman rank test confirms the well-known negative correlation between surface and price per m² (small units trade at a premium per m²), isolating structural heterogeneity from purely spatial heterogeneity before any spatial interpretation.
- **Residual variance remains large.** Roughly 70% of price variance is unexplained by pure geography, motivating the ongoing Phase 4 (see below).

## 📊 Methodology

- **Data cleaning.** Aggregation of DVF transactions by `idmutation` to correct the multi-lot duplication bias (a critical DVF pitfall often overlooked). Filters on resale type (excluding pre-construction sales, VEFA), property type (residential apartments), realistic price bounds, and minimum surface (≥ 9 m²).
- **Structural control.** Spearman rank correlation between surface and price per m² (non-parametric, robust to the heavy-tailed price distribution), followed by a breakdown by surface bracket.
- **Variance decomposition.** OLS with categorical fixed effects (`statsmodels`), reporting adjusted R² to penalize spurious subdivisions.
- **Spatial regionalization.** Agglomerative clustering with Ward linkage under a k-nearest-neighbor contiguity constraint (`scikit-learn`), producing 20 contiguous endogenous clusters, directly comparable to the 20 administrative arrondissements.
- **Geocoding.** Parcel-level coordinates merged from the Etalab geolocated DVF referential via the `l_idpar` key.
- **Visualization.** Interactive `folium` map (with `FastMarkerCluster`) overlaying endogenous clusters and administrative boundaries for direct visual comparison.

## 🗂 Repository structure

- `analysis.ipynb` — full analysis pipeline (Phases 1 to 3)
- `outputs/` — exported figures and `folium` maps
- `requirements.txt` — Python dependencies

> Raw DVF data is not redistributed in this repository. See the Data Access section below to obtain it from the official source.

## 📦 Data Access

- **DVF+ Open Data (Cerema):** [cerema.app.box.com/v/dvfplus-opendata](https://cerema.app.box.com/v/dvfplus-opendata/folder/316766049383) — raw Paris transactions (`mutations_d75.csv`, department 75)
- **DVF geocoded referential (Etalab, data.gouv.fr):** [files.data.gouv.fr/geo-dvf](https://files.data.gouv.fr/geo-dvf/latest/csv/) — parcel-level GPS coordinates, fetched on demand from the notebook
- **Administrative boundaries (Open Data Paris):** district polygons in GeoJSON, fetched on demand from the notebook

## 🚀 Quick Start

1. Clone the repository and install dependencies:
```bash
   pip install -r requirements.txt
```
2. Download `mutations_d75.csv` from the DVF+ link above and place it at the repository root.
3. Open and run `notebooks/analysis.ipynb`. Geocoding and map background data are fetched at runtime.

## 🔭 Ongoing Work

Current work extends the analysis by integrating external variables from the **Paris Open Data** platform (proximity to green spaces, street trees, public transport stations, public facilities, noise exposure) to model the residual price variance left unexplained by pure geography. The aim is to quantify which urban amenities drive micro-local valuation and to what extent they account for the roughly 70% of variance that spatial regionalization alone cannot capture.

## ⚠️ Known Limitations

- **In-sample R² for endogenous clusters.** Clusters are fitted on `prix_m2` and evaluated on `prix_m2`, which mechanically favors the endogenous partition. A more rigorous benchmark would require spatial cross-validation, or clustering on non-price features (amenities, transit, etc.) before evaluating on price.
- **Contiguity approximation.** The contiguity graph is built from a k-nearest-neighbor graph (k = 6) on cadastral centroids rather than strict polygon adjacency (Queen/Rook). This produces occasional cross-Seine artifacts in the interactive map that do not reflect real economic continuities.
- **Missing intrinsic features.** DVF does not record floor, building condition, view, or construction year. A substantial share of the residual variance is therefore structural rather than spatial.
- **Arbitrary outlier bounds.** Price filtering between 1,000 and 30,000 EUR/m² is a conservative heuristic; tighter domain-driven thresholds could marginally change descriptive statistics.

## 📜 License

Released under the **MIT License**.

## 🙋 Acknowledgements

Conducted as a research elective project at **HEC Paris** under the supervision of **Prof. Antonin Bergeaud (HEC Paris)**, whose guidance on spatial econometrics and methodology shaped the direction of this work.

Data provided by **Cerema (DVF+)**, **Etalab / data.gouv.fr**, and **Open Data Paris**.
