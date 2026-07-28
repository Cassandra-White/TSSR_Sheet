# CP3-01 — Installer Debian 13 (serveur)

**Objectif** : installer Debian 13 « trixie » en configuration serveur (sans environnement de bureau), avec un partitionnement simple et le serveur SSH.

**Rattachement REAC** : CP3 « Exploiter des serveurs Linux » — savoir-faire : installer et préparer un serveur Linux.

**Durée** : ~25 min · **Niveau** : débutant.

---

## Prérequis

- Image ISO **Debian 13.6 « trixie » netinst** (édition **amd64**), depuis debian.org.
- Une VM (Proxmox / VMware / Hyper-V) : 1–2 vCPU, ≥ 2 Go RAM, disque ≥ 20 Go, accès réseau (DHCP pendant l'install).
- ISO monté comme périphérique d'amorçage.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian | **13.6 « trixie »** (netinst, amd64) | 24/07/2026 |

---

## Procédure — GUI (installeur Debian)

> Le programme d'installation Debian (menu graphique ou texte). Étapes clés.

1. Démarrer sur l'ISO → **Graphical install**.
2. **Langue**, **pays**, **disposition du clavier**.
3. **Nom de machine (hostname)** : ex. `srv-deb01`. **Domaine** : ex. `lab.local`.
4. **Mot de passe root** : le laisser **vide** pour activer `sudo` sur le premier compte (recommandé) ; sinon définir un mot de passe root fort.
5. Créer le **compte utilisateur** (nom complet, identifiant, mot de passe fort).
6. **Partitionnement** : *Assisté – utiliser un disque entier* → *Tout dans une seule partition* (simple). Pour un vrai serveur, séparer `/var`, `/home`, `/tmp`. Puis **écrire les modifications sur le disque**.
7. **Miroir réseau** (gestionnaire de paquets) : pays le plus proche → `deb.debian.org` ; proxy vide si aucun.
8. **Sélection des logiciels (tasksel)** : **décocher l'environnement de bureau**, **cocher « serveur SSH »** et **« utilitaires usuels du système »**.
9. **GRUB** : installer le chargeur d'amorçage sur le disque principal (ex. `/dev/sda`).
10. **Terminer l'installation** → redémarrer et retirer l'ISO.

## Procédure — CLI (post-installation : vérifs et sudo)

```bash
# Version installée
cat /etc/os-release

# Passer administrateur
su -            # (ou : sudo -i si sudo déjà configuré)

# Mettre à jour l'index des paquets et le système
apt update && apt upgrade

# Si le mot de passe root est défini et que sudo manque :
apt install sudo
usermod -aG sudo <utilisateur>   # ajoute l'utilisateur au groupe sudo
# se déconnecter/reconnecter pour recharger les groupes
```

*(Installation elle-même : à valider en lab. Commandes post-install standard.)*

---

## Vérification (comment savoir que ça marche)

```bash
hostnamectl                     # nom d'hôte + « Operating System: Debian GNU/Linux 13 »
systemctl is-system-running     # doit renvoyer « running »
systemctl status ssh            # le service SSH est actif
# Depuis un autre poste :
ssh <utilisateur>@<ip_du_serveur>
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Pas de réseau pendant l'install | DHCP/miroir injoignable | Vérifier la carte (NAT/pont), reconfigurer le réseau dans l'installeur |
| `sudo: command not found` | root non vide + sudo absent | `su -` puis `apt install sudo` + `usermod -aG sudo` |
| Écran noir au démarrage | GRUB mal installé | Réinstaller GRUB sur le bon disque |
| SSH ne répond pas | Tâche « serveur SSH » non cochée | `apt install openssh-server` puis `systemctl enable --now ssh` |

## Sécurité et bonnes pratiques

- **Root vide + sudo** = traçabilité des actions d'administration.
- **Pas d'environnement de bureau** sur un serveur (surface d'attaque et ressources réduites).
- Mettre à jour dès l'installation (**CP3-11**), puis durcir l'accès SSH (**CP3-09**).

## ⚠️ À ne pas confondre / obsolète

- Utiliser l'ISO **à jour (13.6)**, pas une 13.0 périmée.
- Debian 13 est la **dernière** version à prendre en charge le **x86 32 bits (i386)** : pour un nouveau serveur, installer impérativement l'édition **amd64 (64 bits)**.

---

## Sources (doc officielle)

- [Debian « trixie » — Release Information](https://www.debian.org/releases/trixie/) — consulté le 24/07/2026
- [Debian 13.6 released — News](https://www.debian.org/News/2026/20260711) — consulté le 24/07/2026
- [Guide d'installation Debian](https://www.debian.org/releases/trixie/installmanual) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète (13.6, amd64) · [x] install à valider en lab, CLI post-install standard · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
