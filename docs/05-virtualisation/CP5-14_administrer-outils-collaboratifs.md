# CP5-14 — Administrer/utiliser les outils collaboratifs (visio, messagerie instantanée, partage)

**Objectif** : mettre en œuvre et utiliser des outils collaboratifs (**visioconférence**, **messagerie instantanée**, **partage**) — en Cloud (Teams/Meet) ou auto-hébergés (Jitsi/Nextcloud Talk).

**Rattachement REAC** : CP5 « Maintenir des serveurs dans une infrastructure virtualisée » — savoir-faire : administrer des services collaboratifs.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un tenant **Microsoft 365 / Google Workspace** (**CP5-09**), **ou** une VM pour **Jitsi/Nextcloud Talk**.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Cloud | **Microsoft Teams**, **Google Meet/Chat** | 24/07/2026 |
| Auto-hébergé | **Jitsi Meet**, **Nextcloud Talk** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> Ces outils regroupent **visio**, **chat**, **partage d'écran/fichiers** et **salles/canaux**. Choix **SaaS** (Teams/Meet — simple) vs **auto-hébergé** (Jitsi/Nextcloud Talk — **souveraineté/RGPD**).

---

## Procédure — GUI

### Microsoft Teams (M365)

1. **Centre d'administration Teams** : créer des **équipes/canaux**, définir des **politiques** (réunions, **enregistrement**, invités externes).
2. Usage : réunion (lien), **partage d'écran**, chat, fichiers (adossés à SharePoint — **CP5-10**).

### Google Meet / Chat (Workspace)

- **Admin Console** : politiques de réunion, enregistrement, accès externes.

### Auto-hébergé (Jitsi Meet)

- Déployer **Jitsi** (paquet/Docker) → **salles** accessibles par lien, sans compte ; **Nextcloud Talk** pour intégrer visio + chat au partage de fichiers (**CP5-10**).

---

## Vérification (comment savoir que ça marche)

- Une **réunion** fonctionne (audio/vidéo/partage d'écran), le **chat** et le **partage de fichiers** aussi.
- Un **invité externe** rejoint selon la politique définie (lobby/mot de passe).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Audio/vidéo coupés | Pare-feu / ports média (UDP) | Ouvrir les ports ; **QoS** (**CP7-15**) |
| Micro/caméra KO | Permissions poste | Autoriser dans l'OS/navigateur |
| Visio saccadée | Bande passante | QoS/priorisation ; réduire la qualité |
| Invités bloqués | Politique trop stricte | Ajuster l'accès **externe** |

## Sécurité et bonnes pratiques

- **Contrôler les invités externes**, protéger les salles (**lobby/mot de passe**).
- **Enregistrement** : **consentement** + conservation maîtrisée (**RGPD** — **CT4-01**).
- **Chiffrement** des flux ; pour les données **sensibles**, envisager l'**auto-hébergé** (Jitsi/Nextcloud) pour la souveraineté.

## ⚠️ À ne pas confondre / obsolète

- **Messagerie instantanée** (chat, temps réel) ≠ **messagerie e-mail** (**CP1-14**).
- **SaaS** (Teams/Meet, données éditeur) ≠ **auto-hébergé** (Jitsi/Talk, souveraineté).
- Partage via **espace collaboratif** (droits, versions) → à préférer aux **pièces jointes** e-mail.

---

## Sources (doc officielle)

- [Microsoft Learn — Administration de Teams](https://learn.microsoft.com/en-us/microsoftteams/) — consulté le 24/07/2026
- [Google Workspace — Meet/Chat (admin)](https://support.google.com/a/topic/7302757) — consulté le 24/07/2026
- [Jitsi Meet — Documentation](https://jitsi.github.io/handbook/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI · [x] daté 24/07/2026 · [x] rien d'obsolète (SaaS vs auto-hébergé) · [x] procédure **à tester en lab** · [x] conforme doc Microsoft/Google/Jitsi · [x] vérification présente (réunion) · [x] sécurité (invités, enregistrement RGPD, chiffrement) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
