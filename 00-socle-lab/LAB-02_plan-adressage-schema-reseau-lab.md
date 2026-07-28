# LAB-02 — Définir le plan d'adressage et le schéma réseau du lab (VLAN, sous-réseaux)

**Objectif** : définir un **plan d'adressage IP** cohérent et un **schéma réseau segmenté** (VLAN/sous-réseaux) pour le lab, réutilisé par tous les tutoriels.

**Rattachement REAC** : Socle du lab — support des CP réseau (CP4) et sécurité (CP7).

**Durée** : ~20 min · **Niveau** : intermédiaire · **Type** : Méthode.

---

## Prérequis

- Le lab de virtualisation en place (**LAB-01**) et les notions de **VLSM** (**CP4-01**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Plan d'adressage | **calculé et vérifié en bac à sable** (Python `ipaddress`) | 24/07/2026 |
| Concepts | Méthode | 24/07/2026 |

> Un **plan d'adressage** clair évite les conflits et facilite la **segmentation** (sécurité). On choisit un **bloc privé** (RFC 1918) distinct du réseau domestique, puis on **découpe par usage/VLAN**.

---

## Procédure — Méthode

### 1. Choisir le bloc et la convention

- Bloc du lab : **`10.10.0.0/16`**, découpé en **/24 par VLAN**.
- Convention : **`.1` = passerelle** (le pare-feu pfSense) ; réservations statiques en bas, **plage DHCP** ensuite.

### 2. Plan d'adressage (calculé)

| VLAN | Segment | Réseau | Passerelle | Plage utilisable |
|---|---|---|---|---|
| **1** | Management / PVE | `10.10.1.0/24` | `10.10.1.1` | `.2 → .254` (254 hôtes) |
| **10** | Serveurs (AD/DNS/DHCP) | `10.10.10.0/24` | `10.10.10.1` | `.2 → .254` |
| **20** | Utilisateurs (postes) | `10.10.20.0/24` | `10.10.20.1` | `.2 → .254` |
| **30** | DMZ (web / reverse proxy) | `10.10.30.0/24` | `10.10.30.1` | `.2 → .254` |
| **99** | Admin/infra (iDRAC, switch) | `10.10.99.0/24` | `10.10.99.1` | `.2 → .254` |

> **Vérifié en bac à sable** : les 5 segments sont **cohérents** et **sans chevauchement** (contrôle `overlaps()` → aucun conflit).

### 3. Schéma réseau (logique)

- **pfSense** au cœur : une patte par VLAN (passerelle `.1`), **routage inter-VLAN** (**CP4-06**) + **filtrage** (**CP7-02**) + sortie **Internet** (NAT — **CP7-03**).
- **Commutateur VLAN-aware** (**CP5-05**) reliant l'hôte et les VM ; chaque VM **taguée** sur son VLAN.
- **DHCP** par VLAN (serveur AD ou pfSense — **CP2-13**).

---

## Vérification (comment savoir que c'est bon)

- **Aucun chevauchement** de sous-réseaux (vérifié).
- Chaque VLAN a une **passerelle** unique ; la segmentation correspond aux usages (**CP4-04/CP7-04**).
- Depuis un VLAN, on atteint (ou non) un autre VLAN **selon les règles** du pare-feu.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Conflit d'adresses | Bloc = réseau **domestique** | Choisir un bloc **distinct** (ex. `10.10.0.0/16`) |
| Pas d'inter-VLAN | Routage/filtrage absent | Router + autoriser sur **pfSense** (**CP4-06/CP7-02**) |
| Double DHCP | Deux serveurs sur un VLAN | **Un seul** service DHCP par segment |
| Plan illisible | Non documenté | Tenir la **table** + le **schéma** à jour (**CP4-11**) |

## Sécurité et bonnes pratiques

- **Segmenter = sécuriser** : isoler Utilisateurs / Serveurs / **DMZ** / **Management** (**CP7-04/CP7-12**).
- **Réseau de management séparé** (VLAN 1/99) et filtré.
- **Documenter** (table + schéma) : base de tout diagnostic (**CP4-07/CP4-11**).

## ⚠️ À ne pas confondre / obsolète

- **Réseau plat** (tout dans un /24) → **segmentation VLAN** (sécurité + lisibilité).
- **Adresse réseau**/**broadcast** (non attribuables) ≠ **hôtes** utilisables.
- **VLAN** (segmentation L2) + **sous-réseau** (L3) : les **aligner** (1 VLAN = 1 sous-réseau).

---

## Sources (doc officielle / référence)

- [IETF — RFC 1918 (adresses privées)](https://www.rfc-editor.org/rfc/rfc1918) — consulté le 24/07/2026
- [Python — module `ipaddress`](https://docs.python.org/3/library/ipaddress.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] Type Méthode (plan + schéma) · [x] daté 24/07/2026 · [x] rien d'obsolète (segmentation VLAN) · [x] **plan calculé/vérifié en bac à sable** (aucun chevauchement) · [x] cohérent RFC 1918 · [x] vérification présente · [x] sécurité (segmentation, management isolé) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
