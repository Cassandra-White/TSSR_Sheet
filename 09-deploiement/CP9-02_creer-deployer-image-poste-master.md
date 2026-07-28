# CP9-02 — Créer et déployer une image de poste (master)

**Objectif** : préparer une machine de **référence**, la **généraliser** (Sysprep), **capturer** une image WIM (DISM), puis la **déployer** sur d'autres postes.

**Rattachement REAC** : CP9 « Exploiter et maintenir les services de déploiement des postes » — savoir-faire : créer et déployer une image système.

**Durée** : ~40 min · **Niveau** : avancé.

---

## Prérequis

- Une machine (ou VM) de **référence** Windows 11 24H2 propre.
- L'**ADK** Windows (fournit **WinPE** et **DISM**), une clé/partage pour stocker le WIM.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| OS de référence | **Windows 11 24H2** | 24/07/2026 |
| Outils | **Sysprep**, **DISM**, **WinPE** (ADK) | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **Master = image de référence** capturée après personnalisation puis **généralisée** (Sysprep retire l'identité machine : **SID**, nom, pilotes spécifiques) pour être **déployable en série**.

---

## Procédure — étapes

### 1. Construire la machine de référence

1. Installer Windows 11 24H2, entrer en **mode Audit** (`Ctrl+Shift+F3` à l'OOBE) pour personnaliser sous le compte intégré.
2. Installer applications et réglages communs.

> ⚠️ **Construire SANS accès Internet** : sous 24H2, les mises à jour automatiques du **Store** et **Copilot** **cassent Sysprep**. Couper le réseau pendant build + capture.

### 2. Généraliser (Sysprep) — GUI ou CLI

- **GUI** : `C:\Windows\System32\Sysprep\sysprep.exe` → **OOBE** + cocher **Généraliser** + **Arrêter le système**.
- **CLI** :

```cmd
sysprep /generalize /oobe /shutdown /unattend:C:\unattend.xml
```

### 3. Capturer l'image (DISM, depuis WinPE)

```cmd
Dism /Capture-Image /ImageFile:N:\images\Win11-24H2.wim /CaptureDir:C:\ ^
     /Name:"Win11 24H2 Master" /Compress:max
```

### 4. Déployer l'image (DISM apply)

```cmd
Dism /Apply-Image /ImageFile:N:\images\Win11-24H2.wim /Index:1 /ApplyDir:W:\
bcdboot W:\Windows                 # rendre le volume démarrable
```

> Le déploiement en série se fait ensuite par **PXE** (**CP9-03**), clé USB, ou une solution moderne (**ConfigMgr / Autopilot**).

---

## Vérification (comment savoir que ça marche)

```cmd
Dism /Get-ImageInfo /ImageFile:N:\images\Win11-24H2.wim     # l'image et son index existent
```

- Un poste déployé démarre sur l'**OOBE** (preuve de généralisation), avec un **SID unique** et le nom demandé.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Sysprep échoue | **Appx** provisionnées mises à jour (Store/Copilot) | Build **hors ligne** ; retirer l'appx fautive (log `Panther\setuperr.log`) |
| « nombre de rearm dépassé » | Sysprep généralisé trop de fois | Repartir d'une base ou `skiprearm` |
| BitLocker cassé après Sysprep | Bug 24H2 connu | **Désactiver BitLocker** avant la généralisation |
| Image ne démarre pas | Amorçage non créé | `bcdboot W:\Windows` après apply |

## Sécurité et bonnes pratiques

- **Généraliser** est obligatoire : déployer une image **non** généralisée (SID/nom dupliqués) casse le domaine et WSUS (**CP9-01**).
- **Retirer les données sensibles** (comptes, secrets, historiques) avant capture.
- **Versionner** l'image *golden* et **documenter** son contenu (applis, version).
- Reconstruire régulièrement le master (patchs à jour) pour limiter le rattrapage post-déploiement.

## ⚠️ À ne pas confondre / obsolète

- **Capture** (`/Capture-Image`) ≠ **application** (`/Apply-Image`).
- **MDT (Microsoft Deployment Toolkit) est retiré depuis janvier 2026** : fonctionne encore mais **à éviter pour une nouvelle infra** → **ConfigMgr / Autopilot / WinPE**.
- Sysprep sous 24H2 **exige un build hors ligne** (sinon échec Store/Copilot).

---

## Sources (doc officielle)

- [Microsoft Learn — Sysprep (generalize) a Windows installation](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/sysprep--generalize--a-windows-installation) — consulté le 24/07/2026
- [Microsoft Learn — Capture images with DISM](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/capture-and-apply-an-image) — consulté le 24/07/2026
- [Microsoft Learn — Modern deployment (Autopilot/ConfigMgr)](https://learn.microsoft.com/en-us/windows/deployment/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] étapes GUI et CLI · [x] versions datées (Win 11 24H2) · [x] rien d'obsolète (MDT retiré, build hors ligne 24H2) · [x] procédure **à tester en lab** · [x] conforme doc Microsoft · [x] vérification présente (`Get-ImageInfo`) · [x] sécurité (généralisation, données) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
