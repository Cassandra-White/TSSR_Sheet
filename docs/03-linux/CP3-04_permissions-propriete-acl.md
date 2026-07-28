# CP3-04 — Gérer les permissions et la propriété des fichiers (chmod/chown, ACL)

**Objectif** : lire et régler les permissions Unix (rwx), la propriété (`chown`/`chgrp`), le `umask`, les bits spéciaux (setgid/sticky), et poser des droits fins avec les **ACL POSIX**.

**Rattachement REAC** : CP3 « Exploiter des serveurs Linux » — savoir-faire : maîtriser les droits d'accès sur un serveur Linux.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Debian 13 (**CP3-01**), droits root/sudo. Paquet **`acl`** pour la partie ACL (`sudo apt install acl`).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian | 13.6 « trixie » | 24/07/2026 |
| `chmod` / `umask` / bits spéciaux | coreutils — **testés dans le bac à sable** | 24/07/2026 |
| `setfacl` / `getfacl` | acl — **à tester en lab** (paquet absent du bac à sable) | 24/07/2026 |

---

## Lire les permissions

```bash
ls -l fichier            # ex. -rw-r-----  → u=rw, g=r, o=(rien)
stat -c '%A %a %U:%G' fichier
```

Les 3 blocs `rwx` = **propriétaire / groupe / autres**. Valeurs octales : `r=4`, `w=2`, `x=1`.

## Procédure — CLI

### chmod (octal et symbolique)

```bash
chmod 640 fichier            # rw-r----- (testé : -rw-r-----)
chmod 750 dossier            # rwxr-x---
chmod u+x,g-w,o= fichier     # symbolique
chmod -R g+rX dossier        # X = x seulement sur dossiers / fichiers déjà exécutables
```

### Propriété : chown / chgrp

```bash
sudo chown alice fichier            # change le propriétaire
sudo chown alice:projet fichier     # propriétaire + groupe
sudo chgrp -R projet dossier        # change le groupe récursivement
```

### umask (droits par défaut)

```bash
umask                # afficher (souvent 022)
umask 027            # nouveaux fichiers 640, nouveaux dossiers 750 (testé)
```

### Bits spéciaux

```bash
# setgid sur un dossier partagé : les fichiers créés héritent du GROUPE du dossier
sudo chmod 2775 /srv/partage      # testé : drwxrwsr-x
# sticky bit : dans un dossier commun, seul le propriétaire supprime ses fichiers (cf. /tmp)
sudo chmod +t   /srv/depot        # testé : drwxrwsr-t (ici setgid + sticky)
# setuid (rare, sensible) : exécuter un binaire avec l'identité du propriétaire
```

### ACL POSIX (droits fins par utilisateur/groupe)

```bash
sudo apt install acl
setfacl -m u:bob:rwx fichier        # droit spécifique à un utilisateur
setfacl -m g:projet:rx fichier      # droit spécifique à un groupe
setfacl -d -m u:bob:rwx dossier     # ACL par DÉFAUT (héritée par le contenu créé)
getfacl fichier                     # lister les ACL
setfacl -x u:bob fichier            # retirer une entrée ; -b pour tout effacer
```

Un `+` en fin de ligne `ls -l` (`-rw-rwx---+`) signale la présence d'ACL.

---

## Vérification (sorties obtenues dans le bac à sable)

```
chmod 640         → -rw-r----- (640)
chmod u=rw,g=r,o= → -rw-r----- (640)
chmod 2775 (setgid) → drwxrwsr-x
chmod +t   (sticky) → drwxrwsr-t
umask 027 → nouveau fichier -rw-r----- (640)
```

Pour les ACL : `getfacl fichier` doit lister les lignes `user:bob:rwx`, `group:projet:r-x`, `mask::…`.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| « Permission denied » malgré des droits corrects | Dossier **parent** sans `x` (traversée) | Ajouter `x` sur le chemin ; vérifier le **mask** ACL |
| Les droits ACL semblent ignorés | `mask` ACL trop restrictif | `setfacl -m m:rwx fichier` (ajuster le masque) |
| Fichiers créés avec le mauvais groupe | Pas de **setgid** sur le dossier | `chmod 2775 dossier` |
| ACL « non prise en charge » | Système de fichiers monté sans `acl` | ext4 : support par défaut ; sinon remonter avec l'option `acl` |

## Sécurité et bonnes pratiques

- **Moindre privilège** : éviter absolument `chmod 777` ; privilégier groupe + setgid + ACL.
- Les binaires **setuid/setgid root** sont sensibles : en limiter le nombre, les auditer.
- **sticky bit** sur tout répertoire partagé en écriture (empêche la suppression des fichiers d'autrui).

## ⚠️ À ne pas confondre / obsolète

- Le bit **`x` sur un dossier** = droit de **traverser** (y accéder), pas d'« exécuter ».
- **`X` majuscule** (`chmod g+rX`) ≠ `x` minuscule : `X` n'ajoute `x` que sur les dossiers ou fichiers déjà exécutables.
- `chmod 777` « pour que ça marche » est une mauvaise pratique de sécurité, pas une solution.

---

## Sources (doc officielle)

- [Debian Wiki — Permissions](https://wiki.debian.org/Permissions) — consulté le 24/07/2026
- [chmod(1) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/coreutils/chmod.1.en.html) — consulté le 24/07/2026
- [setfacl(1) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/acl/setfacl.1.en.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées · [x] rien d'obsolète (X vs x, anti-777) · [x] **chmod/umask/bits testés** ; ACL marquée « à tester en lab » · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
