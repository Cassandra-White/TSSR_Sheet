# CP3-12 — Planifier des tâches (cron / systemd timers)

**Objectif** : exécuter des tâches récurrentes sous Debian, avec `cron` (classique) et avec les **timers systemd** (modernes, journalisés).

**Rattachement REAC** : CP3 « Exploiter des serveurs Linux » — savoir-faire : automatiser des tâches planifiées sur un serveur Linux.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Debian 13 (**CP3-01**), droits root/sudo. Service `cron` actif (installé par défaut).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian / systemd | 13.6 « trixie » — systemd 257 | 24/07/2026 |
| `systemd-analyze calendar` | **testé dans le bac à sable** | 24/07/2026 |

---

## Procédure — CLI

### Option A — cron

```bash
crontab -e            # éditer les tâches de l'utilisateur courant
crontab -l            # lister
```

Format (5 champs : **minute heure jour-du-mois mois jour-de-semaine**) :

```text
30 3 * * *   /usr/local/bin/backup.sh      # tous les jours à 03h30
*/15 * * * * /usr/local/bin/collect.sh      # toutes les 15 minutes
0 8 * * 1-5  /usr/local/bin/rapport.sh       # 08h00 du lundi au vendredi
```

Niveau système : `/etc/crontab`, fichiers dans `/etc/cron.d/` (avec un champ **utilisateur** en plus), ou scripts dans `/etc/cron.{hourly,daily,weekly,monthly}/`.

### Option B — timer systemd (recommandé)

`/etc/systemd/system/backup.service` :

```ini
[Unit]
Description=Sauvegarde quotidienne

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
```

`/etc/systemd/system/backup.timer` :

```ini
[Unit]
Description=Planifie la sauvegarde quotidienne

[Timer]
OnCalendar=*-*-* 03:30:00
Persistent=true            # rattrape une exécution manquée (machine éteinte)

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now backup.timer
systemd-analyze calendar "*-*-* 03:30:00"   # vérifier une expression avant de l'utiliser
```

---

## Vérification (sorties obtenues dans le bac à sable)

```
systemd-analyze calendar "daily"            → Normalized: *-*-* 00:00:00 ; Next elapse: 2026-07-25 00:00:00
systemd-analyze calendar "*:0/15"           → toutes les 15 min ; Next elapse: …12:00:00
systemd-analyze calendar "Mon..Fri 08:00"   → Next elapse: Mon 2026-07-27 08:00:00
systemctl list-timers                        → liste les minuteurs et leur prochaine exécution
journalctl -u backup.service                 → trace des exécutions
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| La tâche cron ne s'exécute pas | Chemin relatif / PATH minimal en cron | Utiliser des **chemins absolus** ; définir le PATH dans la crontab |
| Le timer ne se déclenche pas | `daemon-reload`/`enable` oublié | `sudo systemctl daemon-reload` puis `enable --now` |
| `OnCalendar` sans effet | Expression erronée | La valider avec `systemd-analyze calendar "…"` |
| Exécution manquée (machine éteinte) | Pas de rattrapage | `Persistent=true` (timer) ou **anacron** (cron.daily) |

## Sécurité et bonnes pratiques

- **Chemins absolus** et scripts aux **droits restrictifs** (pas d'écriture par tous).
- Ne pas stocker de secrets en clair dans les tâches planifiées.
- Préférer les **timers systemd** pour la **traçabilité** (tout est journalisé dans `journalctl`).

## ⚠️ À ne pas confondre / obsolète

- La syntaxe **cron (5 champs)** ≠ **`OnCalendar`** (systemd) : ne pas les mélanger.
- cron n'a **pas de journal détaillé** par défaut ; les timers systemd, si.
- Pour un poste souvent éteint, `cron` seul rate les tâches → **anacron** ou `Persistent=true`.

---

## Sources (doc officielle)

- [Debian Wiki — cron](https://wiki.debian.org/cron) — consulté le 24/07/2026
- [systemd.timer(5) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/systemd/systemd.timer.5.en.html) — consulté le 24/07/2026
- [systemd.time(7) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/systemd/systemd.time.7.en.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées · [x] rien d'obsolète (timers vs cron) · [x] **`systemd-analyze calendar` testé** · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
