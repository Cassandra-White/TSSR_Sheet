# CP4-15 — Configurer et protéger le Spanning Tree (STP) + port security

**Objectif** : éviter les boucles de commutation (STP/RSTP) et sécuriser les ports d'accès (BPDU Guard, PortFast, port security).

**Rattachement REAC** : CP4 « Exploiter un réseau IP » — savoir-faire : fiabiliser et sécuriser un réseau commuté.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un ou plusieurs switches managés avec des liens redondants (**CP4-03/05**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Commutateur managé | syntaxe **Cisco IOS** (référence) | 24/07/2026 |
| Config | **à tester en lab** (matériel) | 24/07/2026 |

> **STP** empêche les **boucles** (tempêtes de broadcast) en bloquant les liens redondants. **RSTP (802.1w)** converge en secondes ; le STP classique (802.1D) en 30-50 s.

---

## Procédure — CLI (Cisco IOS)

### Spanning Tree (RSTP + racine)

```text
enable
configure terminal
 spanning-tree mode rapid-pvst          ! RSTP (recommandé)
 spanning-tree vlan 10,20,99 root primary   ! définir le root bridge au cœur
```

### Protéger les ports d'accès (terminaux)

```text
 interface range fastEthernet 0/1 - 24
  switchport mode access
  spanning-tree portfast                ! passage direct en forwarding (ports terminaux)
  spanning-tree bpduguard enable        ! coupe le port s'il reçoit un BPDU (anti switch pirate)
 end
 write memory
```

### Port security (limiter les MAC par port)

```text
interface fastEthernet 0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security mac-address sticky
```

---

## Vérification (comment savoir que ça marche)

```text
show spanning-tree              ! root bridge, rôle/état des ports (Forwarding/Blocking)
show spanning-tree summary
show port-security              ! ports protégés, violations, MAC apprises
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Tempête de broadcast / réseau figé | Boucle non bloquée par STP | Vérifier `spanning-tree` actif et le root bridge |
| Port en **err-disabled** | BPDU Guard / violation port-security déclenché | Corriger la cause ; `shutdown`/`no shutdown` ou `errdisable recovery` |
| PortFast provoque une boucle | PortFast mis vers un **autre switch** | Réserver PortFast aux **ports terminaux** |
| MAC non apprise | Port-security saturé | Ajuster `maximum` / nettoyer les MAC sticky |

## Sécurité et bonnes pratiques

- **BPDU Guard** sur **tous** les ports d'accès (PortFast) : bloque un switch pirate branché par un utilisateur.
- **Port security** (limite de MAC) contre le **MAC flooding** et les branchements non autorisés.
- **Root Guard** pour protéger le rôle de **root bridge** ; fixer le root au cœur du réseau.

## ⚠️ À ne pas confondre / obsolète

- **STP 802.1D** (lent) → **RSTP 802.1w** (rapide, recommandé).
- **PortFast** est réservé aux **ports terminaux** : jamais vers un autre switch (risque de boucle).
- **BPDU Guard** (désactive le port sur BPDU) ≠ **BPDU Filter** (ignore les BPDU) : ne pas les confondre.

---

## Sources (doc officielle)

- [Cisco — Spanning Tree Protocol (STP/RSTP)](https://www.cisco.com/c/en/us/tech/lan-switching/spanning-tree-protocol/index.html) — consulté le 24/07/2026
- [Cisco — Port Security](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9300/software/release/17-x/configuration_guide/sec/b_17x_sec_9300_cg/configuring_port_security.html) — consulté le 24/07/2026
- [IEEE 802.1 — Rapid Spanning Tree (802.1w)](https://www.ieee802.org/1/pages/802.1w.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI (IOS) · [x] réf datée · [x] rien d'obsolète (RSTP vs STP) · [x] config à tester en lab · [x] conforme doc · [x] vérification présente · [x] sécurité (BPDU Guard/port security) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
