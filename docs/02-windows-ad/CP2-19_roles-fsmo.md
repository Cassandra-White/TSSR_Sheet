# CP2-19 — Comprendre et transférer les rôles FSMO

**Objectif** : identifier les 5 rôles FSMO, savoir où ils sont, et les transférer proprement (ou les saisir en cas de panne définitive).

**Rattachement REAC** : CP2 — exploitation et disponibilité de l'annuaire Active Directory.

**Durée** : ~15 min · **Niveau** : intermédiaire.

---

## Prérequis

- Au moins deux contrôleurs de domaine (voir **CP2-18**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (Desktop Experience) | 23/07/2026 |

## Les 5 rôles

- **Forêt** : *Maître de schéma* (Schema Master), *Maître d'attribution des noms de domaine* (Domain Naming Master).
- **Domaine** : *Maître RID* (RID Master), *Émulateur PDC* (PDC Emulator — heure, mots de passe, verrouillages), *Maître d'infrastructure* (Infrastructure Master).

---

## Procédure — GUI (méthode prioritaire)

**Localiser** : `netdom query fsmo`.

**Transférer** (se connecter d'abord au DC cible dans chaque console) :
- RID / PDC / Infrastructure : `dsa.msc` → clic droit sur le domaine → **Maîtres d'opérations** (3 onglets) → **Modifier**.
- Domain Naming : **Domaines et approbations Active Directory** → clic droit → **Maître d'opérations**.
- Schéma : enregistrer la console (`regsvr32 schmmgmt.dll`), ajouter le composant **Schéma Active Directory** → clic droit → **Maître d'opérations**.

## Procédure — CLI

```powershell
# Localiser
netdom query fsmo
Get-ADDomain | Select-Object PDCEmulator,RIDMaster,InfrastructureMaster
Get-ADForest | Select-Object SchemaMaster,DomainNamingMaster

# Transfert normal vers SRV-AD02
Move-ADDirectoryServerOperationMasterRole -Identity "SRV-AD02" `
  -OperationMasterRole SchemaMaster,DomainNamingMaster,RIDMaster,PDCEmulator,InfrastructureMaster

# SAISIE (seize) — uniquement si l'ancien titulaire est définitivement HS
Move-ADDirectoryServerOperationMasterRole -Identity "SRV-AD02" -OperationMasterRole PDCEmulator -Force
```

*(À exécuter en lab — non testable dans le bac à sable Linux.)*

---

## Vérification

```powershell
netdom query fsmo   # chaque rôle pointe sur le DC attendu
```

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| Transfert impossible | DC cible injoignable | Vérifier réplication/réseau |
| Rôle bloqué sur un DC HS | Panne définitive | **Saisir** (`-Force`), puis *metadata cleanup* du DC mort |
| Problèmes d'heure/verrouillage | Le PDC Emulator porte le temps | Vérifier le NTP (**CP4-13**) |

## Sécurité et bonnes pratiques

- **Ne jamais saisir** un rôle si l'ancien titulaire peut revenir (risque de corruption) : transférer proprement.
- Après une saisie, **retirer** proprement l'ancien DC (nettoyage des métadonnées).
- Connaître la répartition des rôles et **sauvegarder l'état système** (**CP8-03**).

## ⚠️ À ne pas confondre / obsolète

- Transfert **normal** = PowerShell/console ; `ntdsutil` reste l'outil de **secours** (saisie + nettoyage).
- « Transférer » (propre, les 2 DC en ligne) ≠ « Saisir » (d'urgence, titulaire mort).

---

## Sources (doc officielle)

- [Move-ADDirectoryServerOperationMasterRole (WS2025) — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/activedirectory/move-addirectoryserveroperationmasterrole?view=windowsserver2025-ps) — consulté le 23/07/2026
- [Gérer les rôles FSMO — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/manage-fsmo-roles) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
