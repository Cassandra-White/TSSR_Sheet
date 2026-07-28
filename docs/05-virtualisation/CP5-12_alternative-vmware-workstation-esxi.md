# CP5-12 — (Alternative VMware) Créer et gérer des VM sous VMware Workstation/ESXi

**Objectif** : créer et gérer des VM avec **VMware Workstation Pro** (ou **ESXi**), en connaissant le **statut actuel** (Broadcom) et le positionnement face à Proxmox.

**Rattachement REAC** : CP5 « Maintenir des serveurs dans une infrastructure virtualisée » — savoir-faire : exploiter un autre hyperviseur.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un poste avec **virtualisation matérielle** (VT-x/AMD-V) pour **Workstation**, ou un serveur pour **ESXi**.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Type 2 (poste) | **VMware Workstation Pro 26H1** (**gratuit**) | 24/07/2026 |
| Type 1 (serveur) | **ESXi / vSphere** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> ⚠️ **Actualité (Broadcom) 2026** : **VMware Workstation Pro 26H1 est désormais GRATUIT** (usage commercial/éducatif/personnel, **sans clé**). En revanche, la **version gratuite d'ESXi a été supprimée** → pour du **type 1 gratuit**, **Proxmox VE** (**CP5-01**) est l'alternative de référence.

---

## Procédure — GUI (VMware Workstation Pro)

1. **Create a New Virtual Machine** → choisir une **ISO** (ou installer l'OS plus tard).
2. Sélectionner le **type d'OS**, allouer **CPU/RAM/disque**.
3. **Réseau** : **NAT** (partage l'IP de l'hôte), **Bridged** (sur le LAN), **Host-only** (isolé).
4. Installer les **VMware Tools** dans l'invité (équivalent du guest agent : intégration, pilotes).
5. Utiliser **snapshots** et **clones** pour les tests.

## ESXi (type 1, rappel)

- **ESXi** s'installe **bare-metal** et s'administre via **vSphere Client** ; désormais **sous licence** (plus de version gratuite).

## Comparaison rapide

| Solution | Type | Coût | Usage |
|---|---|---|---|
| **Workstation Pro** | 2 (sur un OS) | **Gratuit** | Poste, **lab**, tests |
| **ESXi/vSphere** | 1 (bare-metal) | **Licence** | Production entreprise |
| **Proxmox VE** | 1 (bare-metal) | **Gratuit** (OSS) | Production / lab |

---

## Vérification (comment savoir que ça marche)

- La VM **démarre** et l'OS s'installe ; les **VMware Tools** fonctionnent (résolution, presse-papier, arrêt propre).
- Un **snapshot** puis un **rollback** ramènent l'état attendu.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| VM ne démarre pas | VT-x/AMD-V désactivé | Activer la virtualisation (BIOS/UEFI) |
| Virtualisation imbriquée KO | *Nested* non activé | Activer *Virtualize Intel VT-x/EPT* |
| Pas de réseau | Mode réseau inadapté | Choisir **NAT/Bridged/Host-only** selon le besoin |
| Intégration limitée | **VMware Tools** absents | Installer les Tools dans l'invité |

## Sécurité et bonnes pratiques

- **Workstation** = idéal **lab/poste** ; pour la **production**, un **type 1** (ESXi licencié **ou** Proxmox gratuit).
- **Snapshots** avant test, mais **≠ sauvegarde** (**CP8-10**).
- **Isoler** les VM de test (réseau host-only/NAT).

## ⚠️ À ne pas confondre / obsolète

- **Type 2** (Workstation, sur un OS) ≠ **type 1** (ESXi/Proxmox, bare-metal).
- **ESXi gratuit = supprimé** (Broadcom) → **Proxmox** pour du type 1 sans licence.
- **VMware Tools** (VMware) ≡ **QEMU Guest Agent** (Proxmox) : intégration invité.

---

## Sources (doc officielle)

- [Broadcom — VMware Workstation Pro (TechDocs 26H1)](https://techdocs.broadcom.com/us/en/vmware-cis/desktop-hypervisors/workstation-pro/26H1.html) — consulté le 24/07/2026
- [Broadcom — Download VMware Desktop Hypervisor (gratuit)](https://knowledge.broadcom.com/external/article/368667/) — consulté le 24/07/2026
- [Broadcom — vSphere / ESXi](https://www.vmware.com/products/vsphere.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI (Workstation) · [x] versions datées (26H1) · [x] **statut Broadcom signalé** (Workstation gratuit, ESXi gratuit supprimé) · [x] procédure **à tester en lab** · [x] conforme doc Broadcom · [x] vérification présente (Tools/snapshot) · [x] sécurité (lab vs prod, snapshot≠backup) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
