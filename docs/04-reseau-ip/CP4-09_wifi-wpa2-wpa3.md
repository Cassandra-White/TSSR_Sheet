# CP4-09 — Configurer et sécuriser un réseau Wi-Fi (WPA2/WPA3, SSID)

**Objectif** : mettre en service un réseau Wi-Fi et le sécuriser (chiffrement WPA3/WPA2, SSID, réseau invité).

**Rattachement REAC** : CP4 « Exploiter un réseau IP » — savoir-faire : déployer et sécuriser un accès sans fil.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un point d'accès / routeur Wi-Fi administrable, accès à son interface d'administration.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Point d'accès Wi-Fi | interface d'admin constructeur — **à tester en lab** | 24/07/2026 |
| Bonnes pratiques | WPA3 (SAE), transition WPA2/WPA3 — vérifiées le 24/07/2026 | 24/07/2026 |

> Administration en **GUI** (console web de l'AP/routeur ou contrôleur type UniFi/Meraki — **CP7-16**).

---

## Procédure — GUI (console d'administration de l'AP)

1. Se connecter à l'admin de l'AP en **HTTPS** ; **changer les identifiants par défaut**.
2. **SSID** : nom neutre (éviter nom/marque/infos perso). Laisser la **diffusion activée** (masquer le SSID n'est **pas** une sécurité).
3. **Sécurité / chiffrement** :
   - **WPA3-Personal (SAE)** si tous les clients sont compatibles ;
   - sinon **WPA2/WPA3 (mode transition)** pour les anciens équipements.
   - **Mot de passe ≥ 16 caractères**, aléatoire.
4. **Désactiver WPS** (code PIN 8 chiffres cassable) et **WEP/WPA(1)/TKIP** (obsolètes).
5. **Réseau invité** : SSID séparé, **VLAN dédié** isolé du LAN (**CP4-04**), avec son propre mot de passe.
6. Choisir les **bandes/canaux** (2,4 / 5 / 6 GHz — Wi-Fi 6E/7) ; **mettre à jour le firmware** de l'AP.
7. *(Entreprise)* **WPA3-Enterprise / 802.1X (RADIUS)** pour une authentification par compte plutôt qu'un mot de passe partagé.

---

## Vérification (comment savoir que ça marche)

- Un client se connecte ; sous Windows : `netsh wlan show interfaces` indique l'**authentification** (WPA3 / WPA2).
- Un client du **réseau invité** accède à Internet mais **pas** au LAN (isolation vérifiée par `ping` d'une ressource interne — doit échouer).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Ancien appareil ne se connecte pas | Mode **WPA3-only** | Passer en **transition WPA2/WPA3** |
| Débit faible / coupures | Canal encombré / interférences | Changer de canal ; privilégier 5/6 GHz |
| L'invité voit le LAN | Isolation absente | Réseau invité sur **VLAN dédié**, isolation client |
| Attaques par PIN | **WPS activé** | Désactiver WPS |

## Sécurité et bonnes pratiques

- **WPA3** (ou WPA2-AES au minimum) ; **jamais WEP/WPA(1)/TKIP**.
- **WPS désactivé**, mot de passe **long et aléatoire**, **firmware à jour**.
- **Réseau invité isolé** (VLAN) ; en entreprise, **WPA3-Enterprise (802.1X)**.
- Suivre les recommandations **ANSSI** pour le Wi-Fi.

## ⚠️ À ne pas confondre / obsolète

- **WEP et WPA(1)/TKIP sont cassés** : à proscrire → **WPA2-AES** minimum, **WPA3** recommandé.
- **Masquer le SSID** et le **filtrage MAC** ne sont **pas** des sécurités fiables (contournables).
- **WPS** est une commodité **dangereuse** : à désactiver.

---

## Sources (doc officielle)

- [Wi-Fi Alliance — Security (WPA3)](https://www.wi-fi.org/discover-wi-fi/security) — consulté le 24/07/2026
- [ANSSI — Recommandations relatives à la sécurité des réseaux Wi-Fi](https://cyber.gouv.fr/publications/recommandations-de-securite-relatives-aux-reseaux-wifi) — consulté le 24/07/2026
- [Cisco — WPA3 Deployment Guide](https://www.cisco.com/c/en/us/td/docs/wireless/controller/9800/technical-reference/wpa3-dg.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI (admin AP) · [x] versions/pratiques datées · [x] rien d'obsolète (WPA3, anti-WEP/WPS) · [x] à tester en lab · [x] conforme doc (Wi-Fi Alliance/ANSSI) · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
