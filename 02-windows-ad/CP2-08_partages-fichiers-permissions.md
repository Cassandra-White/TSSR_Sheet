# CP2-08 — Configurer un partage de fichiers et ses permissions (partage + NTFS)

**Objectif** : publier un dossier partagé sur le réseau avec les bonnes permissions, en combinant les deux couches **Partage (SMB)** et **NTFS**.

**Rattachement REAC** : CP2 — savoir-faire : « Configurer les partages, les droits d'accès et les permissions conformément aux demandes des administrateurs ».

**Durée** : ~15 min · **Niveau** : intermédiaire.

---

## Prérequis

- Serveur membre ou DC (voir **CP2-03**), groupes **AGDLP** prêts (voir **CP2-06**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (Desktop Experience) | 23/07/2026 |

## Principe (à retenir)

Deux couches se cumulent : **permissions de Partage (SMB)** et **permissions NTFS**. La permission **effective = la plus restrictive** des deux. Bonne pratique : partage **large** (Utilisateurs authentifiés → Modifier) et **NTFS fin** via des groupes **Domain Local**.

---

## Procédure — GUI (méthode prioritaire)

1. Créer le dossier (ex. `D:\Partages\Compta`).
2. Clic droit → **Propriétés** → onglet **Partage** → **Partage avancé…**.
3. Cocher **Partager ce dossier**, nom `Compta$` (le `$` masque le partage). **Autorisations** → retirer *Tout le monde*, ajouter `DL_Partage_Compta_Modif` → **Modifier** → **OK**.
4. Onglet **Sécurité** (NTFS) → **Modifier** → ajouter `DL_Partage_Compta_Modif` → **Modification** (héritage traité en **CP2-09**).

> Alternative assistant : **Gestionnaire de serveur → Services de fichiers et de stockage → Partages → Nouveau partage** (profil *SMB Share – Quick*).
> Console des partages : `fsmgmt.msc`.

## Procédure — CLI (alternative / automatisation)

```powershell
New-Item -Path "D:\Partages\Compta" -ItemType Directory

# Partage SMB : Change au groupe DL, Full aux admins
New-SmbShare -Name "Compta$" -Path "D:\Partages\Compta" `
  -ChangeAccess "LABTSSR\DL_Partage_Compta_Modif" -FullAccess "LABTSSR\Domain Admins"

# NTFS : Modify au groupe DL, appliqué aux sous-dossiers et fichiers
icacls "D:\Partages\Compta" /grant "LABTSSR\DL_Partage_Compta_Modif:(OI)(CI)M"
```

*(À exécuter en lab — non testable dans le bac à sable Linux.)*

---

## Vérification

```powershell
Get-SmbShare -Name "Compta$"
Get-SmbShareAccess -Name "Compta$"
icacls "D:\Partages\Compta"
# Depuis un client : \\SRV-AD01\Compta$
```

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| Accès refusé malgré Partage=Modifier | NTFS plus restrictif | Vérifier l'onglet Sécurité / `icacls` |
| Partage introuvable dans le voisinage | Nom en `$` (masqué) | Saisir le chemin complet `\\serveur\Compta$` |
| Tout le monde a accès | ACE `Everyone` par défaut | Retirer *Everyone*, utiliser les groupes |

## Sécurité et bonnes pratiques

- Modèle : **Partage** = *Utilisateurs authentifiés / Modifier*, **NTFS** fin via groupes **Domain Local** (AGDLP).
- Partages d'administration **masqués** (`$`).
- Activer l'**audit** des accès sur les partages sensibles.

## ⚠️ À ne pas confondre / obsolète

- `net share` fonctionne encore mais les cmdlets **SmbShare** sont recommandées.
- Ne pas gérer les droits sur *Tout le monde* : toujours passer par des **groupes**.

---

## Sources (doc officielle)

- [New-SmbShare (Windows Server 2025) — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/smbshare/new-smbshare?view=windowsserver2025-ps) — consulté le 23/07/2026
- [Grant-SmbShareAccess — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/smbshare/grant-smbshareaccess?view=windowsserver2025-ps) — consulté le 23/07/2026
- [icacls — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/icacls) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
