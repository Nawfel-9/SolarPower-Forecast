Pipeline
========================================================================

Le projet se compose de sept phases principales, comme illustré dans la figure ci-dessous:

.. figure:: /Images/Pipeline.png
   :width: 60%
   :align: center
   :alt: SolarPower Forecast's pipeline
   :name: pipeline

1. Collecte & Préparation des données
---------------------------------------

   * Rassembler les données historiques :

      * Irradiance sur le plan des panneaux (POA) pour estimer la production solaire (par panneau ou installation)

      * Consommation électrique de la ville (par semaine/mois)

      * Prix de l'électricité (par semaine/mois)

   * Nettoyage / Normalisation / standardisation si nécessaire.

   * Synchronisation des fréquences temporelles (ex : horaire, journalier)

2. Analyse exploratoire (EDA)
-------------------------------

   * Visualisation des séries temporelles

   * Analyse des tendances, saisonnalités et anomalies

3. Analyse de la structure temporelle
-------------------------------------------

   * Tests de stationnarité (ADF, KPSS)

   * Décomposition des séries (tendance, saisonnalité, résidu)

   * Analyse ACF / PACF

4. Modélisation
------------------

   * Choix du modèle en fonction des caractéristiques :

      * ARIMA / SARIMA / SARIMAX
      * LSTM / GRU (réseaux de neurones récurrents)
      * Prophet (facile à interpréter et robuste aux saisons)

   * Modèles de machine learning (XGBoost, Random Forest, etc.)

   * Entraînement et validation croisée (train/test split)

5. Évaluation du modèle
--------------------------------

   * Calcul des métriques de performance (RMSE, MAE, MAPE, R²)

   * Comparaison des modèles

   * Optimisation des hyperparamètres (GridSearchCV, Optuna...)

6. Prévisions
------------------------

   * Génération de prévisions à court et long terme

   * Visualisation des résultats

7. Simulation économique
------------------------------

   * Calcul de l'énergie autoconsommée grâce aux panneaux

   * Économie = (production autoconsommée × prix de l'électricité)

   * Simulation selon le nombre de panneaux (par palier : 10000, etc.)

   * Visualisation de l'économie potentielle en fonction de l'installation

Flux de travail
---------------------------------------

.. figure:: /Images/Workflow..png
   :width: 60%
   :align: center
   :alt: Flux de travail pour la prévision de la consommation d'énergie et l'évaluation des économies liées aux panneaux solaires
   :name: workflow