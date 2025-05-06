Prévision
======================
Cette section décrit les méthodologies utilisées pour prévoir à la fois la consommation d'énergie de la ville et la production d'énergie solaire. Deux approches de modélisation différentes ont été choisies en fonction de la nature et de la taille des données : un modèle statistique (SARIMA) pour la production solaire et des modèles d'apprentissage profond (LSTM ou Transformers) pour la consommation d'énergie.

Prévision de la Consommation d'Énergie (SARIMA)
-------------------------------------------------------------------

**Objectif**
~~~~~~~~~~~~~~

Prévoir la consommation énergétique future de la ville à partir des données historiques.

**Pourquoi utiliser SARIMA ?**

* Le jeu de données est relativement petit et présente de la saisonnalité et des tendances.
* SARIMA est bien adapté aux séries temporelles univariées avec des composantes saisonnières claires.
* Il offre des paramètres interprétables et des performances stables pour les petits ensembles de données.

**Étapes Suivies**
~~~~~~~~~~~~~~~~~~~~~~

1. **Prétraitement des Données:**
    * Praitement des valeurs manquantes et des valeurs aberrantes.

    * Rééchantillonnage des données (par exemple : moyennes journalières ou mensuelles).

Cette fonction permet de générer des données hebdomadaires à partir de données mensuelles,
en ajoutant une variation aléatoire pour simuler un comportement plus réaliste.

.. code-block:: python

    # Ajouter de l'aléatoire pour rendre les données plus réalistes
    def creer_donnees_hebdomadaires_avec_variation(donnees_mensuelles, date_debut='2010-01-01', variation=0.15):
        np.random.seed(42)
        donnees_hebdomadaires = []
        index_dates = []
        date_courante = pd.to_datetime(date_debut)

        for valeur_mensuelle in donnees_mensuelles:
            # Déterminer le nombre de semaines dans le mois
            semaines = 4 + np.random.choice([0, 1])  # 4 ou 5 semaines

            # Générer des facteurs de variation
            variations = np.random.normal(loc=1.0, scale=variation, size=semaines)
            variations = variations / variations.sum()  # Normalisation

            # Appliquer les variations aux valeurs hebdomadaires
            valeurs_semaine = valeur_mensuelle * variations

            # Ajouter les données hebdomadaires
            for valeur in valeurs_semaine:
                donnees_hebdomadaires.append(valeur)
                index_dates.append(date_courante)
                date_courante += timedelta(days=7)

        return pd.Series(donnees_hebdomadaires, index=index_dates)

-
    * Séparation des données en ensembles d'entraînement, de validation et de test.

2. **Sélection du Modèle**

    **a. Analyse ACF et PACF pour déterminer (p, d, q) et les composantes saisonnières (P, D, Q, s).**

.. figure:: /Images/ACF_PACF.png
   :width: 100%
   :align: center
   :alt: ACF et PACF pour la sélection du modèle SARIMA
   :name: ACF_PACF

   ACF et PACF utilisés pour guider la sélection du modèle SARIMA.

Les graphiques ACF et PACF montrent une forte saisonnalité hebdomadaire (période ``s = 52``) et suggèrent les paramètres suivants :

- **Pour l'énergie consommée** : ``SARIMA(1, 1, 1)(1, 1, 1, 52)``
- **Pour le coût de l'énergie** : ``SARIMA(2, 1, 1)(1, 1, 1, 52)``

Ces choix sont basés sur :

- Un **ralentissement progressif** de l'ACF → composante MA (``q``) et saisonnière (``Q``)
- Une **chute brutale** de la PACF → composante AR (``p``) et saisonnière (``P``)

Les différenciations (``d = 1``, ``D = 1``) permettent de stabiliser la tendance et la saisonnalité.

Le choix final est validé par l'analyse des critères AIC/BIC.

.
    **b. Utilisation de** ``statsmodels`` **ou** ``pmdarima`` **pour une sélection automatique.**

.. note::

   En raison du manque de puissance machine, nous n'avons pas pu utiliser la bibliothèque ``pmdarima`` 
   (30 Go de RAM de kaggle n'ont pas été suffisants).

3. **Resultats et Évaluation**

.. figure:: /Images/SARIMA.png
   :width: 100%
   :align: center
   :alt: SARIMA Forecasting Results
   :name: SARIMA_Results

   Résultat de la prévision SARIMA sur les données de consommation d'énergie.
