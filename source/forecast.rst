.. _forecast:

=======================
Modèles de Prévision
=======================

Cette section décrit l'approche de modélisation hybride adoptée pour le projet. Conscients des caractéristiques distinctes de chaque série temporelle, nous avons sélectionné deux familles de modèles :

1.  Un modèle d'apprentissage profond, **LSTM avec Attention**, pour la production solaire, caractérisée par une haute fréquence et des dynamiques complexes non linéaires.
2.  Des modèles statistiques, **SARIMA**, pour le coût et la consommation d'énergie, qui présentent une saisonnalité annuelle claire et prévisible.

Prévision de la Génération Solaire : LSTM Direct Multi-Step
-------------------------------------------------------------

**Objectif**
^^^^^^^^^^^^

Prévoir la production d'énergie solaire **horaire** (en Watts) pour le système de référence, en se basant sur l'historique des heures précédentes et des caractéristiques temporelles cycliques.

**Justification de l'approche**
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- **Complexité des Données :** La production solaire horaire dépend de cycles multiples (jour/nuit, saisons) et de variations stochastiques (météo) que les réseaux de neurones récurrents comme les LSTM capturent efficacement.
- **Stratégie "Direct Multi-Step" :** Pour garantir la stabilité des prévisions sur de longs horizons, le modèle est entraîné à prédire directement un bloc de 24 heures futures en une seule passe, ce qui limite l'accumulation d'erreurs inhérente aux méthodes purement récursives.
- **Mécanisme d'Attention :** L'ajout d'une couche d'attention permet au modèle de pondérer intelligemment l'influence des différents pas de temps passés, se concentrant sur les informations les plus pertinentes pour formuler sa prédiction.

**Architecture et Entraînement**
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1.  **Préparation des Données :**
    - Les données horaires de génération sont normalisées à l'aide d'un ``StandardScaler``.
    - Les données sont transformées en séquences d'apprentissage où, par exemple, une séquence de **336 heures passées** (``lstm_time_steps``) sert d'entrée pour prédire une séquence de **24 heures futures** (``output_chunk_size``).

2.  **Architecture du Modèle (`models/lstm_model.py`) :**
    - Le modèle est un **LSTM bidirectionnel** qui analyse la séquence d'entrée dans les deux sens pour un enrichissement du contexte.
    - Une **couche d'attention** est appliquée sur les sorties du LSTM pour créer un vecteur de contexte pondéré.
    - La couche de sortie finale est une couche linéaire (`nn.Linear`) qui produit un vecteur de taille ``output_chunk_size``.
    - Une fonction d'activation **ReLU** garantit une production d'énergie toujours positive.

3.  **Processus d'Entraînement (`train/train_lstm.py`) :**
    - Un jeu de **validation** et un mécanisme d'**arrêt anticipé (early stopping)** sont utilisés pour sauvegarder le meilleur modèle et éviter le sur-apprentissage.
    - L'optimiseur **AdamW** avec régularisation (weight decay) et un ordonnanceur de taux d'apprentissage (`ReduceLROnPlateau`) sont employés pour une convergence stable et efficace.


Prévision du Coût et de la Consommation : SARIMA
---------------------------------------------------

**Objectif**
^^^^^^^^^^^^

Prévoir le **coût total hebdomadaire ($)** et la **consommation totale hebdomadaire (kWh)** à partir de leurs données historiques respectives.

**Justification de l'approche**
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- **Saisonnalité Forte :** Les séries de coût et de consommation présentent une saisonnalité annuelle très claire (52 semaines), cas d'usage idéal pour SARIMA.
- **Robustesse et Interprétabilité :** SARIMA est un modèle statistique robuste, interprétable et éprouvé pour les séries temporelles avec des structures saisonnières prévisibles.

**Démarche de Modélisation**
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1.  **Analyse Exploratoire et Identification des Ordres**
    - L'analyse des graphes d'autocorrélation (ACF) et d'autocorrélation partielle (PACF) est utilisée pour déterminer les ordres du modèle.

    .. figure:: /Images/ACF_PACF.png
       :width: 80%
       :align: center
       :alt: Graphes ACF et PACF pour la sélection des ordres SARIMA

    Les graphiques ont révélé une forte saisonnalité annuelle (période :math:`s = 52`) et ont guidé le choix des paramètres suivants (stockés dans ``config.yaml``) :
        - Ordre non saisonnier : :math:`(p,d,q) = (1, 1, 1)`
        - Ordre saisonnier : :math:`(P,D,Q)_s = (1, 1, 0, 52)`

2.  **Entraînement des Modèles (`train_sarima.py`)**
    - Deux modèles SARIMA distincts sont entraînés.
    - Chaque modèle est entraîné sur l'**intégralité de son jeu de données historique** pour que les prévisions futures démarrent immédiatement après la dernière donnée connue.
    - L'instance du modèle entraîné est sauvegardée dans un fichier ``.pkl``.

    .. note::
       La bibliothèque ``pmdarima`` pour la recherche automatique des ordres (Auto-ARIMA) a été envisagée mais n'a pas pu être utilisée en raison de contraintes de mémoire sur l'environnement de test (Kaggle).

3.  **Exemple de Prévision**
    Le graphique ci-dessous illustre la capacité du modèle SARIMA à capturer la tendance et la saisonnalité des données de consommation.

    .. figure:: /Images/SARIMA.png
       :width: 80%
       :align: center
       :alt: Résultats de la prévision SARIMA