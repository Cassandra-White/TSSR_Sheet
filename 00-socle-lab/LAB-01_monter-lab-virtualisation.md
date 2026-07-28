# LAB-01 — Monter le lab de virtualisation (Proxmox VE 9 ou VMware Workstation)

**Objectif** : monter l'**environnement de lab** qui servira à tester **tous** les tutoriels TSSR (VM Windows/Linux, pare-feu, réseau segmenté).

**Rattachement REAC** : Socle du lab (préparation de l'environnement de test) — support de l'ensemble des CP.

**Durée** : ~40 min · **Niveau** : intermédiaire.

---

## Prérequis

- Une machine avec **virtualisation matérielle** (VT-x/AMD-V), **16–32 Go de RAM** conseillés, un **SSD**.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Hyperviseur (recommandé) | **Proxmox VE 9.2** (type 1, gratuit) | 24/07/2026 |
| Alternative | **VMware Workstation Pro 26H1** (type 2, gratuit) | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> Le **lab** est un environnement **isolé** pour s'entraîner sans risque. **Proxmox VE** (**CP5-01**, type 1, gratuit) sur une machine dédiée est le choix recommandé ; **VMware Workstation** (**CP5-12**, type 2, gratuit) convient sur un simple poste. *(La version gratuite d'ESXi n'existe plus.)*

---

## Procédure — Méthode

### 1. Installer l'hyperviseur

- **Proxmox VE 9** (**CP5-01**) sur la machine dédiée, **ou** **VMware Workstation** (**CP5-12**) sur un poste. *(En pédagogie, on peut virtualiser Proxmox lui-même : activer la **virtualisation imbriquée**.)*

### 2. Préparer le réseau du lab

- Créer des **ponts/commutateurs virtuels** + **VLAN** (**CP5-05**) selon le **plan d'adressage** (**LAB-02**).
- Ajouter un **pare-feu pfSense** (**CP7-01**) au cœur pour **router entre VLAN** et vers Internet.

### 3. Créer le « parc » de VM

| VM | Rôle | Renvoi |
|---|---|---|
| **Windows Server 2025** | Contrôleur de domaine (AD/DNS/DHCP) | **CP2** |
| **Windows 11** | Poste client (jonction au domaine) | **CP2-07** |
| **Debian 13** | Serveur Linux (services) | **CP3** |
| **pfSense** | Pare-feu / routeur inter-VLAN | **CP7** |
| *(option)* **PBS**, **GLPI** | Sauvegarde, gestion de parc | **CP8-05**, **CP1-01** |

### 4. Industrialiser

- Créer des **modèles** de VM réutilisables (**LAB-03**) pour déployer vite et de façon homogène.

---

## Vérification (comment savoir que le lab est prêt)

- L'hyperviseur est **opérationnel** ; les **VM de base** démarrent.
- Le **réseau segmenté** fonctionne : un poste du VLAN Utilisateurs **ping** un serveur via le pare-feu (routage inter-VLAN — **CP4-06**).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| VM imbriquées KO | **Nested** désactivé | Activer la virtualisation imbriquée |
| Manque de RAM | Trop de VM à la fois | Démarrer les VM **au besoin** ; Dynamic Memory |
| Pas d'inter-VLAN | Routage/pare-feu absent | Ajouter **pfSense** + règles (**CP7-02**) |
| Conflit réseau | Bloc IP = réseau domestique | Choisir un **bloc distinct** (**LAB-02**) |

## Sécurité et bonnes pratiques

- **Isoler le lab** du réseau domestique/de production (VLAN/NAT dédié).
- **Snapshots** avant les manipulations à risque (**CP8-10**).
- Documenter la topologie (**CP4-11**) et le plan d'adressage (**LAB-02**).

## ⚠️ À ne pas confondre / obsolète

- **Type 1** (Proxmox, bare-metal) ≠ **type 2** (Workstation, sur un OS).
- **ESXi gratuit supprimé** → **Proxmox** pour un type 1 sans licence.
- **Lab** (isolé, jetable) ≠ **production** (à ne jamais confondre les réseaux).

---

## Sources (doc officielle)

- [Proxmox VE — Installation](https://pve.proxmox.com/pve-docs/chapter-pve-installation.html) — consulté le 24/07/2026
- [Broadcom — VMware Workstation Pro (gratuit)](https://techdocs.broadcom.com/us/en/vmware-cis/desktop-hypervisors/workstation-pro/26H1.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] Méthode + renvois CP5-01/12 · [x] versions datées (PVE 9.2 / WS 26H1) · [x] rien d'obsolète (ESXi gratuit supprimé) · [x] procédure **à tester en lab** (c'est le lab) · [x] conforme doc Proxmox/Broadcom · [x] vérification présente (ping inter-VLAN) · [x] sécurité (isolation, snapshots) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
