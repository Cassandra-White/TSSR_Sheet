# CP6-02 — Script PowerShell : créer des utilisateurs AD en masse (import CSV)

**Objectif** : créer automatiquement plusieurs comptes Active Directory à partir d'un fichier CSV.

**Rattachement REAC** : CP6 « Automatiser des tâches à l'aide de scripts » — savoir-faire : automatiser l'administration AD.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un contrôleur de domaine (**CP2-03**), le module **ActiveDirectory** (`Install-WindowsFeature RSAT-AD-PowerShell`), les **OU** cibles créées (**CP2-04**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server / PowerShell | 2025 / 5.1 + 7.x | 24/07/2026 |
| Logique **Import-Csv → champs AD** | **testée** (portable, bac à sable) | 24/07/2026 |

---

## Fichier CSV d'entrée (`utilisateurs.csv`)

```text
Prenom,Nom,Service
Bob,Martin,Compta
Alice,Durand,IT
Jean,Petit,Compta
```

## Procédure — CLI (script PowerShell)

```powershell
Import-Module ActiveDirectory
$motdepasse = ConvertTo-SecureString "Temp!Passw0rd" -AsPlainText -Force   # test uniquement

Import-Csv .\utilisateurs.csv | ForEach-Object {
    $sam = ($_.Prenom.Substring(0,1) + $_.Nom).ToLower()
    $params = @{
        Name                  = "$($_.Prenom) $($_.Nom)"
        GivenName             = $_.Prenom
        Surname               = $_.Nom
        SamAccountName        = $sam
        UserPrincipalName     = "$sam@lab.local"
        DisplayName           = "$($_.Prenom) $($_.Nom)"
        Path                  = "OU=$($_.Service),OU=Utilisateurs,DC=lab,DC=local"
        AccountPassword       = $motdepasse
        Enabled               = $true
        ChangePasswordAtLogon = $true
    }
    New-ADUser @params
    Write-Output "Créé : $sam"
}
```

> Le `@params` = **splatting** (regroupe les paramètres, plus lisible).

---

## Vérification (logique confirmée dans le bac à sable)

Champs dérivés du CSV (miroir exact du script) :

```
Bob Martin    | sam=bmartin | upn=bmartin@lab.local | OU=Compta,OU=Utilisateurs,DC=lab,DC=local
Alice Durand  | sam=adurand | upn=adurand@lab.local | OU=IT,OU=Utilisateurs,DC=lab,DC=local
Jean Petit    | sam=jpetit  | upn=jpetit@lab.local  | OU=Compta,OU=Utilisateurs,DC=lab,DC=local
```

En lab :

```powershell
Get-ADUser -Filter * -SearchBase "OU=Compta,OU=Utilisateurs,DC=lab,DC=local" |
  Select-Object Name, SamAccountName, Enabled
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| `directory object not found` | OU inexistante | Créer l'OU d'abord (**CP2-04**) |
| Mot de passe refusé | Stratégie de complexité | Respecter la politique (**CP2-17**) |
| `already exists` | `SamAccountName` en doublon | Ajouter une règle de dédoublonnage |
| `New-ADUser` inconnu | Module absent | `Install-WindowsFeature RSAT-AD-PowerShell` |

## Sécurité et bonnes pratiques

- **Mot de passe temporaire + `ChangePasswordAtLogon`** (l'utilisateur le change au 1er accès).
- **Ne pas** laisser de mot de passe en clair dans le CSV/script en production (saisie interactive ou coffre-fort).
- Déléguer des **droits minimaux** ; journaliser les créations.

## ⚠️ À ne pas confondre / obsolète

- `ConvertTo-SecureString -AsPlainText` = **pratique de test** : à éviter en production (mot de passe en clair).
- Le **splatting** (`@params`) est préférable à une ligne `New-ADUser` interminable.
- `csvde`/`ldifde` sont des alternatives **anciennes** : `New-ADUser` est la voie moderne.

---

## Sources (doc officielle)

- [New-ADUser — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/activedirectory/new-aduser) — consulté le 24/07/2026
- [Import-Csv — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/import-csv) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées · [x] rien d'obsolète (New-ADUser, splatting) · [x] **logique CSV testée** ; création à valider en lab · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
