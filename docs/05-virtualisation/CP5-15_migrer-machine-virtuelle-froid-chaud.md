# CP5-15 — Migrer une machine virtuelle (à froid et à chaud / live migration)

**Objectif** : migrer une VM **à froid** (éteinte) ou **à chaud** (en marche, sans coupure) au sein d'un cluster Proxmox, et **importer** une VM depuis VMware.

**Rattachement REAC** : CP5 « Maintenir des serveurs dans une infrastructure virtualisée » — savoir-faire : déplacer des VM.

**Durée** : ~25 min · **Niveau** : avancé.

---

## Prérequis

- Pour la migration **à chaud** intra-cluster : un **cluster** Proxmox (**CP5-08**) avec **stockage partagé** (Ceph/NFS).
- Pour l'import **VMware** : accès à l'hôte **ESXi/vCenter**.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Hyperviseur | **Proxmox VE 9.2** (Import Wizard) | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **À froid** = VM **éteinte**, on copie/convertit les disques (fiable, mais **interruption**). **À chaud (live)** = VM **en marche** déplacée **sans coupure** — nécessite un **stockage partagé** (cluster).

---

## Procédure — GUI + CLI

### Migration intra-cluster

```bash
# À chaud (online) — nécessite un stockage partagé
qm migrate 100 node2 --online

# À froid (offline) — VM éteinte
qm migrate 100 node2
```

*(GUI : VM ▸ **Migrate** → nœud cible, mode online/offline.)*

### Import depuis VMware (V2V)

- **Import Wizard** (PVE ≥ 8.1, recommandé) : **Datacenter ▸ Storage ▸ Add ▸ ESXi**, puis importer la VM (conversion **VMDK → qcow2/raw** automatique).

```bash
# Alternatives en ligne de commande
qm importovf 200 machine.ovf local-lvm          # depuis un export OVF (ovftool)
qm importdisk 200 disque.vmdk local-lvm         # importer un disque VMDK
```

> ⚠️ L'**Import Wizard** ne fait **pas** de live-migration d'une VM VMware **en marche** : **éteindre** la VM côté ESXi avant l'import.

---

## Vérification (comment savoir que ça marche)

- La VM **démarre sur la cible** ; disques et réseau fonctionnent.
- **À chaud** : un **ping continu** vers la VM ne se coupe pas pendant la migration.
- Après import VMware : installer le **QEMU Guest Agent** et **retirer les VMware Tools**.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Live migration refusée | Pas de **stockage partagé** | Ceph/NFS accessible par tous (**CP5-08**) |
| VM ne boote pas après import | **UEFI/pilotes** VMware | Régler OVMF/EFI ; pilotes **VirtIO** (**CP5-02**) |
| Réseau KO après migration | MAC/pont/VLAN | Vérifier `net0` (bridge/tag — **CP5-05**) |
| Import lent | Conversion de disque | Prévoir une fenêtre ; réseau rapide |

## Sécurité et bonnes pratiques

- **Sauvegarder avant** (**CP8-05**) et **planifier** (fenêtre de changement — **CP5-11**).
- **Tester** la VM migrée avant de retirer la source.
- Post-migration : **guest agent** installé, **VMware Tools** retirés, vérifs réseau/perfs.

## ⚠️ À ne pas confondre / obsolète

- **À froid** (interruption, simple) ≠ **à chaud/live** (sans coupure, **stockage partagé**).
- **Import Wizard** (moderne, direct ESXi) vs export **manuel** (`ovftool` + `qm importovf`).
- **Migration** (déplacer) ≠ **sauvegarde/restauration** (copier).

---

## Sources (doc officielle)

- [Proxmox VE — Migration (chapter-pvecm)](https://pve.proxmox.com/pve-docs/chapter-pvecm.html) — consulté le 24/07/2026
- [Proxmox VE — Migrate to Proxmox VE (Import Wizard)](https://pve.proxmox.com/wiki/Migrate_to_Proxmox_VE) — consulté le 24/07/2026
- [Proxmox VE — `qm` (importovf/importdisk)](https://pve.proxmox.com/pve-docs/qm.1.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI + CLI (`qm migrate`/`import`) · [x] version datée (PVE 9.2) · [x] rien d'obsolète (Import Wizard) · [x] procédure **à tester en lab** · [x] conforme doc Proxmox · [x] vérification présente (ping continu/boot) · [x] sécurité (sauvegarde, fenêtre, test) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
