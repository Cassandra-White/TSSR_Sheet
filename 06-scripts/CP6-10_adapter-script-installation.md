# CP6-10 — Comprendre et adapter un script d'installation fourni par un éditeur

**Objectif** : lire, comprendre et adapter en toute sécurité un script d'installation d'éditeur (souvent du type `curl … | bash`).

**Rattachement REAC** : CP6 « Automatiser des tâches à l'aide de scripts » — savoir-faire : exploiter et adapter un script fourni.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Debian 13 (**CP3-01**), bases Bash (**CP6-05**), notions de dépôts APT (**CP3-05**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian / Bash | 13.6 « trixie » | 24/07/2026 |
| Démarche | méthode — **à appliquer en lab** | 24/07/2026 |

> ⚠️ **`curl … | sudo bash`** exécute du code distant **en aveugle**, souvent en **root** : à éviter tel quel.

---

## Démarche sûre (au lieu de « curl | bash »)

```bash
# 1. TÉLÉCHARGER sans exécuter
curl -fsSL https://get.exemple.com/install.sh -o install.sh

# 2. LIRE / INSPECTER : que fait le script ?
less install.sh
#   → quels dépôts/clés GPG ajoute-t-il ? quels paquets ? quels services ?
#   → s'exécute-t-il en root ? télécharge-t-il d'autres fichiers ?

# 3. VÉRIFIER la source : HTTPS, éditeur officiel, somme de contrôle si fournie
sha256sum install.sh          # comparer à la valeur publiée par l'éditeur

# 4. EXÉCUTER en connaissance de cause (privilèges minimaux)
bash install.sh
```

## Comprendre un script d'installation (repères)

À la lecture, on cherche :

- l'en-tête **`set -e`** et la **détection d'OS** (`ID`, `VERSION_CODENAME` de `/etc/os-release`) ;
- les **variables** paramétrables (`VERSION=`, `CHANNEL=`, `INSTALL_DIR=`, proxy) ;
- l'ajout d'un **dépôt + clé** (format **deb822 / `signed-by`** — **CP3-05**) ;
- les commandes **`apt install`**, **`systemctl enable --now`** ;
- toute **télémétrie** ou envoi de données à désactiver.

## Adapter le script

- Fixer une **version précise** (reproductibilité) plutôt que « latest ».
- Renseigner un **proxy** (`http_proxy`/`https_proxy`) si le réseau l'impose.
- Retirer/neutraliser ce qui n'est **pas souhaité** (télémétrie, dépôts superflus).
- Conserver la version adaptée dans **Git** (**CP6-09**).

---

## Vérification (comment savoir que ça marche)

- Après lecture, on peut **expliquer** chaque action du script.
- Après exécution : le paquet est installé (`apt list --installed | grep …`), le service tourne (`systemctl status …`), la **version** est celle attendue.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| « OS non supporté » | Détection d'OS trop stricte | Adapter la condition (`VERSION_CODENAME`) |
| Échec de téléchargement | Proxy / réseau | Exporter `http_proxy`/`https_proxy` |
| Dépôt non authentifié | Clé mal posée | Vérifier `signed-by` (**CP3-05**) |
| Service non démarré | `enable --now` absent | `systemctl enable --now <service>` |

## Sécurité et bonnes pratiques

- **Ne jamais** exécuter `curl | sudo bash` sans **lire** le script au préalable.
- Vérifier **HTTPS**, l'**éditeur officiel** et, si possible, une **signature/somme de contrôle**.
- Exécuter avec le **minimum de privilèges** ; se méfier des scripts tiers non officiels.

## ⚠️ À ne pas confondre / obsolète

- Un **installeur officiel d'éditeur** (ex. dépôt Docker) ≠ un **script tiers** trouvé au hasard.
- `curl | bash` = **commodité risquée** : préférer **télécharger → lire → exécuter**.
- Fixer une **version** vaut mieux que « latest » pour un déploiement **reproductible**.

---

## Sources (doc officielle)

- [Docker — Install on Debian (exemple d'installeur éditeur)](https://docs.docker.com/engine/install/debian/) — consulté le 24/07/2026
- [ANSSI — Recommandations (bonnes pratiques d'administration)](https://cyber.gouv.fr/publications) — consulté le 24/07/2026
- [GNU Bash — Manuel de référence](https://www.gnu.org/software/bash/manual/bash.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées · [x] rien d'obsolète (deb822, versions fixées) · [x] démarche à appliquer en lab · [x] conforme bonnes pratiques · [x] vérification présente · [x] **sécurité (anti `curl|bash` aveugle)** · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
