# STO-13 — Gérer les quotas de disque (Windows FSRM / Linux quota)

**Objectif** : limiter la consommation d'espace par **dossier** (Windows FSRM) ou par **utilisateur/groupe** (Linux quota), avec alertes, pour éviter la saturation.

**Rattachement REAC** : CP2 / CP3 / STO — savoir-faire : maîtriser l'usage du stockage.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Windows : **WS 2025** avec un volume de partage. Linux : **Debian 13**, paquet **quota**.
- Droits **administrateur/root**.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows | **FSRM** (File Server Resource Manager, WS 2025) | 24/07/2026 |
| Linux | **quota** (Debian 13) | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **Quota souple (soft)** = seuil d'alerte avec **délai de grâce** ; **quota dur (hard)** = blocage strict. On limite en **espace** (blocs) et parfois en **nombre de fichiers** (inodes).

---

## Procédure — GUI (Windows FSRM)

1. **Server Manager ▸ Add Roles ▸ File and Storage Services ▸ File Server Resource Manager**.
2. Console **`fsrm.msc` ▸ Quota Management ▸ Quotas ▸ Create Quota** : cibler un **dossier**, choisir un **modèle** (ex. *200 MB Limit*), définir les **seuils** (e-mail à 85 %, blocage à 100 %).
3. **File Screening** (optionnel) : **bloquer** des types de fichiers (ex. `.mp3`, `.exe`).
4. **Storage Reports** : rapports d'usage planifiés.

### CLI (PowerShell)

```powershell
Install-WindowsFeature FS-Resource-Manager -IncludeManagementTools
New-FsrmQuota -Path "D:\Shares\Users" -Template "200 MB Limit"
Get-FsrmQuota -Path "D:\Shares\Users"
```

## Procédure — CLI (Linux quota)

```bash
apt install quota
# 1) Activer les quotas sur le FS : ajouter usrquota,grpquota dans /etc/fstab
#    ex.  UUID=...  /home  ext4  defaults,usrquota,grpquota  0 2
mount -o remount /home

# 2) Initialiser et activer
quotacheck -cug /home
quotaon /home

# 3) Définir les limites (soft/hard) et consulter
setquota -u alice 500M 550M 0 0 /home     # 500M soft, 550M hard, 0/0 inodes
edquota -u alice                          # édition interactive
repquota -s /home                         # rapport lisible
```

---

## Vérification (comment savoir que ça marche)

- **Linux** : `repquota -s /home` affiche l'usage et les limites ; un utilisateur qui dépasse le **hard** reçoit *« Disk quota exceeded »*.
- **Windows** : la console FSRM montre le **% d'utilisation** ; l'écriture est **bloquée** au-delà du quota dur.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Quota non appliqué (Linux) | Option fstab / quotacheck manquants | `usrquota` + `remount` + `quotacheck` + `quotaon` |
| Quota ignoré sur **XFS** | XFS ≠ ext4 | Monter avec `pquota`/`uquota`, utiliser `xfs_quota` |
| FSRM absent | Rôle non installé | Installer **FS-Resource-Manager** |
| E-mails d'alerte absents | SMTP FSRM non configuré | Renseigner le serveur SMTP dans FSRM |

## Sécurité et bonnes pratiques

- Les quotas **préviennent la saturation** (un utilisateur ne peut pas remplir tout le disque — mini déni de service).
- Combiner **seuils d'alerte** (soft) + **blocage** (hard) + **rapports**.
- **File Screening** (FSRM) pour bloquer les fichiers indésirables (ex. rançongiciels par extension).

## ⚠️ À ne pas confondre / obsolète

- **FSRM** (quota par **dossier**, granulaire) ≫ ancien **quota NTFS par volume/utilisateur**.
- **Soft** (alerte + grâce) ≠ **hard** (blocage immédiat).
- **ext4** (`quota`) vs **XFS** (`xfs_quota`, project quotas) : outils différents.

---

## Sources (doc officielle)

- [Microsoft Learn — File Server Resource Manager (FSRM)](https://learn.microsoft.com/en-us/windows-server/storage/fsrm/fsrm-overview) — consulté le 24/07/2026
- [Microsoft Learn — Module PowerShell FileServerResourceManager](https://learn.microsoft.com/en-us/powershell/module/fileserverresourcemanager/) — consulté le 24/07/2026
- [Debian — Quota (manuel `edquota`/`repquota`)](https://manpages.debian.org/bookworm/quota/edquota.8.en.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI (FSRM) puis CLI (quota) · [x] versions datées (WS 2025 / Debian 13) · [x] rien d'obsolète (FSRM vs quota NTFS) · [x] procédure **à tester en lab** · [x] conforme doc Microsoft/Debian · [x] vérification présente (`repquota`) · [x] sécurité (anti-saturation, file screening) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
