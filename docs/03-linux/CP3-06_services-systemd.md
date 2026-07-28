# CP3-06 — Gérer les services avec systemd (start/enable/status/journalctl)

**Objectif** : démarrer, arrêter, activer au démarrage et surveiller les services Linux avec `systemctl`, et consulter leurs journaux avec `journalctl`.

**Rattachement REAC** : CP3 « Exploiter des serveurs Linux » — savoir-faire : administrer les services et exploiter les journaux d'un serveur Linux.

**Durée** : ~20 min · **Niveau** : débutant.

---

## Prérequis

- Debian 13 (**CP3-01**), accès console/SSH, droits root/sudo.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian / systemd | 13.6 « trixie » — **systemd 257** | 24/07/2026 |
| `systemctl` / `journalctl` | **commandes testées dans le bac à sable** (systemd 249) | 24/07/2026 |

> Pas de GUI par défaut sur un serveur : gestion en **CLI**.

---

## Procédure — CLI

### Consulter l'état

```bash
systemctl status ssh            # état détaillé d'un service
systemctl is-active ssh         # active / inactive
systemctl is-enabled ssh        # enabled / disabled / static
systemctl list-units --type=service --state=running   # services actifs
systemctl list-unit-files --type=service              # tous + état d'activation
```

### Contrôler un service

```bash
sudo systemctl start  ssh       # démarrer maintenant
sudo systemctl stop   ssh       # arrêter
sudo systemctl restart ssh      # redémarrer
sudo systemctl reload ssh       # recharger la config sans coupure (si supporté)
```

### Activation au démarrage

```bash
sudo systemctl enable  ssh          # démarrage automatique au boot
sudo systemctl enable --now ssh     # active ET démarre immédiatement
sudo systemctl disable ssh          # ne plus démarrer au boot
sudo systemctl mask    ssh          # interdire tout démarrage (unmask pour lever)
```

### Journaux (journalctl)

```bash
journalctl -u ssh               # journal d'un service
journalctl -u ssh -f            # suivi en temps réel
journalctl -b                   # messages depuis le dernier démarrage
journalctl -p err               # niveau erreur et au-dessus
journalctl --since "today" -n 50
journalctl -xe                  # dernières entrées + explications (diagnostic)
```

### Unité personnalisée (rappel)

```bash
sudo nano /etc/systemd/system/monapp.service   # [Unit]/[Service]/[Install]
sudo systemctl daemon-reload                    # après TOUTE modif d'unité
sudo systemctl enable --now monapp
```

---

## Vérification (sorties obtenues dans le bac à sable)

```
systemctl is-active systemd-journald      → active
systemctl is-enabled systemd-journald     → static
systemctl status systemd-journald         → Active: active (running) since ...
systemctl list-timers                     → liste des minuteurs (ex. apt-daily.timer)
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Service en **failed** | Erreur de démarrage | `journalctl -u <svc> -xe` pour la cause exacte |
| Modif d'unité **non prise en compte** | Cache systemd | `sudo systemctl daemon-reload` |
| `Unit not found` | Nom inexact | Vérifier avec `systemctl list-unit-files` |
| `enable` sans effet | Unité **static** (pas de section `[Install]`) | C'est normal : activée via une dépendance/socket |
| `journalctl` ne montre rien | Droits insuffisants | Être root ou membre du groupe `systemd-journal` |

## Sécurité et bonnes pratiques

- **Désactiver les services inutiles** (réduction de la surface d'attaque).
- Contrôler les services **à l'écoute** sur le réseau (`ss -tulpn`, voir **CP3-10**).
- Conserver des **journaux persistants** (`/var/log/journal`) pour la traçabilité.

## ⚠️ À ne pas confondre / obsolète

- **SysVinit est remplacé par systemd** : `service <svc> start`, `/etc/init.d/…`, `chkconfig` sont obsolètes → utiliser **`systemctl`**.
- `is-enabled` = **`static`** ne signifie pas « désactivé » : l'unité n'a pas de cible d'activation propre (déclenchée autrement).
- Toujours `daemon-reload` **après** avoir modifié un fichier d'unité.

---

## Sources (doc officielle)

- [Debian Wiki — systemd](https://wiki.debian.org/systemd) — consulté le 24/07/2026
- [systemctl(1) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/systemd/systemctl.1.en.html) — consulté le 24/07/2026
- [journalctl(1) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/systemd/journalctl.1.en.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées (systemd 257) · [x] rien d'obsolète (systemctl vs SysVinit) · [x] **commandes testées dans le bac à sable** · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
