Irradiance en Plan des Modules (POA)
====================================

Qu'est-ce que l'irradiance POA ?
--------------------------------
L'irradiance en plan des modules (Plane of Array - POA) représente la quantité de rayonnement solaire reçue par une surface inclinée, comme celle d'un panneau photovoltaïque. Contrairement à une surface horizontale, les panneaux solaires sont souvent inclinés pour maximiser la réception du rayonnement. Le calcul de l'irradiance POA est donc essentiel pour estimer précisément la production d’énergie solaire.

L'irradiance POA est constituée de trois composantes principales :

1. **Irradiance directe (beam)** : lumière solaire reçue directement du soleil.
2. **Irradiance diffuse** : lumière dispersée par l’atmosphère.
3. **Irradiance réfléchie (albédo)** : lumière réfléchie par le sol vers le panneau.

Méthode de calcul de l'irradiance POA
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
L'irradiance totale sur le plan des modules, notée :math:`G_{POA}`, est la somme de ces trois composantes :

.. math::

    G_{POA} = G_{b,POA} + G_{d,POA} + G_{r,POA}

Où :

- :math:`G_{b,POA}` est la composante directe,
- :math:`G_{d,POA}` est la composante diffuse,
- :math:`G_{r,POA}` est la composante réfléchie (albédo).

Les composantes sont calculées comme suit :

**1. Angle d'incidence (AOI)**

.. math::

    \cos(\theta) = \cos(\beta) \sin(E) + \sin(\beta) \cos(E) \cos(A_s - A_p)

Avec :
- :math:`E` : Angle d'élévation solaire (calculé comme :math:`90^\circ - SZA`)
- :math:`A_s` : Azimut du soleil
- :math:`A_p` : Azimut du panneau
- :math:`\beta` : Inclinaison du panneau

**2. Irradiance directe**

.. math::

    G_{b,POA} = G_b \cdot \cos(\theta)

Avec :math:`G_b` l'irradiance normale directe (DNI).

**3. Irradiance diffuse (modèle isotropique)**

.. math::

    G_{d,POA} = G_d \cdot \frac{1 + \cos(\beta)}{2}

Avec :math:`G_d` l'irradiance horizontale diffuse (DHI).

**4. Irradiance réfléchie (albédo)**

.. math::

    G_{r,POA} = G \cdot \rho \cdot \frac{1 - \cos(\beta)}{2}

Avec :
- :math:`G` : Irradiance horizontale globale (GHI),
- :math:`\rho` : Albédo (coefficient de réflexion du sol).

Estimation de l'énergie produite à partir de la POA
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Une fois l'irradiance POA calculée, l'énergie électrique générée par les panneaux peut être estimée par :

.. math::

    E = G_{POA} \cdot A \cdot \eta

Où :
- :math:`E` : énergie générée (en Wh),
- :math:`G_{POA}` : irradiance sur le plan du module (en W/m²),
- :math:`A` : surface totale des panneaux (en m²),
- :math:`\eta` : rendement global du système photovoltaïque (efficacité du module et pertes système).

Remarques :
^^^^^^^^^^^
- L'angle d'élévation solaire (:math:`E`) peut être obtenu via :math:`E = 90^\circ - SZA`.
- L'azimut solaire peut être calculé automatiquement avec la bibliothèque Python `pvlib`.

Cette méthode permet d'obtenir une estimation réaliste de l'énergie produite à partir de données météorologiques et des caractéristiques d'installation des panneaux photovoltaïques.

Calcul de la Température de la Cellule et de la Puissance
-----------------------------------------------------------

Dans cette section, nous estimons la température des cellules photovoltaïques ainsi que la puissance générée par les panneaux solaires. Nous utilisons deux modèles reconnus : le **modèle de Ross** pour la température de la cellule et le **modèle PVWatts** pour la puissance DC. Ces modèles s'appuient sur l'irradiance POA et la température ambiante.

1. Calcul de la Température de la Cellule
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
La température d'une cellule photovoltaïque (:math:`T_c`) est estimée à partir de la température ambiante (:math:`T_a`) et de l'irradiance POA (:math:`G_{POA}`), selon le **modèle de Ross** :

.. math::

    T_c = T_a + k_{\text{ross}} \cdot G_{POA}

Où :

- :math:`T_a` : Température ambiante (°C),
- :math:`G_{POA}` : Irradiance reçue en plan des modules (W/m²),
- :math:`k_{\text{ross}}` : Coefficient empirique de température, défini comme :

.. math::

    k_{\text{ross}} = \frac{NOCT - 20}{800}

Avec `NOCT` la température nominale de fonctionnement de la cellule (en général entre 42°C et 48°C). Cette relation suppose des conditions de référence avec un vent modéré et une irradiance de 800 W/m².

2. Calcul de la Puissance DC (Modèle PVWatts)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Une fois la température de la cellule estimée, nous utilisons le **modèle PVWatts** pour calculer la puissance DC produite par les modules photovoltaïques :

.. math::

    P_{\text{DC}} = G_{POA} \cdot P_{\text{nom}} \cdot (1 + \gamma \cdot (T_c - T_{ref}))

Où :

- :math:`P_{\text{DC}}` : Puissance en courant continu (W),
- :math:`P_{\text{nom}}` : Puissance nominale du panneau (W),
- :math:`\gamma` : Coefficient de perte de performance par degré de température (valeur typique : -0.003 à -0.005 /°C),
- :math:`T_{ref}` : Température de référence, souvent fixée à 25°C.

Ce modèle prend en compte la diminution d'efficacité du module avec l'augmentation de la température.

Exemple :
^^^^^^^^^
Pour une petite centrale contenant 10 000 panneaux d'une puissance nominale de 750 W chacun, la puissance installée totale est :

.. math::

    P_{\text{installée}} = 10\ 000 \cdot 750 = 7.5\ MW

À titre de comparaison, la centrale solaire Noor Ouarzazate utilise plus de 200 000 panneaux.

- Panneau de référence utilisé : `SR6-HJT725-750M <https://www.enfsolar.com/pv/panel-datasheet/crystalline/65662>`_

3. Conversion de la Puissance DC en Puissance AC
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
La puissance AC injectée sur le réseau est obtenue en appliquant un facteur d'efficacité représentant l'onduleur :

.. math::

    P_{\text{AC}} = P_{\text{DC}} \cdot \eta

Où :

- :math:`\eta` : Efficacité de conversion de l'onduleur, généralement comprise entre 0.85 et 0.95 selon les modèles.

Ce calcul permet d'estimer la puissance réellement disponible après conversion, prenant en compte les pertes dues aux rendements des équipements électriques (ondulation, câblage, etc.).

Conclusion
----------
Ces étapes — estimation de la température des cellules, calcul de la puissance DC, puis conversion en AC — permettent de simuler la production d'énergie électrique d'une installation photovoltaïque à partir de données météo, en intégrant les pertes thermiques et électriques.


