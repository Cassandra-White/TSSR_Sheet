# CP5-08 — Mettre en place un cluster Proxmox (bases de la haute dispo)

**Objectif** : créer un **cluster Proxmox**, comprendre le **quorum** et activer la **haute disponibilité (HA)** avec **migration à chaud** (live migration).

**Rattachement REAC** : CP5 « Maintenir des serveurs dans une infrastructure virtualisée » — savoir-faire : assurer la continuité de service.

**Durée** : ~35 min · **Niveau** : avancé.

---

## Prérequis

- **≥ 3 nœuds** Proxmox VE 9 (**CP5-01**) — nombre **impair** conseillé.
- Un **réseau dédié** pour Corosync (**latence < 2 ms**) et un **stockage partagé** (Ceph/NFS/iSCSI).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Cluster | **Proxmox VE 9.2** (Corosync) | 24/07/2026 |
| Stockage distribué | **Ceph Squid 19.2** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **Cluster** = plusieurs nœuds administrés ensemble (config partagée). **Corosync** assure la communication (sensible à la **latence** → réseau dédié). Le **quorum** (majorité) évite le *split-brain* → **nombre impair, 3 nœuds minimum** (ou 2 nœuds **+ QDevice** arbitre). La **HA** redémarre une VM sur un autre nœud en cas de panne ; la **migration à chaud** exige un **stockage partagé**.

---

## Procédure — GUI + CLI

### Créer / joindre le cluster

```bash
# Sur le nœud 1 : créer le cluster
pvecm create mon-cluster

# Sur les autres nœuds : rejoindre (IP du nœud 1)
pvecm add 192.168.0.1

pvecm status        # vérifier le quorum (Quorate: Yes)
```

*(GUI : Datacenter ▸ Cluster ▸ Create Cluster / Join Cluster avec le « Join Information ».)*

### Stockage partagé (Ceph)

- **Datacenter ▸ Ceph ▸ Install** (Squid 19.2) → créer les **OSD/pools** ; ou ajouter un **NFS/iSCSI** partagé (**STO-10/11**).

### Haute disponibilité + migration

- **Datacenter ▸ HA** : ajouter les VM critiques à un **groupe HA**.
- **VM ▸ Migrate** : migration **à chaud** vers un autre nœud (stockage partagé requis).

---

## Vérification (comment savoir que ça marche)

- `pvecm status` → **Quorate: Yes**, tous les nœuds visibles.
- Une **migration à chaud** se fait **sans coupure** ; après **panne simulée** d'un nœud, la **HA** redémarre la VM ailleurs.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Perte de quorum | Nombre **pair** / nœud tombé | Ajouter un **QDevice** (arbitre) |
| Cluster instable | **Latence** Corosync | Réseau **dédié** + lien redondant |
| Migration impossible | Pas de **stockage partagé** | Ceph/NFS/iSCSI accessible par tous |
| HA ne bascule pas | **Fencing** / config HA | Vérifier le stockage partagé et les groupes HA |

## Sécurité et bonnes pratiques

- **Réseau Corosync dédié et redondant** (2 liens) ; le séparer du trafic VM/stockage.
- **Stockage partagé résilient** (Ceph min 3 nœuds) ; **tester le failover** régulièrement.
- Documenter le **plan de continuité** (**CP8-01** — PCA/PRA).

## ⚠️ À ne pas confondre / obsolète

- **Cluster** (gestion commune) ≠ **HA** (redémarrage auto) ≠ **live migration** (déplacement à chaud).
- **2 nœuds** sans QDevice = **pas de quorum** fiable.
- **Migration à chaud** nécessite un **stockage partagé** (pas de disque local).

---

## Sources (doc officielle)

- [Proxmox VE — Cluster Manager (pvecm)](https://pve.proxmox.com/pve-docs/chapter-pvecm.html) — consulté le 24/07/2026
- [Proxmox VE — High Availability](https://pve.proxmox.com/pve-docs/chapter-ha-manager.html) — consulté le 24/07/2026
- [Proxmox VE — Deploy Hyper-Converged Ceph](https://pve.proxmox.com/wiki/Deploy_Hyper-Converged_Ceph_Cluster) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI + CLI (`pvecm`) · [x] versions datées (PVE 9.2 / Ceph Squid) · [x] rien d'obsolète (QDevice, quorum impair) · [x] procédure **à tester en lab** · [x] conforme doc Proxmox · [x] vérification présente (`pvecm status`/failover) · [x] sécurité (réseau dédié, failover testé) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
