.. _pipeline:

================================
Pipeline Méthodologique du Projet
================================

Le projet s'articule autour d'un pipeline complet, allant du traitement des données brutes à la simulation financière interactive. Ce pipeline est orchestré par une série de scripts Python et une application Streamlit, comme illustré ci-dessous.

.. figure:: /Images/Pipeline.png
   :width: 80%
   :align: center
   :alt: Schéma du pipeline de SolarPower Forecast
   :name: pipeline-diagram

Le flux de travail peut être décomposé en trois phases fondamentales.


Phase 1 : Pré-traitement et Ingénierie des Données
--------------------------------------------------

L'objectif de cette phase initiale est de transformer les données sources hétérogènes en jeux de données propres, structurés et prêts pour la modélisation.

Génération des Données de Production Solaire
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

La performance du modèle de génération dépend de la qualité de ses données d'entraînement. Nous utilisons une approche de simulation physique pour créer un jeu de données réaliste.

* **Source** : Données météorologiques **horaires** brutes (GHI, DNI, DHI, Température) du `NSRDB <https://nsrdb.nrel.gov/data-viewer>`_ pour New York City.
* **Outil** : Le script ``generated_energy_estimation.py`` utilise la bibliothèque **pvlib**, un standard de l'industrie pour la modélisation de systèmes photovoltaïques.
* **Processus** : Le script simule la production d'un **système de référence** (défini dans ``config.yaml``, par exemple 10 000 panneaux de 750W) en calculant l'irradiance sur le plan des panneaux (POA), la température des cellules, puis la puissance de sortie horaire.
* **Résultat** : Un fichier ``energy_generated.csv`` contenant la série temporelle de la puissance générée, qui servira de cible pour le modèle LSTM.

Préparation des Données de Coût et de Consommation
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* **Source** : Données de `NYC OpenData <https://data.cityofnewyork.us/Housing-Development/Electric-Consumption-And-Cost-2010-Feb-2025-/jr24-e7cr>`_.
* **Processus** : Le script ``consumed_cost_energy_data.py`` nettoie les données et les agrège à une fréquence **hebdomadaire**. Cette agrégation permet de lisser le bruit à court terme et de mieux capturer la forte saisonnalité annuelle de ces variables.
* **Résultat** : Les fichiers ``energy_cost.csv`` et ``energy_consumed.csv``.

Phase 2 : Entraînement des Modèles Prédictifs
----------------------------------------------

Cette phase utilise les données préparées pour entraîner deux types de modèles, chacun spécialisé pour une tâche spécifique.

Modèle de Génération : LSTM avec Mécanisme d'Attention
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Choisi pour sa capacité à modéliser des dépendances temporelles complexes et non-linéaires.

* **Architecture** : Le modèle ``LSTMAttention`` combine un **LSTM Bidirectionnel**, qui analyse la séquence dans les deux directions temporelles, avec un **mécanisme d'attention** qui permet au modèle de se concentrer sur les pas de temps les plus pertinents de l'historique pour effectuer sa prédiction.
* **Stratégie d'Entraînement** : Nous utilisons une stratégie **Direct Multi-Step**. Le modèle apprend à prédire une séquence de **24 heures futures** en une seule fois. Cette méthode est choisie pour sa robustesse sur les longs horizons de prévision, car elle limite l'accumulation des erreurs par rapport à une approche récursive pas à pas.
* **Régularisation** : Un mécanisme d'**arrêt anticipé (early stopping)** surveille la performance sur un jeu de validation et arrête l'entraînement lorsque le modèle ne s'améliore plus, afin d'éviter le sur-apprentissage.
* **Artefacts** : Les fichiers ``lstm_solar_generator.pth`` (poids du modèle) et ``lstm_scaler.pkl`` (normalisateur de données) sont sauvegardés.

Modèles de Coût et Consommation : SARIMA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Choisi pour son efficacité prouvée dans la modélisation de séries temporelles avec de fortes saisonnalités.

* **Architecture** : Deux modèles **SARIMA** :math:`(p,d,q)(P,D,Q)_s` distincts sont entraînés, un pour le coût et un pour la consommation. La composante saisonnière (:math:`s=52`) est fondamentale pour capturer le cycle annuel.
* **Stratégie d'Entraînement** : Les modèles sont entraînés sur l'intégralité de leurs jeux de données respectifs. Cette approche garantit que les prévisions commencent immédiatement après la dernière donnée historique connue, maximisant ainsi la pertinence pour l'utilisateur final.

Phase 3 : Pipeline d'Inférence et de Simulation (Application ``app.py``)
-------------------------------------------------------------------------

C'est le cœur opérationnel de l'application Streamlit, qui s'exécute en temps réel à la demande de l'utilisateur.

1.  **Génération des Prévisions Brutes**

    * **LSTM** : Une méthode de prévision **itérative** est employée. Le modèle prédit un bloc de 24h, puis utilise cette prédiction comme partie de l'entrée pour prédire le bloc suivant.
    * **SARIMA** : Génèrent des prévisions de totaux **hebdomadaires**.

2.  **Alignement Temporel des Données**

    .. note::
        Cette étape est l'une des contributions techniques les plus importantes du projet. Elle permet de fusionner des prévisions de fréquences différentes en une seule vue cohérente.

    La fonction ``align_forecast_data`` unifie toutes les prévisions sur une grille **horaire** :
        * La consommation **hebdomadaire** est convertie en une **moyenne horaire**.
        * Le coût **hebdomadaire** et la consommation sont utilisés pour dériver un **prix moyen horaire** ($/kWh).

3.  **Simulation Économique et Financière**

    * **Mise à l'échelle (Scaling)** : La prévision de génération (issue du système de référence) est mise à l'échelle pour correspondre précisément au système de l'utilisateur (nombre de panneaux, capacité).
    * **Bilan Énergétique Horaire** : Pour chaque heure, le script calcule :
        * L'**autoconsommation** : énergie solaire utilisée directement, évitant un achat.
        * L'**énergie importée** : déficit d'énergie comblé par le réseau.
        * L'**énergie exportée** : surplus d'énergie vendu au réseau.
    * **Calcul des Économies** : Le script fournit une estimation financière complète basée sur ces flux d'énergie.

4.  **Visualisation des Résultats**
    Les résultats finaux (prévisions mises à l'échelle, bilan financier) sont présentés à l'utilisateur via des graphiques interactifs Plotly.

