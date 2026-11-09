# TP 04 — Maîtriser les fusions

## Étape 1 — L'avance rapide

### Prédiction avant la fusion

Je prévois que `git merge feature-a` ne créera pas de commit de fusion. La branche `main` n'a pas avancé depuis la création de `feature-a`, donc Git peut simplement déplacer son pointeur vers le commit de `feature-a`.

### Explication après la fusion

Git n'a pas créé de commit de fusion, car `main` était un ancêtre direct de `feature-a`. Il a simplement avancé le pointeur de `main` jusqu'au dernier commit de `feature-a`.

### Graphe obtenu

```text
* 2803bfe (HEAD -> main, feature-a) Ajoute une ligne dans feature-a
* 84ca8b1 Initialise le fichier
```

## Étape 2 — Commit de fusion avec --no-ff

La fusion avec `--no-ff` crée un commit de fusion même lorsque Git pourrait effectuer une avance rapide. Ici, `main` et `feature-b` avaient divergé : chacune possédait un commit différent depuis leur ancêtre commun.

Le commit de fusion `ab8c74d` possède deux parents, ce qui se vérifie avec `git cat-file -p HEAD`.

### Graphe obtenu

```text
*   ab8c74d (HEAD -> main) Merge branch 'feature-b'
|\
| * 2aceca9 (feature-b) Ajoute autre.txt
* | 923c0d1 Ajoute le README
|/
* 2803bfe (feature-a) Ajoute une ligne dans feature-a
* 84ca8b1 Initialise le fichier
```

### Différence entre les deux fusions

- **Fast-forward :** Git déplace le pointeur de la branche ; aucun commit de fusion n'est créé.
- **--no-ff :** Git crée un commit de fusion qui relie les deux historiques.

## Étape 3 — Le conflit, provoqué puis résolu

### Contenu exact du fichier en conflit

```text
<<<<<<< HEAD
Ligne 1 modifiée sur main
=======
Ligne 1 modifiée sur feature-c
>>>>>>> feature-c
Ligne 2
Ligne 3
Ligne 4 - feature-a
```

### Message d'erreur exact de Git

```text
Auto-merging fichier.txt
CONFLICT (content): Merge conflict in fichier.txt
Automatic merge failed; fix conflicts and then commit the result.
```

### Résolution du conflit

J'ai résolu le conflit en réunissant les deux modifications dans une ligne cohérente, puis j'ai supprimé les marqueurs de conflit.

La commande `git add fichier.txt` indique à Git que le conflit est résolu et prépare le fichier pour le commit. La commande `git commit -m "Resout le conflit sur fichier.txt"` termine la fusion en créant un commit de fusion possédant deux parents.

## Compléments

### Étape 6 — Parents du commit de fusion
Le commit de fusion `ab8c74d` a **deux parents** : `git cat-file -p ab8c74d | grep parent` affiche
```text
parent 923c0d1184ed4049a339aa5f97b1382e60065d20
parent 2aceca979a9bb1d5b8d24f4df8a9ae45df7f153f
```
(le dernier commit de `main`, puis le dernier commit de `feature-b`).

### Étape 10 — Terminer la fusion, et oubli du `git add`
La fusion se termine avec `git add fichier.txt` (marque le conflit comme résolu) puis `git commit` (crée le commit de fusion à deux parents).
Si on oublie le `git add`, `git commit` refuse, et le fichier reste en `UU` dans `git status -s` :
```text
error: Committing is not possible because you have unmerged files.
hint: Fix them up in the work tree, and then use 'git add/rm <file>'
hint: as appropriate to mark resolution and make a commit.
fatal: Exiting because of an unresolved conflict.
```

### Étape 11 — Graphe final
```text
* 9043399 (HEAD -> main) Nettoie le fichier apres la fusion
*   c755123 Resout le conflit sur fichier.txt
|\
| * dc8b6d0 (feature-c) Modifie la ligne 1 sur feature-c
* | b5b5437 Modifie la ligne 1 sur main
|/
*   ab8c74d Merge branch 'feature-b'
|\
| * 2aceca9 (feature-b) Ajoute autre.txt
* | 923c0d1 Ajoute le README
|/
* 2803bfe (feature-a) Ajoute une ligne dans feature-a
* 84ca8b1 Initialise le fichier
```
Le commit de fusion `c755123` a pour parents `b5b5437` (main) et `dc8b6d0` (feature-c). `grep -r '<<<<<<<' .` ne renvoie plus aucun marqueur dans les fichiers du dépôt.

### Étape 12 — learngitbranching (notes, niveau « Rebase Introduction »)
Suite de commandes du niveau (à vérifier sur le site) :
```text
git checkout -b bugFix
git commit
git checkout main
git commit
git checkout bugFix
git rebase main
```
- `rebase` rejoue les commits d'une branche **au-dessus** d'une autre : l'historique devient linéaire, sans commit de fusion.
- C2 devient C2' parce que c'est un **nouveau commit** : même changement et même message, mais un parent différent, donc une empreinte différente. L'ancien C2 n'est plus référencé par la branche.
- Il ne faut pas rebaser des commits déjà publiés (poussés) et partagés avec d'autres : cela réécrit l'historique et oblige les collègues à le réconcilier.
