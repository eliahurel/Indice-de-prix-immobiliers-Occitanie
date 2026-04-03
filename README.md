# Indice-de-prix-immobiliers-Occitanie

Code réalisé par **Mattéo CORREGER** et **Elia HUREL-LUCEF-LOISEL**, dans le cadre de l'UE Conduite de Projet — Master 1 Statistique et Économétrie, Université de Strasbourg (2025/26).  
Supervisé par **Youssef EL YAAKOUBI**. 

### Objectif

L'objectif de ce projet est de construire des **indices de prix immobiliers à "qualité constante"** pour les communes de la région Occitanie, sur la période **2010–2024**.

Une simple moyenne des prix observés ne suffit pas : si l'on vend beaucoup de grandes maisons un mois puis de petits studios le suivant, la moyenne chute sans que le marché ait réellement bougé. Nous utilisons un **modèle hédonique** pour isoler l'évolution pure des prix, en neutralisant les effets de surface, de localisation et de composition des biens vendus.

## Installation

En travaillant avec votre environnement Python, installez les packages nécessaires :
```
pip install pandas numpy matplotlib seaborn scipy polars

```
## Données

Les données proviennent de la base **DVF+ (Demandes de Valeurs Foncières)**, disponible en open data :  
https://datafoncier.cerema.fr/donnees/autres-donnees-foncieres/dvfplus-open-data

Deux sources sont fusionnées pour couvrir l'ensemble de la période :
- **2010–2013** : base nationale (extraction Occitanie)
- **2014–2024** : base DVF+ Occitanie

## Fonctionnement

### Structure du notebook

**Étape 1 — Importation et harmonisation (2010–2024)**

Cette étape prépare l'environnement de calcul. L'utilisation de **Polars** permet de charger et manipuler les millions de lignes de la base DVF+ de manière beaucoup plus rapide que Pandas. Nous préparons également les **matrices creuses (Sparse Matrices)**, essentielles pour stocker des milliers de variables (communes, mois) sans saturer la mémoire vive de l'ordinateur.

**Étape 2 — Filtrage de la base de données**

Le code fusionne les données historiques (2010–2013) avec les données récentes (2014–2024).

- **Filtrage des outliers :** on retire les transactions aberrantes, comme les ventes à 1 € (souvent des transferts familiaux) ou les prix au m² délirants qui fausseraient la moyenne.
- **Segmentation :** le code sépare strictement les **Maisons** des **Appartements**, car ces deux marchés répondent à des logiques de prix différentes.

**Étape 3 — Construction du modèle hédonique**

Pour que le modèle comprenne le temps et l'espace, nous créons des variables binaires (0 ou 1) pour :
- **Chaque mois :** pour capter la tendance globale du marché.
- **Chaque commune :** pour capter le niveau de prix local.
- **Interactions département × année :** pour isoler des dynamiques locales spécifiques.

Le modèle estimé est le suivant :

$$\log(P_{i,c,t}) = \alpha + x_i'\beta + \gamma_{t(i)} + \delta_{d(i)} + \mu_{c(i)} + \theta_{d(i),a(i)} + \varepsilon_i$$

**Étape 4 — Estimation de l'indice à qualité constante**

Le code utilise l'algorithme **LSQR** pour résoudre une régression géante comprenant des centaines de milliers d'observations et des milliers de variables.

- **Objectif :** estimer l'influence de chaque caractéristique (surface, pièces, date, lieu) sur le prix.
- **Résultat :** on obtient un score de fiabilité (R²) qui valide la précision du modèle — 0.49 pour les appartements, 0.43 pour les maisons.

**Étape 5 — Calcul du prix net et agrégation**

Une fois les coefficients obtenus, le code calcule le **prix net** pour chaque vente :

$$\log P_i^{net} = \hat{y}_i - \sum_{k \in \mathcal{H}} \hat{\beta}_k x_{ik}$$

- **Principe :** on retire l'effet de la "qualité" (ex : le surplus de prix dû au fait qu'une maison est très grande) pour ne garder que l'effet pur du marché et du lieu.
- **Médiane :** on regroupe ces prix nets par mois et par commune en calculant la médiane, plus robuste aux ventes exceptionnelles que la moyenne.

**Étape 6 — Complétion des séries et corrections**

- **Petites communes :** pour les villages avec peu de ventes, le code applique une correction par la moyenne des résidus ($\bar{e}_c$) pour éviter des courbes en "dents de scie".
- **Prédiction des trous :** si une commune n'a aucune vente un mois donné, le modèle prédit le prix théorique pour maintenir une courbe continue.

**Étape 7 — Construction de l'indice base 100**

Le code transforme les prix nets en un indice facile à lire, où **janvier 2010 = 100** :

$$\text{Index}_{c,t} = 100 \times \exp\!\left(\log P_{c,t}^{net} - \log P_{c,t_0}^{net}\right)$$

- **Lissage :** une moyenne mobile sur 6 ou 12 mois est appliquée pour éliminer le "bruit" visuel et faire ressortir les vrais cycles du marché.

**Étape 8 — Représentations graphiques**

Production automatique de quatre types de visualisations :
-  **Courbes temporelles** comparatives (métropoles vs villes témoins, maisons vs appartements)
-  **Cartes choroplèthes** du prix médian au m² par commune (2021–2024)
-  **Cartes de l'indice** — évolution depuis 2010 par commune
-  **Heatmaps** commune × temps — dynamiques, ruptures et hétérogénéité spatiale

## Résultats clés

| Période | Dynamique observée |
|---|---|
| 2011–2016 | Baisse commune (~4 %), plus marquée pour les appartements |
| 2018–2020 | Reprise progressive |
| 2020–2022 | Accélération post-COVID, boom des maisons (+10 pts en 2 ans) |
| 2023–2024 | Retournement lié à la remontée des taux d'intérêt |

Toulouse et Montpellier surperforment la moyenne régionale avec une croissance proche de **+20 %** au pic de 2023, contre **+9 %** pour l'Occitanie dans son ensemble.

