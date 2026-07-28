# CP8-11 — Sauvegarder hors-site / vers le Cloud (application de la règle 3-2-1)

**Objectif** : externaliser une copie des sauvegardes vers un **site distant / le Cloud**, **chiffrée** et **immuable**, pour compléter la règle **3-2-1(-1-0)**.

**Rattachement REAC** : CP8 « Sauvegardes et restaurations des éléments de l'infrastructure » — savoir-faire : externaliser les sauvegardes.

**Durée** : ~30 min · **Niveau** : intermédiaire.

---

## Prérequis

- Des sauvegardes locales existantes (**CP8-02 à CP8-06**).
- Un stockage distant **compatible S3** (AWS S3, **Wasabi**, **Backblaze B2**) ou un serveur distant en SSH.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Objet immuable | **S3 Object Lock** (WORM) — AWS/Wasabi/B2 | 24/07/2026 |
| Outils | **restic**, **rclone**, PBS **sync/S3** (4.2) | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **Le « 1 hors site » de la règle 3-2-1**, renforcé par le **« -1-0 »** : une copie **immuable** (*Object Lock* / WORM) qu'**aucun compte**, même compromis, ne peut effacer avant l'échéance → parade **anti-rançongiciel**. Rétention typique **30–90 j**.

---

## Procédure — GUI

1. **Chez le fournisseur** : créer un **bucket** S3 et activer **Object Lock** (mode *Governance* ou *Compliance*) avec une **durée de rétention**.
2. **Proxmox Backup Server** : configurer un **Sync Job** vers un **PBS distant** ou un **datastore S3** (PBS 4.2) pour répliquer les sauvegardes hors site.
3. **Veeam** : créer un **Backup Copy Job** vers l'**object storage** avec **immutabilité** activée.

## Procédure — CLI (restic / rclone vers S3)

### restic (chiffrement + déduplication natifs)

```bash
export RESTIC_REPOSITORY="s3:s3.amazonaws.com/mon-bucket"
export RESTIC_PASSWORD="phrase-secrète-forte"     # chiffrement côté client
restic init                                        # initialiser le dépôt chiffré
restic backup /backup                              # envoyer (incrémental, dédupliqué)
restic snapshots                                   # lister
restic restore latest --target /restore-test       # restaurer
```

### rclone (synchronisation vers objet)

```bash
rclone sync /backup wasabi:mon-bucket --immutable   # copie hors site
# Chiffrement : passer par un remote "crypt" (rclone config) au-dessus du remote S3
```

> **restic** chiffre **avant** l'envoi (les données sont illisibles côté fournisseur) et **déduplique**. **rclone** synchronise ; ajouter un remote **crypt** pour le chiffrement.

---

## Vérification (comment savoir que ça marche)

- La copie distante existe (`restic snapshots`, console du fournisseur).
- **Test de restauration depuis le Cloud** dans un dossier jetable (**CP8-07**).
- **Immutabilité** : tenter de supprimer un objet verrouillé → l'opération est **refusée** avant l'échéance.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Coûts élevés en restauration | Frais de **sortie (egress)** | Choisir un fournisseur sans egress ; restaurer le nécessaire |
| Accès refusé | Clés/API ou horloge | Vérifier identifiants ; NTP (**CP4-13**) |
| Impossible d'appliquer la rétention | Object Lock non activé **à la création** | Recréer le bucket avec **Object Lock** |
| Données irrécupérables | **Clé de chiffrement perdue** | Sauvegarder la clé/phrase **séparément et en sûreté** |

## Sécurité et bonnes pratiques

- **Chiffrer côté client** (restic / remote crypt) : le fournisseur ne voit que des données illisibles.
- **Immutabilité (Object Lock)** + identifiants **en écriture seule/append-only** : même un attaquant ne peut pas purger les sauvegardes.
- **Sauvegarder la clé de chiffrement** hors du Cloud (sa perte = données perdues).
- **Tester** régulièrement une restauration **depuis** le Cloud (pas seulement l'envoi).

## ⚠️ À ne pas confondre / obsolète

- Copie Cloud **non chiffrée** = fuite potentielle → **toujours chiffrer**.
- **Sauvegarde** vers le Cloud ≠ **synchronisation** type Drive/OneDrive (qui **propage** aussi les suppressions/chiffrements).
- Une copie hors site **sans immutabilité** ne protège **pas** d'un rançongiciel qui a les droits d'admin.

---

## Sources (doc officielle)

- [AWS — S3 Object Lock (WORM/immuabilité)](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lock.html) — consulté le 24/07/2026
- [restic — Documentation (dépôts S3, chiffrement)](https://restic.readthedocs.io/en/stable/) — consulté le 24/07/2026
- [rclone — Documentation (S3, crypt)](https://rclone.org/docs/) — consulté le 24/07/2026
- [CISA — #StopRansomware Guide (sauvegardes immuables, 3-2-1-1-0)](https://www.cisa.gov/stopransomware) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (restic/rclone) · [x] versions datées · [x] rien d'obsolète (Object Lock, chiffrement client) · [x] procédure **à tester en lab** · [x] GUI conforme doc fournisseurs · [x] vérification présente (restore + test immutabilité) · [x] sécurité (chiffrement, immuable, clé) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
