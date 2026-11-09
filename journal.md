# Journal TP 05 — Git distant avec GitHub

Dépôt : https://github.com/titouan-rousseau/demo-git-rousseau

## Socle

### Étape 2 — Relier le dépôt du TP 04
```
git remote add origin https://github.com/titouan-rousseau/demo-git-rousseau.git
git push -u origin main
```
```
To https://github.com/titouan-rousseau/demo-git-rousseau.git
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.
```

### Étape 3 — `git remote -v` et `git branch -vv`
```
origin	https://github.com/titouan-rousseau/demo-git-rousseau.git (fetch)
origin	https://github.com/titouan-rousseau/demo-git-rousseau.git (push)
```
```
  feature-a 2803bfe Ajoute une ligne dans feature-a
  feature-b 2aceca9 Ajoute autre.txt
  feature-c dc8b6d0 Modifie la ligne 1 sur feature-c
* main      c615145 [origin/main] Renomme le journal du TP 04
```
- `[origin/main]` est la branche amont (*upstream*) de `main` : la branche distante que `main` suit. C'est la copie locale de l'état de `main` sur le remote `origin`, telle que vue au dernier échange (push, fetch ou pull). `git status`, `git push` et `git pull` sans argument s'en servent comme référence ; s'il y avait un écart, `git branch -vv` afficherait `ahead N` / `behind N`.
- `-u` (`--set-upstream`) enregistre ce lien de suivi dans `.git/config` (`branch.main.remote` et `branch.main.merge`). Le lien est permanent : les `git push` / `git pull` suivants le retrouvent seuls, il n'y a donc pas besoin de le redonner.

### Étape 4 — Les deux protocoles
Second remote ajouté : `git remote add ssh-origin git@github.com:titouan-rousseau/demo-git-rousseau.git`
- **HTTPS** (`origin`) : authentification par compte GitHub via Git Credential Manager (connexion dans le navigateur, puis jeton mémorisé par Windows). Un mot de passe GitHub simple n'est plus accepté : il faut le gestionnaire d'identifiants ou un jeton d'accès personnel.
- **SSH** (`ssh-origin`) : il faut une paire de clés (`ssh-keygen -t ed25519`), la clé publique ajoutée au compte GitHub, et l'empreinte du serveur GitHub acceptée dans `known_hosts`. Sans cela, l'essai échoue :
```
Host key verification failed.
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

### Étape 5 — README poussé
```
[main 79047f2] Ajoute le README
To https://github.com/titouan-rousseau/demo-git-rousseau.git
   c615145..79047f2  main -> main
```
Le commit `79047f2` est visible sur GitHub.

### Étape 6 — Modification en ligne
README modifié depuis l'interface GitHub, commit « Modifie le README en ligne (etape 6) ». Le dépôt local est en retard d'un commit.

### Étape 7 — `git pull`
**Prédiction (09/11/2026, 15h53)** : avance rapide. Le local n'a aucun commit que GitHub n'a pas ; GitHub a un commit de plus. `main` local est donc un ancêtre de `origin/main` : Git avance simplement le pointeur, sans commit de fusion ni conflit.

**Résultat réel** : avance rapide, comme prévu.
```
From https://github.com/titouan-rousseau/demo-git-rousseau
   79047f2..e06a563  main       -> origin/main
Updating 79047f2..e06a563
Fast-forward
 README.md | 1 +
 1 file changed, 1 insertion(+)
```
Prédiction et résultat concordent : aucun commit local n'avait divergé, Git a seulement avancé `main`.

### Étape 8 — `git fetch` seul
Seconde modification en ligne : commit « Seconde modification en ligne (etape 8) ».
Après `git fetch`, le nouveau commit `c70cac8` est **téléchargé mais pas intégré** : il est sur `origin/main` (le souvenir local de GitHub), pas sur `main`. Le répertoire de travail ne change pas.
```
From https://github.com/titouan-rousseau/demo-git-rousseau
   e06a563..c70cac8  main       -> origin/main
