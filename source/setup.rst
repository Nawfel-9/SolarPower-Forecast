Mise en Place et Déploiement du Projet
==================================================

Ces instructions vous guideront pour la mise en place de l'environnement du projet, la préparation des données, l'entraînement du modèle nécessaire et l'exécution de l'application Streamlit.

**1. Prérequis**
--------------------

**Git** : Vous aurez besoin de Git installé pour cloner le dépôt. Vous pouvez le télécharger depuis `https://git-scm.com/ <https://git-scm.com/>`_.

**Python** : Python 3.8 ou une version plus récente est recommandé. Vous pouvez le télécharger depuis `https://www.python.org/ <https://www.python.org/>`_. Pip (l'installateur de paquets Python) sera inclus avec votre installation de Python.

**2. Configuration de l'Environnement et Installation**
-----------------------------------------------------------

**a. Cloner le Dépôt**
~~~~~~~~~~~~~~~~~~~~~~~~~~~
Ouvrez votre terminal ou invite de commandes et exécutez les commandes suivantes :

.. code-block:: bash

    git clone https://github.com/Nawfel-9/SolarPower-Forecast.git
    cd SolarPower-Forecast

**b. Créer et Activer un Environnement Virtuel**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Il est fortement recommandé d'utiliser un environnement virtuel pour gérer les dépendances du projet et éviter les conflits.

Créez l'environnement virtuel (par exemple, nommé venv) :

.. code-block:: bash

    python -m venv venv

(Alternativement, si vous préférez Conda, vous pouvez créer un environnement avec une commande comme : ``conda create -n solarforecast_env python=3.8 -y``)

Activez l'environnement virtuel :

- Windows :

  .. code-block:: bash

      venv\Scripts\activate

- Linux/macOS :

  .. code-block:: bash

      source venv/bin/activate

(Si vous utilisez Conda, activez avec : ``conda activate solarforecast_env``)

Vous devriez voir le nom de l'environnement virtuel (par exemple, (venv) ou (solarforecast_env)) au début de votre invite de terminal.

**c. Installer les Bibliothèques Requises**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Avec votre environnement virtuel activé, installez les paquets Python nécessaires.
Premièrement, installez PyTorch (la commande ci-dessous spécifie la version CPU pour une compatibilité plus large ou vérifiez requirements.txt si vous voulez utiliser le GPU) :

.. code-block:: bash

    pip install --no-cache-dir torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu

Ensuite, installez les dépendances restantes du projet listées dans requirements.txt :

.. code-block:: bash

    pip install -r requirements.txt

**3. Préparation des Données et Entraînement du Modèle (Pour Recréer les Résultats)**
----------------------------------------------------------------------------------------
Suivez ces étapes si vous souhaitez recréer l'ensemble de données à partir de zéro et entraîner le modèle SARIMA. Notez qu'un point de contrôle du modèle LSTM pré-entraîné est déjà inclus dans le dépôt, cette section peut donc être facultative si vous souhaitez uniquement exécuter l'application avec les modèles existants.

**a. Télécharger l'Ensemble de Données**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Téléchargez l'ensemble de données depuis Kaggle : `Solar Power Generation and Consumption Dataset <https://kaggle.com/datasets/77683f114a97ab3ad9f7cfd138528bb1269836a29e085c56e24190f140d3303a>`_
(Un compte Kaggle peut être requis pour le téléchargement.)
Extrayez les fichiers téléchargés et placez-les dans le répertoire ``data/`` de votre dossier de projet SolarPower-Forecast. Créez le répertoire ``data/`` s'il n'existe pas.

**b. Exécuter les Scripts de Prétraitement**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Ces scripts traiteront les données brutes dans un format adapté à l'entraînement du modèle et à l'utilisation de l'application.

.. code-block:: bash

    python consumed_cost_energy_data.py
    python generated_energy_estimation.py

**IMPORTANT** : Avant d'exécuter ces scripts, vous devez ouvrir ``consumed_cost_energy_data.py`` et ``generated_energy_estimation.py`` dans un éditeur de texte. Vérifiez et mettez à jour attentivement tous les chemins de fichiers internes dans ces scripts pour qu'ils pointent correctement vers vos données d'entrée (téléchargées à l'étape 3a) et vos emplacements de sortie souhaités pour les données traitées.

**c. Entraîner le Modèle SARIMA**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Après que les données ont été traitées avec succès, exécutez le script d'entraînement SARIMA. Cette étape ne doit généralement être effectuée qu'une seule fois.

.. code-block:: bash

    python train/train_sarima.py

(Comme mentionné, le modèle LSTM dispose d'un point de contrôle pré-chargé disponible dans le dépôt, donc le ré-entraîner pourrait ne pas être nécessaire pour exécuter l'application.)

**4. Exécuter l'Application**
---------------------------------
Une fois la configuration terminée et, si nécessaire, les étapes de préparation des données et d'entraînement du modèle effectuées, vous pouvez exécuter l'application web Streamlit.
Assurez-vous que votre environnement virtuel est toujours actif.
Dans votre terminal, assurez-vous d'être dans le répertoire racine du projet (SolarPower-Forecast).
Lancez l'application Streamlit en utilisant la commande suivante :

.. code-block:: bash

    streamlit run app.py

Cette commande démarre généralement un serveur web local et ouvre l'application dans votre navigateur web par défaut.

**Note de Dépannage** : Si la commande ``streamlit run app.py`` résulte en une erreur comme "'streamlit' n'est pas reconnu..." ou "commande introuvable : streamlit", cela pourrait indiquer un problème avec le PATH de votre système ou l'environnement virtuel. Dans de tels cas, essayez d'exécuter Streamlit en tant que module Python :

.. code-block:: bash

    python -m streamlit run app.py

**5. Arrêter l'Application**
--------------------------------
Pourarrêter l'application Streamlit, retournez à la fenêtre du terminal où elle s'exécute et appuyez sur Ctrl+C.