.. _poa:

===============================================================
De l'Irradiance Solaire à la Production d'Énergie Électrique
===============================================================

Pour prédire la génération d'énergie, il est fondamental de comprendre et de modéliser la conversion du rayonnement solaire en électricité. Cette section détaille les concepts physiques et leur implémentation dans le projet via la bibliothèque ``pvlib``.

Irradiance en Plan des Modules (POA)
------------------------------------

L'irradiance en plan des modules (Plane of Array - POA) représente la quantité totale de rayonnement solaire qui frappe une surface inclinée, comme celle d'un panneau photovoltaïque. C'est la métrique la plus importante pour estimer la production d'énergie.

L'irradiance POA se décompose en trois composantes :

1.  **Irradiance Directe (*Beam*)** : Rayonnement provenant directement du disque solaire.
2.  **Irradiance Diffuse (*Diffuse*)** : Rayonnement dispersé par les nuages et l'atmosphère.
3.  **Irradiance Réfléchie (*Reflected*)** : Rayonnement réfléchi par le sol et d'autres surfaces vers le panneau.

.. math::

    G_{\text{POA}} = G_{\text{directe, POA}} + G_{\text{diffuse, POA}} + G_{\text{réfléchie, POA}}

Implémentation dans le Projet
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Le calcul manuel de ces composantes nécessite des modèles géométriques et atmosphériques complexes. Heureusement, la bibliothèque ``pvlib`` encapsule cette complexité.

Dans le script ``generated_energy_estimation.py``, nous utilisons la fonction ``pvlib.irradiance.get_total_irradiance`` qui calcule la POA totale en se basant sur la position du soleil, l'orientation du panneau et les composantes du rayonnement mesurées au sol (GHI, DNI, DHI).

.. code-block:: python
   :emphasize-lines: 3

    # Calcul de la position du soleil
    solpos = pvlib.solarposition.get_solarposition(df.index, latitude, longitude)

    # Calcul de l'irradiance totale sur le plan des panneaux (POA)
    poa_irradiance = pvlib.irradiance.get_total_irradiance(
        surface_tilt=surface_tilt,
        surface_azimuth=180, # Orienté plein sud
        solar_zenith=solpos['zenith'],
        solar_azimuth=solpos['azimuth'],
        dni=df['DNI'],
        ghi=df['GHI'],
        dhi=df['DHI'],
        albedo=albedo
    ).fillna(0)

De l'Irradiance à la Puissance Électrique
------------------------------------------
Une fois que nous connaissons la quantité d'énergie solaire qui frappe les panneaux (:math:`G_{\text{POA}}`), nous devons la convertir en puissance électrique. Ce processus dépend de deux facteurs principaux : la **température des cellules** et les **caractéristiques intrinsèques du module**.

1. Température de Cellule
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
La performance d'un panneau photovoltaïque diminue à mesure que sa température augmente. Il est donc crucial de l'estimer.

* **Implémentation** : Le projet utilise le *Sandia Array Performance Model (SAPM)*, un modèle reconnu implémenté dans ``pvlib``. Il estime la température des cellules en fonction de l'irradiance, de la température ambiante et de la vitesse du vent.

.. code-block:: python

    # Estime la température des cellules photovoltaïques
    df['cell_temperature'] = pvlib.temperature.sapm_cell(
        poa_global=poa_irradiance['poa_global'],
        temp_ambient=df['Ambient Temperature'],
        wind_speed=1.0 # Vitesse du vent supposée constante
    )


2. Puissance en Courant Continu (DC)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
La puissance DC finale est calculée en utilisant le modèle **PVWatts**, qui est largement utilisé par le NREL (National Renewable Energy Laboratory). Ce modèle prend en compte la puissance nominale du système, l'irradiance effective reçue et les pertes de performance dues à la température.

* **Implémentation** :

.. code-block:: python

    # Calcule la puissance DC en utilisant le modèle PVWatts
    power_dc = pvlib.pvsystem.pvwatts_dc(
        g_poa_effective=poa_irradiance['poa_global'],
        temp_cell=df['cell_temperature'],
        pdc0=system_pdc0_watts, # Puissance nominale DC du système total
        gamma_pdc=temp_coeff_pdc # Coefficient de perte de puissance par °C
    ).fillna(0)

.. note::
    Les paramètres ``system_pdc0_watts`` (puissance nominale totale du système de référence) et ``temp_coeff_pdc`` (coefficient de température) sont définis dans le fichier ``config.yaml``. Ces valeurs sont typiquement issues de la fiche technique du panneau solaire, comme le `SR6-HJT725-750M <https://www.enfsolar.com/pv/panel-datasheet/crystalline/65662>`_ utilisé comme référence dans ce projet.

3. Conversion en Puissance AC (Optionnelle)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Bien que notre modèle LSTM soit entraîné sur la puissance DC, il est bon de noter que la puissance finale injectée dans le réseau (AC) serait légèrement inférieure en raison des pertes de conversion dans l'onduleur.

.. math::

   P_{\text{AC}} = P_{\text{DC}} \cdot \eta_{\text{onduleur}}

L'efficacité de l'onduleur (:math:`\eta_{\text{onduleur}}`) est prise en compte dans la phase de simulation financière de l'application via le paramètre ``panel_efficiency``.


Conclusion
----------
En combinant ces modèles physiques robustes de la bibliothèque ``pvlib``, nous sommes capables de générer un jeu de données de production d'énergie horaire synthétique mais réaliste. Ce jeu de données de haute qualité est la fondation sur laquelle repose l'entraînement de notre modèle de prévision LSTM.
