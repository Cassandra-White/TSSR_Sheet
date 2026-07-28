# CP9-08 — Gérer le déploiement dans un contexte BYOD

**Objectif** : cadrer l'usage d'appareils **personnels** (BYOD) pour accéder aux ressources de l'entreprise — arbitrer **MDM vs MAM**, séparer **perso/pro**, respecter la **vie privée (RGPD)**.

**Rattachement REAC** : CP9 « Exploiter et maintenir les services de déploiement des postes » — savoir-faire : gérer les terminaux dans un contexte BYOD.

**Durée** : ~20 min · **Niveau** : intermédiaire · **Type** : Méthode.

---

## Prérequis

- Une politique de sécurité et un outil de gestion (ex. **Intune** — **CP9-09**).
- L'implication des **RH/juridique** (charte, RGPD).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Concepts / démarche | Méthode | 24/07/2026 |
| Mise en œuvre | via **Intune** (**CP9-09**) — à tester en lab | 24/07/2026 |

> **BYOD** = *Bring Your Own Device* : l'appareil appartient au salarié. Le défi : **protéger les données pro** **sans** empiéter sur la **vie privée**. Deux stratégies : **MDM** (on gère l'**appareil**) et **MAM** (on protège seulement les **applications** pro).

---

## Procédure — Méthode

### 1. MDM ou MAM ? (le choix structurant)

| Approche | Ce qui est géré | Vie privée | Quand l'utiliser |
|---|---|---|---|
| **MDM** (gestion de l'appareil) | Enrôlement + **profil de travail** Android / **User Enrollment** iOS | Conteneur pro séparé ; l'admin ne voit pas le perso | Besoin de **certificats**, push d'apps, conformité appareil |
| **MAM** (gestion des applications) | Seulement les **apps pro** (Outlook, Teams) via *App Protection Policies* | **Aucune** visibilité sur le perso | BYOD léger, **priorité vie privée** |

### 2. Démarche de déploiement

1. **Charte BYOD** : périmètre, responsabilités, appareils/OS supportés, conditions.
2. **Classer** les données/accès autorisés depuis un appareil perso (limiter aux moins sensibles).
3. **Choisir MDM ou MAM** selon les populations et le besoin.
4. **Accès conditionnel** (**CP9-09**) : n'autoriser que les appareils **conformes** + **MFA**.
5. **RGPD** : séparation stricte perso/pro, **pas de surveillance** des données personnelles, **effacement sélectif** (wipe du **seul** conteneur pro), information/consentement du salarié.
6. **Sensibiliser** (**CP1-10**) et prévoir la procédure **départ/perte** (retrait des accès pro).

---

## Vérification (comment savoir que c'est maîtrisé)

- Un appareil **non conforme** se voit **refuser** l'accès aux ressources pro.
- Un **effacement pro** (wipe sélectif) retire les données de l'entreprise **sans** toucher photos/contacts personnels.
- La charte est **signée** et les accès BYOD sont tracés.

## Dépannage (pièges fréquents)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Refus des utilisateurs (vie privée) | MDM total perçu comme intrusif | Basculer en **MAM** / Work Profile (conteneur) |
| Données pro sur appareil perdu | Pas de wipe sélectif | Activer l'**effacement du conteneur pro** |
| OS non géré | Terminal trop ancien/non supporté | Restreindre la liste des OS autorisés |

## Sécurité et bonnes pratiques

- **Zero Trust** : la confiance dépend de la **conformité** de l'appareil, pas du réseau.
- **Chiffrement**, **MFA**, **code PIN** sur les apps pro ; **wipe sélectif** en cas de perte/départ.
- **Moindre accès** : n'exposer au BYOD que le nécessaire.
- **RGPD/CNIL** : documenter la séparation perso/pro et l'absence de surveillance du perso.

## ⚠️ À ne pas confondre / obsolète

- **MDM** (gère l'appareil) ≠ **MAM** (protège les apps) : le MAM respecte mieux la **vie privée** en BYOD.
- **BYOD** (appareil du salarié) ≠ **COPE/COBO** (appareil d'entreprise).
- Un **MDM total** sur un appareil **personnel** est **intrusif et juridiquement risqué** → préférer **Work Profile/MAM**.

---

## Sources (doc officielle / référence)

- [Microsoft Learn — MAM vs Work Profiles (Android) / App Protection](https://learn.microsoft.com/en-us/intune/app-management/protection/mam-vs-work-profiles-android) — consulté le 24/07/2026
- [Microsoft Learn — App Protection Policies (overview)](https://learn.microsoft.com/en-us/intune/app-management/protection/overview) — consulté le 24/07/2026
- [CNIL — BYOD et vie privée au travail](https://www.cnil.fr/fr/byod-quelles-sont-les-bonnes-pratiques) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] Type Méthode (MDM/MAM + démarche) · [x] daté 24/07/2026 · [x] rien d'obsolète (Zero Trust, MAM) · [x] mise en œuvre **à tester en lab** (Intune) · [x] cohérent doc Microsoft/CNIL · [x] vérification présente · [x] sécurité (vie privée, wipe sélectif, RGPD) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
