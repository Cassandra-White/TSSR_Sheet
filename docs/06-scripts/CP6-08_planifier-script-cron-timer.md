# CP6-08 — Planifier un script sous Linux (cron / systemd timer)

**Objectif** : exécuter automatiquement un script Bash de façon **fiable** (cron ou timer systemd), en le rendant robuste au contexte planifié.

**Rattachement REAC** : CP6 « Automatiser des tâches à l'aide de scripts » — savoir-faire : planifier l'exécution de scripts Linux.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Debian 13 (**CP3-01**), un script prêt (**CP6-06**), bases de la planification (**CP3-12**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian | 13.6 « trixie » | 24/07/2026 |
| `flock` / `systemd-analyze calendar` | **testés dans le bac à sable** | 24/07/2026 |

---

## Procédure — CLI

### Option A — cron

```bash
crontab -e
# min heure jour mois jour-semaine  commande (chemins ABSOLUS + journalisation)
30 2 * * * /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1
```

### Option B — timer systemd (traçabilité via journald — voir CP3-12)

```ini
# /etc/systemd/system/backup.timer  →  [Timer] OnCalendar=*-*-* 02:00:00 / Persistent=true
```

### Rendre le script « planification-safe »

```bash
#!/usr/bin/env bash
set -euo pipefail
PATH=/usr/sbin:/usr/bin:/sbin:/bin        # cron a un PATH minimal → l'expliciter

# Verrou anti-exécution concurrente (empêche deux lancements simultanés)
exec 9>/var/lock/backup.lock
flock -n 9 || { echo "déjà en cours -> sortie"; exit 1; }

# … traitement, avec des CHEMINS ABSOLUS …
```

---

## Vérification (sorties obtenues dans le bac à sable)

```
# Verrou flock
verrou obtenu : traitement en cours...

# Expression de planification
systemd-analyze calendar "*-*-* 02:00:00"
  → Next elapse: Sat 2026-07-25 02:00:00 CEST
```

```bash
systemctl list-timers          # (timer) prochaine exécution
grep backup /var/log/backup.log  # (cron) trace des exécutions
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| OK en manuel, KO en cron | **PATH minimal** / chemins relatifs | Chemins **absolus** + `PATH=` explicite |
| Deux exécutions se chevauchent | Traitement long relancé | **`flock`** (verrou) |
| Aucun journal | Sortie non redirigée | `>> fichier.log 2>&1` (cron) ou `journalctl -u …` (timer) |
| Tâche ratée (machine éteinte) | Pas de rattrapage | `Persistent=true` (timer) / anacron |

## Sécurité et bonnes pratiques

- **Verrou `flock`** pour l'exclusion mutuelle des tâches longues.
- **Journaliser** systématiquement (diagnostic).
- Droits minimaux ; **pas de secrets en clair** dans le script planifié.

## ⚠️ À ne pas confondre / obsolète

- **cron** hérite d'un **PATH minimal** : c'est LE piège (« ça marche en manuel mais pas en planifié »).
- **timer systemd** journalise dans **journald** (avantage sur cron pour la traçabilité).
- `flock` ≠ simple fichier « lock » maison (flock gère proprement les verrous du noyau).

---

## Sources (doc officielle)

- [crontab(5) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/cron/crontab.5.en.html) — consulté le 24/07/2026
- [flock(1) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/util-linux/flock.1.en.html) — consulté le 24/07/2026
- [systemd.timer(5) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/systemd/systemd.timer.5.en.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées · [x] rien d'obsolète (timer + flock) · [x] **flock & calendar testés dans le bac à sable** · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
