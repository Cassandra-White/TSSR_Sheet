# CP5-09 — Administrer une messagerie/bureautique en ligne (Microsoft 365 / Google Workspace)

**Objectif** : administrer une suite bureautique en ligne — créer des **utilisateurs**, attribuer des **licences**, gérer **groupes** et **sécurité** (MFA, DNS mail).

**Rattachement REAC** : CP5 « Maintenir des serveurs dans une infrastructure virtualisée » — savoir-faire : administrer des services Cloud.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un accès **administrateur** à un tenant **Microsoft 365** ou **Google Workspace**.
- Le **domaine** de l'organisation (pour la configuration DNS/mail).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Suite | **Microsoft 365** (admin center) | 24/07/2026 |
| Suite | **Google Workspace** (Admin Console) | 24/07/2026 |
| Procédure appliance | **à tester en lab (tenant)** | 24/07/2026 |

> ⚠️ **Actualité 2026** : depuis le **3 février 2026**, la **MFA est obligatoire** pour se connecter au **centre d'administration Microsoft 365**. Sans MFA configurée, l'admin **ne peut plus se connecter**.

---

## Procédure — GUI

### Microsoft 365 (admin.microsoft.com)

1. **Utilisateurs ▸ Utilisateurs actifs ▸ Ajouter** : nom, identifiant, mot de passe.
2. **Attribuer une licence** (Business/Enterprise) à l'utilisateur.
3. **Groupes** (Microsoft 365 / sécurité / distribution) pour partages et droits.
4. **Sécurité** : **MFA / Accès conditionnel** (Entra — **CP9-09**), **rôles d'admin** au **moindre privilège**.
5. **Domaine/DNS** : enregistrements **MX** + **SPF/DKIM/DMARC** pour le mail (**CP7-14**).

### Google Workspace (admin.google.com)

1. **Répertoire ▸ Utilisateurs ▸ Ajouter**, rattacher à une **unité organisationnelle**.
2. **Licences** et **groupes**.
3. **Sécurité** : **validation en 2 étapes**, politiques Gmail/Drive/Meet, **dates d'expiration** pour les prestataires.

---

## Vérification (comment savoir que ça marche)

- Le nouvel utilisateur **se connecte** (avec **MFA**), sa **licence** est active, la **messagerie** fonctionne (envoi/réception).
- Les enregistrements **MX/SPF/DKIM/DMARC** sont **valides** (pas de rejet des mails).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Admin ne peut plus se connecter | **MFA obligatoire** (M365, fév. 2026) | Enrôler une méthode MFA |
| Mails rejetés/en spam | **SPF/DKIM/DMARC** manquants | Publier les enregistrements DNS |
| App non disponible | **Licence** non attribuée | Assigner la licence adaptée |
| Accès trop large | Rôle admin trop élevé | **Moindre privilège** ; comptes admin dédiés |

## Sécurité et bonnes pratiques

- **MFA pour tous** (obligatoire pour les **admins**) — **CP7-18**.
- **Comptes d'administration dédiés** et **moindre privilège** ; compte **break-glass** de secours.
- **Anti-usurpation mail** : **SPF + DKIM + DMARC** ; conformité **RGPD** (données Cloud — **CT4-01**).

## ⚠️ À ne pas confondre / obsolète

- Administration **sans MFA** = **bloquée** sur M365 (depuis fév. 2026).
- **Groupe M365** (collaboration) ≠ **groupe de sécurité** (droits) ≠ **liste de distribution** (mail).
- Service **Cloud** ≠ dispense de sécurité : la **responsabilité** reste partagée (données, accès).

---

## Sources (doc officielle)

- [Microsoft Learn — Centre d'administration Microsoft 365](https://learn.microsoft.com/en-us/microsoft-365/admin/) — consulté le 24/07/2026
- [Microsoft — MFA obligatoire pour l'admin M365](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-mandatory-multifactor-authentication) — consulté le 24/07/2026
- [Google Workspace — Admin Help](https://support.google.com/a/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI (M365 + Workspace) · [x] daté 24/07/2026 · [x] rien d'obsolète (**MFA admin obligatoire 2026**) · [x] procédure **à tester en lab (tenant)** · [x] conforme doc Microsoft/Google · [x] vérification présente (connexion/mail) · [x] sécurité (MFA, moindre privilège, DMARC) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
