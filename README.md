# Horloge Biologique — Prédiction de l'Âge Épigénétique

## Objectif

Ce projet vise à développer une **horloge biologique** capable d'estimer l'âge d'un individu
à partir de son profil de **méthylation de l'ADN** (bêta-values CpG).

L'approche combine six modèles de Machine Learning et un réseau de neurones profond (TensorFlow / Keras),
assemblés par un méta-learner de stacking, pour prédire l'âge biologique de manière précise et robuste.

L'objectif est d'identifier les sites CpG les plus informatifs, de comparer les performances des
différents modèles entre eux et par rapport aux horloges épigénétiques de référence (Horvath 2013,
Hannum 2013), et de produire un pipeline de prédiction réutilisable sur de nouveaux datasets.

## Jeu de données

Données de méthylation de l'ADN (bêta-values Illumina 450k / EPIC) issues de cohortes publiques.

La matrice de méthylation (`beta_50k.csv`) contient les bêta-values de **~50 000 sites CpG**
mesurés sur plusieurs centaines de sujets, accompagnée d'annotations cliniques
(`annot_projet.csv` : âge chronologique, sexe, ethnie).

Seules des données publiques ont été utilisées dans ce projet.

## Méthodes

- Filtrage des CpG (seuil 20 % de valeurs manquantes) et imputation par la médiane
- Sélection des CpG informatifs par corrélation et **Stability Selection** (LASSO bootstrap, 50 répétitions)
- Encodage des covariables (sexe, ethnie → one-hot, aligné sur les niveaux d'entraînement)
- **Elastic Net** (régression pénalisée L1 + L2)
- **Ridge** (régression pénalisée L2)
- **Lasso** (régression pénalisée L1)
- **Random Forest** (500 arbres)
- **XGBoost** (gradient boosting, sérialisation native `xgb.save.raw`)
- **SVM** (noyau radial, `e1071`)
- **Deep Learning** — réseau dense (128 → 64 → 32 → 1) avec BatchNormalization, Dropout,
  EarlyStopping et ReduceLROnPlateau (TensorFlow / Keras via `reticulate`)
- **Stacking** (méta-learner Elastic Net entraîné sur les prédictions OOF des 6 modèles de base)
- Comparaison aux horloges de référence Horvath et Hannum via le package `methylclock`

Les modèles ont été évalués sur un jeu de test indépendant (20 % des données, split stratifié)
à l'aide du RMSE, de la MAE, de la corrélation de Pearson et du R².

## Résultats

- Comparaison complète de 6 modèles ML + Deep Learning + stacking
- Évaluation des performances sur jeu de test indépendant (RMSE, MAE, Corrélation, R²)
- Référence Horvath (2013) : corrélation publiée **r = 0,96**, MAE ≈ **3,6 ans**
- Analyse de l'importance des CpG sélectionnés (Stability Selection)
- Robustesse aux valeurs manquantes et aux datasets externes (formats GEO / TCGA)
- Pipeline de prédiction `predict_age_from_file()` opérationnel sur tout nouveau CSV de méthylation

Les métriques exactes obtenues sur ce jeu de données sont stockées dans `pipeline_complet.rds`
et affichées automatiquement lors du chargement via `source("load_pipeline.R")`.

## Technologies

- R (≥ 4.1.0)
- Python / TensorFlow / Keras (`reticulate`)
- `glmnet` — Elastic Net / Ridge / Lasso
- `randomForest` — Random Forest
- `xgboost` — Gradient Boosting
- `e1071` — SVM
- `data.table` — lecture et transposition haute performance
- `methylclock` / `methylclockData` — horloges épigénétiques de référence
- `ggplot2` / `kableExtra` — visualisation et rapports

## Rapport interactif (HTML)

Le rapport HTML interactif complet est disponible ici :
https://drive.google.com/file/d/1jiI2ka-PcJ3ST_yijPxMWz-ASISrHSEC/view?usp=drive_link

> Le rapport inclut les visualisations, les courbes d'apprentissage Deep Learning,
> les tableaux de performance et la comparaison aux horloges de référence.

---
