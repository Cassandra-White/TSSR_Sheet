# CP5-01 — Installer l'hyperviseur Proxmox VE 9

**Objectif** : installer l'hyperviseur **Proxmox VE 9** sur un serveur et accéder à la **console web** d'administration.

**Rattachement REAC** : CP5 « Maintenir des serveurs dans une infrastructure virtualisée » — savoir-faire : déployer un hyperviseur.

**Durée** : ~30 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un serveur **x86-64** avec la **virtualisation matérielle** (**Intel VT-x / AMD-V**) **activée dans le firmware**.
- **≥ 8 Go de RAM** (16 Go si **ZFS**), un **SSD** dédié à l'hôte, une connexion **Ethernet filaire** (Wi-Fi non supporté).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Hyperviseur | **Proxmox VE 9.2** (base Debian 13, noyau 6.14, QEMU 10, ZFS 2.3) | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **Proxmox VE** = hyperviseur **type 1** (KVM pour les VM + LXC pour les conteneurs), administré par une **console web** (port **8006**). Base **Debian 13**.

---

## Procédure — GUI (installation ISO)

1. Écrire l'**ISO Proxmox VE 9.2** sur une clé USB, démarrer dessus.
2. Assistant :
   - **Disque cible** (⚠️ **effacé** ; option **ZFS RAID** possible pour la redondance),
   - **pays/clavier**, **mot de passe root** + e-mail,
   - **réseau** : **IP fixe**, **nom d'hôte FQDN**, passerelle, DNS.
3. Redémarrer → ouvrir **`https://<ip-serveur>:8006`** (connexion `root@pam`).

## Procédure — CLI (post-installation)

Pour un usage **communautaire** (sans abonnement), basculer sur le dépôt **no-subscription** :

```bash
# Désactiver le dépôt entreprise, activer le dépôt no-subscription (pve + ceph),
# puis mettre à jour
apt update && apt full-upgrade -y
pveversion                     # vérifier la version installée
```

---

## Vérification (comment savoir que ça marche)

- La **console web** (`:8006`) s'ouvre ; le **nœud** apparaît **en ligne** (vert).
- `pveversion` renvoie **pve-manager/9.2.x** ; `apt update` ne renvoie pas d'erreur 401 (dépôt).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| VM impossibles à démarrer | **VT-x/AMD-V désactivé** | Activer la virtualisation dans le **BIOS/UEFI** |
| Réseau KO | Wi-Fi comme interface principale | Utiliser **Ethernet filaire** |
| `apt` erreur 401 | Dépôt **entreprise** sans abonnement | Passer au dépôt **no-subscription** |
| Console inaccessible | Pare-feu / mauvaise IP | Vérifier IP/port **8006** (HTTPS) |

## Sécurité et bonnes pratiques

- **MFA** sur le compte d'administration (**CP7-18**), **pare-feu Proxmox** activé, accès via un **réseau d'administration**.
- **Ne pas exposer** la console `:8006` directement sur Internet (VPN — **CP7-07**).
- **Mises à jour** régulières ; sauvegarder la configuration.

## ⚠️ À ne pas confondre / obsolète

- **Type 1** (bare-metal, Proxmox/ESXi/Hyper-V) ≠ **type 2** (hôte, VirtualBox/Workstation).
- **Dépôt entreprise** (abonnement) ≠ **no-subscription** (communautaire).
- **Wi-Fi** non supporté comme interface principale.

---

## Sources (doc officielle)

- [Proxmox VE — Installation](https://pve.proxmox.com/pve-docs/chapter-pve-installation.html) — consulté le 24/07/2026
- [Proxmox VE — ISO 9.2](https://www.proxmox.com/en/downloads/proxmox-virtual-environment/iso) — consulté le 24/07/2026
- [Proxmox VE — Package Repositories](https://pve.proxmox.com/wiki/Package_Repositories) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (post-install) · [x] version datée (PVE 9.2) · [x] rien d'obsolète (Debian 13, no-subscription) · [x] procédure **à tester en lab** · [x] conforme doc Proxmox · [x] vérification présente (`pveversion`) · [x] sécurité (MFA, pare-feu, pas d'expo Internet) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
