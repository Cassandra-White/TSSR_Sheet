# LAB-03 — Créer des modèles de VM de base (Windows Server / Debian) réutilisables

**Objectif** : créer des **modèles (templates)** de VM prêts à l'emploi — un **Windows Server** et un **Debian** — pour déployer le lab **vite** et de façon **homogène**.

**Rattachement REAC** : Socle du lab — industrialisation de l'environnement de test.

**Durée** : ~30 min · **Niveau** : intermédiaire.

---

## Prérequis

- Le lab en place (**LAB-01**), le plan d'adressage (**LAB-02**).
- Les techniques de **clonage/cloud-init** (**CP5-04**) et de **généralisation Windows** (**CP9-02**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Modèle Windows | **Windows Server 2025** (Sysprep) | 24/07/2026 |
| Modèle Linux | **Debian 13** (cloud-init) | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> Un **modèle** = une VM de référence **nettoyée** et **généralisée**, convertie en **template** en lecture seule. On la **clone** ensuite en quelques secondes, puis on **personnalise** (nom, IP, domaine) au 1ᵉʳ démarrage.

---

## Procédure — Méthode

### 1. Construire la VM de référence

- Installer l'OS, le **mettre à jour**, installer le **guest agent** (QEMU/VMware) et un **durcissement de base** (**CP7-17**).

### 2. Modèle **Debian** (cloud-init)

```bash
apt install qemu-guest-agent cloud-init
# Nettoyage avant de figer le modèle :
truncate -s 0 /etc/machine-id            # identité unique régénérée au boot
rm -f /etc/ssh/ssh_host_*                # clés d'hôte SSH régénérées
cloud-init clean ; history -c
```

- Puis **convertir en template** et cloner (**CP5-04**) ; **cloud-init** applique nom d'hôte, IP et clé SSH au 1ᵉʳ boot.

### 3. Modèle **Windows Server** (Sysprep)

```cmd
C:\Windows\System32\Sysprep\sysprep.exe /generalize /oobe /shutdown
```

- **Sysprep /generalize** retire le **SID** et l'identité machine (**CP9-02**) → convertir en template, cloner, puis personnaliser via **unattend** (nom, domaine).

### 4. Déployer

- **Cloner** le template (lié/complet), démarrer → identité **unique**, prêt à **rejoindre le domaine** (**CP2-07**).

---

## Vérification (comment savoir que c'est bon)

- Un clone démarre avec un **nom** et une **identité uniques** (Windows : **SID** distinct ; Debian : **machine-id** régénéré), **à jour**.
- Deux clones **ne se disputent pas** la même IP (DHCP) ni le même SID.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| **SID dupliqué** (Windows) | Sysprep **oublié** | **Généraliser** (Sysprep) avant le template — casse sinon WSUS/AD (**CP9-01**) |
| Même IP DHCP (Debian) | **machine-id** dupliqué | Vider `/etc/machine-id` avant de figer |
| Cloud-init sans effet | Drive/paquet absent | Installer **cloud-init** + CloudInit Drive (**CP5-04**) |
| Modèle obsolète | Pas de mises à jour | **Reconstruire** régulièrement (patchs) |

## Sécurité et bonnes pratiques

- **Généralisation obligatoire** : jamais de clone **sans** retirer SID/identité (doublons = pannes de domaine).
- **Pas de secret en dur** : clé SSH (cloud-init) / `unattend` protégé.
- **Modèles à jour** (patchs) et **versionnés/datés** ; un **modèle par OS**.

## ⚠️ À ne pas confondre / obsolète

- **Modèle généralisé** (déployable en série) ≠ **VM clonée telle quelle** (identité dupliquée).
- **Sysprep** (Windows) ≡ **nettoyage machine-id + cloud-init** (Debian) : même objectif d'unicité.
- Refaire chaque VM **à la main** (lent, hétérogène) → **modèles** réutilisables.

---

## Sources (doc officielle)

- [Proxmox VE — VM Templates & Cloud-Init](https://pve.proxmox.com/wiki/Cloud-Init_Support) — consulté le 24/07/2026
- [Microsoft Learn — Sysprep (generalize)](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/sysprep--generalize--a-windows-installation) — consulté le 24/07/2026
- [Debian — cloud-init](https://cloudinit.readthedocs.io/en/latest/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] Méthode + renvois CP5-04/CP9-02 · [x] versions datées (WS 2025 / Debian 13) · [x] rien d'obsolète (généralisation, cloud-init) · [x] procédure **à tester en lab** · [x] conforme doc Proxmox/Microsoft · [x] vérification présente (SID/machine-id uniques) · [x] sécurité (généralisation, pas de secret en dur) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
