# CP6-04 — Planifier un script sous Windows (Planificateur de tâches)

**Objectif** : exécuter automatiquement un script PowerShell à intervalle régulier via le Planificateur de tâches Windows.

**Rattachement REAC** : CP6 « Automatiser des tâches à l'aide de scripts » — savoir-faire : planifier l'exécution de scripts.

**Durée** : ~15 min · **Niveau** : intermédiaire.

---

## Prérequis

- Windows Server 2025 / Windows 11, un script `.ps1` prêt (ex. `C:\Scripts\backup.ps1`), droits administrateur.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows / Planificateur | 2025 — Task Scheduler | 24/07/2026 |
| Config | **à tester en lab** | 24/07/2026 |

---

## Procédure — GUI (Planificateur de tâches)

1. Ouvrir **Planificateur de tâches** (`taskschd.msc`) → **Créer une tâche** (pas « tâche de base »).
2. **Général** : nom ; cocher **Exécuter même si l'utilisateur n'est pas connecté** et **Exécuter avec les autorisations maximales**.
3. **Déclencheurs** → *Nouveau* : ex. **Quotidien**, 03:00.
4. **Actions** → *Nouveau* : Programme = `powershell.exe` ;
   Arguments = `-NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\backup.ps1"`.
5. **Conditions / Paramètres** : réveil, relance en cas d'échec, etc. → **OK**.

## Procédure — CLI (PowerShell)

```powershell
$action  = New-ScheduledTaskAction -Execute "powershell.exe" `
  -Argument '-NoProfile -ExecutionPolicy Bypass -File "C:\Scripts\backup.ps1"'
$trigger = New-ScheduledTaskTrigger -Daily -At 3am
$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" -LogonType ServiceAccount -RunLevel Highest

Register-ScheduledTask -TaskName "Sauvegarde quotidienne" `
  -Action $action -Trigger $trigger -Principal $principal
```

> Équivalent classique : `schtasks /create /tn "Sauvegarde" /tr "powershell -NoProfile -File C:\Scripts\backup.ps1" /sc daily /st 03:00 /ru SYSTEM`.

---

## Vérification (comment savoir que ça marche)

```powershell
Get-ScheduledTask -TaskName "Sauvegarde quotidienne"
Start-ScheduledTask -TaskName "Sauvegarde quotidienne"     # test immédiat
Get-ScheduledTaskInfo -TaskName "Sauvegarde quotidienne"   # LastRunTime / LastTaskResult (0 = OK)
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| La tâche ne se lance pas | Compte/option « si non connecté » | Vérifier le principal et cocher l'option |
| Script bloqué (policy) | ExecutionPolicy | Ajouter `-ExecutionPolicy Bypass` dans l'action |
| Chemin non trouvé | Espaces non protégés | Guillemets autour du `-File "…"` |
| `LastTaskResult` ≠ 0 | Erreur dans le script | Consulter le code retour / les journaux |

## Sécurité et bonnes pratiques

- Exécuter sous un **compte de service dédié** (moindre privilège), pas un compte admin personnel.
- Limiter `-ExecutionPolicy Bypass` **à la tâche** (pas au système entier).
- Protéger le script (droits NTFS) et **journaliser** son exécution.

## ⚠️ À ne pas confondre / obsolète

- **`Register-ScheduledTask`** (PowerShell, moderne) vs **`schtasks`** (classique, toujours valide).
- L'équivalent **Linux** = **cron / systemd timer** (**CP3-12**, **CP6-08**).
- « Créer une tâche » (complète) ≠ « tâche de base » (assistant limité).

---

## Sources (doc officielle)

- [Register-ScheduledTask — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/scheduledtasks/register-scheduledtask) — consulté le 24/07/2026
- [Task Scheduler — Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/taskschd/task-scheduler-start-page) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète (Register-ScheduledTask) · [x] config à tester en lab · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
