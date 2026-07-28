# CP3-10 — Surveiller les ressources et diagnostiquer (htop, df, journaux)

**Objectif** : surveiller CPU, mémoire, disque et réseau d'un serveur Debian et poser un diagnostic quand quelque chose ne va pas.

**Rattachement REAC** : CP3 « Exploiter des serveurs Linux » — savoir-faire : superviser et diagnostiquer un serveur Linux.

**Durée** : ~15 min · **Niveau** : débutant.

---

## Prérequis

- Debian 13 (**CP3-01**), accès console/SSH. Paquets utiles : `htop`, `sysstat` (pour `iostat`).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian | 13.6 « trixie » | 24/07/2026 |
| `df` / `free` / `ss` / `vmstat` / `top` / `uptime` | **testés dans le bac à sable** | 24/07/2026 |

---

## Procédure — CLI

### Charge et processus

```bash
uptime                              # load average (1/5/15 min)
top                                 # temps réel (q pour quitter) ; htop = version conviviale
ps aux --sort=-%cpu | head          # top consommateurs CPU
ps aux --sort=-%mem | head          # top consommateurs mémoire
```

### Mémoire et disque

```bash
free -h                             # RAM / swap
df -h                               # espace par système de fichiers
du -sh /var/* | sort -h             # ce qui occupe le disque (ici /var)
lsblk                               # disques et partitions
```

### Système, entrées/sorties, réseau

```bash
vmstat 1                            # CPU/IO/mémoire en continu (colonne wa = attente disque)
iostat -x 1                         # détail I/O disque (paquet sysstat)
ss -tulpn                           # ports en écoute + processus (surface d'attaque)
```

### Journaux et noyau

```bash
journalctl -p err -b                # erreurs depuis le démarrage (voir CP3-06)
dmesg -T | tail                     # messages noyau récents (matériel, OOM…)
```

---

## Vérification (sorties obtenues dans le bac à sable)

```
df -h        → /dev/nvme0n1p1  9.6G  5.6G  4.0G  59% /
free -h      → Mem: 3.8Gi total, 3.1Gi free, 549Mi buff/cache
uptime       → load average: 0.00, 0.00, 0.00
vmstat 1 1   → colonnes r/b … us sy id wa (id≈99 = CPU au repos)
ss -tuln     → LISTEN 0.0.0.0:3128 …
```

## Dépannage (erreurs fréquentes)

| Symptôme | Piste de diagnostic | Action |
|---|---|---|
| Serveur lent | `top`/`uptime` (charge), `vmstat` (colonne **wa** = I/O) | Identifier le processus, tracer l'I/O |
| « No space left on device » | `df -h` puis `du -sh /var/* \| sort -h` | Purger logs ; `journalctl --vacuum-size=200M` |
| Processus tué sans raison | `dmesg -T \| grep -i oom` | Manque de RAM (OOM killer) : ajuster/limiter |
| Port inattendu ouvert | `ss -tulpn` | Arrêter/désactiver le service concerné |

## Sécurité et bonnes pratiques

- **Auditer régulièrement les ports ouverts** (`ss -tulpn`) : chaque service exposé = surface d'attaque.
- Surveiller le remplissage de **`/var`** (journaux) pour éviter l'arrêt des services.
- Pour un parc, mettre en place une **supervision centralisée** (Zabbix/Centreon — **CP4-17**).

## ⚠️ À ne pas confondre / obsolète

- **`netstat` et `ifconfig`** (paquet *net-tools*) sont obsolètes → **`ss`** et **`ip`**.
- La colonne **« available »** de `free` (pas « free ») indique la mémoire réellement disponible (le cache est réutilisable).
- `htop`/`btop` sont un confort ; `top` est toujours présent par défaut.

---

## Sources (doc officielle)

- [Debian Reference — System tips (supervision)](https://www.debian.org/doc/manuals/debian-reference/ch09.en.html) — consulté le 24/07/2026
- [ss(8) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/iproute2/ss.8.en.html) — consulté le 24/07/2026
- [vmstat(8) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/procps/vmstat.8.en.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées · [x] rien d'obsolète (`ss`/`ip` vs net-tools) · [x] **commandes testées dans le bac à sable** · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
