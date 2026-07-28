# CP8-04 — Sauvegarder et restaurer un serveur Linux (rsync/tar + planification)

**Objectif** : sauvegarder un serveur Linux avec **rsync** (snapshots incrémentaux par liens durs) et **tar** (archives datées), **planifier** les jobs et **restaurer**.

**Rattachement REAC** : CP8 « Sauvegardes et restaurations des éléments de l'infrastructure » — savoir-faire : sauvegarder/restaurer un serveur Linux.

**Durée** : ~30 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un serveur **Debian 13** (**CP3-01**), accès **root**.
- Un espace de destination (disque dédié / NAS / serveur distant en **SSH** — **CP3-09**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Outils | **rsync**, **tar** (Debian 13) | 24/07/2026 |
| Snapshots `--link-dest` + restauration | **testés en bac à sable** | 24/07/2026 |

> **Deux approches complémentaires** : **rsync** garde une **copie navigable** à jour (miroir) et, avec `--link-dest`, un **historique de snapshots** peu coûteux (liens durs). **tar** produit des **archives datées** compressées, faciles à externaliser.

---

## Procédure — CLI

### A. Miroir simple (image fidèle de la source)

```bash
rsync -aAXH --delete /data/ /backup/miroir/   # -a: perms/dates ; -AX: ACL/xattr ; -H: liens durs
```

> ⚠️ `--delete` **propage aussi les suppressions** : un miroir n'est **pas** un historique (un fichier effacé/chiffré par un rançongiciel disparaît aussi de la copie).

### B. Snapshots incrémentaux datés (liens durs — recommandé)

```bash
DEST=/backup/snapshots
LAST=$(ls -1d $DEST/*/ 2>/dev/null | tail -1)     # dernier snapshot
NOW=$DEST/$(date +%F_%H%M)

rsync -aAXH --delete ${LAST:+--link-dest="$LAST"} /data/ "$NOW"/
```

Chaque dossier daté **ressemble à une sauvegarde complète**, mais les fichiers **inchangés sont partagés** (liens durs) → très peu d'espace.

### C. Archive datée (tar) + externalisation SSH

```bash
tar --numeric-owner -czf /backup/data-$(date +%F).tar.gz -C /data .
rsync -aAXH -e ssh /backup/ sauveg@nas:/depot/serveur1/    # copie hors site (règle 3-2-1)
```

### D. Planification (cron ou systemd timer — voir **CP3-12/CP6-08**)

```cron
# /etc/cron.d/backup  → tous les jours à 02h30
30 2 * * *  root  /usr/local/sbin/backup.sh >> /var/log/backup.log 2>&1
```

### E. Restauration

```bash
# Depuis un snapshot (fichier ou arborescence)
rsync -aAXH /backup/snapshots/2026-07-24_0230/etc/hosts /etc/hosts
# Depuis une archive tar
tar -xzf /backup/data-2026-07-24.tar.gz -C /data
```

> **Testé en bac à sable** : après modification d'un fichier et ajout d'un autre, `--link-dest` a **dédupliqué l'inchangé** (même **inode** → une seule copie disque) et **recopié le modifié** (inode différent) ; l'archive `tar` restaurée redonne bien le contenu attendu (`rapport = doc v2`).

---

## Vérification (comment savoir que ça marche)

```bash
ls -l /backup/snapshots/                 # un dossier par exécution
du -sh /backup/snapshots/*               # les snapshots récents pèsent peu (liens durs)
tar -tzf /backup/data-2026-07-24.tar.gz | head   # l'archive est lisible
sha256sum fichier_source fichier_restaure        # intégrité après restauration
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Snapshots qui prennent trop de place | `--link-dest` **cross-filesystem** | Destination sur **le même système de fichiers** (liens durs) |
| Permissions/propriétaires perdus | Options manquantes | Utiliser `-aAXH` (+ `--numeric-owner` en tar) |
| Suppression propagée involontairement | `--delete` sur un miroir | Garder aussi des **snapshots** versionnés |
| Sauvegarde système incohérente | Fichiers ouverts | Exclure `/proc /sys /dev /run` ; envisager un **snapshot LVM** (**STO-06**) |

## Sécurité et bonnes pratiques

- **Externaliser** (SSH/NAS) et respecter **3-2-1** (**CP8-01**) ; garder une copie **hors-ligne**.
- **Chiffrer** les archives sorties du site (GPG) ; protéger la clé SSH de sauvegarde (compte dédié, droits minimaux).
- **Tester les restaurations** (**CP8-07**) et **journaliser** les jobs (sortie cron → log surveillé).
- Pour de la **déduplication + chiffrement** natifs, envisager **BorgBackup** ou **restic** (outils modernes complémentaires).

## ⚠️ À ne pas confondre / obsolète

- **Miroir** (`--delete`) ≠ **snapshots versionnés** (`--link-dest`) : le premier ne protège pas d'une suppression/chiffrement.
- **rsync** copie des **fichiers** ; il ne remplace pas une image **bare-metal** (pour ça : Veeam Agent **CP8-06** ou snapshot VM **CP8-10**).
- Les **liens durs** ne traversent pas les systèmes de fichiers : source de snapshots et cible doivent être sur le **même FS**.

---

## Sources (doc officielle)

- [rsync — Manuel (`--link-dest`)](https://download.samba.org/pub/rsync/rsync.1) — consulté le 24/07/2026
- [GNU tar — Manuel](https://www.gnu.org/software/tar/manual/tar.html) — consulté le 24/07/2026
- [BorgBackup — Documentation (dédup + chiffrement)](https://borgbackup.readthedocs.io/en/stable/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI (rsync/tar) · [x] daté 24/07/2026 · [x] rien d'obsolète (miroir vs snapshots, Borg/restic) · [x] **testé en bac à sable** (`--link-dest` dédup + restauration tar) · [x] conforme doc rsync/tar · [x] vérification présente · [x] sécurité (3-2-1, chiffrement, test) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
