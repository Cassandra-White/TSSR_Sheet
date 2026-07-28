# CP1-07 — Prise de main à distance multiplateforme (RustDesk / AnyDesk)

**Objectif** : prendre la main à distance sur des postes **Windows, Linux, macOS ou mobiles** avec **RustDesk** (auto-hébergeable) ou **AnyDesk/TeamViewer**, en sécurité.

**Rattachement REAC** : CP1 « Assurer le support utilisateur » — savoir-faire : prendre la main à distance en environnement hétérogène.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Deux postes (aidant/aidé), connectivité réseau.
- Pour l'auto-hébergement RustDesk : une petite **VM Linux** (**CP3-01**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Open source | **RustDesk** (serveur OSS **hbbs/hbbr**) | 24/07/2026 |
| Commercial | **AnyDesk** / **TeamViewer** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> Contrairement à RDP/Assistance rapide (**CP1-06**, Windows), ces outils sont **multiplateformes** (Windows/Linux/macOS/Android/iOS), en **chiffrement de bout en bout**. **RustDesk** peut être **auto-hébergé** (souveraineté des données) ; **AnyDesk/TeamViewer** sont des services **commerciaux (SaaS)**, plus simples mais les flux transitent par l'éditeur.

---

## Procédure — GUI

### RustDesk (recommandé auto-hébergé en entreprise)

1. Installer **RustDesk** sur les deux postes (paquet correspondant à l'OS).
2. **Auto-héberger le serveur** : déployer **hbbs** (serveur d'ID/rendez-vous) + **hbbr** (relais) sur une VM Linux (souvent via **Docker**), puis renseigner l'**adresse du serveur** et la **clé publique** dans les clients.
3. L'aidant saisit l'**ID** du poste distant + son **mot de passe** (ponctuel ou permanent) → contrôle.

### AnyDesk / TeamViewer

1. Installer l'application (licence **pro** pour un usage **entreprise**).
2. L'utilisateur communique son **ID** (+ code) ; l'aidant se connecte et l'utilisateur **accepte**.

---

## Vérification (comment savoir que ça marche)

- La session distante s'établit **entre OS différents** ; le **transfert de fichiers** et le **presse-papier** fonctionnent.
- Avec RustDesk auto-hébergé : la connexion passe par **votre** serveur (vérifiable dans les logs hbbs/hbbr).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Relais public lent/bloqué | Dépendance au serveur public | **Auto-héberger** (hbbs/hbbr) |
| Connexion impossible | NAT / pare-feu | Ouvrir les ports du serveur ; tester en LAN/P2P |
| Faux logiciel | **Installeur contrefait** | Télécharger **uniquement** depuis la source **officielle** |
| Licence refusée | Version gratuite en entreprise | Prendre une **licence pro** (AnyDesk/TeamViewer) |

## Sécurité et bonnes pratiques

- **RustDesk : auto-héberger** — les incidents de 2026 visaient le **serveur public** et de **faux installeurs**, pas le logiciel lui-même. Télécharger depuis la **source officielle**, **durcir** les réglages, mot de passe **fort**.
- **Consentement** de l'utilisateur ; méfiance envers les **arnaques au support** (ne jamais donner le contrôle à un inconnu).
- **AnyDesk/TeamViewer** : les données transitent par l'éditeur → considérer **RGPD/souveraineté** (**CT4-01**) ; restreindre l'**accès non surveillé**.

## ⚠️ À ne pas confondre / obsolète

- **RustDesk auto-hébergé** (données chez vous) ≠ **relais public** (dépendance externe).
- Version **gratuite** (usage **personnel**) ≠ **licence pro** (entreprise).
- Multiplateforme (RustDesk/AnyDesk) ≠ **RDP/Quick Assist** (**CP1-06**, Windows).

---

## Sources (doc officielle)

- [RustDesk — Site officiel / self-host](https://rustdesk.com/) — consulté le 24/07/2026
- [RustDesk — Serveur (hbbs/hbbr) sur GitHub](https://github.com/rustdesk/rustdesk-server) — consulté le 24/07/2026
- [AnyDesk — Documentation](https://support.anydesk.com/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI · [x] daté 24/07/2026 · [x] rien d'obsolète (self-host, licence pro) · [x] procédure **à tester en lab** · [x] conforme doc RustDesk/AnyDesk · [x] vérification présente · [x] sécurité (self-host, source officielle, consentement, RGPD) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
