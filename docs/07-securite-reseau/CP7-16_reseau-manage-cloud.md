# CP7-16 — Superviser un réseau managé dans le Cloud (contrôleur UniFi/Meraki/Aruba)

**Objectif** : administrer un parc réseau (points d'accès, commutateurs, passerelle) depuis un **contrôleur Cloud unique** — adopter les équipements, pousser SSID/VLAN, superviser et mettre à jour.

**Rattachement REAC** : CP7 « Maintenir et sécuriser les accès Internet et les interconnexions » — savoir-faire : exploiter et superviser un réseau.

**Durée** : ~30 min · **Niveau** : intermédiaire.

---

## Prérequis

- Des équipements compatibles (ex. **Ubiquiti UniFi**, **Cisco Meraki**, **HPE Aruba**) et un accès Internet.
- Un plan de VLAN/SSID (**CP4-01/CP4-04**) et un compte administrateur Cloud protégé par **MFA** (**CP7-18**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Contrôleur (référence pédagogique) | **UniFi Network Application 9.x** | 24/07/2026 |
| Alternatives | **Cisco Meraki Dashboard**, **Aruba Central** | 24/07/2026 |
| Configuration appliance | **à tester en lab** | 24/07/2026 |

> **Modèle Cloud** : au lieu de configurer chaque équipement en SSH, un **contrôleur** central (hébergé en ligne) pousse la configuration, remonte l'état et gère les mises à jour de **tout le parc**, multi-sites. Meraki/Aruba Instant On/UniFi couvrent ~80 % du marché Wi-Fi managé.

---

## Procédure — GUI (exemple UniFi ; principe identique Meraki/Aruba)

1. **Contrôleur** : accéder à la console (UniFi Cloud / UniFi Network self-hébergé ; **Meraki Dashboard** ou **Aruba Central** pour les autres — licence requise chez Meraki).
2. **Adopter les équipements** : **Devices** → chaque équipement neuf apparaît « *Pending Adoption* » → **Adopt** (il télécharge sa config et son firmware).
3. **Réseaux/VLAN** : **Settings ▸ Networks** → créer les VLAN (ex. Utilisateurs 10, Invités 30, Admin 99).
4. **Wi-Fi** : **Settings ▸ WiFi** → créer les SSID, **WPA3** (ou WPA2/WPA3 mixte), rattacher chaque SSID à son **VLAN** ; activer l'**isolation** pour le SSID invité.
5. **Supervision** : tableau de bord (topologie, clients, débits, latence), **alertes** par mail/push sur panne ou seuil.
6. **Mises à jour** : pousser le firmware sur les équipements depuis la console (planifier hors production).

## Procédure — CLI / API (automatisation multi-sites)

```bash
# Meraki Dashboard API (REST) — exemple : lister les équipements d'un réseau
curl -H "X-Cisco-Meraki-API-Key: <APIKEY>" \
     https://api.meraki.com/api/v1/networks/<netId>/devices
```

> À grande échelle, on **automatise** via l'**API REST** (Meraki, UniFi, Aruba Central) et des outils comme **Ansible/Terraform** pour appliquer une configuration reproductible à des dizaines de sites (*templates*).

---

## Vérification (comment savoir que ça marche)

- Chaque équipement affiche **« Connected »** dans la console, firmware à jour.
- Un client Wi-Fi se connecte au bon **SSID** et reçoit une IP du **VLAN attendu** (vérifier côté DHCP — **CP2-13**).
- Le tableau de bord remonte le trafic et l'état en **temps réel**.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Équipement bloqué en « Adopting » | Adoption L3 / *inform URL* injoignable | Vérifier DNS/route vers le contrôleur, régler l'*inform URL* |
| « Managed by other » | Déjà adopté ailleurs | Réinitialiser l'équipement (*factory reset*) puis réadopter |
| SSID sans le bon VLAN | Trunk/tag manquant côté switch | Vérifier le **trunk 802.1Q** (**CP4-05**) |
| Console injoignable | Panne Internet | Prévoir un accès **local**/local-console de secours |

## Sécurité et bonnes pratiques

- **Protéger le compte Cloud d'administration** par **MFA** (**CP7-18**) — il contrôle tout le parc.
- **Comptes admin limités et nominatifs**, journalisation des actions dans la console.
- **Segmenter** (VLAN invités isolés), **WPA3**, ports d'accès sécurisés (**CP4-15**).
- **Dépendance Internet** : documenter le comportement en cas de coupure (les équipements continuent souvent en autonomie, mais plus d'administration).

## ⚠️ À ne pas confondre / obsolète

- Administration **device-par-device** (SSH sur chaque équipement) ≠ **contrôleur centralisé** (config poussée en masse).
- **WEP/WPA** (obsolètes) → **WPA2/WPA3** (**CP4-09**).
- « Cloud managed » ≠ « données dans le Cloud » : c'est le **plan de gestion** qui est hébergé, pas forcément le trafic utilisateur.

---

## Sources (doc officielle)

- [Ubiquiti — UniFi Network](https://help.ui.com/hc/en-us/categories/6583256751383-UniFi-Network) — consulté le 24/07/2026
- [Cisco Meraki — Documentation](https://documentation.meraki.com/) — consulté le 24/07/2026
- [HPE Aruba Networking Central](https://www.arubanetworks.com/products/network-management-operations/central/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI/API · [x] versions datées · [x] rien d'obsolète (WPA3, contrôleur central) · [x] config **à tester en lab** · [x] GUI conforme doc éditeurs · [x] vérification présente · [x] sécurité (MFA compte Cloud, segmentation) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
