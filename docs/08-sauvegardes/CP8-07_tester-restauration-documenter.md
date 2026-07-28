# CP8-07 — Tester une restauration et documenter le test

**Objectif** : mettre en place un **test de restauration régulier**, dans un environnement isolé, et le **documenter** (fiche de test) pour garantir que les sauvegardes sont réellement exploitables.

**Rattachement REAC** : CP8 « Sauvegardes et restaurations des éléments de l'infrastructure » — savoir-faire : vérifier la restaurabilité et tracer les tests.

**Durée** : ~20 min · **Niveau** : intermédiaire · **Type** : Méthode.

---

## Prérequis

- Des sauvegardes existantes (**CP8-02 à CP8-06**) et leurs procédures de restauration.
- Un **environnement de test isolé** (VM/VLAN séparé) pour ne pas impacter la production.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Concepts (indépendants de l'OS) | Méthode | 24/07/2026 |

> **Règle** : *une sauvegarde non testée n'est pas une sauvegarde*. Le statut « job : Success » **ne prouve pas** qu'on peut **restaurer** : média corrompu, chaîne incrémentale cassée, procédure obsolète, RTO sous-estimé… seul un **test réel** le confirme.

---

## Procédure — Méthode

### 1. Choisir le type de test

| Test | Fréquence conseillée | Ce qu'il valide |
|---|---|---|
| **Restauration de fichier** | Mensuelle | Lisibilité des sauvegardes, procédure de base |
| **Restauration complète / DR** (VM, bare-metal, état AD) | Trimestrielle / annuelle | RTO réel, média de reprise, dépendances |
| **Bascule PRA** (site/infra de secours) | Annuelle | Plan de reprise de bout en bout |

### 2. Restaurer en environnement **isolé**

- Restaurer dans une **VM/VLAN de test** coupé de la production.
- **Cas AD** : ne **jamais** remettre un DC restauré sur le réseau de prod (risque **USN rollback** — **CP8-03**) ; tester dans un réseau cloisonné.

### 3. Mesurer et comparer

- Vérifier l'**intégrité** (checksums, nombre d'enregistrements, ouverture applicative).
- Chronométrer la restauration → c'est le **RTO réel** ; noter l'âge de la sauvegarde → **RPO réel**.

### 4. Documenter — fiche de test

| Champ | Exemple |
|---|---|
| Date / testeur | 24/07/2026 — J. Dupont |
| Système testé | Serveur de fichiers `SRV-FIC` |
| Type de test | Restauration de volume |
| Sauvegarde utilisée (date) | 23/07/2026 22:00 (RPO ≈ 12 h) |
| Durée réelle (RTO) | 41 min (objectif : 1 h ✅) |
| Résultat | Réussi — données intègres |
| Écarts / incidents | Pilote réseau manquant sur le média |
| Actions correctives | Régénérer le média avec pilotes |

### 5. Améliorer

Corriger la **procédure** et les **paramètres** (fréquence, média, RTO/RPO cibles) d'après les écarts constatés, puis re-tester.

---

## Vérification (comment savoir que le test est concluant)

- Les données restaurées sont **complètes et intègres** (comparaison / checksums).
- Le **RTO mesuré ≤ objectif** et la sauvegarde est plus récente que le **RPO**.
- La **fiche de test** est renseignée et archivée (traçabilité audit/examen).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Restauration impossible | Média/chaîne corrompus | Changer de jeu ; revoir la stratégie (**CP8-01**) |
| Test qui percute la prod | Environnement non isolé | Restaurer dans une **VM/VLAN de test** |
| RTO dépassé | Méthode/infra inadaptées | Réplication, snapshots, matériel de secours |
| Procédure fausse le jour J | Jamais testée/à jour | Tester **périodiquement** et **documenter** |

## Sécurité et bonnes pratiques

- **Isoler** l'environnement de test (surtout AD/DC).
- **Purger** les données restaurées après le test (elles peuvent être sensibles — RGPD, **CT4-01**).
- **Planifier** les tests dans le calendrier d'exploitation ; ne pas les reporter indéfiniment.

## ⚠️ À ne pas confondre / obsolète

- « **Job réussi** » ≠ « **restauration possible** » : seul le test le prouve.
- **Test de fichier** (partiel) ≠ **test complet/DR** (reprise réelle) : les deux sont nécessaires.
- Une procédure de restauration **non documentée** ≈ pas de procédure.

---

## Sources (doc officielle / référence)

- [ANSSI — Sauvegardes et plan de reprise](https://cyber.gouv.fr/publications) — consulté le 24/07/2026
- [Veeam — Restore & SureBackup (tests de restauration)](https://helpcenter.veeam.com/docs/backup/vsphere/surebackup_overview.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] Type Méthode (démarche + fiche de test) · [x] daté 24/07/2026 · [x] rien d'obsolète (job≠restore) · [x] non applicable au bac à sable (méthode) — à exécuter en lab · [x] cohérent doc ANSSI/Veeam · [x] vérification présente · [x] sécurité (isolation, purge, RGPD) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
