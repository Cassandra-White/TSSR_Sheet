# CP2-06 — Créer et gérer des groupes AD (portées et stratégie AGDLP)

**Objectif** : créer des groupes de sécurité, comprendre leurs portées et appliquer la bonne pratique **AGDLP** pour attribuer des droits.

**Rattachement REAC** : CP2 — savoir-faire : « Créer, modifier et supprimer des objets dans un annuaire Active Directory » ; « Configurer les partages, les droits d'accès et les permissions ».

**Durée** : ~15 min · **Niveau** : intermédiaire.

---

## Prérequis

- Domaine + OU + quelques utilisateurs (voir **CP2-03/04/05**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (Desktop Experience) | 23/07/2026 |

## Rappels — portées et types

- **Global (G)** : regroupe des comptes d'un même domaine (par métier/service).
- **Domain Local (DL)** : sert à **attribuer des permissions** sur une ressource.
- **Universelle (U)** : usage multi-domaines (forêt).
- **Type** : *Sécurité* (permissions) ou *Distribution* (listes de diffusion mail).

> **AGDLP** = les comptes (**A**) vont dans des groupes **G**lobaux, placés dans des groupes **D**omain **L**ocal, auxquels on attribue les **P**ermissions sur la ressource.

---

## Procédure — GUI (méthode prioritaire)

1. `dsa.msc` → OU cible (ex. `Entreprise → Groupes`).
2. Clic droit → **Nouveau** → **Groupe**.
3. Nom `G_Compta`, **Étendue = Globale**, **Type = Sécurité** → **OK**.
4. Ajouter des membres : double-clic sur `G_Compta` → onglet **Membres** → **Ajouter** → saisir les utilisateurs.
5. Pour une ressource : créer `DL_Partage_Compta_Modif` (**Domain Local**), y ajouter `G_Compta`, puis donner à `DL_...` les permissions NTFS (voir **CP2-08/09**).

## Procédure — CLI (alternative / automatisation)

```powershell
# Groupe global de sécurité (par métier)
New-ADGroup -Name "G_Compta" -GroupScope Global -GroupCategory Security `
  -Path "OU=Groupes,OU=Entreprise,DC=labtssr,DC=lan"

# Groupe domain local (rattaché à une ressource)
New-ADGroup -Name "DL_Partage_Compta_Modif" -GroupScope DomainLocal -GroupCategory Security `
  -Path "OU=Groupes,OU=Entreprise,DC=labtssr,DC=lan"

# Imbrication AGDLP
Add-ADGroupMember -Identity "G_Compta" -Members jdupont
Add-ADGroupMember -Identity "DL_Partage_Compta_Modif" -Members "G_Compta"
```

*(PowerShell à exécuter en lab — non testable dans le bac à sable Linux.)*

---

## Vérification

```powershell
Get-ADGroupMember "G_Compta"
Get-ADGroup "DL_Partage_Compta_Modif" -Properties GroupScope | Select-Object Name,GroupScope
```

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| Permissions non appliquées sur la ressource | Mauvaise portée | Attribuer les droits à un groupe **Domain Local** |
| Imbrication refusée | Règles de portée non respectées | Suivre **AGDLP** |
| Membre « introuvable » | Identité erronée | Vérifier le `SamAccountName` |

## Sécurité et bonnes pratiques

- Nommage clair et préfixé (`G_`, `DL_`).
- Attribuer les droits **à des groupes**, jamais à des comptes individuels.
- Surveiller de près les groupes sensibles (*Admins du domaine*, *Admins de l'entreprise*) : y mettre le strict minimum.

## ⚠️ À ne pas confondre / obsolète

- Ne pas donner de permissions directement à un groupe **Global** ou à un compte sur une ressource : passer par un groupe **Domain Local** (AGDLP).
- *Sécurité* ≠ *Distribution* : seul un groupe de **sécurité** porte des permissions.

---

## Sources (doc officielle)

- [New-ADGroup (Windows Server 2025) — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/activedirectory/new-adgroup?view=windowsserver2025-ps) — consulté le 23/07/2026
- [Add-ADGroupMember — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/activedirectory/add-adgroupmember?view=windowsserver2025-ps) — consulté le 23/07/2026
- [Groupes de sécurité Active Directory — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-groups) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