* c70cac8 (origin/main) Seconde modification en ligne (etape 8)
* e06a563 (HEAD -> main) Modifie le README en ligne (etape 6)
* 79047f2 Ajoute le README
```
`git status` : `Your branch is behind 'origin/main' by 1 commit, and can be fast-forwarded.`
Différence avec l'étape 7 : `git pull` = `git fetch` + intégration (ici une avance rapide de `main`) ; `git fetch` s'arrête à la mise à jour de `origin/main`. Un `git pull` ensuite a fait l'avance rapide `e06a563..c70cac8`, et `git status` affiche `Your branch is up to date with 'origin/main'.`

### Étapes 9 et 10 — Deux postes, commit X
- B : `git clone https://github.com/titouan-rousseau/demo-git-rousseau.git demo-git-B`
- A : commit X `31f515c` « Ajoute une ligne au README (mauvais message) », poussé ; B : `git pull` (avance rapide jusqu'à `31f515c`).

### Étape 11 — `--amend` puis `git push` : rejet
A : `git commit --amend` → nouveau commit X' `ff43dac` ; `git push` est refusé :
```
To https://github.com/titouan-rousseau/demo-git-rousseau.git
 ! [rejected]        main -> main (non-fast-forward)
error: failed to push some refs to 'https://github.com/titouan-rousseau/demo-git-rousseau.git'
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. If you want to integrate the remote changes,
hint: use 'git pull' before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```
X' remplace X localement, mais X est déjà sur GitHub : pousser X' supprimerait X de `main`, ce qui n'est pas une avance rapide.

## Approfondissement

### Étape 12 — B1
B : commit B1 `f9fe4c2` « Ajoute NOTES.md (B1) », poussé (`31f515c..f9fe4c2`).

### Étape 13 — `git push --force-with-lease`
**Prédiction (09/11/2026, 15h59)** : Git va **refuser**. Le dernier échange de A avec GitHub est le push de X : `origin/main` (le « bail ») vaut donc X. Mais B a poussé B1 entre-temps, GitHub est sur B1. Comme `origin/main` ≠ ce que GitHub a réellement, `--force-with-lease` refuse au lieu d'écraser B1.

**Résultat réel** : refus, comme prévu.
```
To https://github.com/titouan-rousseau/demo-git-rousseau.git
 ! [rejected]        main -> main (stale info)
error: failed to push some refs to 'https://github.com/titouan-rousseau/demo-git-rousseau.git'
```
`stale info` : le souvenir de A (`origin/main` = X `31f515c`) ne correspond plus à GitHub (`f9fe4c2` = B1). B1 est protégé.

### Étape 14 — Après `git fetch`
```
* f9fe4c2 (origin/main) Ajoute NOTES.md (B1)
* 31f515c Ajoute une ligne au README (mauvais message)
| * ff43dac (HEAD -> main) Ajoute une ligne au README (commit X corrige)
|/
* c70cac8 Seconde modification en ligne (etape 8)
```
Si l'on relançait maintenant `git push --force-with-lease`, il **réussirait et effacerait B1** : le `fetch` a mis `origin/main` à jour sur `f9fe4c2`, le bail correspond donc à GitHub et Git n'a plus de raison de refuser. Le `fetch` a « renouvelé le bail » sans que A n'intègre B1 : c'est pour ça qu'il faut regarder le graphe avant de forcer.

### Étape 15 — `git rebase origin/main` puis `git push`
```
warning: skipped previously applied commit ff43dac
hint: use --reapply-cherry-picks to include skipped commits
Successfully rebased and updated refs/heads/main.
Everything up-to-date
```
La correction de message n'a pas survécu : X' apporte exactement la même modification que X, déjà présent dans `origin/main`. Git a reconnu un changement « déjà appliqué » (même patch) et a sauté X' ; `main` s'est retrouvé sur `f9fe4c2` et il n'y avait plus rien à pousser. Le message « mauvais message » de X reste dans l'historique publié.
Il fallait corriger le message **avant le premier `git push`** de X : `--amend` ne réécrit sans risque que des commits jamais publiés.

### État final
A et B : `Your branch is up to date with 'origin/main'.`, tête `f9fe4c2` des deux côtés, identique à GitHub (`git ls-remote origin main` → `f9fe4c28fb27823f525cf5a9cb6c728705ea8ba7`). Le commit B1 est intact.

## Issues et traçabilité

### Étape 16 — Issue
Issue #1 « Documenter le TP 05 dans journal.md » : deux lignes de description, label `documentation` (créé d'office par GitHub), milestone « TP 05 » créé pour l'occasion.
https://github.com/titouan-rousseau/demo-git-rousseau/issues/1

### Étape 17 — Fermeture par commit
Ce journal est commité avec le message « Ajoute le journal du TP 05 », qui contient `Closes #1`. Au push sur `main`, GitHub ferme l'Issue #1 et y affiche le commit.

### Étape 18 — Intérêt pour une revue trois mois plus tard
Le lien est dans les deux sens : depuis l'Issue on retrouve le commit qui l'a résolue (donc le diff exact), et depuis le commit on retrouve l'Issue qui explique *pourquoi* il a été fait (le besoin, la discussion, le label, le milestone). Trois mois plus tard, un relecteur n'a pas à reconstituer le contexte de mémoire : la raison du changement et le changement lui-même sont reliés et datés.
