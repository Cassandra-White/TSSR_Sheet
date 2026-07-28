# CP5-10 — Gérer le stockage et le partage en Cloud (SharePoint/OneDrive ou Nextcloud)

**Objectif** : mettre en place un **stockage/partage de fichiers en Cloud** — **SharePoint/OneDrive** (Microsoft 365) ou **Nextcloud** (auto-hébergé) — avec droits, partage et synchronisation.

**Rattachement REAC** : CP5 « Maintenir des serveurs dans une infrastructure virtualisée » — savoir-faire : administrer le stockage/partage Cloud.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un tenant **Microsoft 365** (**CP5-09**), **ou** une VM Linux pour **Nextcloud** (**CP5-02/CP3-01**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Cloud public | **SharePoint / OneDrive** (M365) | 24/07/2026 |
| Auto-hébergé | **Nextcloud** (Docker) | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **Deux modèles** : **SaaS** (SharePoint/OneDrive — simple, mais données chez l'éditeur) ou **auto-hébergé** (**Nextcloud** — **souveraineté/RGPD**, données chez soi). Dans les deux cas : **partage contrôlé** + **synchronisation** multi-appareils.

---

## Procédure — GUI

### Microsoft 365 (SharePoint / OneDrive)

1. **Créer un site SharePoint** (bibliothèque de documents d'équipe) ; **OneDrive** = espace personnel.
2. **Partager** : lien **interne/externe**, droits **lecture/écriture**, **expiration** et **mot de passe**.
3. Installer le **client de synchronisation OneDrive** sur les postes.

### Nextcloud (auto-hébergé)

1. **Déployer Nextcloud** (Docker Compose) sur une VM Linux, derrière **HTTPS** (**CP3-13/CP7-11**).
2. Créer **utilisateurs/groupes**, **dossiers partagés**, **quotas** (**STO-13**).
3. **Partager par lien** (mot de passe + expiration) ; installer le **client de sync** (desktop/mobile).

---

## Vérification (comment savoir que ça marche)

- Un fichier **partagé** est accessible avec **les bons droits** ; un **lien externe** protégé demande le mot de passe.
- La **synchronisation** propage les modifications sur les appareils.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Partage externe impossible | Politique de partage restreinte | Autoriser le partage externe (contrôlé) |
| Conflits de synchro | Édition simultanée | Gérer les versions ; co-édition Office online |
| Nextcloud en HTTP | Pas de TLS | Mettre un **reverse proxy HTTPS** |
| Espace saturé | Quotas | Ajuster les **quotas** (**STO-13**) |

## Sécurité et bonnes pratiques

- **HTTPS/chiffrement** partout ; **MFA** (**CP7-18**) sur les accès.
- **Partages externes contrôlés** : **expiration** + **mot de passe** ; éviter les liens « tout le monde ».
- **RGPD/souveraineté** (**CT4-01**) : Nextcloud garde les données **en interne** ; en SaaS, vérifier la localisation/traitement.

## ⚠️ À ne pas confondre / obsolète

- **Pièce jointe par mail** (lourd, non contrôlé) → **lien de partage** (droits, expiration).
- **SaaS** (données éditeur) ≠ **auto-hébergé** (souveraineté).
- **Partage** (lien/droits) ≠ **synchronisation** (copie locale à jour).

---

## Sources (doc officielle)

- [Microsoft Learn — SharePoint / OneDrive (admin)](https://learn.microsoft.com/en-us/sharepoint/) — consulté le 24/07/2026
- [Nextcloud — Documentation / Installation](https://docs.nextcloud.com/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI (M365 + Nextcloud) · [x] daté 24/07/2026 · [x] rien d'obsolète (lien vs PJ, souveraineté) · [x] procédure **à tester en lab** · [x] conforme doc Microsoft/Nextcloud · [x] vérification présente (partage/sync) · [x] sécurité (HTTPS, MFA, partages contrôlés, RGPD) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
