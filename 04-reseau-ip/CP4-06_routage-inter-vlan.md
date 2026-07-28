# CP4-06 — Configurer le routage inter-VLAN (router-on-a-stick / switch L3)

**Objectif** : permettre la communication entre VLAN par du routage, soit avec un routeur (router-on-a-stick), soit avec un commutateur de niveau 3 (SVI).

**Rattachement REAC** : CP4 « Exploiter un réseau IP » — savoir-faire : router le trafic entre sous-réseaux/VLAN.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- VLAN créés (**CP4-04**) et trunk opérationnel (**CP4-05**), plan d'adressage (**CP4-01**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Équipement | syntaxe **Cisco IOS** (référence) | 24/07/2026 |
| Passerelles inter-VLAN (1 par VLAN) | **validées en namespace Linux** | 24/07/2026 |

> Chaque VLAN a **sa passerelle** (adresse de la sous-interface ou du SVI) ; les hôtes l'utilisent comme *default gateway*.

---

## Procédure — CLI

### Méthode A — Router-on-a-stick (routeur + trunk, sous-interfaces)

```text
interface gigabitEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
interface gigabitEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

> Côté switch, le port relié au routeur doit être en **trunk** (**CP4-05**).

### Méthode B — Switch de niveau 3 (SVI) — recommandé en production

```text
ip routing
interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
interface vlan 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
```

Les postes du VLAN 10 utilisent `192.168.10.1` comme passerelle, ceux du VLAN 20 `192.168.20.1`.

---

## Vérification (comment savoir que ça marche)

```text
show ip route                 ! les réseaux 192.168.10.0/24 et .20.0/24 sont "C" (connected)
show ip interface brief
```

Un poste du VLAN 10 doit pouvoir **pinger** un poste du VLAN 20.

> Preuve du principe (test bac à sable) : les deux passerelles `192.168.10.1/24` (VLAN 10) et `192.168.20.1/24` (VLAN 20) ont été créées et activées sur un même lien taggé.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Aucun routage entre VLAN | `ip routing` absent (switch L3) | Activer `ip routing` |
| Un VLAN ne route pas | Passerelle des hôtes erronée | Configurer la bonne default gateway (IP du SVI) |
| Sous-interface inactive | `encapsulation dot1Q <id>` manquante | Ajouter l'encapsulation sur la sous-interface |
| Trafic bloqué | ACL trop restrictive | Vérifier/ajuster les ACL (**CP7-08**) |

## Sécurité et bonnes pratiques

- **Filtrer** le trafic inter-VLAN avec des **ACL** (**CP7-08**) : tout n'a pas à communiquer.
- Isoler les VLAN sensibles (serveurs, gestion) derrière un **pare-feu** si nécessaire.
- Documenter les passerelles de chaque VLAN.

## ⚠️ À ne pas confondre / obsolète

- **Router-on-a-stick** : simple mais le lien unique est un **goulot d'étranglement** → en production, préférer un **switch L3 (SVI)**.
- Ne pas oublier **`ip routing`** sur le switch L3 (sinon les SVI ne routent pas).
- La **passerelle** d'un hôte = l'IP du **SVI**/sous-interface de son VLAN, pas une autre.

---

## Sources (doc officielle)

- [Cisco — How To Configure InterVLAN Routing on Layer 3 Switches](https://www.cisco.com/c/en/us/support/docs/lan-switching/inter-vlan-routing/41860-howto-L3-intervlanrouting.html) — consulté le 24/07/2026
- [IEEE 802.1Q — VLAN](https://www.ieee802.org/1/pages/802.1Q.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI (2 méthodes) · [x] réf datée · [x] rien d'obsolète (SVI recommandé) · [x] **passerelles validées en namespace** / config à tester en lab · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
