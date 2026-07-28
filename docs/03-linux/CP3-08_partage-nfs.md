# CP3-08 — Configurer un partage NFS

**Objectif** : partager un dossier entre serveurs Linux avec NFS (NFSv4) et le monter côté client de façon permanente.

**Rattachement REAC** : CP3 « Exploiter des serveurs Linux » — savoir-faire : mettre à disposition un partage de fichiers entre systèmes Linux.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Deux machines Debian 13 (**CP3-01**) : un **serveur** et un **client**, IP fixes (**CP3-02**), droits root/sudo.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian / NFS | 13.6 « trixie » — **NFSv4** | 24/07/2026 |
| `/etc/exports` / `exportfs` / `mount` | **à tester en lab** (paquets absents du bac à sable) | 24/07/2026 |

> Service Linux ↔ Linux, sans GUI : configuration en **CLI**.

---

## Procédure — CLI

### Côté serveur

```bash
sudo apt install nfs-kernel-server
sudo mkdir -p /srv/nfs/data
sudo chown nobody:nogroup /srv/nfs/data      # accès neutre (root_squash)
```

Déclarer l'export dans `/etc/exports` :

```text
/srv/nfs/data   192.168.10.0/24(rw,sync,no_subtree_check)
```

> ⚠️ **Syntaxe stricte** : **aucun espace** entre le réseau et la parenthèse.
> `192.168.10.0/24(rw,...)` ✅ — `192.168.10.0/24 (rw,...)` ❌ (donnerait un accès en lecture seule à tout le monde).

Appliquer et vérifier :

```bash
sudo exportfs -ra           # (re)charger /etc/exports
sudo exportfs -v            # lister les exports actifs
sudo systemctl enable --now nfs-server
```

> Ouvrir le pare-feu sur le port **2049** (NFSv4) si actif — voir **CP3-14**.

### Côté client

```bash
sudo apt install nfs-common
sudo mkdir -p /mnt/data
# Montage immédiat (test)
sudo mount -t nfs 192.168.10.5:/srv/nfs/data /mnt/data
```

Montage **permanent** dans `/etc/fstab` :

```text
192.168.10.5:/srv/nfs/data   /mnt/data   nfs   defaults,_netdev   0   0
```

```bash
sudo mount -a               # applique fstab (teste la ligne)
```

---

## Vérification (comment savoir que ça marche)

```bash
# Serveur
sudo exportfs -v
showmount -e localhost              # liste les exports proposés
# Client
showmount -e 192.168.10.5
mount | grep nfs                    # le partage est monté
df -h /mnt/data
touch /mnt/data/test && echo OK     # test d'écriture (selon droits)
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Partage en lecture seule inattendu | **Espace** avant la parenthèse dans `/etc/exports` | Corriger la syntaxe, `exportfs -ra` |
| `access denied by server` | Client hors du réseau autorisé | Vérifier l'IP/sous-réseau ; `exportfs -ra` |
| Le montage échoue | `nfs-common` absent / pare-feu 2049 | Installer `nfs-common` ; ouvrir le port |
| Écriture impossible en root | `root_squash` (défaut) mappe root → `nobody` | Utiliser des UID cohérents entre hôtes |

## Sécurité et bonnes pratiques

- **Restreindre par sous-réseau/hôte** dans `/etc/exports` (jamais `*`).
- **Conserver `root_squash`** (valeur par défaut) ; éviter `no_root_squash`.
- Privilégier **NFSv4** ; en environnement sensible, activer **Kerberos** (`sec=krb5`).
- Ne **jamais** exposer NFS directement sur Internet.

## ⚠️ À ne pas confondre / obsolète

- L'**espace avant la parenthèse** dans `/etc/exports` est le piège classique (change complètement les droits).
- **`no_root_squash`** est dangereux : à proscrire hors cas très précis.
- **NFSv2 est obsolète** ; préférer **NFSv4** (un seul port **2049**, plus simple à filtrer que NFSv3 et son `rpcbind`).

---

## Sources (doc officielle)

- [Debian Wiki — NFS/Server](https://wiki.debian.org/NFS/Server) — consulté le 24/07/2026
- [exports(5) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/nfs-kernel-server/exports.5.en.html) — consulté le 24/07/2026
- [Debian Handbook — NFS File Server](https://www.debian.org/doc/manuals/debian-handbook/sect.nfs-file-server.en.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées (NFSv4) · [x] rien d'obsolète (v4, anti no_root_squash) · [x] config à tester en lab · [x] conforme doc Debian · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
