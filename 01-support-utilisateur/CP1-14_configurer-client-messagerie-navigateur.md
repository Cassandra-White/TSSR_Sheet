# CP1-14 — Configurer un client de messagerie (Outlook/Thunderbird) et un navigateur

**Objectif** : configurer un **client de messagerie** (Outlook/Thunderbird) avec **authentification moderne (OAuth2)** et paramétrer un **navigateur**.

**Rattachement REAC** : CP1 « Assurer le support utilisateur » — savoir-faire : mettre en service les outils bureautiques.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un compte de messagerie (Microsoft 365, ou IMAP/SMTP standard).
- **Outlook** ou **Thunderbird** (128.4.1+ pour OAuth2) installé.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Clients | **Outlook (M365)**, **Thunderbird 128+/145+** | 24/07/2026 |
| Auth | **OAuth 2.0** (authentification moderne) | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> ⚠️ **Actualité 2026** : Microsoft **impose OAuth 2.0** et **désactive l'authentification de base** (Basic Auth) pour IMAP/POP/SMTP sur M365. Un client qui « cesse de marcher » malgré de bons réglages = **Basic Auth bloqué** → passer à un client **compatible OAuth**.

---

## Procédure — GUI (messagerie)

### Configuration automatique

1. Saisir l'**adresse e-mail** → le client tente l'**autoconfiguration** (Thunderbird) / **autodiscover** (Exchange/M365).

> Depuis **juin 2026**, **Outlook ne prend plus en charge l'autodiscover pour IMAP/POP** → **configuration manuelle** requise.

### Configuration manuelle (Microsoft 365, IMAP)

| Paramètre | Entrant (IMAP) | Sortant (SMTP) |
|---|---|---|
| Serveur | `outlook.office365.com` | `smtp.office365.com` |
| Port | **993** | **587** |
| Sécurité | **SSL/TLS** | **STARTTLS** |
| Authentification | **OAuth2** | **OAuth2** |

*(Générique : IMAP 993/SSL, SMTP 587/STARTTLS, POP 995/SSL.)*

## Procédure — GUI (navigateur)

1. **Navigateur par défaut** : *Paramètres ▸ Applications ▸ Applications par défaut*.
2. **Profils/extensions** : extensions **de confiance** uniquement ; nettoyer cache/cookies si besoin.
3. **Proxy** d'entreprise : *Paramètres réseau* ou via **GPO**.

---

## Vérification (comment savoir que ça marche)

- **Envoi + réception** d'un **e-mail de test** réussissent (les deux sens).
- L'**authentification** ouvre bien la fenêtre **OAuth** (connexion + MFA), sans stocker le mot de passe en clair.
- Le navigateur ouvre les liens et accède au web (proxy OK).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Client cesse de fonctionner | **Basic Auth désactivé** (M365) | Utiliser un client **OAuth2** |
| Autoconfig échoue (Outlook) | Autodiscover IMAP/POP **retiré** | **Configurer manuellement** |
| Envoi KO, réception OK | SMTP/port/STARTTLS | Vérifier **587 + STARTTLS + OAuth2** |
| Certificat refusé | TLS/date | Vérifier l'horloge (**CP4-13**), la CA |

## Sécurité et bonnes pratiques

- **OAuth 2.0 + MFA** (**CP7-18**) : plus de mot de passe stocké en clair.
- **IMAP** (synchronisé multi-appareils) **plutôt que POP** (télécharge/supprime).
- **TLS** partout ; extensions de navigateur **vérifiées** ; proxy/filtrage d'entreprise respecté (**CP7-05**).

## ⚠️ À ne pas confondre / obsolète

- **Basic Auth** (désactivée M365 2026) → **OAuth 2.0** (authentification moderne).
- **IMAP** (serveur, synchronisé) ≠ **POP** (local, télécharge) : préférer **IMAP**.
- **Autodiscover Outlook IMAP/POP** : **retiré (juin 2026)** → config manuelle.

---

## Sources (doc officielle)

- [Microsoft Learn — Authentification moderne (OAuth) & fin de Basic Auth](https://learn.microsoft.com/en-us/exchange/clients-and-mobile-in-exchange-online/deprecation-of-basic-authentication-exchange-online) — consulté le 24/07/2026
- [Mozilla — Thunderbird & OAuth Microsoft](https://support.mozilla.org/en-US/kb/microsoft-oauth-authentication-and-thunderbird-202) — consulté le 24/07/2026
- [Microsoft — Paramètres IMAP/POP/SMTP Office 365](https://support.microsoft.com/office/pop-imap-and-smtp-settings-8361e398-8af4-4e97-b147-6c6c4ac95353) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI · [x] versions datées (Outlook/TB 2026) · [x] rien d'obsolète (**OAuth2**, fin Basic Auth, autodiscover retiré) · [x] procédure **à tester en lab** · [x] conforme doc Microsoft/Mozilla · [x] vérification présente (mail test) · [x] sécurité (OAuth+MFA, TLS, IMAP) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
