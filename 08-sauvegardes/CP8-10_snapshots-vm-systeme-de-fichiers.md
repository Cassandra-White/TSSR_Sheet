# CP8-10 — Réaliser et gérer des snapshots (VM / système de fichiers)

**Objectif** : créer, restaurer et supprimer des **snapshots** de VM (Proxmox/Hyper-V) et de système de fichiers (LVM/ZFS), en comprenant qu'un snapshot **n'est pas** une sauvegarde.

**Rattachement REAC** : CP8 « Sauvegardes et restaurations des éléments de l'infrastructure » — savoir-faire : utiliser les snapshots.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un hôte **Proxmox VE 9** (**CP5-01**) avec un stockage compatible snapshots, ou un serveur **Linux** avec **LVM-thin**/**ZFS**.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Virtualisation | **Proxmox VE 9.2** (QCOW2 / ZFS / LVM-thin / Ceph) | 24/07/2026 |
| Système de fichiers | **LVM-thin**, **ZFS** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **Snapshot = photo instantanée** d'un état (mécanisme **copy-on-write**), **rapide** et **réversible**, mais **stockée sur le même support** que la donnée. Idéal comme **filet de sécurité court terme** (avant une mise à jour). ⚠️ Si le stockage tombe, **le snapshot tombe avec** → ce **n'est pas** une sauvegarde.

---

## Procédure — GUI

### Proxmox VE (snapshot de VM)

1. **VM ▸ Snapshots ▸ Take Snapshot** : nom (`avant-maj`), cocher **Include RAM** pour capturer l'état **en cours d'exécution**.
2. **Rollback** : sélectionner le snapshot → **Rollback** → la VM revient à l'état figé.
3. **Supprimer** le snapshot dès qu'il n'est plus utile (il grossit et ralentit la VM).

> Nécessite un format/stockage **snapshot-capable** : **QCOW2**, **ZFS**, **LVM-thin** ou **Ceph**. Le **LVM classique** (non-thin) et le format **raw** sur dossier **ne** gèrent **pas** les snapshots.

### Hyper-V (checkpoint)

- **Hyper-V Manager ▸ Checkpoint** : préférer les **Production Checkpoints** (cohérents via **VSS**) aux *Standard* pour un serveur.

## Procédure — CLI

### Proxmox VE

```bash
qm snapshot 100 avant-maj --vmstate 1     # snapshot de la VM 100 (avec RAM)
qm listsnapshot 100                        # lister
qm rollback 100 avant-maj                  # revenir à l'état
qm delsnapshot 100 avant-maj               # supprimer
```

### LVM-thin

```bash
lvcreate --snapshot --name snap_root vg0/root      # snapshot du volume logique
lvconvert --merge vg0/snap_root                    # restaurer (fusion au prochain montage)
```

### ZFS

```bash
zfs snapshot pool/data@avant-maj      # créer
zfs list -t snapshot                  # lister
zfs rollback pool/data@avant-maj      # restaurer
zfs destroy pool/data@avant-maj       # supprimer
```

---

## Vérification (comment savoir que ça marche)

- Le snapshot apparaît dans la liste (`qm listsnapshot`, `zfs list -t snapshot`).
- Après une modification puis un **rollback**, l'état revient bien à celui figé.
- L'espace consommé par le snapshot **augmente** avec les écritures (copy-on-write).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| « Snapshots non supportés » | Stockage **non compatible** (LVM non-thin, raw/dir) | Utiliser **QCOW2/ZFS/LVM-thin/Ceph** |
| Pool qui se remplit | Snapshot **LVM-thin/ZFS** trop ancien + écritures | Supprimer les snapshots inutiles rapidement |
| VM lente | **Chaîne** de snapshots trop longue | Consolider ; ne pas empiler |
| DC AD cassé après rollback | **USN rollback** (**CP8-03**) | Ne **jamais** rollback un DC par snapshot |

## Sécurité et bonnes pratiques

- **Snapshot ≠ sauvegarde** : toujours **compléter** par une vraie sauvegarde **externe** (PBS — **CP8-05** ; règle 3-2-1 — **CP8-01**).
- Snapshots **courts** (heures/jours) avant une opération risquée, puis **suppression**.
- **Jamais** de rollback de snapshot sur un **contrôleur de domaine** (sauf hyperviseur gérant **VM-GenerationID**).

## ⚠️ À ne pas confondre / obsolète

- **Snapshot** (même stockage, court terme) ≠ **sauvegarde** (copie externe, historisée).
- Hyper-V : l'ancien terme « **snapshot** » est devenu « **checkpoint** » ; pour un serveur, **Production Checkpoint** (VSS).
- **LVM classique** ≠ **LVM-thin** : seul le *thin* fait des snapshots efficaces (le classique fige une taille dédiée).

---

## Sources (doc officielle)

- [Proxmox VE — VM snapshots / storage](https://pve.proxmox.com/wiki/Storage) — consulté le 24/07/2026
- [Microsoft — Hyper-V checkpoints (standard vs production)](https://learn.microsoft.com/en-us/troubleshoot/windows-server/virtualization/hyper-v-snapshots-checkpoints-differencing-disks) — consulté le 24/07/2026
- [OpenZFS — Snapshots](https://openzfs.github.io/openzfs-docs/man/master/8/zfs-snapshot.8.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées (PVE 9.2) · [x] rien d'obsolète (snapshot≠backup, checkpoint) · [x] procédure **à tester en lab** · [x] GUI conforme doc Proxmox/Microsoft · [x] vérification présente · [x] sécurité (compléter par backup, anti USN rollback) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
