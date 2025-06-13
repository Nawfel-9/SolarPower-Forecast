.. _setup:

==================================
Installation et Lancement
==================================

Ce guide vous guidera pour la mise en place de l'environnement, la préparation des données, l'entraînement des modèles et l'exécution de l'application Streamlit.

1. Prérequis
------------

- **Git** : Nécessaire pour cloner le dépôt. Téléchargeable sur `git-scm.com <https://git-scm.com/>`_.
- **Python** : Version 3.9 ou plus récente recommandée. Téléchargeable sur `python.org <https://www.python.org/>`_.

2. Configuration de l'Environnement
------------------------------------

a. Cloner le Dépôt
^^^^^^^^^^^^^^^^^^^
Ouvrez votre terminal ou invite de commandes et exécutez la commande suivante :

.. code-block:: bash

    git clone https://github.com/Nawfel-9/solar_forecasting_project
    cd solar_forecasting_project

b. Créer et Activer un Environnement Virtuel
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
L'utilisation d'un environnement virtuel est une bonne pratique pour isoler les dépendances du projet.

.. code-block:: bash

    # Créer l'environnement
    python -m venv venv

Activez ensuite l'environnement :

- Sur **Windows** :

  .. code-block:: bash

      venv\Scripts\activate

- Sur **Linux/macOS** :

  .. code-block:: bash

      source venv/bin/activate

.. note::
    Si vous utilisez Conda, les commandes équivalentes seraient :
    ``conda create -n solar_env python=3.9 -y`` et ``conda activate solar_env``.

c. Installer les Dépendances
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Une fois votre environnement activé, installez toutes les bibliothèques nécessaires en une seule commande à l'aide du fichier ``requirements.txt``. Ce fichier inclut PyTorch et toutes les autres dépendances.

.. code-block:: bash

    pip install -r requirements.txt


3. Préparation et Entraînement (Optionnel)
--------------------------------------------

Suivez ces étapes si vous souhaitez recréer les modèles à partir des données brutes. Si vous souhaitez simplement lancer l'application avec les modèles pré-entraînis fournis, vous pouvez passer à l'étape 4.

a. Télécharger les Données Brutes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Téléchargez l'ensemble de données depuis Kaggle : `Electric Consumption And Cost <https://kaggle.com/datasets/77683f114a97ab3ad9f7cfd138528bb1269836a29e085c56e24190f140d3303a>`_.

Placez les fichiers CSV téléchargés dans un dossier ``data/`` à la racine de votre projet.

b. Exécuter les Scripts de Prétraitement
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. IMPORTANT::
    Avant d'exécuter, ouvrez les scripts ``config.yaml``. Assurez-vous que les chemins de fichiers à l'intérieur de ce script pointent correctement vers l'emplacement où vous avez sauvegardé les données brutes.

Exécutez les scripts pour préparer les données pour l'entraînement :

.. code-block:: bash

    python consumed_cost_energy_data.py
    python generated_energy_estimation.py

c. Entraîner les Modèles
^^^^^^^^^^^^^^^^^^^^^^^^^^^
Une fois les données prétraitées, vous pouvez entraîner les modèles SARIMA et/ou LSTM.

.. code-block:: bash

    # Entraîner les modèles SARIMA (rapide)
    python train/train_sarima.py

    # Entraîner le modèle LSTM (peut être long)
    # Assurez-vous que run_optuna_search est à 'false' dans config.yaml
    python train/train_lstm.py


4. Lancer l'Application Streamlit
---------------------------------

Assurez-vous que votre environnement virtuel est toujours actif et que vous êtes dans le répertoire racine du projet.

.. code-block:: bash

    streamlit run app.py

Votre navigateur par défaut devrait s'ouvrir à l'adresse de l'application (généralement ``http://localhost:8501``).

.. note::
    Si vous rencontrez une erreur "commande introuvable", essayez de lancer Streamlit en tant que module Python :
    ``python -m streamlit run app.py``

5. Arrêter l'Application
--------------------------

Pour arrêter le serveur Streamlit, retournez au terminal où il s'exécute et appuyez sur ``Ctrl+C``.
