.. _recherche:

============================================
Démarche de Recherche et Itérations du Modèle
============================================

L'élaboration d'un modèle de prévision performant est un processus itératif. Cette page documente le parcours de recherche, les défis rencontrés et les solutions techniques mises en œuvre pour arriver au modèle final utilisé dans ce projet.


Étape 1 : Le Modèle LSTM de Base et la Prévision Récursive
-----------------------------------------------------------
La première approche, la plus intuitive, consistait à utiliser un modèle LSTM standard pour prédire la production d'énergie heure par heure.

**Stratégie et Implémentation :**
- **Architecture :** Un simple LSTM unidirectionnel dont la tâche était de prédire une seule valeur future (``output_size=1``).
- **Prévision Récursive :** La prévision à long terme était générée de manière auto-régressive. Le modèle prédisait l'heure `t+1` à partir de l'historique jusqu'à `t`. Ensuite, cette prédiction pour `t+1` était ajoutée à la séquence d'entrée pour prédire `t+2`, et ainsi de suite.

**Défi Principal : Dégradation de la Prévision (Forecast Decay)**
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Cette méthode s'est avérée très instable sur des horizons de temps longs. Si le modèle était performant pour prédire les prochaines heures, les erreurs s'accumulaient rapidement.

.. figure:: /Images/prediction.png
   :width: 80%
   :align: center
   :alt: Dégradation de la prévision sur un an avec le modèle récursif.

   Illustration de la dégradation : la prévision (en orange) perd rapidement le contexte saisonnier et s'aplatit, incapable de reproduire la courbe annuelle des données historiques (en bleu).

.. admonition:: Analyse de l'Échec

   Le problème provenait du décalage entre l'entraînement et l'inférence :

   1.  **Pendant l'entraînement (Teacher Forcing) :** Le modèle apprend à prédire `t+1` en se basant sur un historique de données 100% réelles et parfaites.
   2.  **Pendant l'inférence :** Le modèle se base sur ses propres prédictions, qui contiennent inévitablement de petites erreurs. Ces erreurs sont réinjectées en boucle, corrompant la séquence d'entrée et faisant "dérailler" la prévision. Le modèle perdait tout contexte saisonnier à long terme.

---

Étape 2 : Tentatives d'Optimisation et Limites de l'Approche
-----------------------------------------------------------------
Avant de changer de stratégie, nous avons tenté d'améliorer le modèle existant.

1.  **Optimisation des Hyperparamètres :** Une recherche avec la bibliothèque **Optuna** a été lancée pour trouver les meilleurs ``learning_rate``, ``dropout``, ``hidden_size``, etc. Bien que cela ait amélioré les performances à court terme, le problème de la dégradation à long terme persistait.
2.  **Augmentation de la Complexité :** Un modèle plus puissant (LSTM Bidirectionnel avec plus de couches) a été testé. Le résultat fut un **sur-apprentissage (overfitting)** encore plus rapide.

.. note::
   **Conclusion de l'Étape 2 :** Le problème n'était pas un manque de puissance du modèle, mais un **manque fondamental d'information en entrée**. Le modèle ne pouvait pas prédire la saisonnalité car il n'avait aucun moyen de "savoir" quel moment de l'année il prévoyait.

---

Étape 3 : La Percée - Ingénierie de Caractéristiques Temporelles
------------------------------------------------------------------
La solution a été de fournir explicitement au modèle un "calendrier" en enrichissant les données.

**Solution Implémentée :**
- **Transformation en problème multivarié :** Au lieu d'une seule série en entrée (la production), le modèle reçoit maintenant **5 caractéristiques (features)** pour chaque heure.
- **Création de caractéristiques cycliques :** Pour éviter que le modèle ne considère l'heure 23 comme étant "loin" de l'heure 0, nous avons utilisé des transformations sinus/cosinus.
    - **Cycle journalier :** :math:`\sin(2\pi \cdot \frac{\text{heure}}{24})` et :math:`\cos(2\pi \cdot \frac{\text{heure}}{24})`
    - **Cycle annuel :** :math:`\sin(2\pi \cdot \frac{\text{jour}}{365.25})` et :math:`\cos(2\pi \cdot \frac{\text{jour}}{365.25})`
- **Mise à jour de la configuration :** Le paramètre ``lstm_params.input_size`` dans ``config.yaml`` a été changé de `1` à `5`.

Le script ``generated_energy_estimation.py`` a été modifié pour ajouter ces caractéristiques au fichier de données d'entraînement.

---

Étape 4 : Changement de Stratégie - Prévision "Direct Multi-Step"
------------------------------------------------------------------------
Avec des données d'entrée plus riches, la stratégie de prévision a également été revue pour être intrinsèquement plus stable.

**Solution Implémentée :**
- **Prédiction par blocs (Chunks) :** Le modèle a été ré-architecturé pour prédire un bloc de **24 heures en une seule fois**.
    - **Architecture :** La couche finale du LSTM (``nn.Linear``) a maintenant une taille de sortie de `24` (via ``output_chunk_size``) au lieu de 1.
    - **Préparation des données :** La fonction ``create_lstm_sequences`` a été modifiée. La cible `y` pour chaque échantillon d'entraînement n'est plus une valeur unique, mais un vecteur de 24 valeurs futures.

.. code-block:: python
   :caption: Logique de create_lstm_sequences pour le multi-step

   # y est maintenant une tranche de 'output_chunk_size' (ex: 24) valeurs
   y.append(data[(i + time_steps):(i + time_steps + output_chunk_size), 0])

- **Avantage :** Cette méthode réduit considérablement l'accumulation d'erreurs. L'erreur est maintenant accumulée de bloc en bloc (toutes les 24 heures) et non plus à chaque heure.

---

Étape 5 : La Synthèse Finale - Architecture Avancée et Optimisation
---------------------------------------------------------------------
La dernière étape a consisté à combiner toutes les leçons apprises pour créer le modèle le plus performant.

**Solution Finale :**
1.  **Architecture Avancée :** Un modèle **LSTM Bidirectionnel avec un Mécanisme d'Attention** a été implémenté dans ``models/lstm_model.py``.
    - Le **LSTM Bidirectionnel** peut désormais être utilisé efficacement, car les caractéristiques temporelles lui fournissent le contexte nécessaire.
    - Le **Mécanisme d'Attention** agit comme une couche de "focus", permettant au modèle de peser dynamiquement l'importance de chaque heure de l'historique. Il peut ainsi apprendre à donner plus de poids aux caractéristiques saisonnières pour les prévisions lointaines, ce qui combat directement la dégradation de la prévision.

2.  **Optimisation Finale avec Optuna :** Une nouvelle recherche d'hyperparamètres a été menée pour trouver la combinaison optimale de ``learning_rate``, ``dropout``, ``hidden_size``, etc., spécifiquement pour cette nouvelle architecture avancée.

**Conclusion de la Recherche**
------------------------------
Le modèle final est une synthèse de ces itérations. Chaque étape a résolu un problème spécifique identifié précédemment. Le résultat est un modèle qui non seulement évite la dégradation des prévisions à long terme grâce aux caractéristiques temporelles et à la stratégie directe, mais qui le fait avec une précision optimisée grâce à une architecture avancée et à des hyperparamètres rigoureusement sélectionnés.
