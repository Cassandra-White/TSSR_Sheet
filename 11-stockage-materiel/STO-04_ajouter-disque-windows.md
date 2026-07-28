# STO-04 — Ajouter un disque sous Windows (initialiser, partitionner, formater, étendre)

**Objectif** : mettre en service un nouveau disque sous Windows — **initialiser** (GPT), **partitionner**, **formater** (NTFS/ReFS) et **étendre** un volume existant.

**Rattachement REAC** : CP2 (exploitation Windows) / STO — savoir-faire : gérer le stockage d'un serveur.

**Durée** : ~15 min · **Niveau** : débutant/intermédiaire.

---

## Prérequis

- **Windows Server 2025** (ou Windows 11), droits **administrateur**.
- Un disque neuf visible côté matériel/hyperviseur.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| OS | **Windows Server 2025** | 24/07/2026 |
| Outils | **Disk Management** / `diskpart` / PowerShell **Storage** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **Trois étapes** : **Initialiser** (choisir le schéma de partition **GPT**), **créer un volume** (partition + formatage), puis éventuellement **étendre** un volume sur de l'espace libre adjacent.

---

## Procédure — GUI (Gestion des disques)

1. Ouvrir **`diskmgmt.msc`** : le nouveau disque apparaît **Non initialisé**.
2. Clic droit sur le disque → **Initialiser le disque** → **GPT**.
3. Clic droit sur l'espace **non alloué** → **Nouveau volume simple** → taille, **lettre**, formatage **NTFS** (ou **ReFS**), nom de volume.
4. **Étendre** : clic droit sur le volume → **Étendre le volume** (nécessite de l'espace **non alloué adjacent à droite**).

## Procédure — CLI (PowerShell)

```powershell
Get-Disk                                             # repérer le n° du nouveau disque
Initialize-Disk -Number 1 -PartitionStyle GPT
New-Partition -DiskNumber 1 -UseMaximumSize -AssignDriveLetter |
  Format-Volume -FileSystem NTFS -NewFileSystemLabel "Data" -Confirm:$false

# Étendre la partition D: à la taille max disponible
Resize-Partition -DriveLetter D -Size (Get-PartitionSupportedSize -DriveLetter D).SizeMax
```

*(Alternative `diskpart` : `select disk`, `convert gpt`, `create partition primary`, `format fs=ntfs quick`, `extend`.)*

---

## Vérification (comment savoir que ça marche)

```powershell
Get-Volume                       # le volume apparaît, sain, avec sa lettre
Get-Disk | ft Number,FriendlyName,PartitionStyle,OperationalStatus
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Disque **Offline** | État hors ligne | `Set-Disk -Number 1 -IsOffline $false` |
| « Étendre » grisé | Pas d'espace **adjacent** / pas NTFS | Libérer l'espace à droite ; NTFS requis (diskpart) |
| Limite ~2 To | Disque en **MBR** | Utiliser **GPT** |
| Formatage refusé | Disque en lecture seule | `Set-Disk -IsReadOnly $false` |

## Sécurité et bonnes pratiques

- **GPT** systématiquement (supporte >2 To, plus robuste que MBR).
- **Sauvegarder** avant tout repartitionnement d'un disque contenant des données (**CP8**).
- **Nommer** clairement les volumes ; documenter l'usage.

## ⚠️ À ne pas confondre / obsolète

- **MBR** (limite 2 To, hérité) → **GPT** (moderne).
- **`diskpart extend`** ne fonctionne qu'en **NTFS** et avec de l'espace adjacent.
- Ajouter un disque **≠** résilience : pour la tolérance de panne, voir **STO-01/03**.

---

## Sources (doc officielle)

- [Microsoft Learn — Initialize new disks](https://learn.microsoft.com/en-us/windows-server/storage/disk-management/initialize-new-disks) — consulté le 24/07/2026
- [Microsoft Learn — Manage disks (Disk Management)](https://learn.microsoft.com/en-us/windows-server/storage/disk-management/overview-of-disk-management) — consulté le 24/07/2026
- [Microsoft Learn — Storage cmdlets (PowerShell)](https://learn.microsoft.com/en-us/powershell/module/storage/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (PowerShell/diskpart) · [x] version datée (WS 2025) · [x] rien d'obsolète (GPT vs MBR) · [x] procédure **à tester en lab** · [x] conforme doc Microsoft · [x] vérification présente (`Get-Volume`) · [x] sécurité (GPT, sauvegarde) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
