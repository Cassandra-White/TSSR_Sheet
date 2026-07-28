# CP8-02 — Sauvegarder et restaurer un serveur Windows (Windows Server Backup)

**Objectif** : installer **Windows Server Backup**, planifier une sauvegarde, puis restaurer un **fichier**, un **volume** ou le **serveur complet** (bare-metal).

**Rattachement REAC** : CP8 « Sauvegardes et restaurations des éléments de l'infrastructure » — savoir-faire : sauvegarder/restaurer un serveur Windows.

**Durée** : ~30 min · **Niveau** : intermédiaire.

---

## Prérequis

- **Windows Server 2025** avec droits **administrateur**.
- Une **cible de sauvegarde** : de préférence un **disque dédié** (idéal), sinon un volume séparé ou un **partage réseau**.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Serveur | **Windows Server 2025** | 24/07/2026 |
| Outil | **Windows Server Backup** (`wbadmin`) | 24/07/2026 |
| Procédure appliance | **à tester en lab** (Windows) | 24/07/2026 |

> ✅ *Sources reconfirmées le 24/07/2026 : WSB reste une **fonctionnalité** à ajouter (Server Manager/`Install-WindowsFeature`) sous Windows Server 2025 ; `-allCritical` = bare-metal, `start systemstaterecovery` pour l'état système.*

> **Sauvegarde de niveau bloc** : WSB crée des images **`.vhdx`**. L'option **`-allCritical`** inclut les volumes critiques + l'**état système** → indispensable pour une **restauration bare-metal**.

---

## Procédure — GUI

### Installer la fonctionnalité

1. **Server Manager ▸ Manage ▸ Add Roles and Features ▸ Features** → cocher **Windows Server Backup** → installer.

### Planifier une sauvegarde

2. Ouvrir **wbadmin.msc** (*Windows Server Backup*) ▸ **Local Backup ▸ Backup Schedule…**.
3. **Backup Configuration** : *Full server* (tout) ou *Custom* (choisir volumes / **Bare metal recovery** / **System State**).
4. **Specify Backup Time** : fréquence (ex. quotidienne 22:00).
5. **Destination** : **disque dédié** (recommandé, formaté et réservé), volume, ou partage réseau. **Confirmer**.

### Restaurer

6. **Local Backup ▸ Recover…** → choisir la **date/version** → type : **Files and Folders**, **Volumes**, **System State** ou **Applications** → suivre l'assistant.
7. **Bare-metal (serveur HS)** : démarrer sur le média d'installation → **Réparer l'ordinateur ▸ Dépannage ▸ Récupération de l'image système** → pointer la sauvegarde.

## Procédure — CLI (`wbadmin`, PowerShell admin)

```powershell
# Installer WSB
Install-WindowsFeature Windows-Server-Backup

# Sauvegarde unique "critique" (bare-metal) vers un disque cible E:
wbadmin start backup -backupTarget:E: -include:C: -allCritical -vssFull -quiet

# Sauvegarde planifiée quotidienne à 22h00 vers un disque dédié
wbadmin enable backup -addtarget:{DiskID} -schedule:22:00 -include:C: -allCritical -quiet

# Lister les sauvegardes disponibles
wbadmin get versions

# Restaurer un fichier/dossier depuis une version
wbadmin start recovery -version:MM/JJ/AAAA-HH:MM -itemtype:File -items:C:\Data -recursive -quiet
```

> `-vssFull` : sauvegarde VSS complète (marque les fichiers comme sauvegardés). Sur **partage réseau**, seule la **dernière** sauvegarde planifiée est conservée → préférer un **disque dédié** pour l'historique.

---

## Vérification (comment savoir que ça marche)

```powershell
wbadmin get versions          # la sauvegarde attendue apparaît (date, type, cible)
wbadmin get status            # état du job en cours
```

- Effectuer une **restauration de test** d'un fichier dans un dossier temporaire et comparer.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Échec « espace insuffisant » | Cible trop petite | Disque dédié plus grand ; réduire le périmètre |
| Erreur **VSS** | Fournisseurs VSS/espace shadow | `vssadmin list writers` ; libérer de l'espace |
| Une seule sauvegarde sur le partage | Limite du **backupTarget réseau** | Utiliser un **disque dédié** pour l'historique |
| BMR impossible | `-allCritical` oublié | Inclure **allCritical** (état système + volumes critiques) |

## Sécurité et bonnes pratiques

- **Disque dédié** en **rotation hors site** (règle **3-2-1** — **CP8-01**), **chiffré** (BitLocker) si transporté.
- Protéger l'accès au partage/dépôt de sauvegarde (comptes dédiés, moindre privilège).
- **Tester la restauration** régulièrement (**CP8-07**) et documenter le RTO réel.
- Sauvegarder aussi l'**état système/AD** sur les contrôleurs de domaine (**CP8-03**).

## ⚠️ À ne pas confondre / obsolète

- **NTBackup / `.bkf`** (hérité) → **Windows Server Backup / `.vhdx`** (niveau bloc).
- Sur **partage réseau**, WSB **n'empile pas** les versions (contrairement à un disque dédié).
- WSB = sauvegarde **locale** du serveur ; pour du multi-site/Cloud, la compléter (Azure Backup, Veeam — **CP8-06/CP8-11**).

---

## Sources (doc officielle / référence)

- [Microsoft Learn — wbadmin (command reference)](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/wbadmin) — consulté le 24/07/2026
- [Microsoft Learn — Back up system state and bare metal (DPM 2025)](https://learn.microsoft.com/en-us/system-center/dpm/back-up-system-state-and-bare-metal?view=sc-dpm-2025) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (`wbadmin`) · [x] version datée (WS 2025) · [x] rien d'obsolète (WSB vs NTBackup) · [x] procédure **à tester en lab** (Windows) · [x] **vérif web reconfirmée (24/07/2026)** · [x] vérification présente (`get versions`) · [x] sécurité (3-2-1, chiffrement, test) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
