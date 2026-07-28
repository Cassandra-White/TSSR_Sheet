# CP1-05 — Alimenter et utiliser la base de connaissances (KB)

**Objectif** : créer et organiser des articles de **base de connaissances** GLPI, publier une **FAQ**, et **capitaliser** les solutions de tickets pour gagner du temps.

**Rattachement REAC** : CP1 « Assurer le support utilisateur » — savoir-faire : documenter et réutiliser les solutions.

**Durée** : ~15 min · **Niveau** : débutant.

---

## Prérequis

- Un **GLPI 11** opérationnel, des **catégories** de connaissances définies.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Outil | **GLPI 11** — Base de connaissances | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> La **base de connaissances (KB)** capitalise le savoir de l'équipe : un incident **résolu une fois** devient un **article réutilisable**. La partie **FAQ publique** permet aux utilisateurs de **se dépanner seuls** (moins de tickets).

---

## Procédure — GUI

### 1. Créer un article

1. **Outils ▸ Base de connaissances ▸ +** : **titre** clair, **contenu** (procédure, captures), **catégorie**.
2. Définir la **cible** : **interne** (techniciens) ou **FAQ publique** (utilisateurs, self-service).

### 2. Capitaliser depuis un ticket

3. Sur un **ticket résolu**, utiliser **« Enregistrer comme solution dans la base de connaissances »** → l'article est créé à partir de la solution.

### 3. Réutiliser

4. Pendant le traitement d'un ticket, **rechercher** un article et le **lier** au ticket (voire l'envoyer au demandeur).

### 4. Organiser et maintenir

5. Classer par **catégories**, tenir à jour (**réviser** les articles obsolètes).

---

## Vérification (comment savoir que ça marche)

- L'article est **trouvable** en recherche et, s'il est public, apparaît dans la **FAQ**.
- Un nouveau ticket similaire se **résout plus vite** grâce à l'article lié.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Article invisible aux utilisateurs | Cible **interne** au lieu de **publique** | Publier en **FAQ** |
| Introuvable en recherche | Titre/mots-clés pauvres | Titre explicite + termes recherchés |
| Solution périmée appliquée | Article **non révisé** | Réviser régulièrement (dates, versions) |
| KB en vrac | Pas de **catégories** | Structurer par catégorie |

## Sécurité et bonnes pratiques

- **Réviser** régulièrement (une KB périmée diffuse de **fausses** solutions — cf. dépréciations WSUS/WDS/MDT **CP9**).
- **Ne pas publier** d'informations sensibles (identifiants, données personnelles) en **FAQ publique**.
- **Capitaliser systématiquement** les incidents récurrents → moins de tickets, réponses homogènes.

## ⚠️ À ne pas confondre / obsolète

- **KB interne** (techniciens) ≠ **FAQ publique** (utilisateurs).
- Un article **non maintenu** = risque de **mauvaise** manipulation.
- La KB **complète** le ticketing (**CP1-03**) : résolution **+** capitalisation.

---

## Sources (doc officielle)

- [GLPI — Base de connaissances](https://help.glpi-project.org/documentation/modules/tools/knowledgebase) — consulté le 24/07/2026
- [GLPI — Documentation (assistance/outils)](https://help.glpi-project.org/documentation/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI · [x] version datée (GLPI 11) · [x] rien d'obsolète (révision KB) · [x] procédure **à tester en lab** · [x] conforme doc GLPI · [x] vérification présente (recherche/FAQ) · [x] sécurité (révision, pas de données sensibles) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
