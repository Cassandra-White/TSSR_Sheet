# CP7-08 — Configurer des ACL sur routeur/commutateur

**Objectif** : filtrer le trafic sur un routeur/commutateur avec des listes de contrôle d'accès (ACL).

**Rattachement REAC** : CP7 « Maintenir et sécuriser les accès Internet et les interconnexions » — savoir-faire : filtrer le trafic au niveau routeur/commutateur.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un routeur / switch L3 managé (**CP4-03/06**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Équipement | syntaxe **Cisco IOS** (référence) | 24/07/2026 |
| Configuration | **à tester en lab** | 24/07/2026 |

## Types d'ACL

| Type | Filtre sur | À placer… |
|---|---|---|
| **Standard** (1-99) | **source** uniquement | près de la **destination** |
| **Étendue** (100-199) | source, destination, protocole, **port** | près de la **source** |
| **Nommée** | idem (lisible) | recommandée |

> Le **masque générique (wildcard)** est l'**inverse** du masque : `0.0.0.255` = « les 24 premiers bits fixes ». Chaque ACL se termine par un **`deny` implicite**.

---

## Procédure — CLI (Cisco IOS)

```text
enable
configure terminal
 ip access-list extended FILTRE-LAN
  permit tcp 192.168.10.0 0.0.0.255 host 192.168.20.10 eq 443   ! LAN -> serveur web
  deny   ip  192.168.10.0 0.0.0.255 192.168.20.0 0.0.0.255      ! LAN -/-> réseau serveurs
  permit ip  any any                                            ! le reste passe
 exit
 interface gigabitEthernet 0/1
  ip access-group FILTRE-LAN in                                 ! sens ENTRANT sur l'interface
end
write memory
```

---

## Vérification (comment savoir que ça marche)

```text
show access-lists                 ! contenu + compteurs de correspondances
show ip interface gig0/1          ! l'ACL est bien appliquée (in/out)
```

Tester les flux : la connexion autorisée passe, la refusée est bloquée.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| **Tout** est bloqué | `deny` implicite en fin | Ajouter un `permit` adapté |
| L'ACL ne filtre pas le bon trafic | Mauvais **sens** (in/out) | Corriger `ip access-group … in/out` |
| Trop/pas assez large | ACL **standard** mal placée | Placer les standard **près de la destination** |
| Ne matche pas | **Wildcard** erroné | Recalculer (inverse du masque) |

## Sécurité et bonnes pratiques

- **Moindre autorisation** ; **commenter** les ACL (`remark`).
- **Journaliser** les refus sensibles (`deny … log`).
- Placer l'ACL **efficacement** (source pour l'étendue, destination pour la standard).

## ⚠️ À ne pas confondre / obsolète

- **Standard** (source seule) ≠ **étendue** (source + destination + port).
- **Wildcard mask** (`0` = bit exact, `255` = quelconque) ≠ masque de sous-réseau.
- Le **`deny` implicite** final : sans `permit`, **tout** est refusé.

---

## Sources (doc officielle)

- [Cisco — Configure and Filter IP Access Lists](https://www.cisco.com/c/en/us/support/docs/security/ios-firewall/23602-confaccesslists.html) — consulté le 24/07/2026
- [Cisco — IP Access List Overview](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/sec_data_acl/configuration/15-mt/sec-data-acl-15-mt-book/sec-access-list-ov.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI (IOS) · [x] réf datée · [x] rien d'obsolète (nommée, wildcard) · [x] config à tester en lab · [x] conforme doc Cisco · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
