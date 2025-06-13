.. _introduction:

=======================
SolarPower Forecast
=======================

.. figure:: /Images/Banner.png
   :width: 60%
   :align: center
   :alt: Logo de SolarPower Forecast
   :name: logo

Contexte
---------
Ce projet a pour objectif de réaliser une prévision temporelle de l'énergie produite par des panneaux solaires, de la consommation électrique, ainsi que du coût de l'électricité. L’objectif final est d’estimer les économies financières réalisables en fonction d'une configuration de panneaux solaires définie par l'utilisateur.

À partir de données historiques, nous modélisons l'évolution de la production et de la consommation d'énergie, en prenant également en compte les variations du prix de l'électricité. Le projet se matérialise par une application **Streamlit** interactive où un utilisateur peut spécifier le nombre de panneaux et leur capacité pour simuler différents scénarios et quantifier les gains économiques associés.

Ce projet s'inscrit dans une logique de transition énergétique et d'optimisation des investissements dans les énergies renouvelables.

Objectifs du Projet
--------------------
- **Développer des modèles de prévision** pour les trois variables clés du projet :

    - La **production d'énergie solaire** (horaire), en utilisant un modèle neuronal de type **LSTM (Long Short-Term Memory)** avec une stratégie de prévision *Direct Multi-Step*.
    - La **consommation d'énergie** (hebdomadaire), en utilisant un modèle statistique **SARIMA**.
    - Le **coût de l'énergie** (hebdomadaire), également en utilisant un modèle **SARIMA**.

- **Analyser les tendances et les facteurs d'influence** (ensoleillement, température, etc.) qui sont utilisés pour estimer la production d'énergie solaire via le script ``generated_energy_estimation.py``.

- **Évaluer la performance des modèles** à l'aide de métriques appropriées (RMSE, MAE) lors de la phase d'entraînement, avec un suivi des courbes de perte sur un jeu de validation.

- **Fournir un outil d'aide à la décision** via une application Streamlit. L'utilisateur fournit une configuration de système solaire (nombre de panneaux, capacité, etc.) et l'application utilise les prévisions pour estimer les économies financières potentielles.

Défis à relever
----------------
La réalisation de ce projet soulève plusieurs défis techniques et méthodologiques :

- **Qualité des données :** L'exactitude des prévisions dépend fortement de la qualité des données historiques. Les données manquantes ou bruitées peuvent impacter les performances des modèles.

- **Hétérogénéité des séries temporelles :** Les trois séries principales ont des fréquences différentes, ce qui complexifie leur alignement pour la simulation. Nous gérons :

    - La **production solaire** à une fréquence **horaire**.
    - La **consommation** et le **coût** de l'énergie à une fréquence **hebdomadaire**.

- **Saisonnalité et variabilité :** La production solaire dépend fortement de la météo et des saisons. Capturer ces variations avec précision est un défi majeur pour le modèle LSTM.

- **Stratégie de prévision à long terme :** Prédire sur de longs horizons (plusieurs mois ou un an) introduit un risque d'accumulation d'erreurs. Nous utilisons une stratégie **Direct Multi-Step** pour le LSTM afin de mitiger ce risque en prédisant des blocs de 24 heures à la fois.

- **Simulation économique réaliste :** Traduire les prévisions en économies financières nécessite de modéliser l'interaction complexe entre la production et la consommation sur une base horaire, en calculant l'**autoconsommation**, l'**énergie importée** du réseau et l'**énergie exportée**. La simulation inclut également une logique de **mise à l'échelle (scaling)** pour adapter la prévision de génération du système de référence à la configuration définie par l'utilisateur.

À propos des données
------------------------
- **Données de consommation et de coût :** Issues de `NYC OpenData <https://data.cityofnewyork.us/Housing-Development/Electric-Consumption-And-Cost-2010-Feb-2025-/jr24-e7cr>`_, ces données fournissent l'historique de la consommation et du coût de l'électricité pour divers bâtiments de la ville de New York. Pour ce projet, elles sont agrégées à une fréquence **hebdomadaire**.
- **Données de production solaire :** Obtenues à partir du `NSRDB Data Viewer <https://nsrdb.nrel.gov/data-viewer>`_, ces données **horaires** incluent des indicateurs clés du rayonnement solaire (GHI, DNI, DHI) et la température, permettant d'estimer la production d'énergie solaire horaire via le script ``generated_energy_estimation.py``.

Bibliothèques Utilisées
---------------------------
* **Analyse de Données et Calculs :** ``numpy``, ``pandas``
* **Modélisation Statistique et Prétraitement :**
    * ``statsmodels``
    * ``scikit-learn``
* **Apprentissage Profond (Deep Learning) :** ``torch``
* **Simulation Photovoltaïque :** ``pvlib``
* **Application Web et Visualisation :** ``streamlit``, ``matplotlib``, ``plotly``
* **Utilitaires et Sauvegarde :** ``joblib``, ``pathlib``, ``datetime``
