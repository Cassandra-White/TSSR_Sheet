# CP9-06 — Mise à jour centralisée sous Linux (dépôt local / apt-cacher-ng)

**Objectif** : centraliser et accélérer les mises à jour Debian/Ubuntu avec un **cache APT** (apt-cacher-ng) et/ou un **dépôt local signé** pour des paquets internes.

**Rattachement REAC** : CP9 « Exploiter et maintenir les services de déploiement des postes » — savoir-faire : centraliser les mises à jour Linux.

**Durée** : ~30 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un serveur **Debian 13** (**CP3-01**) + des clients Debian/Ubuntu.
- **GPG** pour signer un dépôt local (**CP3-05** pour le principe `Signed-By`).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Cache | **apt-cacher-ng** (port **3142**) | 24/07/2026 |
| Dépôt local signé | **apt-ftparchive** + **GPG** — **testé en bac à sable** | 24/07/2026 |

> **Deux besoins distincts** : **apt-cacher-ng** = **cache proxy transparent** (on télécharge une fois depuis Internet, tous les clients servent ensuite localement → bande passante économisée). **Dépôt local** = **héberger ses propres paquets** (`.deb` maison) signés.

---

## Procédure — CLI

### A. Cache APT partagé — apt-cacher-ng

```bash
# Serveur
apt install apt-cacher-ng          # écoute sur :3142, cache dans /var/cache/apt-cacher-ng/

# Client : router apt via le cache
echo 'Acquire::http::Proxy "http://cache.lab.local:3142";' \
  | sudo tee /etc/apt/apt.conf.d/01proxy
sudo apt update && sudo apt upgrade
```

Statistiques/hit-ratio : `http://cache.lab.local:3142/acng-report.html`.

### B. Dépôt local signé (paquets internes) — testé

```bash
# 1) Index des paquets
apt-ftparchive packages pool > dists/stable/main/binary-amd64/Packages
gzip -kf dists/stable/main/binary-amd64/Packages

# 2) Fichier Release (checksums)
apt-ftparchive release dists/stable > dists/stable/Release

# 3) SIGNER le dépôt (clé GPG dédiée)
gpg --default-key repo@lab.local --clearsign -o dists/stable/InRelease dists/stable/Release
gpg --default-key repo@lab.local -abs      -o dists/stable/Release.gpg dists/stable/Release
```

Client (format **deb822** moderne, clé en `Signed-By` — **plus** d'`apt-key`) :

```
# /etc/apt/sources.list.d/local.sources
Types: deb
URIs: http://depot.lab.local/
Suites: stable
Components: main
Signed-By: /usr/share/keyrings/depot-lab.gpg
```

> **Testé en bac à sable** : génération clé GPG → `Packages` (apt-ftparchive) → `Release` → **`InRelease` + `Release.gpg`** → **`gpg --verify` = « Good signature »**. C'est exactement la chaîne de confiance qu'`apt` vérifie côté client.

---

## Vérification (comment savoir que ça marche)

- Avec le cache : `apt update` sur un 2ᵉ client est **quasi instantané** (servi depuis le cache) ; la page `acng-report.html` montre des **hits**.
- Avec le dépôt local : `apt update` **sans avertissement de signature**, et `apt install <paquet-interne>` fonctionne.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| `NO_PUBKEY` / dépôt non signé | Clé absente côté client | Installer la clé + `Signed-By` (deb822) |
| Proxy non pris en compte | Fichier `01proxy` absent/mauvais | Vérifier `/etc/apt/apt.conf.d/01proxy` |
| HTTPS non caché | apt-cacher-ng et TLS | Utiliser le passthrough/`Remap` HTTPS de acng |
| Cache saturé | Rétention | Purger `/var/cache/apt-cacher-ng/` / cron d'entretien |

## Sécurité et bonnes pratiques

- **Signer** tout dépôt local (GPG) et distribuer la clé via **`Signed-By`** — **`apt-key` est déprécié** (**CP3-05**).
- **Restreindre l'accès** au cache/dépôt (réseau interne) et le **journaliser** (**CP7-10**).
- Servir en **HTTPS** quand c'est possible ; garder le miroir **à jour**.

## ⚠️ À ne pas confondre / obsolète

- **Cache** (apt-cacher-ng : accélère l'accès aux dépôts **officiels**) ≠ **dépôt local** (héberge **vos** paquets).
- **`apt-key`** (déprécié/supprimé) → clé en **`Signed-By`** (format **deb822**).
- **`reprepro`/`aptly`** simplifient la gestion d'un dépôt local par rapport à `apt-ftparchive` à la main.

---

## Sources (doc officielle)

- [Debian Wiki — AptCacherNg](https://wiki.debian.org/AptCacherNg) — consulté le 24/07/2026
- [Debian — apt-ftparchive (manuel)](https://manpages.debian.org/bookworm/apt-utils/apt-ftparchive.1.en.html) — consulté le 24/07/2026
- [Debian Wiki — DebianRepository / signature (Signed-By)](https://wiki.debian.org/DebianRepository/UseThirdParty) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI (cache + dépôt local) · [x] daté 24/07/2026 · [x] rien d'obsolète (`Signed-By` vs `apt-key`) · [x] **dépôt local signé testé en bac à sable** (GPG « Good signature ») / apt-cacher-ng à tester en lab · [x] conforme doc Debian · [x] vérification présente · [x] sécurité (signature, accès, HTTPS) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
