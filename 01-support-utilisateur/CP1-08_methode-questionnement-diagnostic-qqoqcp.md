# CP1-08 — Appliquer une méthode de questionnement et de diagnostic d'incident (QQOQCP)

**Objectif** : collecter l'information utile auprès de l'utilisateur avec la méthode **QQOQCP** pour **cadrer** un incident et préparer sa résolution.

**Rattachement REAC** : CP1 « Assurer le support utilisateur » — savoir-faire : diagnostiquer un incident par le questionnement.

**Durée** : ~15 min · **Niveau** : débutant · **Type** : Méthode.

---

## Prérequis

- Un incident signalé par un utilisateur (téléphone, ticket, guichet).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Concepts / démarche | Méthode | 24/07/2026 |

> **QQOQCP** = **Q**uoi, **Q**ui, **O**ù, **Q**uand, **C**omment, **C**ombien, **P**ourquoi. Un cadre simple pour **ne rien oublier** et transformer un « ça ne marche pas » en **description exploitable**.

---

## Procédure — Méthode

### 1. Dérouler le QQOQCP

| Question | À obtenir de l'utilisateur |
|---|---|
| **Quoi ?** | Le problème exact, le **message d'erreur** (au mot près) |
| **Qui ?** | Qui est touché : **1 utilisateur** ou **tout le monde** ? |
| **Où ?** | Quel **poste/application/site/réseau** ? |
| **Quand ?** | **Depuis quand**, à quel moment précis, **reproductible** ? |
| **Comment ?** | Comment ça se manifeste, comment le **reproduire** ? |
| **Combien ?** | **Fréquence**, nombre d'utilisateurs, **impact** métier |
| **Pourquoi ?** | **Qu'est-ce qui a changé** récemment (MAJ, config, matériel) ? |

### 2. Bien questionner

- **Questions ouvertes** (« décrivez-moi… ») plutôt que fermées.
- **Ne pas induire** la réponse ; **reformuler** pour valider (**CT1-01**).
- **Faire reproduire** si possible (observer plutôt que supposer).

### 3. Exploiter

- Ces éléments **qualifient le ticket** (catégorie, **urgence/impact** — **CP1-03**) et alimentent la **démarche structurée de résolution** (**CT2-01**).

---

## Vérification (comment savoir que c'est réussi)

- On dispose d'assez d'éléments pour **formuler des hypothèses** (pas juste « ça bugue »).
- Le **périmètre** (1 poste / tous) et le **« depuis quand / qu'est-ce qui a changé »** sont connus.

## Dépannage (pièges fréquents)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Diagnostic dans le vide | Questions **fermées/orientées** | Ouvrir les questions, laisser décrire |
| On tourne en rond | « Qu'est-ce qui a changé ? » oublié | Toujours chercher le **changement récent** |
| Mauvaise piste | Supposition non vérifiée | **Faire reproduire** le problème |
| Info perdue | Rien noté | Consigner dans le **ticket** (**CP1-03**) |

## Sécurité et bonnes pratiques

- **Ne jamais demander le mot de passe** de l'utilisateur (aucun technicien n'en a besoin).
- **Confidentialité** : ne recueillir que le nécessaire (**CT4-01**).
- **Rassurer** l'utilisateur et rester **factuel** (**CT1-01**).

## ⚠️ À ne pas confondre / obsolète

- **Cadrer** (QQOQCP, décrire) ≠ **résoudre** (tester des hypothèses, **CT2-01**).
- **Symptôme rapporté** ≠ **cause** : creuser avant de conclure.
- Diagnostic **à l'intuition** → **questionnement structuré**.

---

## Sources (référence)

- [AXELOS — ITIL 4 (gestion des incidents)](https://www.axelos.com/certifications/itil-service-management) — consulté le 24/07/2026
- [France compétences — REAC/RNCP37682 TSSR](https://www.francecompetences.fr/recherche/rncp/37682/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] Type Méthode · [x] daté 24/07/2026 · [x] rien d'obsolète · [x] non applicable au bac à sable (méthode) · [x] cohérent ITIL/REAC · [x] vérification présente · [x] sécurité (pas de mot de passe, confidentialité) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
