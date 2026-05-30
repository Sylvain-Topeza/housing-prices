# Analyse Spatiale des Prix Immobiliers à Paris — Régionalisation Endogène du Marché Immobilier

*🇬🇧 [Read in English](README.md)*

Ce dépôt archive un projet de recherche supervisé par le **Professeur Antonin Bergeaud (HEC Paris, Professeur Associé en Économie, Prix du Meilleur Jeune Économiste de France 2025)**.

Il analyse l'hétérogénéité spatiale des prix des logements à Paris à l'aide de la base de données publique des transactions immobilières françaises (DVF+) et remet en question la pertinence économique des 20 arrondissements administratifs en tant qu'unités d'analyse.

## 📖 Aperçu

Les prix de l'immobilier parisien présentent une hétérogénéité extrême, non seulement entre les 20 arrondissements administratifs, mais également **à l'intérieur** de chacun d'eux. Le 15e arrondissement compte à lui seul la population de Bordeaux ; le traiter comme un marché unique occulte les dynamiques micro-locales qui dictent la valorisation immobilière.

En s'appuyant sur les données publiques DVF+, ce projet :

1. Quantifie l'écart entre la dispersion des prix inter- et intra-arrondissement à Paris.
2. Descend à des résolutions spatiales plus fines (sections cadastrales, puis coordonnées à l'échelle de la parcelle).
3. Applique un **clustering agglomératif sous contrainte spatiale** (Lien de Ward avec graphe de contiguïté k-NN) pour générer des "zones économiques" endogènes capables de franchir les frontières administratives.
4. Valide la méthodologie par une massification sur plusieurs années (2021-2024, >120 000 transactions) et une discipline du K (Méthode du coude) pour justifier formellement le nombre de clusters.

## 🔬 Principaux Résultats

- **L'hétérogénéité micro-locale est substantielle.** Le passage de 20 arrondissements à 1 108 sections cadastrales réduit la dispersion des prix intra-unité de 13,3 % et augmente le pouvoir explicatif géographique (R² ajusté) d'environ 40 %.
- **Les frontières administratives sont sous-optimales.** Validés sur une base massive de 4 ans (2021-2024), les clusters endogènes de Ward atteignent un R² ajusté = 26,7 % contre 21,7 % pour les arrondissements officiels. Les zones de marché franchissent les frontières administratives, ce qui est particulièrement visible dans l'Est parisien (continuité 10e–11e–12e).
- **Le cœur central se comporte comme un marché unique.** Les 1er, 2e et 3e arrondissements fusionnent en un seul cluster endogène, tandis que les 15e et 16e se scindent — reflétant la véritable structure micro-locale du marché plutôt qu'un découpage administratif.
- **Discipline du K et échec de la statistique pure.** Les statistiques spatiales pures (Silhouette, Calinski-Harabasz) échouent à segmenter la ville en raison de la nature continue du gradient spatial des prix. Cependant, un arbitrage économétrique (Méthode du coude confrontée à la taille minimale des clusters) justifie formellement $k=20$ comme l'équilibre optimal entre pouvoir explicatif et viabilité économique (évitant les micro-marchés instables).
- **La variance résiduelle reste importante.** Environ 70 % de la variance des prix reste inexpliquée par la géographie pure.

## 📊 Méthodologie

- **Nettoyage des données.** Agrégation des transactions DVF par `idmutation` pour corriger le biais de duplication multi-lots (un piège critique de DVF souvent ignoré). Filtres sur le type de vente (exclusion des ventes en l'état futur d'achèvement), le type de bien (appartements), des bornes de prix réalistes et une surface minimale.
- **Décomposition de la variance.** MCO (Moindres Carrés Ordinaires) avec effets fixes catégoriels (`statsmodels`), en utilisant le R² ajusté pour pénaliser les subdivisions excessives.
- **Régionalisation spatiale.** Clustering agglomératif avec lien de Ward sous contrainte de contiguïté des k-plus proches voisins (`scikit-learn`), produisant 20 clusters endogènes continus.
- **Géocodage.** Coordonnées à l'échelle de la parcelle fusionnées depuis le référentiel DVF géocodé d'Etalab via la clé `l_idpar`.
- **Visualisation.** Cartes interactives `folium` superposant les clusters endogènes (via les transactions et les micro-quartiers) et les frontières administratives pour une comparaison visuelle directe.

## 🗂 Structure du dépôt

- `analysis_EN.ipynb` & `analysis_FR.ipynb` — pipelines d'analyse complets.
- `outputs/` — figures exportées (graphiques de la discipline du K) et cartes `folium`.
- `requirements.txt` — dépendances Python.

> Les données brutes DVF ne sont pas redistribuées dans ce dépôt en raison des limites de taille de GitHub. Voir la section Accès aux Données ci-dessous pour les obtenir depuis la source officielle.

## 📦 Accès aux Données

- **DVF+ Open Data (Cerema) :** [cerema.app.box.com/v/dvfplus-opendata](https://cerema.app.box.com/v/dvfplus-opendata/folder/316766049383) — transactions brutes à Paris (`mutations_d75.csv`).
- **Référentiel géocodé DVF (Etalab, data.gouv.fr) :** [files.data.gouv.fr/geo-dvf](https://files.data.gouv.fr/geo-dvf/latest/csv/) — coordonnées GPS des parcelles (`etalab_2021.csv.gz` à `etalab_2024.csv.gz`).
- **Frontières administratives (Open Data Paris) :** polygones des arrondissements au format GeoJSON, récupérés dynamiquement depuis le notebook.

## 🚀 Démarrage

1. Téléchargez `mutations_d75.csv` depuis le lien Cerema ci-dessus et placez-le à la racine du dépôt.
2. Téléchargez les 4 fichiers Etalab `.csv.gz` (2021 à 2024) et placez-les à la racine du dépôt.
3. Ouvrez et exécutez `analysis_EN.ipynb` ou `analysis_FR.ipynb`. Les fonds de carte sont récupérés lors de l'exécution.

## ⚠️ Limites Connues

- **R² in-sample pour les clusters endogènes.** Les clusters sont ajustés sur `prix_m2` et évalués sur `prix_m2`, ce qui favorise mécaniquement la partition endogène. Un benchmark plus rigoureux nécessiterait une validation croisée spatiale.
- **Approximation de la contiguïté.** Le graphe de contiguïté est construit à partir d'un graphe de k-plus proches voisins (k = 6) sur les centroïdes cadastraux plutôt que sur une stricte adjacence de polygones (Queen/Rook). Cela approxime les voisinages sur la base de la distance plutôt que sur des frontières partagées.
- **Bornes de valeurs aberrantes arbitraires.** Le filtrage des prix entre 1 000 et 30 000 EUR/m² est une heuristique conservatrice ; des seuils plus stricts liés au domaine pourraient modifier marginalement les statistiques descriptives.
- **Absence de caractéristiques intrinsèques.** DVF n'enregistre ni l'étage, ni l'état du bâtiment, ni la vue, ni l'année de construction. Une part substantielle de la variance résiduelle est donc structurelle plutôt que spatiale.

## 📜 Licence

Diffusé sous la **Licence MIT**.

## 🙋 Remerciements

Réalisé dans le cadre d'un projet de recherche à **HEC Paris** sous la supervision du **Professeur Antonin Bergeaud (HEC Paris)**, dont les conseils méthodologiques ont guidé ce travail.

Données fournies par le **Cerema (DVF+)**, **Etalab / data.gouv.fr**, et **Open Data Paris**.
