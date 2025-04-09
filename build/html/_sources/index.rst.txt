.. SolarPower-Forecast documentation master file, created by
   sphinx-quickstart on Tue Apr  8 11:24:11 2025.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

SolarPower Forecast: Optimizing Energy Consumption and Panel Efficiency
========================================================================

Ce **ReadTheDoc** a pour objectif de développer un modèle de prévision de la consommation d'énergie générée par des panneaux solaires photovoltaïques, afin d'optimiser la production d'énergie et de déterminer le nombre de panneaux nécessaires pour répondre à la demande énergétique, tout en assurant une maintenance prédictive.

Objectif:
^^^^^^^^^^^^^^

- **Développer un modèle de prévision** de la consommation d'énergie en utilisant des panneaux solaires photovoltaïques, en exploitant les données historiques de consommation d'énergie et de production solaire.
- **Comparer différentes approches de modélisation** des séries temporelles (ARIMA, LSTM, XGBoost, etc.) pour identifier la méthode la plus performante pour prédire la consommation d'énergie et la production des panneaux solaires.
- **Analyser les tendances et les facteurs d'influence** (ensoleillement, température, encrassement des panneaux, etc.) sur la production d'énergie solaire et la demande en énergie.
- **Évaluer les performances du modèle** à l'aide de métriques appropriées (RMSE, MAE, R²) et optimiser les hyperparamètres pour une meilleure précision dans les prévisions.
- **Utiliser les prévisions pour déterminer le nombre de panneaux solaires nécessaires** afin de répondre aux besoins énergétiques tout en maximisant l'efficacité énergétique.

À propos des données:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- **Données de consommation électrique :** Issues de `NYC OpenData <https://data.cityofnewyork.us/Housing-Development/Electric-Consumption-And-Cost-2010-Feb-2025-/jr24-e7cr>`, ces données fournissent l'historique de la consommation et du coût de l'électricité pour divers bâtiments de la ville de New York.  
- **Données de production solaire :** Obtenues à partir du `NSRDB Data Viewer <https://nsrdb.nrel.gov/data-viewer>`, ces données incluent des indicateurs clés du rayonnement solaire, tels que l'irradiance horizontale globale (GHI), l'irradiance normale directe (DNI) et l'irradiance horizontale diffuse (DHI), permettant d'estimer la production d'énergie solaire.  

Bibliotheques utilisées:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- **sklearn.preprocessing**:
  - ``MinMaxScaler``
  - ``StandardScaler``
- **numpy** (``np``)
- **datetime**:
  - ``datetime``
  - ``timedelta``
- **matplotlib**:
  - ``matplotlib.dates`` (``mdates``)
  - ``matplotlib.pyplot`` (``plt``)
- **seaborn** (``sns``)
- **pandas** (``pd``)
- **pvlib**
- **statsmodels**:
  - ``statsmodels.api`` (``sm``)
  - ``adfuller``
  - ``kpss``
  - ``ARIMA``
  - ``seasonal_decompose``
- **optuna**
- **torch**:
  - ``torch``
  - ``torch.nn`` (``nn``)
  - ``torch.optim`` (``optim``)
  - ``torch.utils.data``:
    - ``DataLoader``
    - ``TensorDataset``
    - ``Dataset``



.. toctree::
   :maxdepth: 2
   :caption: Contents:

