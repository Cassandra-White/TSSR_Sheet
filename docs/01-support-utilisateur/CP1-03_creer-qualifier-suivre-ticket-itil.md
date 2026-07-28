# CP1-03 — Créer, qualifier et suivre un ticket d'incident (cycle de vie ITIL)

**Objectif** : créer un **ticket d'incident** dans GLPI, le **qualifier** (catégorie, urgence, impact → priorité) et le **suivre** jusqu'à la résolution/clôture selon le cycle **ITIL**.

**Rattachement REAC** : CP1 « Assurer le support utilisateur » — savoir-faire : traiter un incident dans un outil de ticketing.

**Durée** : ~20 min · **Niveau** : débutant/intermédiaire.

---

## Prérequis

- Un **GLPI 11** opérationnel (**CP1-01**), des **utilisateurs** et des **catégories** d'incidents définies.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Outil | **GLPI 11** — module Assistance | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **Incident (ITIL)** = interruption ou dégradation d'un service. GLPI est **conforme ITIL** : **priorité = urgence × impact** (matrice **5×5**), statuts **standardisés** et **SLA**.

---

## Procédure — GUI

### 1. Créer le ticket

1. **Assistance ▸ Tickets ▸ +** : **demandeur**, **titre** clair, **description** (symptôme, depuis quand, reproductible — **CT2-01**).
2. Renseigner la **catégorie** et l'**urgence** (souvent fixée par le demandeur).

### 2. Qualifier (triage)

3. Le **technicien** évalue l'**impact** (nombre d'utilisateurs/criticité) → GLPI **calcule la priorité** (urgence × impact).
4. **Attribuer** le ticket à un technicien/groupe ; associer un **SLA** selon la priorité (P1…P5).

### 3. Suivre

5. Ajouter des **suivis** (communication avec le demandeur — **CT1-01**), des **tâches**, le **temps passé**.
6. Passer en **En attente** si l'on dépend d'un tiers (le SLA peut se suspendre).

### 4. Résoudre et clôturer

7. Saisir la **solution**, passer en **Résolu**.
8. Le **demandeur valide** → **Clos** (ou réouverture si non résolu).

> **Cycle des statuts** : **Nouveau → En cours (attribué/planifié) → En attente → Résolu → Clos**.

---

## Vérification (comment savoir que c'est bien fait)

- La **priorité** affichée correspond bien à **urgence × impact**.
- Le **SLA** et la prochaine **échéance/escalade** apparaissent sur le ticket.
- L'**historique** (suivis, tâches, temps) est complet ; le ticket n'est **clos** qu'après **validation**.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Priorité incohérente | Matrice urgence×impact mal réglée | Ajuster la **matrice de priorité** |
| SLA non déclenché | Règle métier absente | Lier **priorité → SLA** (règle métier) |
| Stats faussées | Tickets **sans catégorie** | Rendre la catégorie **obligatoire** |
| Ticket rouvert en boucle | Solution non validée par le demandeur | Documenter la solution, confirmer avant clôture |

## Sécurité et bonnes pratiques

- **Qualifier correctement** (catégorie/priorité) : indispensable pour les **SLA**, l'**escalade** et les **statistiques**.
- **Tout tracer dans le ticket** (suivis) — traçabilité et reprise par un collègue (**CT1-01**).
- **Documenter la solution** dans la **base de connaissances** (**CP1-05**) pour capitaliser.

## ⚠️ À ne pas confondre / obsolète

- **Incident** (rétablir le service) ≠ **demande de service** (*request*) ≠ **problème** (cause racine — **CT2-01**).
- **Urgence** (fixée par le demandeur) ≠ **impact** (évalué par le technicien).
- Ne **jamais clore** sans **validation** du demandeur (sauf règle définie).

---

## Sources (doc officielle)

- [GLPI — Gérer les tickets](https://help.glpi-project.org/documentation/modules/assistance/tickets/ticketmanagement) — consulté le 24/07/2026
- [GLPI — Cycle de vie d'un ticket](https://help.glpi-project.org/documentation/modules/assistance/tickets/ticketlifecycle) — consulté le 24/07/2026
- [GLPI — Matrice de calcul de la priorité](https://glpi-user-documentation.readthedocs.io/fr/latest/modules/assistance/prioritymatrix.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI · [x] version datée (GLPI 11) · [x] rien d'obsolète (ITIL, urgence×impact) · [x] procédure **à tester en lab** · [x] conforme doc GLPI · [x] vérification présente (priorité/SLA/historique) · [x] sécurité (qualification, traçabilité) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
