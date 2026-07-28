# STO-02 — Configurer un RAID logiciel sous Linux (mdadm)

**Objectif** : créer et gérer une grappe **RAID logicielle** avec **mdadm**, la rendre persistante, la surveiller et **remplacer** un disque défaillant.

**Rattachement REAC** : CP3 (exploitation Linux) / STO — savoir-faire : mettre en œuvre un stockage résilient logiciel.

**Durée** : ~30 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un serveur **Debian 13** (**CP3-01**), accès **root**, au moins **3–4 disques** identiques (ou partitions).
- Paquet **mdadm** installé.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Outil | **mdadm** (Debian 13) | 24/07/2026 |
| Principe de parité (RAID 5) | **testé en bac à sable** (XOR) | 24/07/2026 |
| Assemblage sur disques | **à tester en lab** (nécessite root/loop) | 24/07/2026 |

> **RAID logiciel** = le **noyau Linux** gère la grappe (pas de contrôleur dédié) : gratuit, portable, souple, mais consomme un peu de **CPU**. `mdadm` crée des périphériques `/dev/mdN`.

---

## Procédure — CLI

### Créer la grappe

```bash
# RAID5 : 4 disques actifs + 1 disque de secours (spare)
mdadm --create /dev/md0 --level=5 --raid-devices=4 \
      /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1 --spare-devices=1 /dev/sdf1

cat /proc/mdstat                 # état/progression (ex. [UUUU] = tous actifs)
mdadm --detail /dev/md0          # détail : niveau, état, disques
```

### Rendre persistant (au démarrage)

```bash
mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf   # enregistrer la grappe
sudo update-initramfs -u                                    # inclure au boot
```

### Formater et monter (voir **STO-05**)

```bash
mkfs.ext4 /dev/md0
echo '/dev/md0 /data ext4 defaults 0 2' | sudo tee -a /etc/fstab && sudo mount -a
```

### Remplacer un disque défaillant

```bash
mdadm --manage /dev/md0 --fail /dev/sdc1 --remove /dev/sdc1   # sortir le disque HS
mdadm --manage /dev/md0 --add  /dev/sdg1                      # ajouter le neuf -> rebuild auto
```

### Surveiller

```bash
mdadm --monitor --scan --mail=admin@lab.local --daemonise    # alerte e-mail sur incident
```

> **Parité RAID 5 testée** (bac à sable) : `P = d1 XOR d2 XOR d3` ; après perte de `d2`, `d2 = P XOR d1 XOR d3` redonne la **valeur exacte** → la grappe se reconstruit. Les commandes `mdadm` (root/loop requis) sont **à exécuter en lab**.

---

## Vérification (comment savoir que ça marche)

```bash
cat /proc/mdstat            # [UUUU] = OK ; [U_UU] + "recovery" = reconstruction en cours
mdadm --detail /dev/md0 | grep -E "State|Failed|Spare"
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Grappe absente au boot | `mdadm.conf`/initramfs non mis à jour | `--detail --scan` + `update-initramfs -u` |
| Reconstruction lente | I/O intense / gros disques | Ajuster `sync_speed_*` ; patience ; sauvegarder d'abord |
| Disque marqué *faulty* | Panne / câble | Remplacer, `--add` ; vérifier **SMART** (**STO-09**) |
| 2ᵉ panne au rebuild | **URE** (gros disques) | Préférer **RAID 6** (`--level=6`) ou **RAID 10** |

## Sécurité et bonnes pratiques

- **Surveiller** (`mdadm --monitor` + **SMART**) et prévoir un **spare**.
- **Persister** la config (`mdadm.conf` + initramfs) sinon la grappe peut ne pas remonter.
- **Gros disques** : **RAID 6/10** plutôt que RAID 5 (fenêtre de rebuild).
- **RAID ≠ sauvegarde** (**CP8-01**) : garder de vraies sauvegardes externes.

## ⚠️ À ne pas confondre / obsolète

- **RAID logiciel (mdadm)** ≠ **RAID matériel** (contrôleur — **STO-01**) ≠ **RAID du FS** (ZFS/btrfs — **STO-12**).
- **RAID 5** sur gros disques = risqué → **RAID 6/10**.
- Ne pas oublier la **persistance** : une grappe non déclarée peut ne pas se réassembler.

---

## Sources (doc officielle)

- [mdadm(8) — Manuel](https://manpages.debian.org/bookworm/mdadm/mdadm.8.en.html) — consulté le 24/07/2026
- [Linux RAID Wiki — RAID setup](https://raid.wiki.kernel.org/index.php/RAID_setup) — consulté le 24/07/2026
- [Debian Wiki — mdadm](https://wiki.debian.org/mdadm) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI (mdadm) · [x] daté 24/07/2026 · [x] rien d'obsolète (RAID6/10, SMART) · [x] **parité testée en bac à sable** / assemblage à tester en lab · [x] conforme doc mdadm/kernel · [x] vérification présente (`/proc/mdstat`) · [x] sécurité (monitor, spare, RAID≠backup) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
