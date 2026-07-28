# STO-11 — Ajouter/étendre un stockage de virtualisation Proxmox (LVM-thin, ZFS, Ceph)

**Objectif** : ajouter ou étendre un stockage dans **Proxmox VE** et **choisir le bon type** (LVM-thin, ZFS, Ceph) selon le besoin (snapshots, intégrité, haute disponibilité).

**Rattachement REAC** : CP5 (infrastructure virtualisée) / STO — savoir-faire : gérer le stockage d'un hyperviseur.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un hôte **Proxmox VE 9** (**CP5-01**), un ou plusieurs disques disponibles.
- Pour Ceph : un **cluster** d'au moins **3 nœuds**.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Virtualisation | **Proxmox VE 9.2** | 24/07/2026 |
| Types | **LVM-thin / ZFS / Ceph RBD** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> Proxmox distingue le stockage **bloc** (LVM, LVM-thin, ZFS, iSCSI, **Ceph RBD**) et **fichier** (Directory, NFS, CIFS, ZFS, CephFS). Le **type** conditionne les **snapshots**, la **réplication** et la **haute disponibilité**.

---

## Choisir le bon type

| Type | Points forts | Snapshots | Usage |
|---|---|---|---|
| **LVM-thin** | Léger, **COW** local, peu d'overhead | Oui | Hôte simple, local |
| **ZFS** | **Intégrité** (checksums), compression, réplication | Oui | Intégrité maximale, local |
| **Ceph RBD** | **Distribué**, **HA**, pas de SPOF | Oui | **Cluster** multi-nœuds |
| Directory/NFS/CIFS | Simple/partagé (fichier) | Selon (qcow2) | Sauvegardes, ISO, partagé |

> **Nouveau PVE 9** : *snapshots as volume chains* (qcow2) — snapshots **vendor-agnostic** pour LVM « épais »/iSCSI/NFS/CIFS (aperçu technique).

---

## Procédure — GUI

1. **Datacenter ▸ Storage ▸ Add** → choisir le type (LVM-Thin, ZFS, RBD…).
2. Renseigner le VG/thinpool ou le pool ZFS/Ceph, et le **Content** (Disk image, Container, Backup, ISO…).
3. Le stockage devient disponible pour créer des **disques de VM/CT**.

## Procédure — CLI (`pvesm`)

```bash
# Ajouter un stockage LVM-thin
pvesm add lvmthin local-thin --vgname pve --thinpool data --content images,rootdir

# Ajouter un pool ZFS existant comme stockage
pvesm add zfspool tank-vm --pool tank/vm --content images,rootdir

pvesm status                         # état des stockages (actif/espace)
```

### Étendre

- **LVM-thin** : ajouter un disque au **VG** puis étendre le **thinpool** (**STO-06**).
- **ZFS** : `zpool add` / remplacer par de plus gros disques (**STO-12**).
- **Ceph** : ajouter des **OSD** (disques) au cluster.

---

## Vérification (comment savoir que ça marche)

- `pvesm status` liste le stockage **actif** avec sa capacité.
- On peut **créer un disque de VM** dessus et prendre un **snapshot** (si supporté).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Stockage **inactif** | Nœud/réseau/pool absent | Vérifier le VG/pool, la connectivité (Ceph/NFS) |
| Pas de snapshot possible | Type sans COW (LVM épais/dir raw) | LVM-**thin**/ZFS/RBD, ou volume-chains (PVE 9) |
| ZFS gourmand en RAM | ARC | Dimensionner la RAM ; **ECC** recommandé |
| Ceph « HEALTH_WARN » | OSD/quorum | Vérifier l'état du cluster et le nombre de nœuds |

## Sécurité et bonnes pratiques

- **Réseau de stockage** dédié (Ceph/iSCSI/réplication).
- **ZFS** : **scrub** régulier + **ECC RAM** recommandée (**STO-12**).
- **Ceph** : min **3 nœuds** pour la HA ; surveiller la santé.
- **Stockage ≠ sauvegarde** : garder **PBS** (**CP8-05**) en plus.

## ⚠️ À ne pas confondre / obsolète

- **LVM-thin** (local, léger) vs **ZFS** (intégrité, local) vs **Ceph** (distribué, HA).
- **RAID matériel sous ZFS** = déconseillé : donner les disques en **HBA/pass-through** à ZFS.
- Snapshot de VM (**CP8-10**) ≠ sauvegarde externe (**CP8-05**).

---

## Sources (doc officielle)

- [Proxmox VE — Storage (pvesm)](https://pve.proxmox.com/pve-docs/chapter-pvesm.html) — consulté le 24/07/2026
- [Proxmox VE — Storage: LVM / LVM-thin](https://pve.proxmox.com/wiki/Storage:_LVM) — consulté le 24/07/2026
- [Proxmox VE — Deploy Hyper-Converged Ceph](https://pve.proxmox.com/wiki/Deploy_Hyper-Converged_Ceph_Cluster) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (`pvesm`) · [x] version datée (PVE 9.2) · [x] rien d'obsolète (volume-chains, HBA pour ZFS) · [x] procédure **à tester en lab** · [x] conforme doc Proxmox · [x] vérification présente (`pvesm status`) · [x] sécurité (réseau dédié, ≠backup) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
