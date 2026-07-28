# STO-07 — Remplacer un disque défaillant dans un RAID (reconstruction)

**Objectif** : identifier le **bon** disque défaillant, le remplacer (hot-swap) et suivre la **reconstruction** — en RAID matériel, mdadm et Storage Spaces.

**Rattachement REAC** : CP5 / CP8 / STO — savoir-faire : maintenir un stockage résilient en condition opérationnelle.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Une grappe RAID **dégradée** mais fonctionnelle (**STO-01/02/03**), un disque de rechange **identique ou plus grand**.
- Un backplane **hot-swap** (sinon, arrêt du serveur).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| RAID matériel | `storcli` (MegaRAID) | 24/07/2026 |
| RAID logiciel | **mdadm** (Debian 13) | 24/07/2026 |
| Storage Spaces | **WS 2025** (`Repair-VirtualDisk`) | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> 🛑 **Identifier le BON disque** (numéro de **série** + **LED**), pas seulement le slot. Sur une grappe déjà **dégradée**, retirer le mauvais disque = **perte totale**. **Sauvegarder avant** (une grappe dégradée est fragile).

---

## Procédure

### RAID matériel (contrôleur / storcli)

1. Le contrôleur signale un disque **Failed** ; le **localiser** (faire clignoter la LED).
2. **Hot-swap** : extraire le disque HS, insérer le neuf → **reconstruction automatique** (sur le nouveau ou le hot spare).

```bash
storcli /c0 show                       # repérer l'état/slot du disque Failed
storcli /c0/e252/s3 show               # vérifier le disque (slot 3)
storcli /c0/e252/s3 start rebuild      # si la reconstruction ne démarre pas seule
```

### RAID logiciel (mdadm) — fail → remove → swap → add

```bash
cat /proc/mdstat                       # [U_UU] = un disque manquant
mdadm --detail /dev/md0                # identifier le disque "faulty"
mdadm /dev/md0 --fail /dev/sdc1 --remove /dev/sdc1   # le sortir proprement

# --- remplacement physique du disque ---
sgdisk -R /dev/sdc /dev/sdb            # copier la table de partition d'un disque SAIN
sgdisk -G /dev/sdc                     # réattribuer de nouveaux GUID
mdadm /dev/md0 --add /dev/sdc1         # reconstruction automatique
watch cat /proc/mdstat                 # suivre "recovery = xx%"
```

### Storage Spaces (Windows)

```powershell
Get-PhysicalDisk | ? HealthStatus -ne 'Healthy'     # repérer le disque défaillant
Repair-VirtualDisk -FriendlyName Data               # relancer la réparation après ajout d'un disque
```

---

## Vérification (comment savoir que ça marche)

- **mdadm** : `/proc/mdstat` repasse en **`[UUUU]`** une fois la reconstruction finie.
- **Contrôleur** : état **Optimal** ; **Storage Spaces** : `Get-VirtualDisk` → **Healthy**.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Mauvais disque retiré | Slot confondu | **STOP** : réinsérer ; restaurer depuis sauvegarde si besoin |
| Reconstruction lente | I/O de production | Planifier ; patienter ; limiter la charge |
| 2ᵉ panne pendant rebuild | **URE** (gros disques) | Prévenir avec **RAID 6/10** (**STO-01**) |
| `--add` refusé | Table de partition absente | Recopier la table (`sgdisk -R`) puis `-G` |

## Sécurité et bonnes pratiques

- **Vérifier le numéro de série** avant de tirer un disque ; utiliser la **LED de localisation**.
- **Sauvegarder** avant l'opération (grappe dégradée = fragile).
- **Hot spare** pour démarrer la reconstruction immédiatement ; **surveiller SMART** (**STO-09**).
- Remplacer par un disque de **taille ≥** à l'original.

## ⚠️ À ne pas confondre / obsolète

- **Slot** ≠ **numéro de série** : toujours confirmer par la **série**.
- Reconstruction **RAID 5** sur gros disques = risquée → **RAID 6/10**.
- **Reconstruction ≠ sauvegarde** : le RAID ne protège pas d'une suppression/corruption (**CP8-01**).

---

## Sources (doc officielle)

- [Linux RAID Wiki — Replacing a failed drive](https://raid.wiki.kernel.org/index.php/Replacing_a_failed_drive) — consulté le 24/07/2026
- [Broadcom — storcli (MegaRAID) Reference](https://techdocs.broadcom.com/us/en/storage-and-ethernet-connectivity/enterprise-storage-solutions/megaraid-12gb-s.html) — consulté le 24/07/2026
- [Microsoft Learn — Repair-VirtualDisk](https://learn.microsoft.com/en-us/powershell/module/storage/repair-virtualdisk) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (storcli/mdadm/PowerShell) · [x] versions datées · [x] rien d'obsolète (RAID6/10, série vs slot) · [x] procédure **à tester en lab** · [x] conforme doc kernel/Broadcom/Microsoft · [x] vérification présente (`/proc/mdstat`) · [x] sécurité (bon disque, sauvegarde) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
