# CP6-09 — Documenter et versionner ses scripts (bonnes pratiques + Git de base)

**Objectif** : documenter proprement ses scripts et gérer leur historique avec Git.

**Rattachement REAC** : CP6 « Automatiser des tâches à l'aide de scripts » — savoir-faire : documenter et versionner le code.

**Durée** : ~25 min · **Niveau** : débutant.

---

## Prérequis

- `git` installé (`sudo apt install git`), un ou plusieurs scripts à versionner.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Git | présent — **workflow exécuté dans le bac à sable** | 24/07/2026 |

---

## Documenter un script (bonnes pratiques)

En-tête normalisé + README :

```bash
#!/usr/bin/env bash
# backup.sh — Sauvegarde compressée + rotation d'un dossier.
# Usage   : ./backup.sh (voir variables SRC/DEST/RETENTION en tête)
# Auteur  : équipe TSSR   Date : 2026-07-24
set -euo pipefail
```

## Procédure — CLI (Git de base)

```bash
git config --global user.name  "TSSR"
git config --global user.email "tssr@lab.local"

git init                                  # nouveau dépôt
printf '*.log\n*.tar.gz\n.env\n' > .gitignore   # ne PAS versionner logs/archives/secrets
git add .
git commit -m "Ajout du script de sauvegarde"
git log --oneline                          # historique
git diff                                   # modifications non validées

# Travailler sur une branche
git switch -c amelioration-rotation
# … modifications … puis :
git commit -am "Rotation par âge"
git switch main && git merge amelioration-rotation

# Dépôt distant
git remote add origin https://git.exemple.local/tssr/scripts.git
git push -u origin main
```

**Workflow réellement exécuté (bac à sable) :**

```
git log --oneline   → 91bf037 Ajout du script de sauvegarde
git diff --stat     →  backup.sh | 1 +
                       1 file changed, 1 insertion(+)
```

## Procédure — GUI

**VS Code** (onglet *Source Control*) ou **GitHub Desktop** : mêmes opérations (stage, commit, push) en clics.

---

## Vérification (comment savoir que ça marche)

```bash
git status                 # "working tree clean" après commit
git log --oneline --graph  # l'historique et les branches
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| `Author identity unknown` | Git non configuré | `git config --global user.name/user.email` |
| Un secret/log a été committé | `.gitignore` posé trop tard | `git rm --cached fichier` + committer |
| `rejected (non-fast-forward)` au push | Dépôt distant en avance | `git pull --rebase` puis `git push` |
| Conflit de fusion | Mêmes lignes modifiées | Résoudre les marqueurs `<<<<`, committer |

## Sécurité et bonnes pratiques

- **Ne jamais committer de secrets** (mots de passe, clés, `.env`) → `.gitignore` **dès le départ**.
- Messages de commit **clairs** ; commits **atomiques**.
- Dépôt **privé** pour du code interne ; relire avant `push`.

## ⚠️ À ne pas confondre / obsolète

- **`git switch`** / **`git restore`** (modernes, explicites) remplacent avantageusement `git checkout` (surchargé).
- **`commit`** (local) ≠ **`push`** (envoi au distant) : deux étapes distinctes.
- La branche par défaut est aujourd'hui **`main`** (anciennement `master`).

---

## Sources (doc officielle)

- [Git — Documentation](https://git-scm.com/doc) — consulté le 24/07/2026
- [Pro Git (livre officiel, FR)](https://git-scm.com/book/fr/v2) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI puis GUI · [x] versions datées · [x] rien d'obsolète (`switch`/`restore`, `main`) · [x] **workflow Git exécuté dans le bac à sable** · [x] conforme doc · [x] vérification présente · [x] sécurité (pas de secrets) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
