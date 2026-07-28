# CP1-04 — Configurer les SLA, priorités et l'escalade dans GLPI

**Objectif** : définir des **SLA** (et **OLA**) avec délais **TTO/TTR**, un **calendrier** d'heures ouvrées, des **niveaux d'escalade**, et les lier à la **priorité** des tickets.

**Rattachement REAC** : CP1 « Assurer le support utilisateur » — savoir-faire : appliquer les engagements de service.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un **GLPI 11** opérationnel, des **priorités** de tickets configurées (**CP1-03**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Outil | **GLPI 11** — Niveaux de service | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **SLA** = engagement **fournisseur ↔ client** ; **OLA** = engagement **interne entre équipes**. Les deux mesurent un **TTO** (*Time To Own* : délai avant **attribution**) et un **TTR** (*Time To Resolve* : délai avant **résolution**).

---

## Procédure — GUI

### 1. Créer un SLA

1. **Configuration ▸ Niveaux de service ▸ +** : nommer (ex. *P1 — Critique*), définir un **TTR** (ex. 4 h) et/ou un **TTO**.
2. Associer un **calendrier** d'heures ouvrées (sinon le calcul se fait **24/7** par défaut).

### 2. Ajouter des niveaux d'escalade

3. Dans le SLA, **ajouter un niveau d'escalade** déclenché **avant** ou **après** l'échéance (ex. **−1 h** avant le TTR).
4. Configurer ses **critères** (optionnels) et ses **actions** (notifier le responsable, réattribuer au N2…).

### 3. Lier la priorité au SLA (règle métier)

5. **Configuration ▸ Règles ▸ Règles métier sur les tickets** : *si priorité = 5 → assigner SLA « P1 »*, etc. → application **automatique** à la création.

### 4. Définir le calendrier

6. **Configuration ▸ Calendriers** : jours/heures ouvrés + jours fériés → les délais SLA se calculent sur ce temps de travail.

---

## Vérification (comment savoir que ça marche)

- Un ticket **P1** reçoit automatiquement le **SLA P1** ; l'**échéance TTR** s'affiche sur le ticket.
- À l'approche de l'échéance, le **niveau d'escalade** se déclenche (notification/réattribution).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Délais comptés **24/7** | Aucun **calendrier** associé | Associer un calendrier d'heures ouvrées |
| SLA non appliqué | Règle métier absente/mauvais ordre | Vérifier la **règle** priorité→SLA |
| Escalade jamais déclenchée | Critères trop restrictifs | Revoir les **critères** du niveau |
| TTO/TTR confondus | Mauvais délai réglé | **TTO** = attribution, **TTR** = résolution |

## Sécurité et bonnes pratiques

- **Calendrier réaliste** (heures ouvrées réelles) pour des délais crédibles.
- Escalade vers **les bonnes personnes** (manager, N2) avec **notifications**.
- Suivre les **indicateurs SLA** (respect/dépassement) pour piloter le service.

## ⚠️ À ne pas confondre / obsolète

- **SLA** (externe, client) ≠ **OLA** (interne, équipes).
- **TTO** (*Time To Own*) ≠ **TTR** (*Time To Resolve*).
- Sans **calendrier**, GLPI compte en **24/7** (délais faussés).

---

## Sources (doc officielle)

- [GLPI — Service Levels (SLA/OLA)](https://help.glpi-project.org/documentation/modules/configuration/service_levels) — consulté le 24/07/2026
- [GLPI — Tutoriel : configurer les SLA](https://help.glpi-project.org/tutorials/helpdesk/service_levels) — consulté le 24/07/2026
- [GLPI — OLA Management (spécification)](https://github.com/glpi-project/spec/wiki/OLA-Management) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI · [x] version datée (GLPI 11) · [x] rien d'obsolète (SLA/OLA, TTO/TTR) · [x] procédure **à tester en lab** · [x] conforme doc GLPI · [x] vérification présente (échéance/escalade) · [x] sécurité (calendrier, escalade ciblée) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
