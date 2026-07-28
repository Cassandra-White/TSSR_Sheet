# CP7-09 — Configurer le routage statique et dynamique (OSPF)

**Objectif** : router le trafic entre réseaux avec des routes statiques, puis avec le protocole dynamique OSPF.

**Rattachement REAC** : CP7 « Maintenir et sécuriser les accès Internet et les interconnexions » — savoir-faire : mettre en œuvre le routage.

**Durée** : ~25 min · **Niveau** : avancé.

---

## Prérequis

- Des routeurs / switches L3 (**CP4-06**) reliant plusieurs sous-réseaux.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Équipement | syntaxe **Cisco IOS** (référence) | 24/07/2026 |
| Configuration | **à tester en lab** | 24/07/2026 |

> **Statique** : routes définies à la main (petit réseau, simple, mais aucune adaptation). **Dynamique (OSPF)** : les routeurs s'échangent les routes et **convergent** automatiquement.

---

## Procédure — CLI (Cisco IOS)

### Routage statique

```text
configure terminal
 ip route 192.168.20.0 255.255.255.0 10.0.0.2        ! atteindre 192.168.20.0/24 via 10.0.0.2
 ip route 0.0.0.0 0.0.0.0 203.0.113.1                ! route par défaut (vers Internet)
```

### Routage dynamique — OSPF (aire fédératrice 0)

```text
 router ospf 1
  network 10.0.0.0      0.0.0.255 area 0
  network 192.168.10.0  0.0.0.255 area 0
  passive-interface gigabitEthernet 0/1              ! pas d'OSPF vers les LAN utilisateurs
 exit
end
write memory
```

> Sur **pfSense / Linux**, l'équivalent OSPF/BGP est le paquet **FRR**.

---

## Vérification (comment savoir que ça marche)

```text
show ip route                     ! S = statique, O = OSPF, S* = défaut
show ip ospf neighbor             ! adjacences en état "FULL"
```

Un ping traverse bien les sous-réseaux distants.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Voisins OSPF absents | Aire/sous-réseau/hello incohérents, **MTU** | Aligner aire, réseaux et timers |
| Route statique inactive | Next-hop injoignable | Vérifier l'accessibilité du saut suivant |
| Boucle / route sous-optimale | Métriques/coûts | Ajuster le coût OSPF |
| OSPF sur un LAN utilisateur | `passive-interface` oublié | Passiver les interfaces terminales |

## Sécurité et bonnes pratiques

- **Authentifier OSPF** (`message-digest`) pour empêcher l'injection de fausses routes.
- **`passive-interface`** sur toutes les interfaces sans voisin routeur.
- Contrôler les **redistributions** entre protocoles.

## ⚠️ À ne pas confondre / obsolète

- **Statique** (manuel, fixe) vs **dynamique** (auto, adaptatif) : OSPF pour les réseaux qui évoluent.
- **OSPF** (IGP *link-state*, moderne) ≠ **RIP** (*distance-vector*, ancien, à éviter).
- **Aire 0** = **backbone** OSPF : toutes les autres aires s'y raccordent.

---

## Sources (doc officielle)

- [Cisco — OSPF Configuration Guide](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/iproute_ospf/configuration/15-mt/iro-15-mt-book.html) — consulté le 24/07/2026
- [Cisco — Configuring a Gateway of Last Resort (routes statiques)](https://www.cisco.com/c/en/us/support/docs/ip/routing-information-protocol-rip/16448-default.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI (IOS) · [x] réf datée · [x] rien d'obsolète (OSPF vs RIP) · [x] config à tester en lab · [x] conforme doc Cisco · [x] vérification présente · [x] sécurité (auth OSPF) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
