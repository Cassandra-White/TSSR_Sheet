# CP5-05 — Configurer le réseau virtuel (ponts, VLAN dans Proxmox)

**Objectif** : configurer le réseau virtuel de Proxmox — **pont Linux** (vmbr0), **pont VLAN-aware** et **tag VLAN** par VM — pour segmenter le trafic.

**Rattachement REAC** : CP5 « Maintenir des serveurs dans une infrastructure virtualisée » — savoir-faire : gérer le réseau des VM.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un hôte **Proxmox VE 9** (**CP5-01**), un **switch** dont le port uplink est configurable en **trunk 802.1Q** (**CP4-05**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Hyperviseur | **Proxmox VE 9.2** (ifupdown2) | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> Le **pont Linux** (`vmbr0`) est un **switch virtuel** reliant les VM à la carte physique. Un pont **VLAN-aware** transporte **plusieurs VLAN** sur un seul uplink : on **tague chaque VM** au niveau de sa carte (le port du switch doit être en **trunk**).

---

## Procédure — GUI

1. **Nœud ▸ System ▸ Network** : `vmbr0` a pour **Bridge port** la carte physique (ex. `eno1`) et porte l'**IP de management**.
2. Cocher **VLAN aware** sur `vmbr0`.
3. Sur la **carte réseau de la VM** (Hardware ▸ Network Device) : renseigner le **VLAN Tag** (ex. `10`).
4. **Apply Configuration** (appliqué par **ifupdown2**, qui ne change que le nécessaire — sûr en SSH).

## Procédure — CLI (`/etc/network/interfaces`)

```ini
auto vmbr0
iface vmbr0 inet static
    address 192.168.0.2/24
    gateway 192.168.0.1
    bridge-ports eno1
    bridge-vlan-aware yes
    bridge-vids 2-4094          # VLAN autorisés à traverser le pont
```

```bash
ifreload -a                     # appliquer (ifupdown2)
```

> Côté **switch** : le port relié à l'hôte doit être en **trunk** (802.1Q) laissant passer les VLAN utilisés (**CP4-05**). Pour des topologies étendues, Proxmox VE 9 propose la couche **SDN**.

---

## Vérification (comment savoir que ça marche)

- Une VM avec **VLAN Tag 10** obtient une IP du **sous-réseau du VLAN 10** (DHCP de ce VLAN — **CP2-13**).
- `ifreload -a` s'applique **sans couper** la connexion de management (ifupdown2).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| VM sans réseau taggé | Pont **non VLAN-aware** | Cocher **VLAN aware** / `bridge-vlan-aware yes` |
| Un seul VLAN passe | Port switch en **access** | Passer le port uplink en **trunk** (**CP4-05**) |
| Perte de management | Mauvaise édition à chaud | ifupdown2 (`ifreload -a`) plutôt que ifdown/ifup |
| Mauvais sous-réseau | Tag VLAN erroné | Corriger le **VLAN Tag** de la carte VM |

## Sécurité et bonnes pratiques

- **Segmenter** par VLAN (utilisateurs, serveurs, **DMZ**, management — **CP4-04/CP7-04**).
- **Isoler le réseau de management** de Proxmox du trafic des VM.
- **Filtrer** entre VLAN au niveau du pare-feu/routeur (**CP4-06/CP7-02**).

## ⚠️ À ne pas confondre / obsolète

- **Un pont par VLAN** (lourd) → **pont VLAN-aware** unique (tag par VM).
- **Access** (1 VLAN) ≠ **Trunk** (plusieurs VLAN) côté switch.
- **ifupdown2** (applique le delta, sûr) vs ancien **ifup/ifdown**.

---

## Sources (doc officielle)

- [Proxmox VE — Network Configuration](https://pve.proxmox.com/wiki/Network_Configuration) — consulté le 24/07/2026
- [Proxmox VE — SDN](https://pve.proxmox.com/pve-docs/chapter-pvesdn.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (`interfaces`/`ifreload`) · [x] version datée (PVE 9.2) · [x] rien d'obsolète (VLAN-aware, ifupdown2, SDN) · [x] procédure **à tester en lab** · [x] conforme doc Proxmox · [x] vérification présente · [x] sécurité (segmentation, management isolé) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
