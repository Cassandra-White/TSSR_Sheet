# CP4-04 — Créer et affecter des VLAN sur un switch

**Objectif** : créer des VLAN sur un commutateur managé et affecter des ports en mode **accès** pour segmenter le réseau.

**Rattachement REAC** : CP4 « Exploiter un réseau IP » — savoir-faire : segmenter un réseau commuté en VLAN.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un switch managé accessible (**CP4-03**), plan d'adressage/VLAN défini (**CP4-01**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Commutateur managé | syntaxe **Cisco IOS** (référence) | 24/07/2026 |
| Principe 802.1Q (1 sous-réseau/VLAN) | **validé en namespace Linux** (voir CP4-05) | 24/07/2026 |

> Un **VLAN** = un domaine de diffusion (broadcast) séparé, en général **un sous-réseau IP par VLAN**.

---

## Procédure — GUI (interface web)

1. Menu **VLAN Management** → **Create VLAN** : saisir l'**ID** (ex. 10) et le **nom** (ex. POSTES).
2. **Port to VLAN** : affecter les ports concernés en **Untagged/Access** au bon VLAN.
3. Enregistrer la configuration.

## Procédure — CLI (Cisco IOS)

```text
enable
configure terminal
 vlan 10
  name POSTES
 vlan 20
  name SERVEURS
 exit
 interface range fastEthernet 0/1 - 12
  switchport mode access
  switchport access vlan 10
 interface range fastEthernet 0/13 - 24
  switchport mode access
  switchport access vlan 20
end
write memory
```

---

## Vérification (comment savoir que ça marche)

```text
show vlan brief              ! liste des VLAN et des ports affectés
show interfaces status       ! statut et VLAN de chaque port
```

Deux postes de VLAN différents ne doivent **pas** communiquer directement (il faut du routage — **CP4-06**).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Port dans le mauvais VLAN | Mauvaise affectation | `switchport access vlan <id>` sur le bon port |
| VLAN absent après affectation | VLAN non créé | Créer le VLAN **avant** de l'affecter |
| Le port ne « prend » pas le VLAN | Port en mode trunk | `switchport mode access` |

## Sécurité et bonnes pratiques

- **Un VLAN par usage** (postes, serveurs, gestion, voix, DMZ).
- Placer les **ports inutilisés** dans un VLAN « poubelle » non routé et les **désactiver** (`shutdown`).
- **Ne pas utiliser le VLAN 1** (par défaut) pour la production.

## ⚠️ À ne pas confondre / obsolète

- **Access** (port dans **un** VLAN, trafic **untagged**) ≠ **Trunk** (plusieurs VLAN **taggés** — **CP4-05**).
- Le VLAN doit **exister** avant d'y affecter un port.
- **VTP** (propagation automatique des VLAN) est pratique mais risqué : à manier avec prudence (un switch mal configuré peut effacer la base VLAN).

---

## Sources (doc officielle)

- [IEEE 802.1Q — Bridges and Bridged Networks (VLAN)](https://www.ieee802.org/1/pages/802.1Q.html) — consulté le 24/07/2026
- [Cisco — Catalyst : Basic Switch Configuration (best practices)](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst3850/software/release/16-1/best_practices_guide/BP_Cat3850/BP_basic_config.pdf) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI (web) puis CLI (IOS) · [x] réf datée · [x] rien d'obsolète (access vs trunk) · [x] principe validé en namespace / config à tester en lab · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
