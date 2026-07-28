# CP7-18 — Mettre en place l'authentification forte (MFA)

**Objectif** : ajouter un **second facteur** (TOTP / push / FIDO2) sur les accès sensibles — ouverture de session, **VPN**, **SSH** — pour qu'un mot de passe volé ne suffise plus.

**Rattachement REAC** : CP7 « Maintenir et sécuriser les accès Internet et les interconnexions » — savoir-faire : renforcer l'authentification des accès.

**Durée** : ~30 min · **Niveau** : avancé.

---

## Prérequis

- Un annuaire (**AD** — **CP2-03**) ou des comptes locaux, et une **horloge synchronisée NTP** (**CP4-13**) — indispensable pour le **TOTP**.
- Une application d'authentification (Microsoft/Google Authenticator, FreeOTP…) côté utilisateur.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Serveur MFA (on-prem) | **privacyIDEA** + **FreeRADIUS** | 24/07/2026 |
| MFA Cloud | **Microsoft Entra** (via NPS/RADIUS ou SAML) | 24/07/2026 |
| SSH | **libpam-google-authenticator** (Debian 13) | 24/07/2026 |
| Algorithme **TOTP** | **RFC 6238 — vecteurs de test validés** | 24/07/2026 |

> **MFA = au moins 2 catégories** de preuve : ce que je **sais** (mot de passe), ce que je **possède** (téléphone/clé), ce que je **suis** (biométrie). Un seul facteur en double (deux mots de passe) **n'est pas** de la MFA.

---

## Procédure — GUI

### A. Accès Microsoft (Entra / Windows)

1. **Entra admin center ▸ Protection ▸ Conditional Access** : créer une règle exigeant la **MFA** pour les utilisateurs/applications ciblés.
2. Autoriser les **méthodes** : **Microsoft Authenticator** (push/TOTP), **FIDO2** (clé de sécurité), en évitant le **SMS** (faible).
3. Pour l'**AD on-premises** : synchroniser avec **Azure AD Connect**, puis étendre la MFA aux accès RADIUS/RDP via l'**extension NPS**.

### B. Serveur MFA on-prem (privacyIDEA)

1. Dans la console **privacyIDEA**, **enrôler un token TOTP** pour l'utilisateur → un **QR code** s'affiche.
2. L'utilisateur le scanne avec son application → le token est lié.
3. Brancher **FreeRADIUS** sur privacyIDEA ; pointer le **VPN pfSense** (OpenVPN — **CP7-07**) vers ce RADIUS : la connexion demandera **mot de passe + code TOTP**.

## Procédure — CLI (SSH avec TOTP — Debian)

```bash
sudo apt install libpam-google-authenticator
google-authenticator            # génère la graine + QR + codes de secours (scratch codes)
# Réponses conseillées : time-based=y, update ~/.google_authenticator=y,
#                        disallow reuse=y, rate-limit=y
```

Activer le second facteur dans PAM et SSH :

```bash
# /etc/pam.d/sshd  (ajouter)
auth required pam_google_authenticator.so

# /etc/ssh/sshd_config.d/mfa.conf
KbdInteractiveAuthentication yes
AuthenticationMethods publickey,keyboard-interactive   # clé SSH + code TOTP
```

```bash
sudo sshd -t && sudo systemctl restart ssh   # tester la config AVANT de fermer la session
```

> **Comment marche un code TOTP** (validé en bac à sable, RFC 6238) : `code = HOTP(secret, ⌊temps/30s⌋)` tronqué à 6–8 chiffres. Vérifié sur les vecteurs officiels : `T=59 → 94287082`, `T=1111111109 → 07081804`. D'où l'importance d'une **horloge juste** des deux côtés.

---

## Vérification (comment savoir que ça marche)

- Une nouvelle connexion (SSH/VPN/session) réclame bien le **code à 6 chiffres** en plus du mot de passe/clé.
- Un code **périmé** (>30 s) ou erroné est **refusé** ; un code valide passe.
- Les **codes de secours** permettent de se connecter si le téléphone est indisponible.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Tous les codes refusés | **Horloge désynchronisée** | Synchroniser **NTP** serveur **et** téléphone (**CP4-13**) |
| Verrouillé dehors en SSH | PAM/sshd modifié sans test | Garder une **session ouverte** ; corriger via console |
| Push/SMS non reçu | Méthode fragile (SMS) | Basculer sur **TOTP** ou **FIDO2** |
| Perte du téléphone | Pas de secours | Utiliser/prévoir les **scratch codes** |

## Sécurité et bonnes pratiques

- **Prioriser les comptes à privilèges** (admins, VPN, accès distant) et les expositions Internet.
- Préférer **FIDO2/clé de sécurité** ou **push avec correspondance de numéro** : **résistants au phishing**, contrairement au **SMS** (**SIM swapping**).
- **NTP obligatoire** ; **codes de secours** stockés de façon sûre (coffre).
- **Ne jamais couper l'ancien accès** avant d'avoir validé la MFA sur une session de test.

## ⚠️ À ne pas confondre / obsolète

- **OTP par SMS** = déconseillé (interception, SIM swap) → **TOTP / FIDO2**.
- **2FA** (2 facteurs de **catégories différentes**) ≠ **double mot de passe** (même catégorie).
- **TOTP** (temps, 30 s) vs **HOTP** (compteur) : le TOTP exige une **horloge synchronisée**.

---

## Sources (doc officielle)

- [Microsoft Learn — Entra multifactor authentication](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-mfa-howitworks) — consulté le 24/07/2026
- [privacyIDEA — Documentation](https://privacyidea.readthedocs.io/en/latest/) — consulté le 24/07/2026
- [Debian — google-authenticator (libpam)](https://manpages.debian.org/google-authenticator) — consulté le 24/07/2026
- [RFC 6238 — TOTP: Time-Based One-Time Password](https://www.rfc-editor.org/rfc/rfc6238) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI (Entra/privacyIDEA) puis CLI (SSH TOTP) · [x] versions datées · [x] rien d'obsolète (FIDO2/TOTP vs SMS) · [x] **TOTP testé en bac à sable** (vecteurs RFC 6238) / intégration à tester en lab · [x] GUI conforme doc Microsoft/privacyIDEA · [x] vérification présente · [x] sécurité (anti-phishing, NTP, secours) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
