# Exposition aux conflits armés et dépenses militaires

## Présentation du projet
Ce projet de recherche de données analyse l'impact de la proximité géographique d'un conflit armé majeur sur les dépenses militaires des pays voisins. 

L'intuition première (surtout depuis le regain d'intensité de la guerre russo-ukrainienne en 2022) suggère qu'être proche géographiquement d'une guerre pousse mécaniquement un pays à se réarmer. En analysant plus de 30 ans de données mondiales, cette étude montre que la réalité est plus nuancée : il n'y a pas d'accélération systématique des dépenses militaires liée à la seule distance kilométrique (ex: moins de 1 000 km). L'augmentation budgétaire constatée dans des cas emblématiques (Ukraine 2014 et 2022, Irak 2003) s'explique en réalité davantage par des dynamiques géopolitiques et des jeux d'alliances (comme l'objectif des 2% du PIB de l'OTAN) que par la simple contagion spatiale.

## Sources de données
L'analyse repose sur le croisement de trois sources de données de référence :
- **SIPRI (Stockholm International Peace Research Institute)** : Base de données des dépenses militaires (millions de dollars constants).
- **UCDP (Uppsala Conflict Data Program)** : Panel des conflits armés dans le monde, avec identification des belligérants, théâtres d'opération et niveaux d'intensité.
- **CShapes 2.0** : Données géographiques, frontières historiques et coordonnées des capitales pour le calcul des distances.

## Structure du projet
Le code est conçu pour lire les données brutes, effectuer le traitement géospatial et générer des visualisations et un panel d'analyse complet.

```text
├── data/
│   ├── raw/            # Fichiers bruts (SIPRI .xlsx, UCDP .csv, CShapes .geojson) non fournis
│   └── processed/      # Tables intermédiaires générées (panel_pays_annee.csv, etc.)
├── notebooks/          # Fichiers sources
│   └── main.ipynb      # Code source Python 
├── figures/            # Graphiques, cartes et tableaux générés par le programme
├── NOTES.md            # Journal de bord méthodologique détaillé
├── README.md           # Présentation du projet
└── requirements.txt    # Dépendances Python
```

## Méthodologie et Analyse
1. **Appariement (Mapping)** : Harmonisation des identifiants de pays (codes Gleditsch & Ward) entre les dépenses SIPRI et les zones de conflits UCDP.
2. **Identification des chocs** : Un pays subit un "choc" s'il se retrouve soudainement à moins de 1 000 km d'un conflit de forte intensité (intensité 2 : > 1000 morts/an) après au moins 3 ans de paix dans ce rayon.
3. **Exclusion des belligérants** : Seuls les "voisins spectateurs" (non-belligérants directs) sont analysés.
4. **Étude d'événement (Event Study)** : Suivi de la croissance des dépenses militaires de l'année `k = -3` à `k = +3` par rapport à l'année du choc (`k = 0`).

## Installation et Exécution
Assurez-vous d'avoir Python installé, puis installez les dépendances requises :
```bash
pip install -r requirements.txt
```
Placez les fichiers de données brutes dans le dossier `data/raw/`, puis exécutez le script d'analyse :
```bash
python export.py
```