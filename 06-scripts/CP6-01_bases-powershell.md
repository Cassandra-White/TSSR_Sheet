# CP6-01 — Bases de PowerShell (variables, structures, ExecutionPolicy)

**Objectif** : écrire ses premiers scripts PowerShell (variables, pipeline, structures de contrôle) et régler la politique d'exécution.

**Rattachement REAC** : CP6 « Automatiser des tâches à l'aide de scripts » — savoir-faire : maîtriser les bases du scripting Windows.

**Durée** : ~25 min · **Niveau** : débutant.

---

## Prérequis

- Windows Server 2025 / Windows 11, droits administrateur.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows PowerShell | **5.1** (intégré à Windows) | 24/07/2026 |
| PowerShell (multiplateforme) | **7.6 LTS** (7.x) | 24/07/2026 |
| Exemples | **à tester en lab** (pas de `pwsh` dans le bac à sable) | 24/07/2026 |

---

## Procédure — CLI

### Politique d'exécution (ExecutionPolicy)

```powershell
Get-ExecutionPolicy -List
# Recommandé : scripts locaux autorisés, scripts distants signés
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### Variables et types

```powershell
$nom  = "Bob"
$age  = [int]30
$vms  = @("srv-ad01","srv-web01")          # tableau
$user = @{ Nom = "Bob"; Role = "Admin" }    # table de hachage (hashtable)
$vms.Count           # 2
$user.Role           # Admin
```

### Découvrir (les 3 réflexes)

```powershell
Get-Command *service*        # trouver une cmdlet (Verbe-Nom)
Get-Help Get-Service -Examples
Get-Service | Get-Member     # propriétés/méthodes d'un objet
```

### Pipeline, conditions, boucles

```powershell
# Pipeline : filtrer puis sélectionner
Get-Service | Where-Object { $_.Status -eq 'Running' } | Select-Object Name, Status

# Condition
if ($age -ge 18) { "Majeur" } elseif ($age -gt 0) { "Mineur" } else { "?" }

# Boucle
foreach ($vm in $vms) { Write-Output "VM : $vm" }
```

### Fonction et exécution d'un script

```powershell
function Get-Uptime {
    (Get-Date) - (Get-CimInstance Win32_OperatingSystem).LastBootUpTime
}
# Enregistrer dans script.ps1 puis :  .\script.ps1
```

---

## Vérification (comment savoir que ça marche)

```powershell
$PSVersionTable.PSVersion     # version de PowerShell
Get-ExecutionPolicy -List     # la politique CurrentUser = RemoteSigned
.\script.ps1                  # le script s'exécute sans blocage
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| « l'exécution de scripts est désactivée » | ExecutionPolicy `Restricted` | `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser` |
| « terme non reconnu » | Cmdlet/module absent | `Get-Command`, importer le module |
| `$_` semble vide | Hors contexte pipeline | `$_` n'existe que dans `Where/ForEach-Object` |
| Accents cassés | Encodage | Enregistrer le `.ps1` en **UTF-8 (BOM)** |

## Sécurité et bonnes pratiques

- **RemoteSigned** plutôt que `Bypass`/`Unrestricted` ; **signer** les scripts en production.
- Ne **jamais exécuter** un script non fiable sans le lire.
- Activer la **journalisation** (transcription / *script block logging*) sur les serveurs.

## ⚠️ À ne pas confondre / obsolète

- **Windows PowerShell 5.1** (intégré, .NET Framework, `powershell.exe`) ≠ **PowerShell 7.x** (`pwsh`, multiplateforme, .NET moderne) : privilégier **7.x** pour les nouveaux scripts.
- **`Write-Host`** (affichage écran) ≠ **`Write-Output`** (envoie dans le **pipeline**).
- Les cmdlets suivent la forme **Verbe-Nom** (`Get-`, `Set-`, `New-`, `Remove-`).

---

## Sources (doc officielle)

- [Microsoft — about_Execution_Policies](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies) — consulté le 24/07/2026
- [Microsoft — Discover PowerShell](https://learn.microsoft.com/en-us/powershell/scripting/learn/ps101/01-getting-started) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées (5.1 + 7.6 LTS) · [x] rien d'obsolète (5.1 vs 7.x, Host/Output) · [x] exemples à tester en lab · [x] conforme doc · [x] vérification présente · [x] sécurité (ExecutionPolicy) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
