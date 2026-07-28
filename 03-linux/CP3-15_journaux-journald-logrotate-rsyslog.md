# CP3-15 — Gérer et centraliser les journaux (journald, logrotate, rsyslog)

**Objectif** : consulter les journaux, les rendre persistants et maîtrisés en taille (journald), les faire tourner (logrotate) et, au besoin, les centraliser (rsyslog).

**Rattachement REAC** : CP3 « Exploiter des serveurs Linux » — savoir-faire : exploiter et conserver les journaux d'un serveur Linux.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Debian 13 (**CP3-01**), droits root/sudo.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian / journald | 13.6 « trixie » — journald par défaut | 24/07/2026 |
| `logrotate` / `rsyslogd -N1` | **testés dans le bac à sable** | 24/07/2026 |

> ⚠️ Sous Debian 12/13, **rsyslog n'est plus installé par défaut** : le journal système est **systemd-journald**. Il n'y a donc **pas de `/var/log/syslog`** tant que rsyslog n'est pas installé — on consulte les journaux avec **`journalctl`**.

---

## Procédure — CLI

### journald (journal par défaut)

```bash
# Consulter (voir aussi CP3-06)
journalctl -u ssh -b -p err
journalctl --disk-usage            # place occupée par le journal

# Rendre le journal persistant + borner sa taille : /etc/systemd/journald.conf
#   [Journal]
#   Storage=persistent
#   SystemMaxUse=500M
sudo mkdir -p /var/log/journal
sudo systemctl restart systemd-journald

# Purger
sudo journalctl --vacuum-size=200M     # garder 200 Mo max
sudo journalctl --vacuum-time=30d      # garder 30 jours
```

### logrotate (rotation des fichiers `/var/log/*`)

Fichier type dans `/etc/logrotate.d/monapp` :

```text
/var/log/monapp/*.log {
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 0640 root adm
}
```

```bash
logrotate --debug /etc/logrotate.d/monapp   # simulation (n'écrit rien)
```

### rsyslog (optionnel : syslog classique / centralisation)

```bash
sudo apt install rsyslog           # recrée /var/log/syslog
rsyslogd -N1                       # valider la configuration
```

Centraliser vers un collecteur — `/etc/rsyslog.d/remote.conf` :

```text
*.*   @@192.168.10.5:514           # @@ = TCP (fiable) ; @ = UDP
```

---

## Vérification (sorties obtenues dans le bac à sable)

```
logrotate --debug …  → "rotating pattern: /var/log/monapp/*.log after 1 days (7 rotations)"
                       "empty log files are not rotated"
rsyslogd -N1         → "config validation run (level 1) … End of config validation run. Bye."
journalctl --disk-usage → taille occupée par le journal
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Pas de `/var/log/syslog` | Comportement normal (journald) | Utiliser `journalctl` ; installer `rsyslog` si besoin |
| Journal qui remplit le disque | Pas de borne | `SystemMaxUse=` + `journalctl --vacuum-*` |
| logs non centralisés | Mauvais protocole/port | `@@host:514` (TCP), ouvrir 514 sur le collecteur |
| logrotate n'agit pas | Déclencheur absent | `logrotate.timer` (systemd) ou `cron.daily` |

## Sécurité et bonnes pratiques

- **Journaux persistants** pour la traçabilité (obligations légales — **CT4-01**).
- **Centraliser** les logs des serveurs critiques (analyse, corrélation d'incidents).
- Restreindre l'accès aux journaux (groupe `systemd-journal`).

## ⚠️ À ne pas confondre / obsolète

- Sous Debian 12/13, **`/var/log/syslog` n'existe plus par défaut** → `journalctl`.
- rsyslog : **`@@`** = TCP, **`@`** = UDP (ne pas confondre).
- La rotation logrotate est aujourd'hui déclenchée par un **timer systemd** (`logrotate.timer`), plus seulement par cron.

---

## Sources (doc officielle)

- [Debian Wiki — Rsyslog](https://wiki.debian.org/Rsyslog) — consulté le 24/07/2026
- [journald.conf(5) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/systemd/journald.conf.5.en.html) — consulté le 24/07/2026
- [logrotate(8) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/logrotate/logrotate.8.en.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées · [x] rien d'obsolète (journald par défaut, pas de syslog) · [x] **logrotate/rsyslog testés dans le bac à sable** · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
