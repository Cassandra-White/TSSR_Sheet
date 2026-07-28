# CP3-03 — Gérer les utilisateurs, les groupes et sudo

**Objectif** : créer, modifier et supprimer des utilisateurs et des groupes sous Debian, et accorder des droits d'administration via `sudo`.

**Rattachement REAC** : CP3 « Exploiter des serveurs Linux » — savoir-faire : gérer les comptes et les droits sur un serveur Linux.

**Durée** : ~20 min · **Niveau** : débutant.

---

## Prérequis

- Debian 13 installé (**CP3-01**), accès console/SSH, droits **root/sudo**.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian | 13.6 « trixie » | 24/07/2026 |
| `useradd` / `groupadd` / `usermod` / `visudo -c` | shadow-utils / sudo — **testés dans le bac à sable** | 24/07/2026 |

> Pas d'interface graphique par défaut sur un serveur Debian (console web **Cockpit** disponible en option) : la gestion se fait en **CLI**.

---

## Procédure — CLI

### Créer un utilisateur

```bash
# Méthode Debian (interactif, crée le home + demande le mot de passe) — recommandée
sudo adduser bob

# Méthode bas niveau (scriptable)
sudo useradd -m -s /bin/bash -c "Bob Martin" bob   # -m: home ; -s: shell ; -c: commentaire
sudo passwd bob                                     # définir le mot de passe
```

### Groupes

```bash
sudo groupadd projet
sudo usermod -aG projet bob      # -aG : AJOUTE au groupe (⚠️ sans -a, -G REMPLACE tous les groupes)
sudo gpasswd -d bob projet       # retirer bob du groupe
```

### Modifier / verrouiller

```bash
sudo usermod -c "Bob R. Martin" -s /bin/bash bob
sudo usermod -L bob     # verrouiller (login impossible) ; -U pour déverrouiller
sudo passwd -l bob      # équivalent verrouillage du mot de passe
```

### Supprimer

```bash
sudo deluser bob            # (Debian) ; ajouter --remove-home pour effacer le home
sudo userdel -r bob         # bas niveau, -r supprime le home
sudo delgroup projet        # ou : groupdel projet
```

### Droits d'administration avec sudo

```bash
# Le plus simple : ajouter l'utilisateur au groupe "sudo" (Debian)
sudo usermod -aG sudo bob          # effectif à la prochaine ouverture de session

# Règles fines : TOUJOURS via un fichier dans /etc/sudoers.d/ validé par visudo
sudo visudo -f /etc/sudoers.d/sysadmin
```

Contenu type (validé avec `visudo -c`) :

```text
%sysadmin ALL=(ALL:ALL) ALL
alice ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx
```

```bash
sudo visudo -c        # vérifie la syntaxe de toute la configuration sudo
```

---

## Vérification (sorties obtenues dans le bac à sable)

```bash
id bob
getent passwd bob        # bob:x:1000:1000:Bob Martin:/home/bob:/bin/bash
getent group projet      # projet:x:1001:bob
groups bob
```

`visudo -c` renvoie `parsed OK` quand la configuration sudo est saine.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| L'utilisateur a **perdu ses groupes** | `usermod -G` **sans `-a`** (remplacement) | Toujours utiliser `usermod -aG` |
| `sudo` refuse encore après ajout | Session pas rechargée | Se déconnecter/reconnecter, ou `newgrp sudo` |
| Fichier sudoers cassé = plus de sudo | Édition directe de `/etc/sudoers` | **Toujours `visudo`** (bloque la sauvegarde si erreur) |
| `deluser: command not found` | Distribution non-Debian | Utiliser `userdel -r` |

## Sécurité et bonnes pratiques

- **Moindre privilège** : limiter `NOPASSWD` à des commandes précises, jamais à `ALL`.
- Éditer la config sudo **uniquement via `visudo`** ; préférer des fichiers dans `/etc/sudoers.d/`.
- **Verrouiller** les comptes inutilisés (`usermod -L`) plutôt que de les laisser actifs.
- Désactiver la connexion **root en SSH** (**CP3-09**).

## ⚠️ À ne pas confondre / obsolète

- `usermod -G groupe` **sans `-a`** écrase l'appartenance aux groupes : piège classique → toujours `-aG`.
- `adduser`/`deluser` (scripts conviviaux **Debian**) ≠ `useradd`/`userdel` (outils bas niveau POSIX).
- Sous Debian, le groupe d'admin est **`sudo`** (et non `wheel` comme sur Red Hat).

---

## Sources (doc officielle)

- [Debian Wiki — sudo](https://wiki.debian.org/sudo) — consulté le 24/07/2026
- [useradd(8) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/passwd/useradd.8.en.html) — consulté le 24/07/2026
- [sudoers(5) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/sudo/sudoers.5.en.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI (pas de GUI serveur par défaut) · [x] versions datées · [x] rien d'obsolète (`-aG`, groupe `sudo`) · [x] **commandes testées dans le bac à sable** · [x] conforme doc Debian · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
