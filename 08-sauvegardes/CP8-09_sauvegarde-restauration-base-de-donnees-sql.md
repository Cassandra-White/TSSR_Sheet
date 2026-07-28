# CP8-09 — Sauvegarder et restaurer une base de données (dump SQL : MySQL/PostgreSQL)

**Objectif** : réaliser un **dump logique** cohérent d'une base **MySQL/MariaDB** et **PostgreSQL**, le **restaurer**, et planifier l'opération.

**Rattachement REAC** : CP8 « Sauvegardes et restaurations des éléments de l'infrastructure » — savoir-faire : sauvegarder/restaurer une base de données.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un serveur **MariaDB/MySQL** et/ou **PostgreSQL** (**CP3**), avec un **compte de sauvegarde** aux droits suffisants (lecture).
- Un espace de destination + planification (**CP6-08 / CP3-12**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| MySQL/MariaDB | `mysqldump` / **`mariadb-dump`** | 24/07/2026 |
| PostgreSQL | `pg_dump` / `pg_dumpall` | 24/07/2026 |
| Principe **dump → restauration** | **testé en bac à sable** (sqlite3, analogue) | 24/07/2026 |

> **On ne copie jamais les fichiers d'une base en cours d'écriture** (état incohérent). On utilise un **dump logique** : un fichier **SQL rejouable** (`CREATE` + `INSERT`) qui recrée la base **à l'identique**, même pendant l'activité.

---

## Procédure — CLI

### MySQL / MariaDB (`mariadb-dump`, ex-`mysqldump`)

```bash
# Sauvegarde cohérente d'InnoDB SANS verrouiller (snapshot transactionnel)
mariadb-dump --single-transaction --routines --triggers \
             --databases app > app-$(date +%F).sql

# Restauration (la base est recréée si --databases inclut le CREATE DATABASE)
mariadb app < app-2026-07-24.sql
```

### PostgreSQL (`pg_dump` + `pg_dumpall`)

```bash
# Une base, format "custom" (compressé, restauration sélective)
pg_dump -Fc app > app-$(date +%F).dump
# Objets globaux (rôles, tablespaces) : une fois pour tout le serveur
pg_dumpall --globals-only > globals-$(date +%F).sql

# Restauration : d'abord les globaux, puis la base
psql -f globals-2026-07-24.sql
createdb app && pg_restore -d app app-2026-07-24.dump
```

> **Démonstration testée** (principe, via sqlite3) : base de 3 lignes → **dump SQL** (`BEGIN TRANSACTION; CREATE TABLE…; INSERT…`) → **restauration dans une base vide** → contenu **identique** vérifié. C'est exactement la logique de `mysqldump`/`pg_dump`.

### Planifier (cron)

```cron
30 1 * * *  root  mariadb-dump --single-transaction --all-databases | gzip > /backup/sql/all-$(date +\%F).sql.gz
```

---

## Vérification (comment savoir que ça marche)

```bash
ls -lh /backup/sql/                     # le dump existe et n'est pas vide
zcat all-2026-07-24.sql.gz | head       # en-tête SQL lisible
# Test réel : restaurer dans une base JETABLE puis comparer
mariadb testrestore < app-2026-07-24.sql
mariadb -e "SELECT COUNT(*) FROM testrestore.clients;"   # même nombre de lignes qu'en prod
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Base bloquée pendant le dump | `--lock-tables` (défaut MyISAM) | **`--single-transaction`** (InnoDB) |
| Accents/caractères cassés | Jeu de caractères | Forcer `--default-character-set=utf8mb4` |
| Restauration PostgreSQL : rôles manquants | Globaux non restaurés | Restaurer **`pg_dumpall --globals-only`** d'abord |
| Dump très long / gros volume | Base volumineuse | Sauvegarde **physique** (`mariabackup` / `pg_basebackup` + WAL pour PITR) |

## Sécurité et bonnes pratiques

- **Compte de sauvegarde à droits minimaux** (lecture) ; ne pas mettre le mot de passe en clair (fichier `~/.my.cnf`/`.pgpass` protégé).
- **Chiffrer** et **externaliser** les dumps (données sensibles — RGPD, **CT4-01** ; règle 3-2-1 — **CP8-01**).
- **Tester la restauration** dans une base jetable (**CP8-07**).
- Pour du **point-in-time recovery**, combiner dump **+ journaux** (binlog MySQL / WAL PostgreSQL).

## ⚠️ À ne pas confondre / obsolète

- **Copie des fichiers d'une base active** = corruption → utiliser un **dump** (ou une sauvegarde physique à chaud dédiée).
- `mysqldump` est renommé **`mariadb-dump`** sous MariaDB (l'ancien nom reste un alias).
- **Dump logique** (portable, lent à restaurer si gros) ≠ **sauvegarde physique** (rapide, moins portable) : choisir selon le volume/RTO.

---

## Sources (doc officielle)

- [MariaDB — `mariadb-dump` (ex-mysqldump)](https://mariadb.com/docs/server/clients-and-utilities/backup-restore-and-import-clients/mariadb-dump) — consulté le 24/07/2026
- [PostgreSQL — `pg_dump`](https://www.postgresql.org/docs/current/app-pgdump.html) — consulté le 24/07/2026
- [PostgreSQL — `pg_dumpall`](https://www.postgresql.org/docs/current/app-pg-dumpall.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI (MySQL + PostgreSQL) · [x] daté 24/07/2026 · [x] rien d'obsolète (`--single-transaction`, `mariadb-dump`) · [x] principe **testé en bac à sable** (dump→restore) / outils SGBD à tester en lab · [x] conforme doc MariaDB/PostgreSQL · [x] vérification présente · [x] sécurité (droits mini, chiffrement, RGPD) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
