# CP3-07 — Configurer un partage de fichiers Samba (interopérabilité Windows)

**Objectif** : partager un dossier depuis un serveur Debian et y accéder depuis un poste Windows 11 (protocole SMB).

**Rattachement REAC** : CP3 « Exploiter des serveurs Linux » (+ CP2, interopérabilité) — savoir-faire : mettre à disposition un partage de fichiers multiplateforme.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Debian 13 (**CP3-01**), droits root/sudo, IP fixe (**CP3-02**).
- Un poste **Windows 11** sur le même réseau.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian / Samba | 13.6 « trixie » — Samba 4.x | 24/07/2026 |
| `smb.conf` / `testparm` / accès Windows | **à tester en lab** (paquets absents du bac à sable) | 24/07/2026 |

> Configuration du serveur en **CLI** (`smb.conf`) ; consommation du partage en **GUI** côté Windows.

---

## Procédure — CLI (configuration du serveur)

```bash
sudo apt install samba
sudo mkdir -p /srv/partage/commun
sudo groupadd sambausers
sudo chgrp -R sambausers /srv/partage/commun
sudo chmod 2770 /srv/partage/commun        # setgid : héritage du groupe
```

Éditer `/etc/samba/smb.conf` :

```ini
[global]
   workgroup = WORKGROUP
   server string = Serveur de fichiers
   server min protocol = SMB2_10          # SMB1 interdit
   server max protocol = SMB3
   map to guest = never

[Commun]
   path = /srv/partage/commun
   browseable = yes
   read only = no
   valid users = @sambausers
   create mask = 0660
   directory mask = 0770
```

Valider, créer le compte Samba, (re)démarrer :

```bash
testparm                       # vérifie la syntaxe de smb.conf
sudo useradd -M -s /usr/sbin/nologin -G sambausers bob   # compte système sans shell
sudo smbpasswd -a bob          # définit le mot de passe SMB
sudo smbpasswd -e bob          # active le compte
sudo systemctl enable --now smbd nmbd
sudo systemctl restart smbd nmbd
```

> Ouvrir le pare-feu pour Samba (ports **445**/139) si actif — voir **CP3-14**.

## Procédure — GUI (accès depuis Windows 11)

1. Ouvrir l'**Explorateur de fichiers** → barre d'adresse : `\\<IP-du-serveur>\Commun`.
2. Saisir les identifiants **Samba** (`bob` + mot de passe défini par `smbpasswd`).
3. Pour un accès permanent : **Ce PC → … → Connecter un lecteur réseau** → lettre + `\\<IP>\Commun` → *Se reconnecter à l'ouverture de session*.

---

## Vérification (comment savoir que ça marche)

```bash
testparm -s                         # config sans erreur
smbclient -L //localhost -U bob     # liste les partages visibles
sudo smbstatus                      # connexions clientes actives
```

Côté Windows : le dossier `Commun` s'ouvre et permet lecture/écriture selon les droits.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| « Accès refusé » | Compte non activé / droits Unix | `smbpasswd -e bob` ; vérifier `valid users` et droits sur le dossier |
| Partage invisible depuis Windows | Pare-feu / `browseable = no` | Ouvrir le port **445** ; taper directement `\\IP\Commun` |
| Windows refuse de se connecter | Ancien client réclamant SMB1 | **Ne pas réactiver SMB1** ; mettre le client à jour |
| Mauvais groupe sur les fichiers créés | Pas de setgid | `chmod 2770` sur le dossier partagé |

## Sécurité et bonnes pratiques

- **Protocole minimum SMB2** (idéalement SMB3) ; **jamais SMB1**.
- **Comptes Samba dédiés** sans shell (`/usr/sbin/nologin`), droits Unix restrictifs.
- Interdire l'accès invité (`map to guest = never`) ; ne jamais exposer Samba sur Internet.
- Chiffrement possible : `smb encrypt = required` pour les données sensibles.

## ⚠️ À ne pas confondre / obsolète

- **SMB1/NT1 est supprimé par défaut** depuis Samba 4.11 : **ne pas le réactiver** (vulnérabilités type *WannaCry*).
- L'ancien outil graphique **`system-config-samba` n'existe plus** : on édite `smb.conf` (ou via **Cockpit**).
- Un utilisateur Samba a **deux mots de passe distincts** : compte Unix (système) et compte SMB (`smbpasswd`).

---

## Sources (doc officielle)

- [Debian Wiki — Samba](https://wiki.debian.org/Samba) — consulté le 24/07/2026
- [smb.conf(5) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/samba/smb.conf.5.en.html) — consulté le 24/07/2026
- [smbpasswd(8) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/samba-common-bin/smbpasswd.8.en.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI (serveur) puis GUI (accès Windows) · [x] versions datées · [x] rien d'obsolète (SMB2+/anti-SMB1) · [x] config à tester en lab · [x] conforme doc Samba · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
