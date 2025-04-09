SolarPower Forecast 
======================

Context:
^^^^^^^^^^^^^^^^^^^^
Ce projet a pour objectif de réaliser une prévision temporelle de l'énergie produite par des panneaux solaires, de la consommation électrique d'une ville, ainsi que du coût de l'électricité. L’objectif final est d’estimer les économies financières réalisables en fonction du nombre de panneaux solaires installés.

À partir de données historiques, nous modélisons l'évolution de la production et de la consommation d'énergie, en prenant également en compte les variations du prix de l'électricité. Ces prévisions permettent de simuler différents scénarios d'installation de panneaux solaires et de quantifier les gains économiques associés.

Ce projet s'inscrit dans une logique de transition énergétique et d'optimisation des investissements dans les énergies renouvelables.





Objectif:
^^^^^^^^^^^^^^
- **Développer un modèle de prévision** de la consommation d'énergie en utilisant des panneaux solaires photovoltaïques, en exploitant les données historiques de consommation d'énergie et de production solaire.
- **Comparer différentes approches de modélisation** des séries temporelles (ARIMA, LSTM, XGBoost, etc.) pour identifier la méthode la plus performante pour prédire la consommation d'énergie et la production des panneaux solaires.
- **Analyser les tendances et les facteurs d'influence** (ensoleillement, température, encrassement des panneaux, etc.) sur la production d'énergie solaire et la demande en énergie.
- **Évaluer les performances du modèle** à l'aide de métriques appropriées (RMSE, MAE, R²) et optimiser les hyperparamètres pour une meilleure précision dans les prévisions.
- **Utiliser les prévisions pour déterminer le nombre de panneaux solaires nécessaires** afin de répondre aux besoins énergétiques tout en maximisant l'efficacité énergétique.

À propos des données:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
- **Données de consommation électrique :** Issues de `NYC OpenData <https://data.cityofnewyork.us/Housing-Development/Electric-Consumption-And-Cost-2010-Feb-2025-/jr24-e7cr>`_, ces données fournissent l'historique de la consommation et du coût de l'électricité pour divers bâtiments de la ville de New York.  
- **Données de production solaire :** Obtenues à partir du `NSRDB Data Viewer <https://nsrdb.nrel.gov/data-viewer>`_, ces données incluent des indicateurs clés du rayonnement solaire, tels que l'irradiance horizontale globale (GHI), l'irradiance normale directe (DNI) et l'irradiance horizontale diffuse (DHI), permettant d'estimer la production d'énergie solaire.  

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