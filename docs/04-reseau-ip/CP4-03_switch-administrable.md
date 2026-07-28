# CP4-03 — Configurer un switch administrable (accès console/web, IP de gestion)

**Objectif** : accéder à un commutateur managé (console et interface web) et lui attribuer une **IP de gestion** sécurisée dans un **VLAN de gestion** dédié.

**Rattachement REAC** : CP4 « Exploiter un réseau IP » — savoir-faire : mettre en service et administrer un commutateur.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un switch **administrable** (managé), un **câble console** (RJ45-USB/DB9) et un émulateur de terminal (PuTTY, `screen`, `minicom`).
- Le plan d'adressage du VLAN de gestion (**CP4-01**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Commutateur managé | syntaxe **Cisco IOS** (référence pédagogique) | 24/07/2026 |
| Config switch | **à tester en lab** (matériel) | 24/07/2026 |

> La syntaxe exacte varie selon le constructeur (Cisco, HPE/Aruba, Netgear…) ; les **principes** sont identiques.

---

## Procédure — GUI (interface web)

1. Se connecter au switch : IP par défaut du constructeur (ou obtenue en DHCP) via **HTTPS** dans le navigateur.
2. S'authentifier avec les identifiants par défaut → **changer immédiatement le mot de passe**.
3. Créer un **VLAN de gestion** (ex. VLAN 10) et y placer l'**IP de gestion** + la passerelle.
4. **Activer SSH et HTTPS**, **désactiver Telnet et HTTP** (protocoles en clair).

## Procédure — CLI (console, style Cisco IOS)

```text
enable
configure terminal
 hostname SW-CoeurA
 vlan 10
  name GESTION
 exit
 interface vlan 10
  ip address 192.168.99.2 255.255.255.0
  no shutdown
 exit
 ip default-gateway 192.168.99.1
 username admin secret MotDePasseFort
 ip domain-name lab.local
 crypto key generate rsa modulus 2048
 ip ssh version 2
 line vty 0 4
  transport input ssh          ! SSH uniquement (pas de Telnet)
  login local
 exit
 no ip http server             ! désactive HTTP en clair
 ip http secure-server         ! active HTTPS
end
write memory                    ! sauvegarder la configuration
```

---

## Vérification (comment savoir que ça marche)

```text
show ip interface brief         ! l'interface VLAN 10 est "up" avec son IP
show vlan brief                 ! le VLAN de gestion existe
show running-config             ! relire la conf appliquée
```

Depuis un poste **du VLAN de gestion** : `ssh admin@192.168.99.2` doit aboutir (Telnet doit être refusé).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Pas d'accès web/SSH | Poste hors du VLAN de gestion / IP erronée | Se placer dans le bon VLAN ; vérifier l'IP de gestion |
| SSH refusé | Clé RSA absente / SSHv1 / `login` non configuré | `crypto key generate rsa` + `ip ssh version 2` + `login local` |
| Config perdue au redémarrage | `write memory` oublié | Sauvegarder (`copy running-config startup-config`) |
| Accès en Telnet possible | Telnet non désactivé | `transport input ssh` sous `line vty` |

## Sécurité et bonnes pratiques

- **VLAN de gestion dédié**, séparé du trafic des utilisateurs.
- **SSHv2 + HTTPS uniquement** ; désactiver **Telnet** et **HTTP**.
- **Changer les identifiants par défaut** et le **VLAN natif/par défaut** (ne pas rester sur VLAN 1).
- **Sauvegarder la configuration** du switch (**CP8-08**).

## ⚠️ À ne pas confondre / obsolète

- **Telnet / HTTP** (en clair) sont **proscrits** → **SSH / HTTPS**.
- **SSHv1** est vulnérable → **SSHv2** obligatoire.
- Laisser tout le monde sur le **VLAN 1 par défaut** est une mauvaise pratique de sécurité.

---

## Sources (doc officielle)

- [Cisco — Catalyst : Basic Switch Configuration (best practices)](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst3850/software/release/16-1/best_practices_guide/BP_Cat3850/BP_basic_config.pdf) — consulté le 24/07/2026
- [Cisco — Configurer SSH sur les commutateurs](https://www.cisco.com/c/en/us/support/docs/security-vpn/secure-shell-ssh/4145-ssh.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI (web) puis CLI (console) · [x] versions/ref datées · [x] rien d'obsolète (SSHv2/HTTPS, anti-Telnet) · [x] config à tester en lab · [x] conforme doc constructeur · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
