# STO-12 — Créer et gérer un pool ZFS (pool, dataset, snapshot)

**Objectif** : créer un **pool ZFS** résilient (mirror/raidz), organiser des **datasets**, prendre des **snapshots**, régler les propriétés (compression/quota) et vérifier l'intégrité (**scrub**).

**Rattachement REAC** : CP3 / CP5 / STO — savoir-faire : mettre en œuvre un stockage avancé.

**Durée** : ~30 min · **Niveau** : avancé.

---

## Prérequis

- Un système avec **OpenZFS** (Debian avec `zfsutils-linux`, ou Proxmox VE), accès **root**.
- Plusieurs disques (identifiés par **`/dev/disk/by-id/`**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Stockage | **OpenZFS 2.4** | 24/07/2026 |
| Capacité/redondance des vdev | **calcul testé en bac à sable** | 24/07/2026 |
| Commandes `zpool`/`zfs` | **à tester en lab** (root/kernel) | 24/07/2026 |

> **ZFS** = système de fichiers **+** gestionnaire de volumes : **checksums** (auto-réparation avec redondance), **compression**, **snapshots** quasi gratuits, **réplication** (`send/receive`). Un **pool** agrège des **vdev** (mirror ou **raidz**) ; les **datasets** héritent des propriétés.

---

## Capacité et tolérance des vdev (ex. 6 disques de 4 To)

| Layout | Capacité utile | Tolérance | Remarque |
|---|---|---|---|
| **mirror** (3× 2-way) | 12 To | 1 par miroir | perf lecture, 50 % capacité |
| **raidz1** | 20 To | 1 disque | capacité, **risqué** sur gros disques |
| **raidz2** | 16 To | **2 disques** | **recommandé** (rebuild long) |
| **raidz3** | 12 To | 3 disques | archivage critique |

*(Valeurs calculées en bac à sable.)*

---

## Procédure — CLI

```bash
# 1) Créer un pool raidz2 (tolère 2 pannes) — TOUJOURS par by-id
zpool create tank raidz2 \
  /dev/disk/by-id/wwn-0x... /dev/disk/by-id/wwn-0x... ... (6 disques)
zpool status ; zpool list

# 2) Datasets + propriétés
zfs create tank/data
zfs set compression=zstd tank/data     # compression recommandée
zfs set quota=100G tank/data

# 3) Snapshots
zfs snapshot tank/data@avant-maj
zfs list -t snapshot
zfs rollback tank/data@avant-maj

# 4) Intégrité (mensuel) et réplication
zpool scrub tank                       # vérifie/répare (self-healing)
zfs send tank/data@avant-maj | ssh nas zfs receive backup/data   # réplication hors-site
```

> **Extension** (OpenZFS ≥ 2.3) : `zpool attach` permet d'**ajouter un disque à un vdev raidz** (expansion). La **topologie** d'un raidz existant ne se **change** pas (pas de raidz1→raidz2 sans reconstruction).

---

## Vérification (comment savoir que ça marche)

```bash
zpool status tank        # état ONLINE, "errors: No known data errors"
zfs list                 # datasets + espace utilisé/compressé
zpool status | grep scrub  # dernier scrub "repaired 0B ... no errors"
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Pool ne remonte pas | Disques nommés en **`/dev/sdX`** | Créer par **`by-id`** (stable) |
| Pool **DEGRADED** | Disque défaillant | `zpool replace tank <ancien> <neuf>` |
| RAM saturée | **ARC** (cache ZFS) | Limiter `zfs_arc_max` ; **ECC** recommandée |
| Pas de réduction possible | raidz non réductible | Prévoir la topologie ; expansion via `attach` |

## Sécurité et bonnes pratiques

- **Snapshots réguliers** (protection rançongiciel) + **`zfs send`** hors-site (**CP8-11**).
- **Scrub mensuel** (détecte/corrige la corruption silencieuse — *bit rot* — avec redondance).
- **ECC RAM** recommandée ; **raidz2** pour les gros disques.
- **RAID-Z ≠ sauvegarde** : garder de vraies sauvegardes (**CP8**).

## ⚠️ À ne pas confondre / obsolète

- **pool** (agrégat) / **vdev** (mirror ou raidz) / **dataset** (système de fichiers) : trois niveaux.
- **raidz1** sur gros disques = risqué → **raidz2** (comme RAID 6).
- Créer par **`by-id`**, jamais par `/dev/sdX` (réordonnancement au boot).

---

## Sources (doc officielle)

- [OpenZFS — zpool-create(8)](https://openzfs.github.io/openzfs-docs/man/master/8/zpool-create.8.html) — consulté le 24/07/2026
- [OpenZFS — zfs-snapshot(8) / datasets](https://openzfs.github.io/openzfs-docs/man/master/8/zfs-snapshot.8.html) — consulté le 24/07/2026
- [OpenZFS — Documentation](https://openzfs.github.io/openzfs-docs/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] version datée (OpenZFS 2.4) · [x] rien d'obsolète (raidz2, by-id, expansion 2.3) · [x] **capacité testée en bac à sable** / `zpool` à tester en lab · [x] conforme doc OpenZFS · [x] vérification présente (`zpool status`/scrub) · [x] sécurité (snapshots, scrub, ≠backup) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
