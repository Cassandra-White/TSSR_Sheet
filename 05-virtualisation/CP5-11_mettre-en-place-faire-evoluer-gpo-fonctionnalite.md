# CP5-11 — Mettre en place / faire évoluer une GPO ou une fonctionnalité serveur (demande d'évolution)

**Objectif** : traiter une **demande d'évolution** — mettre en place ou modifier une **GPO** ou une **fonctionnalité serveur** — avec une démarche de **gestion du changement**.

**Rattachement REAC** : CP5 « Maintenir des serveurs dans une infrastructure virtualisée » — savoir-faire : faire évoluer une configuration en production de façon maîtrisée.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un domaine **AD** (**CP2-03**), un environnement de **test** (OU pilote / lab).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Serveur | **Windows Server 2025** (GPO/rôles) | 24/07/2026 |
| Démarche | Gestion du changement (ITIL) | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> Une **demande d'évolution** ne s'applique **jamais directement en production** : on **qualifie**, on **teste**, on **planifie** (avec **rollback**), on **implémente**, on **vérifie**, on **documente** et on **communique**.

---

## Procédure — démarche (change enablement)

### 1. Qualifier la demande

- Comprendre le **besoin réel** (QQOQCP — **CP1-08**), le **périmètre** et l'**objectif**.

### 2. Évaluer impact et risque

- **Qui/quoi** est affecté, **réversibilité**, fenêtre possible, dépendances.

### 3. Tester en préproduction / lab

- Appliquer d'abord à une **OU pilote** (GPO) ou sur un **serveur de test**.

### 4. Planifier

- **Fenêtre de changement**, **plan de retour arrière** (snapshot/sauvegarde — **CP8-10**), **communication** aux utilisateurs.

### 5. Implémenter

- **GPO** : créer/modifier, **lier** à la bonne OU, gérer **filtrage** (sécurité/WMI) et **précédence** (**CP2-11/CP2-17**).
- **Fonctionnalité serveur** :

```powershell
Install-WindowsFeature -Name <RoleOuFeature> -IncludeManagementTools
```

### 6. Vérifier, documenter, communiquer

- `gpresult /h` / tests fonctionnels ; **documenter** (**CP1-09**) ; **informer** (**CT1-01**) ; clôturer/suivre.

---

## Vérification (comment savoir que c'est réussi)

- La demande est **satisfaite**, **testée**, **sans régression** ; la GPO/le rôle s'applique là où il faut (`gpresult`).
- Le **changement est tracé** (qui/quoi/quand) et **communiqué**.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Régression en prod | **Non testé** en amont | Toujours **piloter** avant (OU/lab) |
| GPO non appliquée | Lien/filtrage/précédence | Vérifier l'OU, le filtrage, l'ordre (**CP2-11**) |
| Impossible de revenir | Pas de **rollback** | Snapshot/sauvegarde **avant** (**CP8-10**) |
| Utilisateurs surpris | Pas de **communication** | Prévenir avant, expliquer après (**CT1-01**) |

## Sécurité et bonnes pratiques

- **Tester avant la production** ; **snapshot/sauvegarde** avant (**CP8-10**) ; **rollback** prêt.
- **Un changement à la fois** et **documenté** (**CT2-01/CP1-09**).
- Respecter la **fenêtre de changement** et la **communication** (**CT1-01**).

## ⚠️ À ne pas confondre / obsolète

- **Demande d'évolution** (changement planifié) ≠ **incident** (rétablir un service — **CP1-03**).
- **Modifier en prod sans test** = pratique à proscrire → **gestion du changement**.
- **GPO** (config du parc) ≠ **fonctionnalité/rôle** serveur (capacité du serveur).

---

## Sources (doc officielle / référence)

- [Microsoft Learn — Group Policy](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/group-policy/group-policy-overview) — consulté le 24/07/2026
- [Microsoft Learn — Install-WindowsFeature](https://learn.microsoft.com/en-us/powershell/module/servermanager/install-windowsfeature) — consulté le 24/07/2026
- [AXELOS — ITIL 4 : Change Enablement](https://www.axelos.com/certifications/itil-service-management) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] Démarche + CLI (GPO/rôle) · [x] version datée (WS 2025) · [x] rien d'obsolète (gestion du changement) · [x] procédure **à tester en lab** · [x] conforme doc Microsoft/ITIL · [x] vérification présente (`gpresult`) · [x] sécurité (test, snapshot, rollback) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
