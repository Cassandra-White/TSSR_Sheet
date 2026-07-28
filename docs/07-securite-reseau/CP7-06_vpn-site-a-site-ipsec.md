# CP7-06 — Configurer un VPN site-à-site (IPsec)

**Objectif** : relier deux sites distants par un tunnel IPsec chiffré, pour que leurs réseaux locaux communiquent en sécurité via Internet.

**Rattachement REAC** : CP7 « Maintenir et sécuriser les accès Internet et les interconnexions » — savoir-faire : interconnecter des sites de façon sécurisée.

**Durée** : ~30 min · **Niveau** : avancé.

---

## Prérequis

- **Deux** pfSense (site A et site B) avec IP publiques et des **sous-réseaux LAN différents** (ex. A = 192.168.10.0/24, B = 192.168.20.0/24).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| pfSense CE | 2.8.1 | 24/07/2026 |
| Configuration | **à tester en lab** | 24/07/2026 |

> IPsec = **Phase 1** (IKE : authentification + canal sécurisé) puis **Phase 2** (ESP : chiffre le trafic des sous-réseaux). Paramètres **identiques et miroir** des deux côtés.

---

## Procédure — GUI (à répéter sur les deux sites)

### Phase 1 (IKE)

**VPN → IPsec → Tunnels → Add P1** :

- **Key Exchange** : **IKEv2**
- **Remote Gateway** : IP publique du site distant
- **Authentication** : *Mutual PSK* → **Pre-Shared Key** longue et aléatoire
- **Encryption** : **AES-256-GCM** · **Hash** : SHA-256 · **DH Group** : **14** (2048 bits) ou plus

### Phase 2 (ESP)

**Add P2** :

- **Local Network** : LAN local · **Remote Network** : LAN distant
- **Protocol** : ESP · **Encryption** : AES-256-GCM · **PFS Group** : 14

### Règles de pare-feu

**Firewall → Rules → IPsec** : autoriser le trafic entre les deux LAN.

> Sur le site B : mêmes paramètres, en **inversant** *Local*/*Remote* et la *Remote Gateway*.

---

## Vérification (comment savoir que ça marche)

- **Status → IPsec** : le tunnel est **Established** (P1 + P2 « up »).
- Un poste du LAN A **pingue** un poste du LAN B (et inversement).
- Les **SA** (associations de sécurité) sont visibles.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Phase 1 ne monte pas | PSK/algos/IP différents | Aligner **exactement** P1 des deux côtés |
| Phase 1 OK, Phase 2 KO | Sous-réseaux local/remote incohérents | Vérifier les réseaux P2 (miroir) |
| Tunnel up mais pas de ping | Règles firewall **IPsec** / NAT | Autoriser le trafic ; **pas de NAT** sur le trafic VPN |
| Coupures | MTU / fragmentation | Ajuster le MSS |

## Sécurité et bonnes pratiques

- **IKEv2 + AES-GCM + PFS** ; **PSK forte** (ou **certificats/PKI** — **CP7-11**).
- **Bannir** les algos faibles (**DES/3DES, MD5, SHA1, DH < 14**).
- Restreindre les **sous-réseaux** échangés au strict nécessaire ; journaliser.

## ⚠️ À ne pas confondre / obsolète

- **IKEv1 → IKEv2** ; **3DES/DES/MD5/SHA1** sont obsolètes → **AES-256-GCM / SHA-256**.
- **IPsec site-à-site** (relie deux réseaux) ≠ **VPN nomade** OpenVPN/WireGuard (relie un poste — **CP7-07**).
- Le trafic du tunnel **ne doit pas être NATé** (sinon les réseaux ne se voient pas correctement).

---

## Sources (doc officielle)

- [Netgate — IPsec Site-to-Site VPN Example](https://docs.netgate.com/pfsense/en/latest/recipes/ipsec-s2s-psk.html) — consulté le 24/07/2026
- [Netgate — IPsec](https://docs.netgate.com/pfsense/en/latest/vpn/ipsec/index.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI · [x] version datée · [x] rien d'obsolète (IKEv2, AES-GCM) · [x] config à tester en lab · [x] conforme doc Netgate · [x] vérification présente · [x] sécurité (crypto moderne) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
