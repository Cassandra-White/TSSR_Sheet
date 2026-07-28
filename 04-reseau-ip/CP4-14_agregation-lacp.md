# CP4-14 — Configurer l'agrégation de liens (LACP / EtherChannel) sur switch

**Objectif** : regrouper plusieurs liens physiques entre deux équipements en un seul lien logique (bande passante cumulée + redondance) avec **LACP** (802.3ad / 802.1AX).

**Rattachement REAC** : CP4 « Exploiter un réseau IP » — savoir-faire : agréger des liens entre commutateurs.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Deux équipements reliés par **plusieurs liens identiques** (même vitesse/duplex/VLAN).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Commutateur managé | syntaxe **Cisco IOS** (référence) | 24/07/2026 |
| Agrégation **802.3ad (LACP)** | **validée en namespace Linux** (bond `lacp_active on`) | 24/07/2026 |

> **LACP** (standard) négocie dynamiquement l'agrégat. Modes : `active`/`passive`. **PAgP** est propriétaire Cisco ; `on` = statique (sans négociation, risqué).

---

## Procédure — CLI (Cisco IOS) — des DEUX côtés

```text
enable
configure terminal
 interface range gigabitEthernet 0/1 - 2
  channel-group 1 mode active          ! LACP (active des deux côtés, ou active/passive)
  exit
 interface port-channel 1
  switchport mode trunk                 ! ex. si le lien transporte des VLAN
  switchport trunk allowed vlan 10,20,99
end
write memory
```

> Côté **hôte** (serveur), l'équivalent est un *bond* Linux `mode 802.3ad` ou un *teaming* Windows (**CP2-22**) — même principe LACP.

---

## Vérification (comment savoir que ça marche)

```text
show etherchannel summary        ! l'agrégat "Po1" ; ports en (P) = bundled
show lacp neighbor               ! l'autre extrémité est vue en LACP
```

> Preuve du mécanisme (test bac à sable) : un *bond* `mode 802.3ad` avec `lacp_active on` a été créé côté Linux — c'est la moitié « hôte » d'un agrégat LACP.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| L'agrégat ne monte pas | Modes incohérents (`passive`/`passive`) | Au moins un côté en `active` |
| Un lien reste hors bundle | Vitesse/duplex/VLAN différents | Uniformiser les paramètres des liens |
| Instabilité | Un côté LACP, l'autre `on` (statique) | Même méthode des deux côtés |

## Sécurité et bonnes pratiques

- **Cohérence stricte** des deux côtés (méthode, VLAN, vitesse).
- L'agrégat apparaît comme **un seul lien logique** pour STP → **évite les boucles** tout en gardant la redondance.
- Documenter le port-channel (**CP4-11**).

## ⚠️ À ne pas confondre / obsolète

- **LACP** (standard **802.3ad/802.1AX**) ≠ **PAgP** (propriétaire Cisco). Préférer LACP.
- Mode **`on`** (statique) = pas de négociation → risque de boucle si l'autre côté n'est pas configuré : préférer **`active`**.
- **Agrégation (LACP)** ≠ **trunk (802.1Q)** : complémentaires (un port-channel peut être un trunk).

---

## Sources (doc officielle)

- [Cisco — Configuring EtherChannel / LACP](https://www.cisco.com/c/en/us/support/docs/lan-switching/etherchannel/12023-4.html) — consulté le 24/07/2026
- [IEEE 802.1AX — Link Aggregation](https://www.ieee802.org/1/pages/802.1ax.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI (IOS) · [x] réf datée · [x] rien d'obsolète (LACP vs PAgP/`on`) · [x] **802.3ad validé en namespace** / config switch à tester en lab · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
