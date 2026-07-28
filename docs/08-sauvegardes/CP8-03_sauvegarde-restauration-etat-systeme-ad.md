# CP8-03 — Sauvegarder et restaurer l'état système / Active Directory

**Objectif** : sauvegarder l'**état système** d'un contrôleur de domaine et restaurer AD, en distinguant restauration **non autoritaire**, **autoritaire** et **corbeille AD**.

**Rattachement REAC** : CP8 « Sauvegardes et restaurations des éléments de l'infrastructure » — savoir-faire : sauvegarder/restaurer l'annuaire et l'état système.

**Durée** : ~35 min · **Niveau** : avancé.

---

## Prérequis

- Un **contrôleur de domaine** Windows Server 2025 (**CP2-03**), **Windows Server Backup** installé (**CP8-02**).
- Le **mot de passe DSRM** (défini à la promotion du DC) connu.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Contrôleur de domaine | **Windows Server 2025** — AD DS | 24/07/2026 |
| Outils | `wbadmin`, **DSRM**, `ntdsutil`, **Corbeille AD** | 24/07/2026 |
| Procédure appliance | **à tester en lab** (Windows/AD) | 24/07/2026 |

> ✅ *Sources reconfirmées le 24/07/2026 : la restauration **autoritaire** (`ntdsutil`) incrémente l'**USN de +100 000** pour réimposer les objets ; si la **Corbeille AD** est activée, `Restore-ADObject` suffit (ni DSRM ni restauration autoritaire).*

> **État système** d'un DC = registre, fichiers de démarrage, **base AD (NTDS.dit)** + **SYSVOL**, base de certificats… **Non autoritaire** : le DC restauré **rattrape** les autres par réplication. **Autoritaire** : les objets restaurés sont **réimposés** aux autres DC.

---

## Procédure — GUI

### Sauvegarder l'état système

1. **wbadmin.msc ▸ Backup Once/Schedule** → configuration **Custom** → cocher **System State** (ou **Bare metal recovery**) → cible **disque dédié** → lancer.

### Restaurer (DC en panne) — non autoritaire

2. Redémarrer le DC en **DSRM** (au boot, **F8** → *Directory Services Restore Mode*), ouvrir une session avec le **compte administrateur DSRM**.
3. **wbadmin.msc ▸ Recover** → sélectionner la version → **System State** → restaurer → redémarrer : le DC **réplique** les changements récents des autres DC.

### Restaurer un objet supprimé par erreur — Corbeille AD (recommandé)

4. **Active Directory Administrative Center ▸ Deleted Objects** → clic droit sur l'objet → **Restore** (aucun redémarrage, pas de DSRM).

## Procédure — CLI

```powershell
# --- Sauvegarde de l'état système ---
wbadmin start systemstatebackup -backupTarget:E: -quiet

# --- Restauration non autoritaire (en DSRM) ---
wbadmin get versions
wbadmin start systemstaterecovery -version:MM/JJ/AAAA-HH:MM -quiet   # puis redémarrage

# --- Restauration AUTORITAIRE d'une OU supprimée (en DSRM, après la non-auth, avant reboot) ---
ntdsutil
  activate instance ntds
  authoritative restore
    restore subtree "OU=Compta,DC=lab,DC=local"
    quit
  quit
```

**Corbeille AD** (moderne, sans DSRM — à activer une fois, irréversible) :

```powershell
Enable-ADOptionalFeature 'Recycle Bin Feature' -Scope ForestOrConfigurationSet `
  -Target "lab.local"
Get-ADObject -Filter 'Name -like "*Dupont*"' -IncludeDeletedObjects | Restore-ADObject
```

> **Réinitialiser le mot de passe DSRM** si oublié : `ntdsutil` → `set dsrm password` → `reset password on server null`.

---

## Vérification (comment savoir que ça marche)

```powershell
dcdiag /v                       # santé du DC
repadmin /replsummary           # réplication OK entre DC
Get-ADUser -Identity jdupont    # l'objet restauré est bien présent
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Mot de passe DSRM inconnu | Non noté à la promotion | `ntdsutil ▸ set dsrm password` |
| Sauvegarde refusée à la restauration | Plus vieille que la **durée de tombstone** (180 j) | Utiliser une sauvegarde **récente** |
| Objet resupprimé après réplication | Restauration **non** autoritaire | Marquer **autoritaire** (`ntdsutil`) ou **Corbeille AD** |
| Réplication cassée après un *rollback* | **USN rollback** (snapshot restauré) | Ne **jamais** restaurer un DC par snapshot ; refaire proprement |

## Sécurité et bonnes pratiques

- **Protéger le mot de passe DSRM** (coffre) et le tester périodiquement.
- Conserver au moins **une sauvegarde d'état système récente par DC** (< durée de tombstone).
- **Ne pas restaurer un DC par snapshot/clonage** (risque d'**USN rollback**) — passer par l'état système.
- Activer la **Corbeille AD** en amont : elle évite la lourdeur d'une restauration autoritaire.

## ⚠️ À ne pas confondre / obsolète

- **Non autoritaire** (le DC rattrape les autres) ≠ **autoritaire** (le DC réimpose ses objets).
- Pour une **suppression accidentelle**, préférer la **Corbeille AD** à la restauration autoritaire.
- **Snapshot rollback** d'un DC = dangereux (**USN rollback**) ; réservé aux hyperviseurs gérant **VM-GenerationID**.

---

## Sources (doc officielle / référence)

- [Microsoft Learn — AD Forest Recovery : sauvegarder un serveur complet](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/forest-recovery-guide/ad-forest-recovery-backing-up-a-full-server) — consulté le 24/07/2026
- [Microsoft Learn — Authoritative restore (ntdsutil, USN +100 000)](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc732211(v=ws.11)) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (`wbadmin`/`ntdsutil`) · [x] version datée (WS 2025) · [x] rien d'obsolète (Corbeille AD, anti USN rollback) · [x] procédure **à tester en lab** (AD) · [x] **vérif web reconfirmée (24/07/2026)** · [x] vérification présente (`dcdiag`/`repadmin`) · [x] sécurité (DSRM, tombstone) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
