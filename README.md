# Untitled.ipynb — Ordonnancement de production (Job-Shop Scheduling)

Notebook du cours SPL (S7) résolvant un problème d'**ordonnancement d'atelier (job-shop scheduling)** : 13 pièces (`P1`…`P13`), chacune décomposée en phases d'usinage séquentielles réalisées sur différentes machines, à ordonnancer de façon à minimiser le temps de cycle total (makespan).

## Données — `ordonnancement_donnees.csv`

Une ligne par phase d'une pièce :

| Colonne | Description |
|---|---|
| `Pièce` | identifiant produit (P1…P13) |
| `Phase` | numéro d'opération (10, 20, 30…), donne l'ordre de passage |
| `Désignation` | nom de l'opération (ébauche, reprise, rectification, tournage…) |
| `Machine` | poste de travail utilisé (TR1-3, CU1-6, RE1-3, TCN1-2) |
| `T Unitaire (H)` | temps unitaire par pièce produite (heures) |
| `T Série (H)` | temps de série/réglage fixe (heures) |

Les quantités par pièce sont fixées dans le notebook (`quantites = [30,30,30,30,30,30,30,10,10,10,10,10,10]`), associées aux produits triés par numéro.

## Modèle d'optimisation

Le notebook utilise **OR-Tools CP-SAT** (`ortools.sat.python.cp_model`) :

- **Variables** : pour chaque tâche `(Pièce, Phase, Machine)`, une variable `start` et `end` (en secondes, bornées à `[0, 86400]`, soit 24h), et un `IntervalVar` associé.
- **Durée d'une tâche** : `T Série × 3600 + T Unitaire × Quantité × 3600` (secondes).
- **Contrainte de précédence** : pour chaque pièce, la phase `i` doit se terminer avant que la phase `i+1` ne commence (`end[i] <= start[i+1]`).
- **Contrainte de ressource** : une même machine ne peut traiter qu'une tâche à la fois (`AddNoOverlap` sur les intervalles par machine).
- **Objectif** : minimiser le `makespan` = `max(end_times)` de toutes les tâches (`model.Minimize(makespan)`).

Le solveur CP-SAT renvoie, si une solution optimale est trouvée, le makespan en heures et les horaires de début/fin de chaque tâche.

## Visualisation

La deuxième cellule construit un **diagramme de Gantt** (matplotlib) à partir d'une liste de tâches `(Produit, Machine, début, fin)` codées en dur (résultats d'une exécution précédente du solveur), une barre horizontale par tâche/machine, colorée par produit, avec une légende par produit.

> Note : les horaires de cette cellule sont des valeurs recopiées manuellement (résultat figé d'une résolution), pas recalculées dynamiquement à partir de la sortie du solveur de la cellule précédente.

## Dépendances

```bash
pip install ortools matplotlib plotly pandas
```

## Pour exécuter

Le chemin du CSV est codé en dur dans le notebook (`/Users/rachidaitjalloul/Desktop/ÉTUDES/S7/SPL/ordonnancement_donnees.csv`) — à adapter au chemin local avant exécution, ou à remplacer par un chemin relatif (`ordonnancement_donnees.csv`).
