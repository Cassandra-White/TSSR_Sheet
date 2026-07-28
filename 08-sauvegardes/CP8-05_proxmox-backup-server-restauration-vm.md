# CP8-05 — Installer Proxmox Backup Server et restaurer une VM

**Objectif** : installer **Proxmox Backup Server (PBS)**, créer un **datastore**, l'intégrer à **Proxmox VE**, sauvegarder une VM (déduplication) puis la **restaurer**.

**Rattachement REAC** : CP8 « Sauvegardes et restaurations des éléments de l'infrastructure » — savoir-faire : sauvegarder/restaurer des VM.

**Durée** : ~35 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un hôte **Proxmox VE 9** (**CP5-01**) avec des VM.
- Une machine (ou VM) dédiée pour **PBS** + un **disque de stockage** pour le datastore.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Sauvegarde | **Proxmox Backup Server 4.2** | 24/07/2026 |
| Virtualisation | **Proxmox VE 9.2** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **PBS** stocke les sauvegardes en **blocs dédupliqués + compressés + chiffrables** : après la 1ʳᵉ sauvegarde, les suivantes sont **incrémentales** (seuls les blocs modifiés), tout en restant **restaurables intégralement**. Nouveauté 4.2 : cible **S3** pour l'externalisation.

---

## Procédure — GUI

### Installer PBS et créer un datastore

1. Installer PBS (ISO dédié, ou `apt` sur Debian 13) → accéder à la console web **`https://<pbs>:8007`**.
2. **Administration ▸ Datastore ▸ Add Datastore** : nom (`store1`) + chemin (ex. `/mnt/datastore/store1`).
3. (Sécurité) noter l'**empreinte** (fingerprint) affichée : elle sera demandée côté PVE.

### Rattacher PBS à Proxmox VE

4. Sur PVE : **Datacenter ▸ Storage ▸ Add ▸ Proxmox Backup Server** → hôte PBS, **Datastore** `store1`, utilisateur (`backup@pbs` ou `root@pam`), mot de passe, **Fingerprint**.

### Sauvegarder une VM

5. **Datacenter ▸ Backup ▸ Add** : job planifié (heure, VM ciblées, **Storage = PBS**, mode **Snapshot**, rétention) — ou **VM ▸ Backup ▸ Backup now**.

### Restaurer

6. Sélectionner le stockage **PBS ▸ Backups** → choisir la sauvegarde → **Restore** (VM entière, éventuellement sous un nouvel ID).
7. **File Restore** : restaurer **un seul fichier** depuis l'intérieur d'une sauvegarde de VM.

## Procédure — CLI

```bash
# Sur PBS : créer le datastore
proxmox-backup-manager datastore create store1 /mnt/datastore/store1

# Sur PVE : sauvegarder la VM 100 vers le stockage PBS (mode snapshot)
vzdump 100 --storage pbs --mode snapshot

# Restaurer la sauvegarde vers un nouvel ID de VM (ex. 999)
qmrestore <volid-de-la-sauvegarde-PBS> 999

# Sur PBS : maintenance de l'espace (récupération des blocs orphelins)
proxmox-backup-manager garbage-collection start store1
```

> Une tâche **Verify** recalcule les empreintes pour garantir l'intégrité ; **Prune + Garbage Collection** appliquent la rétention et libèrent l'espace.

---

## Vérification (comment savoir que ça marche)

- La sauvegarde apparaît dans **PBS ▸ Datastore ▸ Content** avec un statut **OK** ; la tâche **Verify** est **verte**.
- Une **restauration de test** (vers un nouvel ID) démarre et la VM boote.
- Les sauvegardes suivantes sont **rapides/petites** (incrémentales, dédupliquées).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Ajout de stockage refusé | **Fingerprint** erroné | Recopier l'empreinte exacte affichée par PBS |
| Datastore plein | Rétention non appliquée | **Prune** + **Garbage Collection** |
| Sauvegarde échoue | Droits/utilisateur PBS | Vérifier les permissions du **DatastoreBackup** |
| Restauration écrase une VM | Même VMID réutilisé | Restaurer vers un **nouvel ID** puis basculer |

## Sécurité et bonnes pratiques

- **Chiffrement côté client** (clé de datastore) : les blocs sont illisibles sans la clé — **sauvegarder la clé** séparément.
- **Externaliser** via **Sync/Remote** ou **S3** (3-2-1 — **CP8-01**) ; idéalement une copie **hors-ligne/immuable**.
- Planifier des **Verify jobs** réguliers et **tester une restauration** (**CP8-07**).
- PBS sur un **hôte séparé** de l'hyperviseur (ne pas tout perdre en cas de sinistre de l'hôte PVE).

## ⚠️ À ne pas confondre / obsolète

- **PBS** (dédup, incrémental *forever*, restauration fichier) ≫ ancien **vzdump** vers un disque local en **full** répété.
- **Snapshot PVE** (état ponctuel sur l'hôte) ≠ **sauvegarde PBS** (copie externe, historisée) — voir **CP8-10**.
- Une sauvegarde **non vérifiée/non externalisée** ne satisfait pas la règle 3-2-1.

---

## Sources (doc officielle)

- [Proxmox Backup Server — Documentation](https://pbs.proxmox.com/docs/) — consulté le 24/07/2026
- [Proxmox — PBS 4.2 (nouveautés, S3)](https://proxmox.com/en/about/company-details/press-releases/proxmox-backup-server-4-2) — consulté le 24/07/2026
- [Proxmox VE — Backup and Restore](https://pve.proxmox.com/wiki/Backup_and_Restore) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées (PBS 4.2 / PVE 9.2) · [x] rien d'obsolète (dédup vs vzdump full) · [x] procédure **à tester en lab** · [x] GUI conforme doc Proxmox · [x] vérification présente (Verify) · [x] sécurité (chiffrement, S3/offsite) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
