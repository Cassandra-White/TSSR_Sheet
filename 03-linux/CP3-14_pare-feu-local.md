# CP3-14 — Configurer un pare-feu local (nftables / ufw)

**Objectif** : filtrer le trafic entrant d'un serveur Debian avec une politique par défaut restrictive, tout en gardant l'accès SSH.

**Rattachement REAC** : CP3 « Exploiter des serveurs Linux » — savoir-faire : sécuriser un serveur Linux par un pare-feu local.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Debian 13 (**CP3-01**), droits root/sudo, **accès console de secours** (au cas où).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian | 13.6 « trixie » — **nftables par défaut** | 24/07/2026 |
| Jeu de règles nftables | **testé (appliqué + listé) en namespace réseau** | 24/07/2026 |

> Deux approches au choix : **ufw** (simple) ou **nftables** (natif). **Ne jamais mélanger** plusieurs outils de pare-feu.

---

## Procédure — CLI

### Option A — ufw (le plus simple)

```bash
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw limit 22/tcp            # SSH (limit = anti-bruteforce léger) — AVANT d'activer !
sudo ufw allow 80,443/tcp        # web si nécessaire
sudo ufw enable                  # activer (et au démarrage)
sudo ufw status verbose
```

### Option B — nftables (natif Debian)

Éditer `/etc/nftables.conf` (jeu de règles validé) :

```nft
#!/usr/sbin/nft -f
flush ruleset
table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;
        ct state established,related accept
        iif "lo" accept
        ct state invalid drop
        tcp dport 22 accept
        ip protocol icmp accept
    }
    chain forward { type filter hook forward priority 0; policy drop; }
    chain output  { type filter hook output priority 0; policy accept; }
}
```

```bash
sudo apt install nftables
sudo nft -f /etc/nftables.conf          # appliquer
sudo systemctl enable --now nftables    # recharger au démarrage
```

> Ouvrir le SSH (port 22) **avant** d'appliquer une politique `drop`, sous peine de se couper l'accès.

---

## Vérification (sortie obtenue lors du test)

```bash
sudo ufw status verbose          # (option A)
sudo nft list ruleset            # (option B)
```

Extrait réel du `nft list ruleset` appliqué :

```
table inet filter {
    chain input {
        type filter hook input priority filter; policy drop;
        ct state established,related accept
        iif "lo" accept
        ct state invalid drop
        tcp dport 22 accept
        ip protocol icmp accept
    }
}
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| **Verrouillé hors du serveur** | Politique `drop` sans règle SSH | Reprendre par la **console** ; autoriser 22 **avant** d'activer |
| Règles perdues au redémarrage | Service non activé | `sudo systemctl enable nftables` / `sudo ufw enable` |
| Comportement imprévisible | **Plusieurs outils** actifs | N'en garder **qu'un** (ufw **ou** nftables **ou** iptables) |
| Un service reste injoignable | Port non ouvert | Ajouter la règle correspondante |

## Sécurité et bonnes pratiques

- **Politique par défaut `deny`/`drop`** en entrée ; n'ouvrir **que** les ports nécessaires.
- **Journaliser** les paquets refusés pour l'analyse (règle `log`).
- Compléter par **Fail2ban** (**CP3-18**) contre le bruteforce SSH.

## ⚠️ À ne pas confondre / obsolète

- **nftables remplace iptables** : sous Debian, la commande `iptables` est en réalité le wrapper **`iptables-nft`**. Éviter **`iptables-legacy`**.
- **Choisir un seul outil + un seul backend** ; mélanger ufw, nftables et iptables crée des conflits.
- **Toujours autoriser SSH avant** de passer la politique d'entrée en `drop`.

---

## Sources (doc officielle)

- [Debian Wiki — nftables](https://wiki.debian.org/nftables) — consulté le 24/07/2026
- [ufw(8) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/ufw/ufw.8.en.html) — consulté le 24/07/2026
- [nftables Wiki — Quick reference](https://wiki.nftables.org/wiki-nftables/index.php/Quick_reference-nftables_in_10_minutes) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées (nftables défaut) · [x] rien d'obsolète (nft vs iptables-legacy) · [x] **ruleset nftables testé en namespace** · [x] conforme doc Debian · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
