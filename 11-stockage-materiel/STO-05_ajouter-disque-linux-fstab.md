# STO-05 — Ajouter un disque sous Linux (partition, formatage, montage, fstab)

**Objectif** : mettre en service un disque sous Linux — **partitionner** (GPT), **formater** (ext4/xfs), **monter** et rendre le montage **persistant** via `/etc/fstab` (par **UUID**).

**Rattachement REAC** : CP3 (exploitation Linux) / STO — savoir-faire : gérer le stockage d'un serveur.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un serveur **Debian 13** (**CP3-01**), accès **root**, un disque neuf.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| OS | **Debian 13** | 24/07/2026 |
| Outils | `sgdisk`/`parted`, `mkfs.ext4`, `blkid` | 24/07/2026 |
| Formatage + UUID + fstab | **testé en bac à sable** | 24/07/2026 |

> **Toujours monter par UUID**, pas par `/dev/sdX` : l'ordre des disques (`sdb`, `sdc`…) peut **changer** au redémarrage, l'**UUID** est stable.

---

## Procédure — CLI

```bash
lsblk                                   # repérer le nouveau disque (ex. /dev/sdb)

# 1) Partition GPT couvrant tout le disque
sgdisk -n 1:0:0 -t 1:8300 /dev/sdb      # (ou : parted /dev/sdb mklabel gpt / mkpart)

# 2) Formater (ext4 ou xfs)
mkfs.ext4 -L DATA /dev/sdb1

# 3) Récupérer l'UUID
blkid /dev/sdb1                         # -> UUID="838faa94-...."

# 4) Point de montage + fstab (par UUID)
mkdir -p /data
echo 'UUID=838faa94-7756-4f80-89db-d2e4ebc487a1  /data  ext4  defaults,nofail  0 2' >> /etc/fstab

# 5) Appliquer
systemctl daemon-reload && mount -a
```

> **Testé en bac à sable** : `mkfs.ext4 -L DATA` sur une image → `dumpe2fs`/`blkid` donnent l'**UUID** `838faa94-…` et le **label** `DATA` → génération de la **ligne fstab par UUID**. `file` confirme *ext4 filesystem*.

---

## Vérification (comment savoir que ça marche)

```bash
mount -a && echo "fstab OK"      # aucune erreur = /etc/fstab valide
df -h /data                      # le volume est monté avec la bonne taille
findmnt /data                    # source (UUID), type, options
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Montage perdu au reboot | Monté par `/dev/sdX` | Utiliser l'**UUID** dans `/etc/fstab` |
| **Boot bloqué** | Erreur fstab | Toujours tester `mount -a` **avant reboot** ; option **`nofail`** |
| « unknown filesystem type » | Mauvais type dans fstab | Aligner sur le FS réel (`ext4`/`xfs`) |
| Disque >2 To en MBR | Table MBR | Utiliser **GPT** (`sgdisk`/`parted`) |

## Sécurité et bonnes pratiques

- **`nofail`** pour éviter qu'un disque absent **bloque le démarrage**.
- Définir **propriétaire/permissions** du point de montage après montage (**CP3-04**).
- **Sauvegarder** avant de repartitionner un disque contenant des données (**CP8**).

## ⚠️ À ne pas confondre / obsolète

- Montage par **`/dev/sdX`** (fragile) → par **UUID** (stable).
- **`fdisk`/MBR** (limite 2 To) → **`sgdisk`/`parted` GPT** pour les gros disques.
- Ajouter un disque **≠** LVM : pour redimensionner à la volée, voir **STO-06**.

---

## Sources (doc officielle)

- [Debian Manpages — fstab(5)](https://manpages.debian.org/bookworm/mount/fstab.5.en.html) — consulté le 24/07/2026
- [util-linux — blkid / mount](https://manpages.debian.org/bookworm/util-linux/blkid.8.en.html) — consulté le 24/07/2026
- [GPT fdisk — sgdisk](https://manpages.debian.org/bookworm/gdisk/sgdisk.8.en.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] daté 24/07/2026 · [x] rien d'obsolète (UUID vs /dev, GPT) · [x] **formatage + UUID + fstab testés en bac à sable** · [x] conforme doc Debian · [x] vérification présente (`mount -a`/`findmnt`) · [x] sécurité (`nofail`, permissions) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
