# STO-06 — Gérer le stockage avec LVM (PV/VG/LV, étendre un volume à chaud)

**Objectif** : mettre en place **LVM** (Physical Volume → Volume Group → Logical Volume) et **étendre** un volume **et son système de fichiers** à chaud, sans interruption.

**Rattachement REAC** : CP3 (exploitation Linux) / STO — savoir-faire : gérer un stockage flexible.

**Durée** : ~30 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un serveur **Debian 13** (**CP3-01**), accès **root**, paquet **lvm2**, un ou plusieurs disques.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Outils | **LVM2**, `resize2fs` (Debian 13) | 24/07/2026 |
| Croissance du FS (`resize2fs`) | **testée en bac à sable** | 24/07/2026 |
| Commandes LVM sur disques | **à tester en lab** (root/loop) | 24/07/2026 |

> **LVM** apporte la **souplesse** que les partitions classiques n'ont pas : **PV** (disque/partition marqué) → **VG** (pool agrégeant les PV) → **LV** (volume « logique » redimensionnable, éventuellement étalé sur plusieurs disques). On peut **agrandir à chaud**, déplacer, faire des **snapshots**.

---

## Procédure — CLI

### Créer la pile LVM

```bash
pvcreate /dev/sdb /dev/sdc              # marquer les disques comme Physical Volumes
vgcreate vg_data /dev/sdb /dev/sdc      # créer le Volume Group (pool)
lvcreate -L 20G -n lv_app vg_data       # créer un Logical Volume de 20 Go
mkfs.ext4 /dev/vg_data/lv_app           # formater, puis monter (fstab par UUID — STO-05)
```

### Étendre à chaud (LV **et** système de fichiers)

```bash
# (au besoin) ajouter un disque au pool
pvcreate /dev/sdd && vgextend vg_data /dev/sdd

# Étendre le LV de +50 Go ET le FS en une commande (-r = resize du FS)
lvextend -L +50G -r /dev/vg_data/lv_app
```

> ⚠️ **Piège classique** : `lvextend` **seul** agrandit le **LV** mais **pas** le système de fichiers → `lvs` grossit mais `df` non. Il faut **`resize2fs`** (ext4) / **`xfs_growfs`** (xfs), ou l'option **`-r`**.

> **Testé en bac à sable** : après agrandissement du support, `resize2fs` fait passer un ext4 de **16384 → 32768 blocs** (croissance effective du FS) — c'est l'étape que `lvextend -r` réalise automatiquement.

---

## Vérification (comment savoir que ça marche)

```bash
pvs ; vgs ; lvs                  # vue d'ensemble PV / VG / LV
df -h /mnt/app                   # la NOUVELLE taille apparaît (preuve que le FS a suivi)
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| `lvs` a grossi mais **pas `df`** | FS **non** redimensionné | `resize2fs`/`xfs_growfs` (ou `lvextend -r`) |
| VG plein | Plus d'espace libre | `pvcreate` + `vgextend` avec un nouveau disque |
| Impossible de réduire un XFS | **XFS ne se réduit pas** | Utiliser ext4 (offline) ou recréer |
| PV « in use » | Déjà partitionné/monté | Démonter/nettoyer avant `pvcreate` |

## Sécurité et bonnes pratiques

- **Snapshots LVM** avant une opération risquée (rollback — **CP8-10**).
- **LVM ≠ redondance** : le poser **au-dessus** d'un RAID (**STO-01/02**) pour la tolérance de panne.
- **Sauvegarder** avant un *reshape*/réduction ; l'agrandissement est sûr, la réduction est risquée.

## ⚠️ À ne pas confondre / obsolète

- **PV** (support) ≠ **VG** (pool) ≠ **LV** (volume) : trois niveaux.
- `lvextend` (agrandit le LV) **≠** `resize2fs` (agrandit le FS) — les **deux** sont nécessaires (`-r` combine).
- **XFS** : croissance seulement (`xfs_growfs`), **pas** de réduction ; **ext4** peut réduire (hors ligne).

---

## Sources (doc officielle)

- [Debian Manpages — lvextend(8)](https://manpages.debian.org/bookworm/lvm2/lvextend.8.en.html) — consulté le 24/07/2026
- [Debian Manpages — resize2fs(8)](https://manpages.debian.org/bookworm/e2fsprogs/resize2fs.8.en.html) — consulté le 24/07/2026
- [Red Hat — Configuring and managing LVM](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/configuring_and_managing_logical_volumes/index) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] daté 24/07/2026 · [x] rien d'obsolète (extension à chaud, `-r`) · [x] **`resize2fs` testé en bac à sable** / pile LVM à tester en lab · [x] conforme doc LVM/Debian · [x] vérification présente (`lvs`/`df`) · [x] sécurité (snapshot, LVM≠RAID) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
