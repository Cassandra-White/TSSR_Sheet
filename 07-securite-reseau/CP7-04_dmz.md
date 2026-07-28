# CP7-04 — Mettre en place une DMZ

**Objectif** : créer une zone démilitarisée (DMZ) pour héberger les services exposés à Internet en les **isolant** du réseau interne.

**Rattachement REAC** : CP7 « Maintenir et sécuriser les accès Internet et les interconnexions » — savoir-faire : segmenter et cloisonner le réseau.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- pfSense opérationnel (**CP7-01**), une **3ᵉ interface** (ou un VLAN) disponible, notions de règles/NAT (**CP7-02/03**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| pfSense CE | 2.8.1 | 24/07/2026 |
| Configuration | **à tester en lab** | 24/07/2026 |

> Principe : les serveurs joignables depuis Internet (web, relais mail…) vivent en **DMZ**. Si un serveur DMZ est compromis, l'attaquant **n'atteint pas** le LAN.

---

## Procédure — GUI

1. **Interfaces → Assignments** : ajouter l'interface (**OPT1**), l'**activer**, lui donner une IP (ex. `192.168.30.1/24`), DHCP optionnel. La renommer **DMZ**.
2. **Exposer un service** : port forward WAN → serveur DMZ (**CP7-03**).
3. **Règles DMZ** (Firewall → Rules → DMZ), dans cet ordre :
   - **Bloquer DMZ → LAN** (alias des réseaux privés `RFC1918`) — isolation clé.
   - **Autoriser DMZ → Internet** (ports nécessaires : DNS, HTTP/S, mises à jour).

---

## Vérification (comment savoir que ça marche)

- Un serveur DMZ est **joignable depuis Internet** (son service exposé répond).
- Depuis la DMZ, une tentative de connexion **vers le LAN échoue** (isolation).
- **Diagnostics → States / System Logs** confirment les flux.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| La DMZ atteint le LAN | Règle de blocage absente/mal placée | Placer **Block DMZ→RFC1918** en haut |
| Service injoignable de l'extérieur | Port forward / règle WAN | Vérifier NAT + règle associée (**CP7-03**) |
| Pas d'Internet en DMZ | Règle sortante manquante | Autoriser DMZ → Internet (ports utiles) |

## Sécurité et bonnes pratiques

- **Cloisonner** strictement : DMZ ↛ LAN.
- **Durcir** les serveurs exposés (**CP7-17**) ; n'exposer que le nécessaire.
- **Journaliser** les flux entrants/sortants de la DMZ.

## ⚠️ À ne pas confondre / obsolète

- Une **vraie DMZ** (segment filtré par le pare-feu) ≠ le **« DMZ host »** des box grand public (qui expose **tout** un hôte — à éviter).
- La DMZ est une zone **semi-fiable** : on la traite comme potentiellement hostile vis-à-vis du LAN.

---

## Sources (doc officielle)

- [Netgate — Interface Configuration (OPT interfaces)](https://docs.netgate.com/pfsense/en/latest/interfaces/index.html) — consulté le 24/07/2026
- [Netgate — Firewall Rule Basics](https://docs.netgate.com/pfsense/en/latest/firewall/index.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI · [x] version datée · [x] rien d'obsolète (vraie DMZ vs DMZ host) · [x] config à tester en lab · [x] conforme doc Netgate · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
