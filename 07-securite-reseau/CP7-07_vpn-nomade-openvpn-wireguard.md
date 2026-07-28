# CP7-07 — Configurer un VPN nomade (OpenVPN / WireGuard) + client

**Objectif** : permettre à un utilisateur distant (télétravail) d'accéder au réseau de l'entreprise via un VPN d'accès, avec OpenVPN ou WireGuard sur pfSense.

**Rattachement REAC** : CP7 « Maintenir et sécuriser les accès Internet et les interconnexions » — savoir-faire : fournir un accès distant sécurisé.

**Durée** : ~35 min · **Niveau** : avancé.

---

## Prérequis

- pfSense opérationnel (**CP7-01**) avec une IP publique ; une PKI/CA pour OpenVPN (**CP7-11**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| pfSense CE | 2.8.1 — **OpenVPN intégré**, **WireGuard** (paquet, depuis 2.7) | 24/07/2026 |
| Configuration / clés | **à tester en lab** | 24/07/2026 |

---

## Option A — OpenVPN (auth par utilisateur possible)

1. **VPN → OpenVPN → Wizards → Remote Access** : type d'authentification (*Local User Access*, RADIUS ou **LDAP/AD**).
2. Sélectionner/créer la **CA** et le **certificat serveur** (**CP7-11**).
3. Assistant : interface **WAN**, **UDP**, port **1194**, **réseau tunnel** (ex. `10.8.0.0/24`), réseau local à joindre, DNS.
4. L'assistant crée les **règles** (WAN 1194 + OpenVPN).
5. Créer les **utilisateurs** (+ certificats) ; installer le paquet **OpenVPN Client Export** et exporter le profil `.ovpn`.
6. **Client** : installer *OpenVPN Connect*, importer le `.ovpn`, se connecter.

## Option B — WireGuard (moderne, rapide, par clés)

1. **System → Package Manager** : installer **WireGuard**.
2. **VPN → WireGuard → Tunnels → Add** : génère les **clés serveur**, écoute **UDP 51820**.
3. **Peers → Add** : clé **publique du client**, *Allowed IPs*.
4. Règle **firewall** WAN (51820) + onglet WireGuard.
5. **Client** : app WireGuard → générer une paire de clés → config (Endpoint = IP publique:51820, clé publique du serveur, *AllowedIPs* = LAN ou `0.0.0.0/0`).

> **WireGuard** n'authentifie **pas par utilisateur** (paires de clés) : pour une auth **annuaire/MFA**, choisir **OpenVPN**.

---

## Vérification (comment savoir que ça marche)

- **Status → OpenVPN / WireGuard** : le client est **connecté** (handshake WireGuard récent).
- Le client obtient une **IP du tunnel** et **atteint le LAN** (ping d'une ressource interne).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Pas de connexion | Port UDP fermé (1194/51820) | Ouvrir le port sur le WAN ; vérifier le NAT |
| Connecté mais pas d'accès LAN | Routes / *Allowed IPs* / règles | Vérifier les réseaux poussés et les règles VPN |
| OpenVPN : `TLS handshake failed` | Certificat / heure | Vérifier la **PKI** (**CP7-11**) et l'heure (**CP4-13**) |
| Pas de résolution | DNS non poussé | Distribuer le DNS interne |

## Sécurité et bonnes pratiques

- **OpenVPN** : certificats + TLS + authentification utilisateur ; ajouter la **MFA** (**CP7-18**).
- **WireGuard** : **protéger la clé privée** ; algorithmes modernes par défaut.
- N'ouvrir que le **port VPN** ; restreindre les réseaux accessibles.

## ⚠️ À ne pas confondre / obsolète

- **WireGuard** (rapide, clés, pas d'auth utilisateur) vs **OpenVPN** (auth annuaire/MFA, certificats).
- **PPTP** est **obsolète et non sûr** : à proscrire.
- VPN **nomade** (relie un poste) ≠ VPN **site-à-site** (relie deux réseaux — **CP7-06**).

---

## Sources (doc officielle)

- [Netgate — OpenVPN Remote Access](https://docs.netgate.com/pfsense/en/latest/recipes/openvpn-ra.html) — consulté le 24/07/2026
- [Netgate — WireGuard Remote Access](https://docs.netgate.com/pfsense/en/latest/recipes/wireguard-ra.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI + client · [x] versions datées · [x] rien d'obsolète (WireGuard/OpenVPN, anti-PPTP) · [x] config à tester en lab · [x] conforme doc Netgate · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
