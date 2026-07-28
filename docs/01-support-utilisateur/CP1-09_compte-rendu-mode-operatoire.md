# CP1-09 — Rédiger un compte rendu d'intervention et un mode opératoire

**Objectif** : produire un **compte rendu d'intervention** clair et un **mode opératoire** reproductible, à partir de gabarits réutilisables.

**Rattachement REAC** : CP1 « Assurer le support utilisateur » — savoir-faire : formaliser et transmettre.

**Durée** : ~15 min · **Niveau** : débutant · **Type** : Méthode.

---

## Prérequis

- Une intervention réalisée (à restituer) ou une procédure à formaliser.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Concepts / gabarits | Méthode | 24/07/2026 |

> Deux documents complémentaires : le **compte rendu** raconte **ce qui a été fait** (une fois) ; le **mode opératoire** explique **comment refaire** (à chaque fois). Voir aussi **CT1-01** pour la posture de communication.

---

## Procédure — Méthode

### 1. Gabarit — Compte rendu d'intervention

| Rubrique | Contenu |
|---|---|
| **Date / heure** | Quand (début/fin) |
| **Technicien / demandeur / n° ticket** | Traçabilité (**CP1-03**) |
| **Contexte** | Environnement, poste, service |
| **Symptôme** | Ce qui était constaté (factuel) |
| **Diagnostic** | Cause identifiée (**CT2-01**) |
| **Actions réalisées** | Ce qui a été fait, dans l'ordre |
| **Résultat** | Résolu / contourné / en cours |
| **Reste à faire** | Suites éventuelles |
| **Temps passé** | Durée |

### 2. Gabarit — Mode opératoire

1. **Objectif** (une phrase : ce que le lecteur saura faire).
2. **Prérequis** (droits, accès, outils).
3. **Étapes numérotées** (une action par étape, reproductibles ; captures si utile).
4. **Vérification** (comment confirmer que ça marche).
5. **En cas d'échec / retour arrière** (rollback).

### 3. Bonnes pratiques de rédaction

- **Factuel, daté, concis** ; **une action = une étape**.
- Rédigé pour **un tiers** : il doit pouvoir **reprendre**/**rejouer** sans vous.
- Consigné au bon endroit : **ticket** (compte rendu — **CP1-03**) et **base de connaissances** (mode opératoire — **CP1-05**).

---

## Vérification (comment savoir que c'est réussi)

- Un **collègue** comprend le compte rendu et pourrait **reprendre** le dossier.
- Le **mode opératoire** est **rejoué avec succès** par quelqu'un d'autre, sans vous.

## Dépannage (pièges fréquents)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Compte rendu inexploitable | Vague / non daté | Faits, dates, actions précises |
| Mode opératoire qui échoue | Étapes manquantes / non testé | **Faire tester** par un tiers |
| Trop long / illisible | Remplissage | Aller à l'essentiel, numéroter |
| Jargon | Public non ciblé | Adapter au lecteur (**CT1-01**) |

## Sécurité et bonnes pratiques

- **Pas d'identifiants/mots de passe** en clair dans les documents.
- **Traçabilité** : tout dans le ticket / la KB (pas de savoir « dans la tête »).
- **Versionner/dater** les modes opératoires (les tenir à jour — **CP1-05**).

## ⚠️ À ne pas confondre / obsolète

- **Compte rendu** (ce qui a été fait, ponctuel) ≠ **mode opératoire** (comment refaire, réutilisable).
- Un mode opératoire **non testé** ≈ pas de mode opératoire.
- Document **non daté/non versionné** = risque d'appliquer une procédure **périmée**.

---

## Sources (référence)

- [AXELOS — ITIL 4 (documentation et connaissance)](https://www.axelos.com/certifications/itil-service-management) — consulté le 24/07/2026
- [France compétences — REAC/RNCP37682 TSSR](https://www.francecompetences.fr/recherche/rncp/37682/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] Type Méthode (gabarits) · [x] daté 24/07/2026 · [x] rien d'obsolète · [x] non applicable au bac à sable (méthode) · [x] cohérent ITIL/REAC · [x] vérification présente (reprise/rejeu) · [x] sécurité (pas d'identifiants, traçabilité) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
