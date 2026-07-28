# STO-09 — Surveiller la santé des disques (SMART) et anticiper les pannes

**Objectif** : lire l'état **SMART** des disques, lancer des **auto-tests** et **alerter** automatiquement pour remplacer un disque **avant** qu'il ne lâche.

**Rattachement REAC** : CP2 / CP3 / STO — savoir-faire : surveiller le matériel et anticiper les pannes.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- **smartmontools** (`smartctl`, `smartd`) sous Linux ; outil équivalent sous Windows.
- Accès **root**/admin ; disques compatibles **SMART** (la plupart).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Linux | **smartmontools** (`smartctl`/`smartd`) | 24/07/2026 |
| Windows | PowerShell **Storage** / CrystalDiskInfo | 24/07/2026 |
| Procédure appliance | **à tester en lab** (disque réel) | 24/07/2026 |

> **SMART** = autodiagnostic intégré au disque. Il expose des **attributs** (secteurs réalloués, secteurs en attente, température, heures de fonctionnement…). Les attributs de type **Pre-fail** qui se dégradent = **signal de remplacement anticipé**.

---

## Procédure — CLI (Linux, `smartctl`)

```bash
smartctl -H /dev/sda            # verdict global : PASSED / FAILED
smartctl -a /dev/sda            # tous les attributs SMART
smartctl -t short /dev/sda      # auto-test court (~2 min) ; -t long pour l'approfondi
smartctl -l selftest /dev/sda   # résultats des auto-tests
```

Attributs à surveiller en priorité :

| Attribut | Signification | Seuil d'alerte |
|---|---|---|
| **Reallocated_Sector_Ct** | Secteurs défectueux réalloués | **> 0** et **en hausse** |
| **Current_Pending_Sector** | Secteurs instables en attente | **> 0** |
| **Offline_Uncorrectable** | Secteurs irrécupérables | **> 0** |
| **Temperature_Celsius** | Température | selon le disque (trop chaud) |

Alertes automatiques avec le démon **smartd** (`/etc/smartd.conf`) :

```conf
/dev/sda -a -o on -S on -s (S/../.././02|L/../../6/03) -m admin@lab.local
```

## Procédure — GUI / Windows

```powershell
Get-PhysicalDisk | Get-StorageReliabilityCounter |
  ft DeviceId, Temperature, ReadErrorsTotal, Wear
```

- Ou outils graphiques : **CrystalDiskInfo** / utilitaires constructeur (Dell, HPE…).

---

## Vérification (comment savoir que ça marche)

- `smartctl -H` renvoie **PASSED** ; un **auto-test** apparaît « Completed without error » dans `-l selftest`.
- **smartd** est actif (`systemctl status smartd`) et envoie bien une alerte de test.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| « SMART support is: Disabled » | SMART désactivé | `smartctl -s on /dev/sda` |
| Rien derrière un contrôleur RAID | Disques masqués | `smartctl -d megaraid,N -a /dev/bus/0` |
| Disque USB non lu | Pont USB | `-d sat` / `-d scsi` selon le boîtier |
| Verdict PASSED mais secteurs qui montent | Dégradation lente | Remplacer **avant** le FAILED |

## Sécurité et bonnes pratiques

- **Remplacement anticipé** dès que **Reallocated/Pending Sectors** augmentent (ne pas attendre le FAILED).
- **Superviser** (smartd + intégration Zabbix — **CP4-17**) et **journaliser**.
- SMART **complète** le RAID (**STO-01/02**) et les **sauvegardes** (**CP8**) : il ne les remplace pas.
- Surveiller la **température** (refroidissement) — facteur de panne majeur.

## ⚠️ À ne pas confondre / obsolète

- **PASSED** ne signifie pas « parfait » : lire les **attributs** (un disque peut passer tout en se dégradant).
- Attributs **Pre-fail** (critiques) ≠ **Old_age** (usure normale).
- SMART **prévient** souvent mais **pas toujours** (pannes brutales) → **RAID + sauvegarde** restent indispensables.

---

## Sources (doc officielle)

- [smartmontools — smartctl / smartd](https://www.smartmontools.org/wiki/TocDoc) — consulté le 24/07/2026
- [smartctl(8) — Manuel](https://manpages.debian.org/bookworm/smartmontools/smartctl.8.en.html) — consulté le 24/07/2026
- [Microsoft Learn — Get-StorageReliabilityCounter](https://learn.microsoft.com/en-us/powershell/module/storage/get-storagereliabilitycounter) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI puis GUI/Windows · [x] daté 24/07/2026 · [x] rien d'obsolète (attributs Pre-fail, smartd) · [x] procédure **à tester en lab** (disque réel) · [x] conforme doc smartmontools/Microsoft · [x] vérification présente (`-H`/`-l selftest`) · [x] sécurité (remplacement anticipé, RAID+backup) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
