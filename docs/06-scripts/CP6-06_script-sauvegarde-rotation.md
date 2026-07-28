# CP6-06 — Script Bash : sauvegarde et rotation de fichiers/journaux

**Objectif** : écrire un script qui sauvegarde un dossier dans une archive compressée et supprime les archives trop anciennes (rotation).

**Rattachement REAC** : CP6 « Automatiser des tâches à l'aide de scripts » — savoir-faire : automatiser des sauvegardes.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Debian 13 (**CP3-01**), bases Bash (**CP6-05**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian / Bash / `tar` / `find` | 13.6 « trixie » | 24/07/2026 |
| Script (création + rotation) | **exécuté dans le bac à sable** | 24/07/2026 |

---

## Procédure — CLI (script testé)

```bash
#!/usr/bin/env bash
set -euo pipefail

SRC="/srv/donnees"          # dossier à sauvegarder
DEST="/var/backups/app"     # destination des archives
RETENTION=7                 # conserver 7 jours

mkdir -p "$DEST"
archive="$DEST/backup-$(date +%Y%m%d-%H%M%S).tar.gz"

# 1. Sauvegarde compressée (-C : chemins relatifs dans l'archive)
tar -czf "$archive" -C "$SRC" .
echo "Créé : $archive ($(du -h "$archive" | cut -f1))"

# 2. Rotation : supprimer les archives de plus de RETENTION jours
find "$DEST" -name 'backup-*.tar.gz' -mtime +"$RETENTION" -delete
```

**Test réalisé (bac à sable) :**

```
Créé : /tmp/backups/backup-20260724-181215.tar.gz (4.0K)
Avant rotation :
  backup-20260724-181215.tar.gz
  backup-vieux.tar.gz          (daté de 10 jours)
Après rotation (vieux supprimé) :
  backup-20260724-181215.tar.gz
```

> **Planifier** ce script quotidiennement : cron ou timer systemd (**CP3-12** / **CP6-08**).

---

## Vérification (comment savoir que ça marche)

```bash
ls -1 "$DEST"                       # une nouvelle archive horodatée
tar -tzf "$archive" | head          # lister le contenu de l'archive
# Restaurer un test : tar -xzf "$archive" -C /tmp/restore
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Chemins cassés dans l'archive | Chemins absolus | Utiliser `tar -C "$SRC" .` |
| Rotation trop/pas assez agressive | `-mtime +N` = **strictement** > N×24 h | Ajuster `RETENTION` |
| Erreur sur un chemin avec espaces | Variable non quotée | `"$archive"`, `"$DEST"` |
| Disque plein | Trop d'archives | Réduire la rétention / cible externe |

## Sécurité et bonnes pratiques

- Appliquer la règle **3-2-1** : copie **hors-site** (**CP8-11**), pas seulement locale.
- **Tester la restauration** régulièrement (**CP8-07**) — une sauvegarde non testée ne vaut rien.
- **Chiffrer** les archives sensibles ; restreindre les droits sur `$DEST`.

## ⚠️ À ne pas confondre / obsolète

- `find -mtime +7` = fichiers modifiés il y a **plus de 7 × 24 h** (pas « 7 fichiers »).
- Rotation « maison » (par âge) ≠ **logrotate** (**CP3-15**) : logrotate pour les journaux, ce script pour des données.
- `tar -C` (change de répertoire) évite d'embarquer toute l'arborescence absolue.

---

## Sources (doc officielle)

- [tar(1) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/tar/tar.1.en.html) — consulté le 24/07/2026
- [find(1) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/findutils/find.1.en.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées · [x] rien d'obsolète · [x] **script (sauvegarde + rotation) exécuté dans le bac à sable** · [x] conforme doc · [x] vérification présente · [x] sécurité (3-2-1, test restauration) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
