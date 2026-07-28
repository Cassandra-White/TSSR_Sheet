# CP3-09 — Configurer et durcir l'accès SSH (clés, bonnes pratiques)

**Objectif** : se connecter en SSH par **clé** (sans mot de passe) et durcir le serveur `sshd` (pas de root, pas de mot de passe).

**Rattachement REAC** : CP3 « Exploiter des serveurs Linux » — savoir-faire : sécuriser l'accès distant d'un serveur Linux.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Debian 13 (**CP3-01**) avec `openssh-server`, droits root/sudo, un compte utilisateur.
- Un poste client (Linux/Windows) avec un client SSH.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian / OpenSSH | 13.6 « trixie » — **OpenSSH 10** | 24/07/2026 |
| `ssh-keygen` (ed25519) | **testé dans le bac à sable** | 24/07/2026 |

---

## Procédure — CLI

### 1. Générer une paire de clés (sur le poste client)

```bash
ssh-keygen -t ed25519 -C "bob@poste-admin"
# Entrée pour le chemin par défaut (~/.ssh/id_ed25519), + une passphrase forte
```

Clé publique et empreinte obtenues (test réel) :

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPQuX8Tx...+pvi bob@poste-admin
256 SHA256:iX6gYsPgp/7j8S9Fdrboy5B8h7XsGdK3e0Y7eI/Wfv0 bob@poste-admin (ED25519)
```

### 2. Déposer la clé publique sur le serveur

```bash
ssh-copy-id bob@192.168.10.20          # méthode simple
# (équivalent manuel : ajouter la clé à ~/.ssh/authorized_keys)
#   chmod 700 ~/.ssh ; chmod 600 ~/.ssh/authorized_keys
```

### 3. Durcir le serveur — fichier drop-in dédié

```bash
sudo nano /etc/ssh/sshd_config.d/60-hardening.conf
```

```text
PermitRootLogin no
PasswordAuthentication no
KbdInteractiveAuthentication no
PubkeyAuthentication yes
AllowUsers bob
# Port 22        # (option : changer le port = simple obfuscation)
```

### 4. Valider et recharger — SANS se couper l'accès

```bash
sudo sshd -t                       # vérifie la syntaxe (nécessite les clés d'hôte du serveur)
sudo systemctl reload ssh          # applique (le service Debian s'appelle « ssh »)
```

> ⚠️ **Garder la session actuelle ouverte** et tester une **nouvelle** connexion par clé avant de fermer, pour ne pas se verrouiller dehors.

---

## Vérification (comment savoir que ça marche)

```bash
ssh -v bob@192.168.10.20                       # doit s'authentifier par clé (publickey)
sudo sshd -T | grep -Ei 'permitrootlogin|passwordauthentication'
# attendu : permitrootlogin no / passwordauthentication no
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| `Permission denied (publickey)` | Clé non déposée / mauvais compte | `ssh-copy-id` ; vérifier `AllowUsers` |
| Clé refusée malgré tout | Droits `~/.ssh` incorrects | `chmod 700 ~/.ssh` ; `chmod 600 authorized_keys` ; bon propriétaire |
| Plus aucun accès après reload | Verrouillé dehors | Reprendre la main par **console/KVM** de la VM |
| `sshd -t` : erreur de syntaxe | Faute dans le drop-in | Corriger la ligne signalée |

## Sécurité et bonnes pratiques

- **Clés ed25519** + passphrase ; désactiver **mot de passe** et **root** en SSH.
- Ajouter **Fail2ban** (**CP3-18**) et un **pare-feu** (**CP3-14**).
- Restreindre les comptes autorisés (`AllowUsers`/`AllowGroups`) ; suivre les recommandations **ANSSI**.

## ⚠️ À ne pas confondre / obsolète

- **OpenSSH 10 (Debian 13) ne supporte plus les clés DSA** ; `ssh-rsa` (SHA-1) est déconseillé → **ed25519**.
- Ne pas éditer aveuglément `/etc/ssh/sshd_config` : il inclut `sshd_config.d/*.conf` en tête → **utiliser un drop-in** (la dernière valeur lue l'emporte).
- Sous Debian, le service est **`ssh`** (`systemctl reload ssh`), pas `sshd`.

---

## Sources (doc officielle)

- [Debian Wiki — SSH](https://wiki.debian.org/SSH) — consulté le 24/07/2026
- [sshd_config(5) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/openssh-server/sshd_config.5.en.html) — consulté le 24/07/2026
- [ssh-keygen(1) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/openssh-client/ssh-keygen.1.en.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées (OpenSSH 10) · [x] rien d'obsolète (ed25519, anti-DSA) · [x] **ssh-keygen testé** ; `sshd -t` documenté · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
