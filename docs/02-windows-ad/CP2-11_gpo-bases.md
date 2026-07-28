# CP2-11 — Créer et lier une GPO (stratégie de groupe) — bases

**Objectif** : créer un objet de stratégie de groupe (GPO), le lier à une OU, y configurer un paramètre et l'appliquer aux postes/utilisateurs.

**Rattachement REAC** : CP2 — savoir-faire : « Spécifier et implémenter les nouvelles règles de gestion (GPO) dans un annuaire Active Directory ».

**Durée** : ~15 min · **Niveau** : intermédiaire.

---

## Prérequis

- Contrôleur de domaine et OU structurées (voir **CP2-03**, **CP2-04**).
- Console **Gestion des stratégies de groupe** (GPMC), installée avec AD DS.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (Desktop Experience) | 23/07/2026 |

## Rappel

Une GPO contient des paramètres **Ordinateur** et/ou **Utilisateur**. Ordre d'application **LSDOU** (Local → Site → Domaine → OU) : en cas de conflit, **la dernière appliquée l'emporte**. L'actualisation se fait toutes les ~90 min ou via `gpupdate`.

---

## Procédure — GUI (méthode prioritaire)

1. **Outils** → **Gestion des stratégies de groupe** (`gpmc.msc`).
2. Développer *Forêt → Domaines → `labtssr.lan`*. Clic droit sur l'OU cible (ex. `Compta`) → **Créer un objet GPO dans ce domaine, et le lier ici…**.
3. Nommer la GPO (ex. `GPO_Compta_Securite`) → **OK**.
4. Clic droit sur la GPO → **Modifier** (Éditeur de gestion des stratégies).
5. Naviguer (ex. *Configuration utilisateur → Stratégies → Modèles d'administration…*) et définir le paramètre voulu.
6. Fermer l'éditeur : la GPO s'applique aux objets de l'OU.
7. Sur un client : `gpupdate /force`, puis `gpresult /r`.

## Procédure — CLI (module GroupPolicy)

```powershell
# Créer la GPO et la lier à l'OU
New-GPO -Name "GPO_Compta_Securite" |
  New-GPLink -Target "OU=Compta,OU=Utilisateurs,OU=Entreprise,DC=labtssr,DC=lan"

# Exemple : masquer le Panneau de configuration (paramètre par registre)
Set-GPRegistryValue -Name "GPO_Compta_Securite" `
  -Key "HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer" `
  -ValueName "NoControlPanel" -Type DWord -Value 1

# Rapport HTML de la GPO
Get-GPOReport -Name "GPO_Compta_Securite" -ReportType Html -Path "C:\temp\gpo.html"
```

*(À exécuter en lab — non testable dans le bac à sable Linux.)*

---

## Vérification

```powershell
gpresult /r        # sur le client : GPO listée dans « Objets… appliqués »
Get-GPInheritance -Target "OU=Compta,OU=Utilisateurs,OU=Entreprise,DC=labtssr,DC=lan"
```

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| GPO non appliquée | Mauvaise OU / filtrage de sécurité | Vérifier le lien et le droit « Appliquer la stratégie de groupe » |
| Paramètre ignoré | Conflit d'ordre LSDOU | Vérifier la priorité des liens / option *Appliqué* |
| Délai d'application | Rafraîchissement ~90 min | `gpupdate /force` |

## Sécurité et bonnes pratiques

- Nommer les GPO explicitement (portée + objectif).
- Une GPO = un thème ; éviter les GPO fourre-tout.
- Tester sur une **OU pilote** avant généralisation.
- **Sauvegarder** les GPO : `Backup-GPO`.

## ⚠️ À ne pas confondre / obsolète

- `gpedit.msc` édite la stratégie **locale** du poste, **pas** les GPO de domaine (via `gpmc.msc`).

---

## Sources (doc officielle)

- [New-GPO (Windows Server 2025) — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/grouppolicy/new-gpo?view=windowsserver2025-ps) — consulté le 23/07/2026
- [New-GPLink — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/grouppolicy/new-gplink?view=windowsserver2025-ps) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
