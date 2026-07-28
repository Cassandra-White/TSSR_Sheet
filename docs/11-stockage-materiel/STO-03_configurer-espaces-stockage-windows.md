# STO-03 — Configurer les Espaces de stockage Windows (Storage Spaces)

**Objectif** : agréger des disques dans un **pool**, créer un **disque virtuel résilient** (miroir/parité) et un volume **ReFS/NTFS** avec **Storage Spaces**.

**Rattachement REAC** : CP2 (exploitation Windows) / STO — savoir-faire : mettre en œuvre un stockage résilient Windows.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- **Windows Server 2025** avec plusieurs disques **non initialisés** (poolables).
- Droits **administrateur**.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Serveur | **Windows Server 2025** | 24/07/2026 |
| Stockage | **Storage Spaces** + **ReFS** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **Storage Spaces** = le **RAID logiciel de Windows** : on regroupe des disques dans un **pool**, puis on crée des **espaces** (disques virtuels) avec une **résilience** : **Simple** (RAID 0), **Miroir** (2/3 copies, type RAID 1), **Parité** (type RAID 5/6). Sur WS 2025 + **ReFS**, la **parité accélérée par miroir** combine perf (miroir) et capacité (parité).

---

## Procédure — GUI (Server Manager)

1. **File and Storage Services ▸ Volumes ▸ Storage Pools ▸ New Storage Pool** : nommer, sélectionner les **disques physiques**.
2. **New Virtual Disk** : choisir la **résilience** (Mirror pour le critique, Parity pour la capacité) et le **provisionnement** (thin/fixed).
3. **New Volume** : formater en **ReFS** (recommandé pour la résilience/intégrité) ou NTFS, lettre/point de montage.

## Procédure — CLI (PowerShell)

```powershell
# Disques disponibles pour le pool
Get-PhysicalDisk -CanPool $true

# Créer le pool
New-StoragePool -FriendlyName Pool1 -StorageSubSystemFriendlyName "Windows Storage*" `
  -PhysicalDisks (Get-PhysicalDisk -CanPool $true)

# Disque virtuel en miroir + volume ReFS
New-VirtualDisk -StoragePoolFriendlyName Pool1 -FriendlyName Data `
  -ResiliencySettingName Mirror -Size 500GB -ProvisioningType Thin
Get-VirtualDisk Data | Get-Disk | Initialize-Disk -PassThru |
  New-Partition -AssignDriveLetter -UseMaximumSize |
  Format-Volume -FileSystem ReFS
```

---

## Vérification (comment savoir que ça marche)

```powershell
Get-StoragePool Pool1 | Format-List FriendlyName, HealthStatus
Get-VirtualDisk       | Format-Table FriendlyName, ResiliencySettingName, HealthStatus
```

- **HealthStatus = Healthy** ; test : retirer un disque → **Degraded** → **réparation** si capacité de réserve disponible.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Disque non poolable | Déjà partitionné/initialisé | Effacer les partitions (`Reset-PhysicalDisk`/nettoyer) |
| Écritures en **parité** lentes | Nature de la parité | **Mirror-accelerated parity** / **tier SSD** |
| Volume non réparé après panne | Pas de **capacité de réserve** | Prévoir un disque/reserve pour l'auto-repair |
| Perf/résilience insuffisante | Mauvais choix de résilience | **Miroir** pour le critique, **parité** pour le volumineux |

## Sécurité et bonnes pratiques

- **Miroir** pour les données **critiques**, **parité** pour la **capacité** (archives/sauvegardes).
- Prévoir de la **capacité de réserve** (auto-repair) ou un disque de rechange.
- **ReFS** : flux d'intégrité, résilience aux corruptions (mieux que NTFS pour de gros volumes).
- **Storage Spaces ≠ sauvegarde** : compléter par **CP8**.

## ⚠️ À ne pas confondre / obsolète

- **Storage Spaces** (1 serveur) ≠ **Storage Spaces Direct (S2D)** (cluster, hyperconvergé).
- **Miroir** (perf/redondance) ≠ **Parité** (capacité) ; **Mirror-accelerated parity** combine les deux (WS 2025 + ReFS).
- Storage Spaces = **logiciel** : ne pas mélanger avec un **RAID matériel** sous-jacent (laisser les disques en **HBA/pass-through**).

---

## Sources (doc officielle)

- [Microsoft Learn — Storage Spaces overview](https://learn.microsoft.com/en-us/windows-server/storage/storage-spaces/overview) — consulté le 24/07/2026
- [Microsoft Learn — Fault tolerance & storage efficiency](https://learn.microsoft.com/en-us/windows-server/storage/storage-spaces/fault-tolerance) — consulté le 24/07/2026
- [Microsoft Learn — Mirror-accelerated parity (ReFS)](https://learn.microsoft.com/en-us/windows-server/storage/refs/mirror-accelerated-parity) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (PowerShell) · [x] version datée (WS 2025) · [x] rien d'obsolète (ReFS, mirror-accelerated parity) · [x] procédure **à tester en lab** · [x] conforme doc Microsoft · [x] vérification présente (HealthStatus) · [x] sécurité (miroir/critique, réserve, ≠backup) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
