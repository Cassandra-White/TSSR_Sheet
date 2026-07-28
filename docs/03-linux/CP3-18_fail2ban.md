# CP3-18 — Sécuriser SSH avec Fail2ban (anti-bruteforce)

**Objectif** : bannir automatiquement les adresses IP qui multiplient les tentatives de connexion SSH échouées.

**Rattachement REAC** : CP3 « Exploiter des serveurs Linux » — savoir-faire : protéger l'accès distant d'un serveur Linux contre le bruteforce.

**Durée** : ~15 min · **Niveau** : intermédiaire.

---

## Prérequis

- Debian 13 (**CP3-01**) avec SSH (**CP3-09**), droits root/sudo.
- Un **accès de secours** (console) au cas où l'on se bannirait soi-même.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian / Fail2ban | 13.6 « trixie » — Fail2ban 1.x | 24/07/2026 |
| Config `jail.local` | **à tester en lab** (paquet absent du bac à sable) | 24/07/2026 |

> Sur Debian 13, Fail2ban lit par défaut le **journal systemd** (backend `systemd`) et bannit via **nftables** — cohérent avec **CP3-15** (journald) et **CP3-14** (nftables).

---

## Procédure — CLI

```bash
sudo apt install fail2ban
```

Créer `/etc/fail2ban/jail.local` (ne **jamais** éditer `jail.conf`, écrasé aux mises à jour) :

```ini
[DEFAULT]
bantime   = 1h
findtime  = 10m
maxretry  = 5
backend   = systemd                 # Debian 13 : lecture du journal (pas d'auth.log)
banaction = nftables-multiport      # cohérent avec le pare-feu par défaut
ignoreip  = 127.0.0.1/8 192.168.10.0/24   # ne jamais bannir le réseau d'admin

[sshd]
enabled = true
```

Activer et (re)charger :

```bash
sudo systemctl enable --now fail2ban
sudo systemctl restart fail2ban       # après toute modification de jail.local
```

---

## Vérification (comment savoir que ça marche)

```bash
sudo fail2ban-client status            # liste des jails actives (doit inclure « sshd »)
sudo fail2ban-client status sshd       # Currently failed / Total banned / Banned IP list
#   la ligne « Journal matches » confirme la lecture du journal systemd
sudo fail2ban-client set sshd unbanip 203.0.113.10   # débannir une IP
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| `status sshd` : **0 failures** malgré des attaques | Mauvais backend (recherche d'`auth.log`) | `backend = systemd` (Debian 13) |
| Bannissement sans effet | `banaction` iptables sur système nftables | `banaction = nftables-multiport` |
| **Auto-bannissement** | Son IP non exclue | Ajouter le réseau d'admin à `ignoreip` |
| Fail2ban ne démarre pas | Erreur de syntaxe `jail.local` | `journalctl -u fail2ban` ; corriger |

## Sécurité et bonnes pratiques

- Fail2ban est une **2ᵉ barrière** : d'abord durcir SSH (clés, pas de mot de passe — **CP3-09**).
- Augmenter la sanction des récidivistes (`bantime.increment = true`).
- Renseigner `ignoreip` avec les IP/réseaux d'administration ; garder un **accès console**.
- Ne remplace pas un **pare-feu** (**CP3-14**), il le complète.

## ⚠️ À ne pas confondre / obsolète

- Sous Debian 13, **`/var/log/auth.log` n'existe plus par défaut** → backend **systemd** (les tutos anciens pointant `logpath = /var/log/auth.log` donnent « 0 failures »).
- Utiliser **`nftables-multiport`** et non une action iptables (nftables est le backend par défaut).
- Toujours personnaliser via **`jail.local`**, jamais `jail.conf`.

---

## Sources (doc officielle)

- [jail.conf(5) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/fail2ban/jail.conf.5.en.html) — consulté le 24/07/2026
- [fail2ban-client(1) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/fail2ban/fail2ban-client.1.en.html) — consulté le 24/07/2026
- [Fail2ban — dépôt et wiki officiels](https://github.com/fail2ban/fail2ban/wiki) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées · [x] rien d'obsolète (**backend systemd, nftables**) · [x] config à tester en lab · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
