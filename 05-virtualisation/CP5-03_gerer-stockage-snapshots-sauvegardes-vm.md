# CP5-03 — Gérer stockage, snapshots et sauvegardes de VM

**Objectif** : gérer le **stockage** des VM Proxmox, prendre des **snapshots** (rollback rapide) et **sauvegarder** les VM (vzdump vers PBS/NFS).

**Rattachement REAC** : CP5 « Maintenir des serveurs dans une infrastructure virtualisée » — savoir-faire : protéger les VM.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un hôte **Proxmox VE 9** avec des VM (**CP5-02**) et un stockage adapté.
- Idéalement un **Proxmox Backup Server** (**CP8-05**) ou un partage **NFS**.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Hyperviseur | **Proxmox VE 9.2** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **Trois notions à ne pas confondre** : le **stockage** (où vivent les disques de VM), les **snapshots** (photo instantanée, **court terme**, même stockage) et les **sauvegardes** (copie **externe** historisée). Voir **STO-11**, **CP8-10** et **CP8-05**.

---

## Procédure — GUI

### Stockage

- **Datacenter ▸ Storage ▸ Add** : choisir le type (**LVM-thin/ZFS/Ceph** pour les snapshots, NFS/dir pour les fichiers) et le **Content** (**STO-11**).

### Snapshots (rollback rapide)

1. **VM ▸ Snapshots ▸ Take Snapshot** : nom, **Include RAM** pour l'état en cours d'exécution.
2. **Rollback** pour revenir ; **supprimer** dès que possible (le snapshot grossit).

### Sauvegardes (vzdump)

3. **Datacenter ▸ Backup ▸ Add** : job planifié (heure, VM, **Storage = PBS/NFS**, **Mode = Snapshot**, rétention) — ou **VM ▸ Backup ▸ Backup now**.

## Procédure — CLI

```bash
# Snapshot avec état mémoire
qm snapshot 100 avant-maj --vmstate 1
qm rollback 100 avant-maj

# Sauvegarde à chaud (zéro interruption) vers le stockage PBS
vzdump 100 --storage pbs --mode snapshot --compress zstd
```

> **Modes vzdump** : **snapshot** (à chaud, **sans interruption**, mode recommandé en production — nécessite un stockage compatible COW), **suspend**, **stop**.

---

## Vérification (comment savoir que ça marche)

- Le **snapshot** apparaît dans la liste et le **rollback** ramène l'état attendu.
- La **sauvegarde** apparaît dans le stockage (PBS : chunks dédupliqués + tâche **Verify** verte).
- Une **restauration de test** (**CP8-07**) redémarre la VM.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Snapshot impossible | Stockage **sans COW** (LVM épais/dir raw) | LVM-**thin**/ZFS/Ceph (**STO-11**) |
| Sauvegarde échoue | Stockage/droit/espace | Vérifier la cible et l'espace ; PBS/NFS |
| Datastore plein (PBS) | Rétention non appliquée | **Prune + Garbage Collection** (**CP8-05**) |
| VM lente | **Chaîne** de snapshots trop longue | Consolider / supprimer les snapshots |

## Sécurité et bonnes pratiques

- **Snapshot ≠ sauvegarde** (**CP8-10**) : la sauvegarde doit être **externe** et historisée.
- Appliquer **3-2-1** (**CP8-01**) : sauvegardes **hors site** (PBS distant / S3 — **CP8-11**), **vérifiées** et **testées**.
- **Guest agent** activé (**CP5-02**) pour des sauvegardes **cohérentes**.

## ⚠️ À ne pas confondre / obsolète

- **Snapshot** (même stockage, court terme) ≠ **sauvegarde vzdump/PBS** (externe).
- **Mode snapshot** (à chaud) ≠ **snapshot de VM** : le mode vzdump *snapshot* fait une sauvegarde sans arrêt.
- **PVE 9** : *snapshots as volume chains* (qcow2) élargissent les stockages compatibles (**STO-11**).

---

## Sources (doc officielle)

- [Proxmox VE — Backup and Restore (vzdump)](https://pve.proxmox.com/pve-docs/chapter-vzdump.html) — consulté le 24/07/2026
- [Proxmox VE — Storage](https://pve.proxmox.com/pve-docs/chapter-pvesm.html) — consulté le 24/07/2026
- [Proxmox Backup Server — Documentation](https://pbs.proxmox.com/docs/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (`qm`/`vzdump`) · [x] version datée (PVE 9.2) · [x] rien d'obsolète (mode snapshot, volume chains) · [x] procédure **à tester en lab** · [x] conforme doc Proxmox · [x] vérification présente (Verify/restore) · [x] sécurité (snapshot≠backup, 3-2-1) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
