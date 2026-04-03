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

Après nettoyage : **564 506 transactions maisons** et **397 690 transactions appartements**.

## Fonctionnement

### Structure du notebook

**Étape 1 : Importation et harmonisation (2010-2024)**

Cette étape prépare l'environnement de calcul. L'utilisation de Polars permet de charger et manipuler les millions de lignes de la base DVF+ de manière beaucoup plus rapide que Pandas. Nous préparons également les matrices creuses (Sparse Matrices), essentielles pour stocker des milliers de variables (communes, mois) sans saturer la mémoire vive de l'ordinateur.

**Étape 2 : Filtrage de la base de donnée**

Le code fusionne les données historiques (2010-2013) avec les données récentes (2014-2024).

Filtrage des "Outliers" : On retire les transactions aberrantes, comme les ventes à 1 € (souvent des transferts familiaux) ou les prix au $m^2$ délirants qui fausseraient la moyenne.

Segmentation : Le code sépare strictement les Maisons des Appartements, car ces deux marchés répondent à des logiques de prix différentes.

**Étape 3 : Construction du Modèle Hédonique**

Pour que le modèle comprenne le temps et l'espace, nous créons des variables binaires (0 ou 1) pour :

- Chaque mois : pour capter la tendance globale.
- Chaque commune : pour capter le niveau de prix local.
- Interactions département-année : pour isoler des dynamiques locales spécifiques.

**Étape 4 :  Création de l'indice "à qualité constante"**

Le code utilise l'algorithme LSQR pour résoudre une régression géante comprenant des centaines de milliers d'observations et des milliers de variables.

Objectif : Estimer l'influence de chaque caractéristique (surface, pièces, date, lieu) sur le prix.

Résultat : On obtient un score de fiabilité ($R^2$) qui valide la précision du modèle.

**Étape 5 : Compléter les périodes manquantes**

Une fois les coefficients obtenus, le code calcule le prix net pour chaque vente.

Principe : On retire l'effet de la "qualité" (ex: on retire le surplus de prix dû au fait qu'une maison est très grande) pour ne garder que l'effet pur du marché et du lieu.

Médiane : On regroupe ces prix nets par mois et par commune en calculant la médiane, plus robuste aux ventes exceptionnelles que la moyenne

**Étape 6 : Construction de l'indice base 100**

Petites Communes : Pour les villages avec peu de ventes, le code applique une correction par la moyenne des résidus pour éviter des courbes en "dents de scie".

Prédiction des trous : Si une commune n'a aucune vente un mois donné, le modèle "prédit" le prix théorique pour maintenir une courbe continue.

**Étape 7 : Représentation graphique**

Le code transforme les prix nets en un indice facile à lire, où Janvier 2010 = 100.

Lissage : Une moyenne mobile sur 6 ou 12 mois est appliquée pour éliminer le "bruit" visuel et faire ressortir les vrais cycles.

Sorties graphiques : Production automatique des courbes comparatives, des cartes choroplèthes, une des prix au $m^2$ et des heatmaps de croissance.
