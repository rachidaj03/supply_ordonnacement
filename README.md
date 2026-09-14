# Ordonnancement de production (Job-Shop Scheduling)

Notebook du cours SPL (S7) qui résout un problème d'**ordonnancement d'atelier (job-shop scheduling)** : 13 pièces (`P1` à `P13`), chacune décomposée en phases d'usinage séquentielles réalisées sur différentes machines, à ordonnancer de manière à minimiser le temps de cycle total (makespan).

## Données — `ordonnancement_donnees.csv`

Une ligne par phase d'une pièce :

| Colonne | Description |
|---|---|
| `Pièce` | identifiant produit (P1 à P13) |
| `Phase` | numéro d'opération (10, 20, 30...), donne l'ordre de passage |
| `Désignation` | nom de l'opération (ébauche, reprise, rectification, tournage...) |
| `Machine` | poste de travail utilisé (TR1-3, CU1-6, RE1-3, TCN1-2) |
| `T Unitaire (H)` | temps unitaire par pièce produite (heures) |
| `T Série (H)` | temps de série/réglage fixe (heures) |

Les quantités par pièce sont fixées directement dans le notebook (`quantites = [30,30,30,30,30,30,30,10,10,10,10,10,10]`), associées aux produits triés par numéro.

## Modèle d'optimisation

Le notebook s'appuie sur **OR-Tools CP-SAT** (`ortools.sat.python.cp_model`) :

- **Variables** : pour chaque tâche `(Pièce, Phase, Machine)`, une variable `start` et `end` (en secondes, bornées à `[0, 86400]`, soit 24h), avec un `IntervalVar` associé.
- **Durée d'une tâche** : `T Série × 3600 + T Unitaire × Quantité × 3600` (en secondes).
- **Contrainte de précédence** : pour chaque pièce, la phase `i` doit se terminer avant que la phase `i+1` ne commence (`end[i] <= start[i+1]`).
- **Contrainte de ressource** : une même machine ne peut traiter qu'une tâche à la fois (`AddNoOverlap` sur les intervalles par machine).
- **Objectif** : minimiser le `makespan`, c'est-à-dire `max(end_times)` sur l'ensemble des tâches (`model.Minimize(makespan)`).

Si une solution optimale est trouvée, le solveur CP-SAT renvoie le makespan en heures ainsi que les horaires de début et de fin de chaque tâche.

## Visualisation

La deuxième cellule construit un **diagramme de Gantt** (matplotlib) à partir d'une liste de tâches `(Produit, Machine, début, fin)` codées en dur (ce sont les résultats d'une exécution précédente du solveur) : une barre horizontale par tâche/machine, colorée par produit, avec une légende par produit.

> Note : les horaires de cette cellule sont recopiés manuellement (résultat figé d'une résolution passée), et ne sont pas recalculés dynamiquement à partir de la sortie du solveur de la cellule précédente.

## Dépendances

```bash
pip install ortools matplotlib plotly pandas
```

## Pour exécuter

Le chemin du CSV est codé en dur dans le notebook (`/Users/rachidaitjalloul/Desktop/ÉTUDES/S7/SPL/ordonnancement_donnees.csv`). Il faut donc l'adapter au chemin local avant exécution, ou le remplacer par un chemin relatif (`ordonnancement_donnees.csv`).
