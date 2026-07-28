# CP3-17 — Monter un partage réseau côté client (CIFS/NFS) + fstab

**Objectif** : monter, depuis un client Debian, un partage **Windows/Samba (CIFS)** et un partage **NFS**, de façon manuelle puis **permanente** via `/etc/fstab`.

**Rattachement REAC** : CP3 « Exploiter des serveurs Linux » — savoir-faire : raccorder un serveur Linux à des partages réseau.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Debian 13 (**CP3-01**), droits root/sudo.
- Un partage Samba (**CP3-07**) et/ou NFS (**CP3-08**) accessibles.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian | 13.6 « trixie » | 24/07/2026 |
| `mount.cifs` / `mount -t nfs` / `fstab` | **à tester en lab** (paquets absents du bac à sable) | 24/07/2026 |

---

## Procédure — CLI

### Partage Windows/Samba (CIFS)

```bash
sudo apt install cifs-utils
sudo mkdir -p /mnt/commun

# Fichier d'identifiants (jamais le mot de passe en clair dans fstab)
sudo tee /root/.smbcreds >/dev/null <<'EOF'
username=bob
password=MotDePasseFort
domain=WORKGROUP
EOF
sudo chmod 600 /root/.smbcreds

# Montage manuel (test)
sudo mount -t cifs //192.168.10.20/Commun /mnt/commun \
  -o credentials=/root/.smbcreds,vers=3.0,uid=1000,gid=1000
```

### Partage NFS

```bash
sudo apt install nfs-common
sudo mkdir -p /mnt/data
sudo mount -t nfs 192.168.10.5:/srv/nfs/data /mnt/data
```

### Rendre permanent — `/etc/fstab`

```text
//192.168.10.20/Commun  /mnt/commun  cifs  credentials=/root/.smbcreds,vers=3.0,uid=1000,gid=1000,_netdev  0  0
192.168.10.5:/srv/nfs/data  /mnt/data  nfs  defaults,_netdev  0  0
```

```bash
sudo systemctl daemon-reload
sudo mount -a          # applique et teste les lignes fstab
```

> Pour ne **pas bloquer le démarrage** si le serveur est absent, monter à la demande :
> `x-systemd.automount,noauto` (en plus de `_netdev`).

---

## Vérification (comment savoir que ça marche)

```bash
mount | grep -E 'cifs|nfs'      # les partages apparaissent montés
df -h /mnt/commun /mnt/data
touch /mnt/commun/test && echo OK   # test d'écriture (selon droits)
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| `mount error(112)` / refus CIFS | Version SMB inadaptée | Préciser `vers=3.0` (ou `3.1.1`) |
| Démarrage bloqué si serveur down | Montage réseau synchrone | Ajouter `_netdev` + `x-systemd.automount,noauto` |
| Droits incorrects sur CIFS | Mapping absent | Options `uid=`,`gid=`,`file_mode=`,`dir_mode=` |
| NFS ne monte pas | `nfs-common` absent / pare-feu | Installer `nfs-common` ; ouvrir le port 2049 |

## Sécurité et bonnes pratiques

- **Fichier `credentials` en `chmod 600`** : jamais de mot de passe en clair dans `/etc/fstab`.
- Imposer **SMB3** (`vers=3.0`+) ; refuser SMB1.
- `_netdev` sur tout montage réseau pour attendre la connectivité.

## ⚠️ À ne pas confondre / obsolète

- Ne **jamais** écrire le mot de passe en clair dans `fstab` → fichier **`credentials`**.
- **`vers=1.0` (SMB1) est proscrit** (vulnérabilités) → `vers=3.0` minimum.
- `_netdev` (et de préférence `x-systemd.automount`) sont **indispensables** pour les montages réseau, sinon risque de blocage au boot.

---

## Sources (doc officielle)

- [mount.cifs(8) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/cifs-utils/mount.cifs.8.en.html) — consulté le 24/07/2026
- [nfs(5) — options de montage — Debian Manpages (trixie)](https://manpages.debian.org/trixie/nfs-common/nfs.5.en.html) — consulté le 24/07/2026
- [Debian Wiki — fstab](https://wiki.debian.org/fstab) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées · [x] rien d'obsolète (SMB3, credentials, _netdev) · [x] à tester en lab · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
