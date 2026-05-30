# Spatial Analysis of Paris Housing Prices — Endogenous Regionalization of the Real Estate Market

*🇫🇷 [Lire en français](README_FR.md)*

This repository archives a research project supervised by **Prof. Antonin Bergeaud (HEC Paris, Associate Professor of Economics, 2025 Best Young French Economist Award)**.

It analyzes the spatial heterogeneity of housing prices in Paris using the public French real estate transaction database (DVF+) and challenges the economic relevance of the 20 administrative *arrondissements* as units of analysis.

## 📖 Overview

Parisian housing prices exhibit extreme heterogeneity not only across the 20 administrative arrondissements but **within** each of them. The 15th arrondissement alone has the population of Bordeaux; treating it as a single market ignores the micro-local dynamics that drive real estate valuation.

Building on the DVF+ public dataset, this project:

1. Quantifies the gap between inter- and intra-district price dispersion in Paris
2. Descends to finer spatial resolutions (cadastral sections, then parcel-level coordinates)
3. Applies **spatially-constrained agglomerative clustering** (Ward with k-NN contiguity graph) to generate endogenous "economic zones" that may cross administrative boundaries
4. Validates the methodology through multi-year massification (2021-2024, >120,000 transactions) and K-discipline (Elbow method) to formally justify the number of clusters.

## 🔬 Main Findings

- **Micro-local heterogeneity is substantial.** Moving from 20 arrondissements to 1,108 cadastral sections reduces intra-unit price dispersion by 13.3% and increases geographic explanatory power (adjusted R²) by roughly 40%.
- **Administrative boundaries are suboptimal.** Validated on a massive 4-year dataset (2021-2024), the endogenous Ward clusters reach adjusted R² = 26.7% vs. 21.7% for the official arrondissements. Market-based zones cross administrative borders, particularly in eastern Paris (10e–11e–12e continuity).
- **The central core behaves as one market.** The 1st, 2nd, and 3rd arrondissements merge into a single endogenous cluster, whereas the 15th and 16th split — reflecting genuine micro-local market structure rather than administrative delineation.
- **K-Discipline and Pure Statistics Failure.** Pure spatial statistics (Silhouette, Calinski-Harabasz) fail to segment the city due to the continuous nature of the spatial price gradient. However, an econometric trade-off (Elbow method vs. minimum cluster size) formally justifies $k=20$ as the optimal balance between explanatory power and economic viability (avoiding unstable micro-markets).
- **Residual variance remains large.** Roughly 70% of price variance is unexplained by pure geography.

## 📊 Methodology

- **Data cleaning.** Aggregation of DVF transactions by `idmutation` to correct the multi-lot duplication bias (a critical DVF pitfall often overlooked). Filters on resale type (excluding pre-construction sales), property type (residential apartments), realistic price bounds, and minimum surface.
- **Variance decomposition.** OLS with categorical fixed effects (`statsmodels`), reporting adjusted R² to penalize spurious subdivisions.
- **Spatial regionalization.** Agglomerative clustering with Ward linkage under a k-nearest-neighbor contiguity constraint (`scikit-learn`), producing 20 contiguous endogenous clusters.
- **Geocoding.** Parcel-level coordinates merged from the Etalab geolocated DVF referential via the `l_idpar` key.
- **Visualization.** Interactive `folium` maps overlaying endogenous clusters (via transactions and micro-neighborhoods) and administrative boundaries for direct visual comparison.

## 🗂 Repository Structure

- `analysis_EN.ipynb` & `analysis_FR.ipynb` — full analysis pipelines
- `outputs/` — exported figures (K-discipline graphs) and `folium` maps
- `requirements.txt` — Python dependencies

> Raw DVF data is not redistributed in this repository due to GitHub size limits. See the Data Access section below to obtain it from the official source.

## 📦 Data Access

- **DVF+ Open Data (Cerema):** [cerema.app.box.com/v/dvfplus-opendata](https://cerema.app.box.com/v/dvfplus-opendata/folder/316766049383) — raw Paris transactions (`mutations_d75.csv`)
- **DVF geocoded referential (Etalab, data.gouv.fr):** [files.data.gouv.fr/geo-dvf](https://files.data.gouv.fr/geo-dvf/latest/csv/) — parcel-level GPS coordinates (`etalab_2021.csv.gz` to `etalab_2024.csv.gz`).
- **Administrative boundaries (Open Data Paris):** district polygons in GeoJSON, fetched on demand from the notebook.

## 🚀 Quick Start

1. Download `mutations_d75.csv` from the Cerema link above and place it at the repository root.
2. Download the 4 Etalab `.csv.gz` files (2021 to 2024) and place them at the repository root.
3. Open and run `analysis_EN.ipynb` or `analysis_FR.ipynb`. Map background data is fetched at runtime.

## ⚠️ Known Limitations

- **In-sample R² for endogenous clusters.** Clusters are fitted on `prix_m2` and evaluated on `prix_m2`, which mechanically favors the endogenous partition. A more rigorous benchmark would require spatial cross-validation.
- **Contiguity approximation.** The contiguity graph is built from a k-nearest-neighbor graph (k = 6) on cadastral centroids rather than strict polygon adjacency (Queen/Rook). This approximates neighborhoods based on distance rather than shared borders.
- **Arbitrary outlier bounds.** Price filtering between 1,000 and 30,000 EUR/m² is a conservative heuristic; tighter domain-driven thresholds could marginally change descriptive statistics.
- **Missing intrinsic features.** DVF does not record floor, building condition, view, or construction year. A substantial share of the residual variance is therefore structural rather than spatial.

## 📜 License

Released under the **MIT License**.

## 🙋 Acknowledgements

Conducted as a research project at **HEC Paris** under the supervision of **Prof. Antonin Bergeaud**, whose methodological guidance shaped the direction of this work.

Data provided by **Cerema (DVF+)**, **Etalab / data.gouv.fr**, and **Open Data Paris**.
