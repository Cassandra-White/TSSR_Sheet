# CP6-03 — Script PowerShell : administration (services, journaux, rapports)

**Objectif** : automatiser des tâches d'exploitation courantes : contrôle des services, extraction des journaux, génération de rapports.

**Rattachement REAC** : CP6 « Automatiser des tâches à l'aide de scripts » — savoir-faire : automatiser l'exploitation Windows.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Windows Server 2025 / Windows 11, PowerShell 5.1 ou 7.x, droits administrateur.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows / PowerShell | 2025 / 5.1 + 7.x | 24/07/2026 |
| Exemples | **à tester en lab** | 24/07/2026 |

---

## Procédure — CLI

### Services

```powershell
# Services en démarrage automatique mais actuellement arrêtés (anomalie)
Get-Service | Where-Object { $_.Status -eq 'Stopped' -and $_.StartType -eq 'Automatic' }

# Relancer un service
Restart-Service -Name Spooler
```

### Journaux (Get-WinEvent — moderne)

```powershell
# Erreurs et alertes du journal Système des dernières 24 h
Get-WinEvent -FilterHashtable @{ LogName='System'; Level=1,2; StartTime=(Get-Date).AddDays(-1) } |
  Select-Object TimeCreated, Id, LevelDisplayName, ProviderName -First 20
```

### Rapports (export CSV / HTML)

```powershell
Get-Service | Select-Object Name, Status, StartType |
  Export-Csv .\services.csv -NoTypeInformation -Encoding UTF8

Get-CimInstance Win32_LogicalDisk -Filter "DriveType=3" |
  Select-Object DeviceID,
    @{N='Libre(Go)';E={[math]::Round($_.FreeSpace/1GB,1)}},
    @{N='Total(Go)';E={[math]::Round($_.Size/1GB,1)}} |
  ConvertTo-Html -Title "Disques" | Out-File .\disques.html
```

---

## Vérification (comment savoir que ça marche)

```powershell
Test-Path .\services.csv          # le rapport est généré
Get-WinEvent -MaxEvents 5 -LogName System   # les journaux sont accessibles
Get-Service -Name Spooler         # le service ciblé est dans l'état attendu
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| `No events were found` | Filtre trop strict | Élargir `Level`/`StartTime` |
| `Accès refusé` aux journaux | Droits insuffisants | Exécuter en administrateur |
| Accents cassés dans le CSV | Encodage | `Export-Csv -Encoding UTF8` |
| Colonnes vides | Mauvaise propriété | `Get-Member` pour trouver le bon nom |

## Sécurité et bonnes pratiques

- Exécuter en **lecture seule** quand c'est possible (rapports) ; agir avec parcimonie sur les services.
- **Ne pas exposer** les rapports (données sensibles) ; les stocker de façon protégée.
- Coupler avec le **Planificateur** (**CP6-04**) pour des rapports périodiques.

## ⚠️ À ne pas confondre / obsolète

- **`Get-EventLog`** et **`Get-WmiObject`** sont **dépréciés** → **`Get-WinEvent`** et **`Get-CimInstance`**.
- `Export-Csv` nécessite **`-NoTypeInformation`** (inutile de garder l'en-tête de type) et **`-Encoding UTF8`** pour les accents.
- Les **propriétés calculées** (`@{N=…;E={…}}`) servent à mettre en forme les rapports.

---

## Sources (doc officielle)

- [Get-WinEvent — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent) — consulté le 24/07/2026
- [Get-CimInstance — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/cimcmdlets/get-ciminstance) — consulté le 24/07/2026
- [Export-Csv — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/export-csv) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées · [x] rien d'obsolète (Get-WinEvent/CimInstance) · [x] exemples à tester en lab · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
