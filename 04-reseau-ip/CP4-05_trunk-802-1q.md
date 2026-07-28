# CP4-05 — Configurer un lien trunk 802.1Q entre commutateurs

**Objectif** : transporter plusieurs VLAN sur un seul lien entre deux commutateurs grâce à l'étiquetage **802.1Q** (trunk).

**Rattachement REAC** : CP4 « Exploiter un réseau IP » — savoir-faire : interconnecter des commutateurs en préservant les VLAN.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Deux switches managés avec des VLAN créés (**CP4-04**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Commutateur managé | syntaxe **Cisco IOS** (référence) | 24/07/2026 |
| Étiquetage 802.1Q | **validé dans le bac à sable** (namespace réseau) | 24/07/2026 |

> **Trunk** = lien qui transporte **plusieurs VLAN taggés 802.1Q**. **Access** = lien d'**un seul** VLAN, non taggé. Le **VLAN natif** circule **non taggé** sur le trunk.

---

## Procédure — GUI (interface web)

1. Sélectionner le port d'interconnexion → mode **Trunk**.
2. Définir les **VLAN autorisés** (tagged) sur le trunk.
3. Définir le **VLAN natif** (untagged) — identique des deux côtés.

## Procédure — CLI (Cisco IOS) — à faire des DEUX côtés

```text
enable
configure terminal
 interface gigabitEthernet 0/1
  switchport trunk encapsulation dot1q     ! sur les modèles qui le demandent
  switchport mode trunk
  switchport trunk allowed vlan 10,20,99
  switchport trunk native vlan 99
  switchport nonegotiate                    ! désactive DTP (sécurité)
end
write memory
```

---

## Vérification (comment savoir que ça marche)

```text
show interfaces trunk                 ! mode trunk, VLAN autorisés, VLAN natif
show interfaces gigabitEthernet 0/1 switchport
```

> Preuve du mécanisme (test bac à sable) : deux sous-interfaces **802.1Q** (id **10** → 192.168.10.1/24, id **20** → 192.168.20.1/24) créées et adressées sur **un même lien** — c'est exactement ce qu'un trunk transporte.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Un VLAN ne passe pas le trunk | Absent de `allowed vlan` | Ajouter le VLAN à la liste autorisée |
| Avertissement « native VLAN mismatch » | VLAN natif différent des deux côtés | Aligner `native vlan` sur les deux switches |
| Lien ne monte pas en trunk | Mode/encapsulation incohérents | `switchport mode trunk` + `encapsulation dot1q` des deux côtés |

## Sécurité et bonnes pratiques

- **Limiter `allowed vlan`** au strict nécessaire (réduit la surface d'attaque).
- **Changer le VLAN natif** (ne pas laisser VLAN 1) et n'y placer **aucun** trafic utile → limite le **VLAN hopping**.
- **Désactiver DTP** (`switchport nonegotiate`) : ne pas négocier les trunks automatiquement.

## ⚠️ À ne pas confondre / obsolète

- **ISL** (encapsulation propriétaire Cisco) est **obsolète** → **802.1Q** (standard).
- **VLAN natif** = trafic **non taggé** sur le trunk : un mismatch est une faille (VLAN hopping).
- **DTP en auto** = risque : préférer un trunk **statique** + `nonegotiate`.

---

## Sources (doc officielle)

- [IEEE 802.1Q — VLAN Tagging](https://www.ieee802.org/1/pages/802.1Q.html) — consulté le 24/07/2026
- [Cisco — Inter-Switch Link and IEEE 802.1Q Trunking](https://www.cisco.com/c/en/us/support/docs/lan-switching/8021q/17056-741-4.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI (web) puis CLI (IOS) · [x] réf datée · [x] rien d'obsolète (802.1Q vs ISL, anti-DTP) · [x] **802.1Q validé en namespace** / config à tester en lab · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
