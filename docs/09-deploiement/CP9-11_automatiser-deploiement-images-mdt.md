# CP9-11 — Automatiser le déploiement d'images avec MDT (Microsoft Deployment Toolkit)

**Objectif** : comprendre le fonctionnement de **MDT** (séquences de tâches, partage de déploiement) pour l'**existant**, et savoir qu'il est **retiré** → migrer vers les alternatives modernes.

**Rattachement REAC** : CP9 « Exploiter et maintenir les services de déploiement des postes » — savoir-faire : automatiser le déploiement d'images.

**Durée** : ~30 min · **Niveau** : avancé.

---

## Prérequis

- Un serveur Windows, l'**ADK** + **WinPE add-on**, MDT installé (existant).
- Un `install.wim` Windows, pilotes et applications à intégrer.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Outil | **MDT** (dernière build **8456**, 2019) | 24/07/2026 |
| Amorçage | ADK + **WinPE** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> 🛑 **STATUT CRITIQUE : MDT est officiellement RETIRÉ depuis le 6 janvier 2026.** Plus **aucune** mise à jour, correctif de sécurité ni support ; **aucune compatibilité** garantie avec les nouvelles versions de Windows. Les installations **existantes fonctionnent encore**, mais **ne pas bâtir de nouvelle infrastructure dessus**. Migrer vers **Windows Autopilot** (cloud) ou **ConfigMgr/SCCM OSD** (on-prem) ; pour le simple, **WinPE + DISM** (**CP9-02**).

---

## Procédure — GUI (pour maintenir l'existant)

1. **Deployment Workbench** → créer un **Deployment Share** (dépôt central).
2. **Importer** : un **Operating System** (`install.wim`), des **Applications**, des **Out-of-Box Drivers** (par modèle).
3. Créer une **Task Sequence** (*Standard Client*) : partitionnement → application de l'image → **injection de pilotes** → installation d'apps → **jonction au domaine**.
4. Régler **`Bootstrap.ini`** / **`CustomSettings.ini`** (paramètres, règles).
5. **Update Deployment Share** → génère l'image de boot **`LiteTouch.wim`** (WinPE).
6. Démarrer le client dessus (via **WDS/PXE** — **CP9-03** — ou USB) → assistant **LiteTouch (LTI)**.

## Procédure — CLI (PowerShell MDT)

```powershell
Import-Module "C:\Program Files\Microsoft Deployment Toolkit\bin\MicrosoftDeploymentToolkit.psd1"
New-PSDrive -Name DS001 -PSProvider MDTProvider -Root "D:\DeploymentShare"
Import-MDTOperatingSystem -Path "DS001:\Operating Systems" -SourcePath "E:\Win11" -DestinationFolder "Win11-24H2"
Update-MDTDeploymentShare -Path "DS001:"   # régénère LiteTouch.wim
```

> **LTI (LiteTouch)** = déploiement semi-assisté (MDT seul). **ZTI (ZeroTouch)** = entièrement automatisé (MDT **+ ConfigMgr**).

---

## Vérification (comment savoir que ça marche)

- La **Task Sequence** se déroule jusqu'au bureau (OS + pilotes + apps + domaine).
- Journaux MDT : `BDD.log` / `SMSTS.log` (`C:\...\SMSTSLog`) en cas d'échec.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Échec sur Windows récent | **MDT retiré** (pas de compat 24H2+) | Migrer vers **ConfigMgr/Autopilot** |
| Pas de pilotes | Drivers non importés/ciblés | Importer par **modèle** (sélection profil) |
| Boot PXE KO | WDS/`LiteTouch.wim` non ajouté | Recréer et ajouter le boot à WDS (**CP9-03**) |
| Version ADK/MDT incompatible | Mismatch | Aligner ADK + WinPE (attention : plus de mises à jour MDT) |

## Sécurité et bonnes pratiques

- **`CustomSettings.ini`/`Bootstrap.ini`** peuvent contenir des **identifiants de jonction** → **protéger** le partage, éviter les comptes privilégiés (rappel **CVE-2026-0386**, **CP9-03**).
- **Planifier la migration** hors de MDT (produit non supporté = risque de sécurité).
- Isoler le réseau de déploiement (**VLAN dédié**) et journaliser.

## ⚠️ À ne pas confondre / obsolète

- **MDT retiré (janv. 2026)** : à connaître pour l'existant, **pas** pour du neuf → **Autopilot / ConfigMgr / WinPE+DISM**.
- **LTI** (MDT seul, assisté) ≠ **ZTI** (MDT + ConfigMgr, automatisé).
- MDT **orchestre** WinPE/DISM/WDS ; il ne les remplace pas.

---

## Sources (doc officielle)

- [Microsoft Learn — MDT : avis de retrait immédiat](https://learn.microsoft.com/en-us/troubleshoot/mem/configmgr/mdt/mdt-retirement) — consulté le 24/07/2026
- [Microsoft Learn — MDT support lifecycle](https://learn.microsoft.com/en-us/troubleshoot/windows-server/setup-upgrade-and-drivers/deployment-toolkit-support) — consulté le 24/07/2026
- [Microsoft Learn — Déploiement moderne (Autopilot / ConfigMgr OSD)](https://learn.microsoft.com/en-us/windows/deployment/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (PowerShell MDT) · [x] version datée (build 8456) · [x] **RETRAIT signalé + alternatives** · [x] procédure **à tester en lab** · [x] conforme doc Microsoft · [x] vérification présente (logs) · [x] sécurité (identifiants, migration) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
