# STO-08 — Retirer proprement un disque (démontage, retrait du RAID/VG)

**Objectif** : retirer un disque **sans perte ni corruption** — migrer les données, le sortir des couches logiques (RAID/LVM/pool), puis l'extraire physiquement.

**Rattachement REAC** : CP2 / CP3 / STO — savoir-faire : maintenir le stockage d'un serveur.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Identifier le disque à retirer (**série**/LED) et son **rôle** (isolé, RAID, LVM, pool).
- De l'espace libre suffisant si des données doivent être **migrées**.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Linux | **mdadm** / **LVM** / `wipefs` (Debian 13) | 24/07/2026 |
| Windows | **Storage Spaces** (`Remove-PhysicalDisk`) | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **Règle** : on ne **débranche jamais** un disque en cours d'usage. On **migre/démonte**, on le **sort des grappes/pools**, on **efface les métadonnées**, **puis** on l'extrait.

---

## Procédure — CLI

### Linux — disque isolé

```bash
umount /data                              # démonter
sed -i '\#/data#d' /etc/fstab             # retirer la ligne fstab
wipefs -a /dev/sdb                        # effacer les signatures de FS
```

### Linux — disque membre d'un VG LVM

```bash
pvmove /dev/sdb                           # MIGRER les données vers les autres PV du VG
vgreduce vg_data /dev/sdb                 # sortir le PV du Volume Group
pvremove /dev/sdb                         # supprimer l'étiquette LVM
wipefs -a /dev/sdb
```

### Linux — disque membre d'un RAID mdadm

```bash
mdadm /dev/md0 --fail /dev/sdb1 --remove /dev/sdb1
mdadm --grow /dev/md0 --raid-devices=3    # si on réduit la grappe (sinon remplacer — STO-07)
```

### Windows — retrait d'un pool Storage Spaces

```powershell
# Le pool doit avoir assez d'espace libre pour "drainer" le disque
Remove-PhysicalDisk -StoragePoolFriendlyName Pool1 `
  -PhysicalDisks (Get-PhysicalDisk -FriendlyName "Disk 3")
```

---

## Vérification (comment savoir que c'est propre)

```bash
pvs ; vgs                       # le disque n'apparaît plus dans le VG
lsblk                           # le disque est libre (aucune partition montée/active)
```

- Windows : `Get-PhysicalDisk` ne liste plus le disque dans le pool → extraction **sûre**.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| `pvmove` échoue | Pas assez d'espace libre dans le VG | Ajouter un PV d'abord (`vgextend`) |
| « device is busy » | Montage/process actif | `umount`, `lsof`/`fuser` sur le point de montage |
| Retrait de pool refusé | Capacité libre insuffisante | Libérer de l'espace / ajouter un disque |
| Disque encore vu « en RAID » | Métadonnées résiduelles | `wipefs -a` / `mdadm --zero-superblock` |

## Sécurité et bonnes pratiques

- **Migrer/sauvegarder** avant tout retrait (**CP8**).
- **Effacer les données** avant réforme/réaffectation (rémanence — **RGPD**, **DEEE** **CP1-12**) : `wipefs`, effacement sécurisé, voire destruction.
- Vérifier **l'absence de montage/usage actif** avant l'extraction physique.

## ⚠️ À ne pas confondre / obsolète

- **Retrait à chaud d'un disque en service** = corruption → **migrer/sortir des grappes d'abord**.
- `vgreduce` (sortir du VG) ≠ `pvremove` (supprimer l'étiquette) : faire **les deux**.
- « Formater » ≠ « **effacer** » : pour la réforme, un simple format ne suffit pas.

---

## Sources (doc officielle)

- [Debian Manpages — pvmove(8) / vgreduce(8)](https://manpages.debian.org/bookworm/lvm2/pvmove.8.en.html) — consulté le 24/07/2026
- [Debian Manpages — wipefs(8)](https://manpages.debian.org/bookworm/util-linux/wipefs.8.en.html) — consulté le 24/07/2026
- [Microsoft Learn — Remove-PhysicalDisk](https://learn.microsoft.com/en-us/powershell/module/storage/remove-physicaldisk) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI (Linux + Windows) · [x] daté 24/07/2026 · [x] rien d'obsolète (migrer avant retrait) · [x] procédure **à tester en lab** · [x] conforme doc LVM/Microsoft · [x] vérification présente (`pvs`/`lsblk`) · [x] sécurité (migration, effacement RGPD/DEEE) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
