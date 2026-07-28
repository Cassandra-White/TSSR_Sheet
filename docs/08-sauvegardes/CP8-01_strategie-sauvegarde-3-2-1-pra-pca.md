# CP8-01 — Définir une stratégie de sauvegarde (3-2-1 ; totale/incrémentale/différentielle ; PRA/PCA)

**Objectif** : savoir bâtir une stratégie de sauvegarde robuste — règle **3-2-1(-1-0)**, choix des **types** de sauvegarde, objectifs **RPO/RTO**, articulation **PRA/PCA**.

**Rattachement REAC** : CP8 « Sauvegardes et restaurations des éléments de l'infrastructure » — savoir-faire : définir une politique de sauvegarde.

**Durée** : ~20 min · **Niveau** : intermédiaire · **Type** : Méthode.

---

## Prérequis

- Inventaire des données/services et de leur **criticité** (**CP4-11**).
- Un espace de stockage cible pour les sauvegardes (local + hors site).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Démonstration full/incr/diff | **tar** (Debian 13, bac à sable) | 24/07/2026 |
| Concepts (indépendants de l'OS) | Méthode | 24/07/2026 |

> ✅ *Sources reconfirmées le 24/07/2026 : la variante **3-2-1-1-0** (copie **immuable** + **0** erreur de restauration testée) est bien celle promue par le guide **#StopRansomware de la CISA**.*

---

## Procédure — Méthode

### 1. La règle 3-2-1 (socle)

- **3** copies des données (l'original + 2 sauvegardes),
- sur **2** supports **différents** (ex. disque interne + NAS/bande),
- dont **1 hors site** (autre bâtiment / Cloud).

**Extension moderne 3-2-1-1-0** (anti-rançongiciel) : **+1** copie **hors-ligne/immuable** (*air-gap*), **0** erreur → sauvegardes **vérifiées et testées**.

### 2. Les types de sauvegarde

| Type | Ce qui est copié | Place/temps | Restauration |
|---|---|---|---|
| **Totale** | Tout, à chaque fois | Lente, volumineuse | La plus simple/rapide (1 jeu) |
| **Incrémentale** | Ce qui a changé depuis **la dernière sauvegarde** (quelle qu'elle soit) | Rapide, peu de place | Full **+ toute la chaîne** d'incréments |
| **Différentielle** | Ce qui a changé depuis **la dernière totale** | Croît chaque jour | Full **+ la dernière diff** seulement |

Schéma courant : **totale hebdo + incrémentales (ou diff) quotidiennes** ; rotation **GFS** (Fils/Père/Grand-père) pour la rétention.

### 3. Fixer les objectifs RPO / RTO

- **RPO** (*Recovery Point Objective*) = **perte de données** maximale tolérée → **dicte la fréquence** (RPO 1 h ⇒ sauvegarde au moins horaire).
- **RTO** (*Recovery Time Objective*) = **temps de remise en service** maximal → **dicte la méthode/l'infra** (réplication, snapshots, matériel de secours).

### 4. PRA vs PCA

- **PCA** (Plan de **Continuité** d'Activité / BCP) : le service **ne s'arrête pas** (redondance, haute dispo).
- **PRA** (Plan de **Reprise** d'Activité / DRP) : le service **redémarre après sinistre** dans un délai défini (le RTO).

### 5. Démarche à appliquer

Inventorier les données critiques → classer par criticité → **fixer RPO/RTO par service** → choisir types + fréquence + supports (**3-2-1**) → **planifier + chiffrer** → **tester les restaurations** (**CP8-07**) → documenter.

## Démonstration (bac à sable — full / incrémentale / différentielle)

```bash
# J1 : sauvegarde TOTALE (snapshot d'état .snar)
tar --listed-incremental=snap.snar -cf full.tar -C data .

# J2 (b.txt modifié, c.txt ajouté)
# INCREMENTALE : repart de l'état de la veille  -> ne contient que b.txt + c.txt
# DIFFERENTIELLE : repart de l'état de la TOTALE -> b.txt + c.txt (et grossira J3, J4…)
```

> **Testé** : la totale contient `a.txt`+`b.txt` ; l'incrémentale et la différentielle du J2 ne contiennent que les **fichiers modifiés/ajoutés** (`b.txt`, `c.txt`). Restauration vérifiée : après extraction full **puis** diff, `b.txt = modif-j2` et `c.txt = nouveau`. Intégrité contrôlée par **SHA-256** de l'archive.

---

## Vérification (comment savoir que la stratégie tient)

- Une **restauration de test** aboutit dans le **RTO** visé, avec des données au plus vieilles que le **RPO**.
- Contrôle d'**intégrité** (checksum) des archives ; alertes en cas d'échec de job.
- La règle **3-2-1** est effectivement respectée (au moins 1 copie **hors site** et 1 **hors-ligne**).

## Dépannage (pièges fréquents)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Restauration impossible | Sauvegarde **jamais testée** | Tester régulièrement (**CP8-07**) |
| Chaîne incrémentale cassée | Un maillon manquant/corrompu | Préférer **différentielle** si restauration simple prioritaire |
| Rançongiciel chiffre aussi les sauvegardes | Pas de copie **hors-ligne/immuable** | Ajouter le **-1-0** (air-gap/immuable) |

## Sécurité et bonnes pratiques

- **Chiffrer** les sauvegardes (surtout hors site/Cloud) et **protéger l'accès** au dépôt.
- **Copie hors-ligne/immuable** = meilleure défense contre les rançongiciels.
- **Tester** les restaurations et **documenter** la procédure (temps réel = futur RTO).

## ⚠️ À ne pas confondre / obsolète

- **RAID ≠ sauvegarde** : le RAID protège d'une **panne disque**, pas d'une suppression/corruption/rançongiciel.
- **Incrémentale** (dernière sauvegarde) ≠ **différentielle** (dernière **totale**).
- Une **sauvegarde non testée** n'est **pas** une sauvegarde.

---

## Sources (doc officielle / référence)

- [CISA — #StopRansomware Guide (règle 3-2-1-1-0, sauvegardes immuables)](https://www.cisa.gov/stopransomware) — consulté le 24/07/2026
- [GNU tar — Incremental dumps](https://www.gnu.org/software/tar/manual/html_node/Incremental-Dumps.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] Type Méthode (concepts + démarche) · [x] daté 24/07/2026 · [x] rien d'obsolète (3-2-1-1-0, RAID≠backup) · [x] démonstration **testée en bac à sable** (tar full/incr/diff + restauration) · [x] **vérif web reconfirmée (24/07/2026)** · [x] vérification présente · [x] sécurité (chiffrement, immuable) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
