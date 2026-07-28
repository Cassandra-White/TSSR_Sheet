# CT2-01 — Mettre en œuvre une démarche structurée de résolution de problème

**Objectif** : diagnostiquer et résoudre un incident **avec méthode** (pas à l'intuition) : cadrer, formuler et tester des hypothèses, trouver la **cause racine**, documenter.

**Rattachement REAC** : Compétences transversales — « Mettre en œuvre une démarche de résolution de problème ».

**Durée** : ~20 min · **Niveau** : intermédiaire · **Type** : Méthode.

---

## Prérequis

- Un incident à traiter et l'accès aux **journaux**/outils de diagnostic.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Concepts / démarche | Méthode | 24/07/2026 |

> **Incident** (ITIL) = **rétablir le service vite** (contournement accepté). **Problème** = **trouver la cause racine** pour éviter la récurrence. Les deux se traitent, mais avec des objectifs différents.

---

## Procédure — démarche structurée

### 1. Cadrer le problème (QQOQCP)

**Quoi, Qui, Où, Quand, Comment, Combien, Pourquoi** : symptôme précis, périmètre (1 poste ou tous ?), **depuis quand**, **reproductible** ?

### 2. Recueillir les faits

- **Journaux**, messages d'erreur exacts, et surtout : **« qu'est-ce qui a changé ? »** (mise à jour, config, matériel récent).

### 3. Formuler et tester des hypothèses — **une variable à la fois**

- **Diviser pour régner** / approche **par couches** (modèle **OSI** pour le réseau — **CP4-07**) pour **isoler**.
- Ne **changer qu'une chose** à la fois et **observer** l'effet.

### 4. Analyser la cause racine (RCA)

| Outil | À quoi il sert |
|---|---|
| **QQOQCP** | Cadrer et décrire le problème |
| **5 pourquoi** | Creuser la chaîne de causes (problème simple) |
| **Ishikawa** (arêtes de poisson) | Catégoriser les causes possibles (Matériel, Méthode, Main-d'œuvre, Milieu, Moyens, Mesure) |
| **Diviser pour régner / OSI** | Isoler la zone en cause |

### 5. Corriger, vérifier, documenter

- Appliquer la solution, **vérifier** le rétablissement, **documenter** dans la **base de connaissances** (**CP1-05**).

### 6. Prévenir la récurrence

- Action **corrective durable** (pas seulement le contournement) ; suivre si le problème réapparaît.

---

## Vérification (comment savoir que c'est réussi)

- Le service est **rétabli** **et** la **cause racine** est identifiée (pas seulement le symptôme).
- La résolution est **documentée** et **reproductible** ; le problème **ne réapparaît pas**.

## Dépannage (pièges fréquents)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Le problème revient | Symptôme traité, **pas la cause** | Aller jusqu'à la **cause racine** (5 pourquoi) |
| Impossible de savoir ce qui a marché | **Plusieurs changements** en même temps | **Une variable à la fois** |
| Perte de temps | Pas de cadrage | Commencer par **QQOQCP** |
| Fausse piste | **Biais de confirmation** | Tester objectivement, envisager d'autres causes |

## Sécurité et bonnes pratiques

- **Sauvegarder/snapshot avant** de modifier (**CP8-10**) ; privilégier des changements **réversibles**.
- **Un changement à la fois**, tracé ; **communiquer** l'avancement (**CT1-01**).
- Documenter pour **capitaliser** (KB) et accélérer les prochaines fois.

## ⚠️ À ne pas confondre / obsolète

- **Incident** (rétablir vite) ≠ **problème** (cause racine, ITIL).
- **Symptôme** (ce qu'on voit) ≠ **cause** (l'origine réelle).
- Bidouille **à l'intuition non tracée** → démarche **structurée + documentée**.

---

## Sources (référence)

- [AXELOS — ITIL 4 : Problem Management / RCA](https://www.axelos.com/certifications/itil-service-management) — consulté le 24/07/2026
- [ASQ — Fishbone (Ishikawa) & 5 Whys](https://asq.org/quality-resources/fishbone) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] Type Méthode · [x] daté 24/07/2026 · [x] rien d'obsolète (RCA, ITIL 4) · [x] non applicable au bac à sable (méthode) · [x] cohérent ITIL/ASQ · [x] vérification présente (cause racine + non-récurrence) · [x] sécurité (snapshot, un changement à la fois) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
