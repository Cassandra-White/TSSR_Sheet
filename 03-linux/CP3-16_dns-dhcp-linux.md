# CP3-16 — Installer un service DNS/DHCP sous Linux (dnsmasq / bind9 / Kea)

**Objectif** : fournir la résolution de noms (DNS) et l'attribution d'adresses (DHCP) sur un réseau de lab avec un service Linux.

**Rattachement REAC** : CP3 « Exploiter des serveurs Linux » — savoir-faire : mettre en service DNS/DHCP sur un serveur Linux.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Debian 13 (**CP3-01**), IP fixe (**CP3-02**), droits root/sudo.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian | 13.6 « trixie » | 24/07/2026 |
| dnsmasq / bind9 / Kea | config **à tester en lab** (paquets absents du bac à sable) | 24/07/2026 |

> ⚠️ **Actualité majeure** : **`isc-dhcp-server` est retiré de Debian 13** (le projet ISC DHCP est en fin de vie). Pour du DHCP, utiliser **dnsmasq** (simple) ou **Kea** (successeur d'ISC). Ne **pas** suivre les vieux tutos `dhcpd` / `/etc/dhcp/dhcpd.conf`.

---

## Procédure — CLI

### Option A — dnsmasq (DNS + DHCP en un seul service, idéal lab)

```bash
sudo apt install dnsmasq
sudo tee /etc/dnsmasq.d/lab.conf >/dev/null <<'EOF'
interface=enp1s0
domain=lab.local
# DHCP
dhcp-range=192.168.10.100,192.168.10.200,255.255.255.0,12h
dhcp-option=option:router,192.168.10.1
dhcp-option=option:dns-server,192.168.10.5
# DNS statique
address=/srv.lab.local/192.168.10.5
EOF
dnsmasq --test                       # valider la configuration
sudo systemctl restart dnsmasq
```

> ⚠️ **Conflit du port 53** avec `systemd-resolved` : désactiver son écoute (`DNSStubListener=no` dans `/etc/systemd/resolved.conf`) ou arrêter `systemd-resolved`.

### Option B — bind9 (DNS autoritaire, zones)

```bash
sudo apt install bind9 bind9-utils
# Déclarer les zones dans /etc/bind/named.conf.local + fichiers de zone
sudo named-checkconf                 # vérifier la config
sudo named-checkzone lab.local /etc/bind/db.lab.local
sudo systemctl reload named
```

### Option C — Kea (DHCP autonome, remplaçant d'isc-dhcp)

```bash
sudo apt install kea-dhcp4-server
# Config JSON : /etc/kea/kea-dhcp4.conf (subnet4, pools, option-data)
sudo systemctl enable --now kea-dhcp4-server
```

---

## Vérification (comment savoir que ça marche)

```bash
dnsmasq --test                          # "syntax check OK"
dig @192.168.10.5 srv.lab.local         # la résolution répond
journalctl -u dnsmasq -b                # baux DHCP attribués
# Côté client : ip a  → une adresse de la plage est obtenue
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| dnsmasq ne démarre pas | Port 53 pris par `systemd-resolved` | `DNSStubListener=no` puis redémarrer |
| Pas de bail DHCP côté client | Interface/plage erronée, pare-feu | Vérifier `interface=`/`dhcp-range` ; ouvrir UDP 67/68 |
| Adresses en conflit | **Deux serveurs DHCP** sur le LAN | N'en garder qu'un sur le segment |
| Résolution KO | Mauvais `dns-server` distribué | Corriger `dhcp-option:dns-server` |

## Sécurité et bonnes pratiques

- Cantonner le **DHCP au bon segment** ; éviter d'exposer un DHCP « pirate » sur le réseau.
- DNS : **pas de récursion ouverte** vers l'extérieur ; restreindre aux clients internes.
- Journaliser les attributions (traçabilité).

## ⚠️ À ne pas confondre / obsolète

- **`isc-dhcp-server` n'existe plus dans Debian 13** → **Kea** ou **dnsmasq** (un assistant de migration Kea existe).
- **dnsmasq vs systemd-resolved** : les deux veulent le **port 53** → n'en activer qu'un.
- `dnsmasq` = pratique en lab (DNS+DHCP) ; `bind9` = DNS autoritaire « sérieux » ; `Kea` = DHCP à grande échelle.

---

## Sources (doc officielle)

- [Debian Wiki — DHCP_Server](https://wiki.debian.org/DHCP_Server) — consulté le 24/07/2026
- [Notes de version Debian 13 — points d'attention (retrait isc-dhcp)](https://www.debian.org/releases/stable/release-notes/issues.html) — consulté le 24/07/2026
- [dnsmasq(8) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/dnsmasq-base/dnsmasq.8.en.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées · [x] rien d'obsolète (**isc-dhcp retiré → Kea/dnsmasq**) · [x] config à tester en lab · [x] conforme doc Debian · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
