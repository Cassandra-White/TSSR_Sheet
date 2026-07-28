# CP3-05 — Installer et gérer des paquets (APT) et les dépôts

**Objectif** : installer, mettre à jour, rechercher et supprimer des paquets avec APT, et ajouter proprement un dépôt tiers (format **deb822** + clé **Signed-By**).

**Rattachement REAC** : CP3 « Exploiter des serveurs Linux » — savoir-faire : gérer les logiciels et les sources de paquets d'un serveur Linux.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Debian 13 (**CP3-01**), accès réseau, droits root/sudo.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian / APT | 13.6 « trixie » — **APT 3.0** | 24/07/2026 |
| `apt` / `apt-cache` / `dpkg` / format `.sources` | **commandes génériques testées dans le bac à sable** (APT 2.4) | 24/07/2026 |

> Debian 13 introduit **APT 3.0** : solveur *solver3*, sortie **en couleurs** (rouge = suppression, vert = ajout), commande `apt modernize-sources`. Les commandes ci-dessous sont identiques ; les nouveautés 3.0 sont signalées.

---

## Procédure — CLI

### Opérations de base

```bash
sudo apt update                 # rafraîchir l'index des paquets
sudo apt upgrade                # mettre à jour les paquets installés
sudo apt full-upgrade           # + gérer les changements de dépendances
sudo apt install nginx          # installer
sudo apt remove nginx           # désinstaller (garde la config)
sudo apt purge nginx            # désinstaller + config
sudo apt autoremove             # retirer les dépendances orphelines
```

### Rechercher / inspecter

```bash
apt search <mot-clé>
apt show <paquet>
apt list --installed
apt-cache policy <paquet>       # versions et dépôt d'origine (testé)
dpkg -l                         # paquets installés (testé)
dpkg -L <paquet>                # fichiers fournis par un paquet (testé)
dpkg -S /chemin/fichier         # à quel paquet appartient un fichier
```

### Figer une version / installer un .deb

```bash
sudo apt-mark hold <paquet>     # empêcher la mise à jour ; unhold pour lever ; showhold pour lister
sudo dpkg -i paquet.deb && sudo apt install -f   # -f répare les dépendances manquantes
```

### Ajouter un dépôt tiers (méthode Debian 13 — deb822 + Signed-By)

```bash
# 1. Déposer la clé du dépôt (ASCII-armored) dans le répertoire dédié
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://deb.exemple.com/key.asc -o /etc/apt/keyrings/exemple.asc

# 2. Déclarer le dépôt au format .sources
sudo tee /etc/apt/sources.list.d/exemple.sources >/dev/null <<'EOF'
Types: deb
URIs: https://deb.exemple.com/debian
Suites: trixie
Components: main
Signed-By: /etc/apt/keyrings/exemple.asc
EOF

# 3. Prendre en compte le nouveau dépôt
sudo apt update
```

> Migrer d'anciens fichiers `sources.list` (une ligne `deb …`) vers le format `.sources` : **`sudo apt modernize-sources`** (APT 3.0).

---

## Vérification

```bash
apt-cache policy <paquet>       # doit montrer le dépôt ajouté comme source candidate
apt policy                      # priorités des dépôts
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| `NO_PUBKEY` / dépôt non authentifié | Clé absente ou `Signed-By` erroné | Vérifier le chemin de la clé dans le `.sources` |
| `apt-key` renvoie « command not found / deprecated » | `apt-key` **supprimé** | Utiliser `Signed-By` + `/etc/apt/keyrings/` |
| Dépendances cassées après un `.deb` | Dépendances manquantes | `sudo apt install -f` |
| Paquet non mis à jour | Il est en **hold** | `sudo apt-mark unhold <paquet>` |

## Sécurité et bonnes pratiques

- N'ajouter que des **dépôts de confiance** ; **une clé par dépôt** dans `/etc/apt/keyrings/` (ne pas polluer le trousseau global `trusted.gpg.d`).
- Toujours en **HTTPS** ; vérifier l'empreinte de la clé fournie par l'éditeur.
- Appliquer régulièrement les mises à jour de sécurité (**CP3-11**).

## ⚠️ À ne pas confondre / obsolète

- **`apt-key` est supprimé** (dernier support Debian 12) → gestion des clés par **`Signed-By`**.
- Le format **une ligne `deb …`** est **déprécié** → préférer **deb822 `.sources`**.
- Depuis Debian 12, le composant non-libre est scindé : **`non-free`** (paquets) **et `non-free-firmware`** (micrologiciels).
- Dans un **script**, préférer **`apt-get`** (sortie stable) ; `apt` est fait pour l'usage interactif.

---

## Sources (doc officielle)

- [Debian Wiki — SourcesList (format deb822)](https://wiki.debian.org/SourcesList) — consulté le 24/07/2026
- [sources.list(5) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/apt/sources.list.5.en.html) — consulté le 24/07/2026
- [What's new in APT 3.0 — LWN.net](https://lwn.net/Articles/1017315/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées (APT 3.0) · [x] rien d'obsolète (deb822, Signed-By, apt-key supprimé) · [x] **commandes génériques testées** · [x] conforme doc Debian · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
