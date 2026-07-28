# CP7-01 — Installer et mettre en route un pare-feu pfSense

**Objectif** : installer pfSense CE, assigner les interfaces WAN/LAN et accéder à la console web pour la première configuration.

**Rattachement REAC** : CP7 « Maintenir et sécuriser les accès Internet et les interconnexions » — savoir-faire : mettre en service un pare-feu périmétrique.

**Durée** : ~30 min · **Niveau** : intermédiaire.

---

## Prérequis

- Une VM/machine avec **deux interfaces réseau** (WAN + LAN), ≥ 1 vCPU / 1 Go RAM / 8 Go disque.
- L'**ISO pfSense CE 2.8.1** (netgate/pfsense.org).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| pfSense CE | **2.8.1** | 24/07/2026 |
| Alternative | OPNsense **26.7** | 24/07/2026 |
| Installation | **à tester en lab** | 24/07/2026 |

---

## Procédure — Console (installation)

1. Démarrer sur l'ISO → accepter → **Install** (ZFS ou UFS) → choisir le disque → installer → **Reboot** (retirer l'ISO).
2. **Assignation des interfaces** : associer la carte **WAN** (vers la box/Internet, en DHCP) et la carte **LAN** (réseau interne).
3. La **LAN** prend par défaut `192.168.1.1/24`. Le menu console permet de la modifier (option *Set interface IP address*).

## Procédure — GUI (première configuration)

1. Depuis un poste **du LAN** : `https://192.168.1.1` — identifiants par défaut **admin / pfsense**.
2. **Setup Wizard** : nom d'hôte, serveurs **DNS**, **fuseau horaire**, configuration **WAN** (DHCP/PPPoE/statique) et **LAN**, puis **changer le mot de passe admin**.
3. La règle LAN par défaut **« LAN → any »** autorise déjà la sortie Internet.

---

## Vérification (comment savoir que ça marche)

- Un poste du LAN obtient une IP et **accède à Internet**.
- Le **Dashboard** pfSense affiche l'état des interfaces et la **version 2.8.1**.
- `Status → Interfaces` : WAN a une IP, LAN est active.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Pas d'accès à `192.168.1.1` | Poste pas sur le LAN / mauvaise IP | Se connecter côté **LAN**, vérifier l'IP du poste |
| WAN sans adresse | DHCP box absent | Vérifier le câblage WAN, le mode d'obtention |
| Interfaces inversées | Mauvaise assignation | Réassigner WAN/LAN dans la console |
| Verrouillé dehors | — | La règle **anti-lockout** protège l'accès admin depuis le LAN |

## Sécurité et bonnes pratiques

- **Changer le mot de passe admin** par défaut ; accès d'administration en **HTTPS**.
- **Jamais** d'administration exposée côté **WAN**.
- Tenir pfSense **à jour** (correctifs de sécurité).

## ⚠️ À ne pas confondre / obsolète

- **pfSense CE** (gratuit, communautaire) vs **pfSense Plus** (Netgate).
- Le **WAN bloque tout le trafic entrant par défaut** (comportement voulu et sûr).
- **OPNsense** est une alternative équivalente (fork), même logique WAN/LAN.

---

## Sources (doc officielle)

- [Netgate — Installing pfSense](https://docs.netgate.com/pfsense/en/latest/install/index.html) — consulté le 24/07/2026
- [pfSense — Téléchargement (Community Edition)](https://www.pfsense.org/download/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] Console puis GUI · [x] version datée (2.8.1) · [x] rien d'obsolète · [x] install à tester en lab · [x] conforme doc Netgate · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
