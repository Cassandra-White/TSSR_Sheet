# CP4-16 — Configurer l'adressage IPv6 (bases)

**Objectif** : comprendre les types d'adresses IPv6 et configurer une interface en IPv6, côté Windows et Linux.

**Rattachement REAC** : CP4 « Exploiter un réseau IP » — savoir-faire : mettre en œuvre l'adressage IPv6.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un poste Windows 11 / Server 2025 et/ou un serveur Debian 13.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian (`ip -6`) | 13.6 — **testé en namespace** | 24/07/2026 |
| Windows | 11 24H2 / Server 2025 | 24/07/2026 |

## Rappels IPv6

- Adresse **128 bits** en hexadécimal ; abréviation : zéros non significatifs omis, **`::`** une seule fois.
- Types : **link-local `fe80::/10`** (automatique, non routée, obligatoire) · **global unicast `2000::/3`** (routable, préfixe **/64**) · **unique local `fc00::/7`** (privée) · **multicast `ff00::/8`** (il n'y a **pas de broadcast** en IPv6).
- Autoconfiguration : **SLAAC** (via *Router Advertisement*), **DHCPv6**, ou **manuel**.

---

## Procédure — GUI (Windows)

`ncpa.cpl` → *Propriétés* de la carte → **Protocole Internet version 6 (TCP/IPv6)** → adresse manuelle ou automatique.

## Procédure — CLI

### Linux (Debian)

```bash
ip -6 addr                                   # afficher
sudo ip -6 addr add 2001:db8:10::1/64 dev enp1s0
sudo ip -6 route add default via 2001:db8:10::254
ping -6 2001:db8:10::254
```

### Windows

```powershell
Get-NetIPAddress -AddressFamily IPv6
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 2001:db8:10::10 `
  -PrefixLength 64 -DefaultGateway 2001:db8:10::1
ping -6 2001:db8:10::1
```

---

## Vérification (sorties obtenues en namespace)

```
ip -6 addr → inet6 2001:db8:10::1/64 scope global
             inet6 fe80::bc4f:d7ff:fee0:a858/64 scope link      (link-local auto)
ip -6 route → 2001:db8:10::/64 ... ; 2001:db8:20::/64 via 2001:db8:10::254
```

Une adresse **link-local `fe80::`** apparaît toujours ; une **globale** n'apparaît que si SLAAC/DHCPv6/manuel l'a fournie.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Seulement une `fe80::`, pas de globale | Pas de RA/SLAAC/DHCPv6 | Vérifier le routeur (RA) ou fixer une adresse manuelle |
| IPv4 OK, IPv6 KO | Pare-feu IPv6 distinct | Ouvrir les règles **IPv6** (séparées de l'IPv4) |
| Résolution incohérente | Windows **préfère IPv6** | Ne pas désactiver IPv6 sans raison |

## Sécurité et bonnes pratiques

- **Ne pas désactiver IPv6** sans raison (des composants Windows en dépendent).
- Prévoir un **pare-feu IPv6** dédié (règles distinctes de l'IPv4).
- Sur les postes, activer les **privacy extensions** (adresses temporaires).

## ⚠️ À ne pas confondre / obsolète

- **Pas de broadcast** en IPv6 : remplacé par **multicast** et **NDP** (Neighbor Discovery, qui remplace **ARP**).
- Le préfixe réseau standard est **/64** (l'hôte occupe 64 bits).
- **Désactiver IPv6** « pour simplifier » est une **mauvaise pratique**.

---

## Sources (référence)

- [RFC 4291 — IP Version 6 Addressing Architecture](https://www.rfc-editor.org/rfc/rfc4291) — consulté le 24/07/2026
- [Microsoft — Configure IPv6 (New-NetIPAddress)](https://learn.microsoft.com/en-us/powershell/module/nettcpip/new-netipaddress) — consulté le 24/07/2026
- [Debian Wiki — IPv6](https://wiki.debian.org/IPv6) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI (Windows) puis CLI (Win + Linux) · [x] versions datées · [x] rien d'obsolète (NDP vs ARP) · [x] **`ip -6` testé en namespace** · [x] conforme RFC · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
