# CP3-02 — Configurer le réseau (IP fixe, DNS) sous Debian 13

**Objectif** : attribuer une adresse IP fixe, une passerelle et des serveurs DNS à un serveur Debian 13.

**Rattachement REAC** : CP3 « Exploiter des serveurs Linux » — savoir-faire : configurer l'accès réseau d'un serveur Linux.

**Durée** : ~15 min · **Niveau** : débutant.

---

## Prérequis

- Debian 13 installé (**CP3-01**), accès console ou SSH, droits **root/sudo**.
- Plan d'adressage connu : IP, masque, passerelle, DNS (hors plage DHCP).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian | 13.6 « trixie » | 24/07/2026 |
| Commandes `ip` / `ip route` | iproute2 — **testées en namespace réseau** (bac à sable) | 24/07/2026 |

---

## Repérer l'interface

```bash
ip -c link        # noms d'interfaces (ex. enp1s0, ens18) — plus « eth0 »
ip -c addr        # adresses actuelles
```

> Debian nomme les interfaces de façon **prévisible** (`enpXsY`/`ensXX`), pas `eth0`.

## Procédure — méthode par défaut Debian : `/etc/network/interfaces` (ifupdown)

> Une installation **serveur** Debian utilise **ifupdown** par défaut. On édite le fichier de configuration.

```bash
sudo nano /etc/network/interfaces
```

Bloc pour une IP fixe (adapter le nom d'interface) :

```text
auto enp1s0
iface enp1s0 inet static
    address 192.168.10.20/24
    gateway 192.168.10.1
    dns-nameservers 192.168.10.10 1.1.1.1
```

Appliquer :

```bash
sudo systemctl restart networking
# ou, ciblé sur une interface :
sudo ifdown enp1s0 && sudo ifup enp1s0
```

- `address … /24` : notation **CIDR** (remplace l'ancien couple `address` + `netmask`).
- `dns-nameservers` requiert le paquet **resolvconf** ; sinon, renseigner directement `/etc/resolv.conf`.

## Procédure — alternative : systemd-networkd (moderne, non activé par défaut)

```bash
sudo tee /etc/systemd/network/20-lan.network >/dev/null <<'EOF'
[Match]
Name=enp1s0

[Network]
Address=192.168.10.20/24
Gateway=192.168.10.1
DNS=192.168.10.10
DNS=1.1.1.1
EOF

sudo systemctl enable --now systemd-networkd systemd-resolved
# Éviter le conflit avec ifupdown :
sudo mv /etc/network/interfaces /etc/network/interfaces.bak
```

> Avec systemd-resolved, `/etc/resolv.conf` pointe vers le résolveur local `127.0.0.53`.

---

## Vérification (commandes validées dans le bac à sable)

```bash
ip -4 addr show enp1s0      # l'IP 192.168.10.20/24 doit apparaître
ip route                    # ligne « default via 192.168.10.1 »
resolvectl status           # (systemd-resolved) serveurs DNS actifs
cat /etc/resolv.conf        # (ifupdown) nameservers
ping -c2 192.168.10.1       # passerelle joignable
ping -c2 debian.org         # résolution DNS + accès Internet
```

Sortie type obtenue lors du test (interface de lab `eth-lab`) :

```
2: eth-lab: <BROADCAST,NOARP,UP,LOWER_UP> ... inet 192.168.10.10/24 scope global eth-lab
default via 192.168.10.1 dev eth-lab
192.168.10.0/24 dev eth-lab proto kernel scope link src 192.168.10.10
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Config non appliquée | Mauvais nom d'interface | Vérifier avec `ip link`, corriger le bloc |
| Pas de résolution DNS | `/etc/resolv.conf` vide / resolvconf absent | Installer `resolvconf` ou éditer `/etc/resolv.conf` |
| Perte réseau après reboot | ifupdown **et** systemd-networkd actifs | N'en garder **qu'un** ; désactiver l'autre |
| « Address already in use » | IP déjà attribuée sur le LAN | Choisir une autre IP (hors DHCP), vérifier avec `arping` |

## Sécurité et bonnes pratiques

- Réserver l'IP fixe **hors de la plage DHCP** pour éviter les conflits.
- **Documenter** l'adressage (**CP4-01**).
- Après connectivité, penser au pare-feu local (**CP3-14**).

## ⚠️ À ne pas confondre / obsolète

- **`ifconfig` est obsolète** (paquet *net-tools*) → utiliser **`ip a`** / `ip addr`. De même : `route` → **`ip route`**, `netstat` → **`ss`**.
- Ne pas activer **ifupdown** et **systemd-networkd** sur la même interface.
- Sous Debian, **pas de netplan** (outil spécifique à Ubuntu).

---

## Sources (doc officielle)

- [Debian Reference — Chapter 5. Network setup](https://www.debian.org/doc/manuals/debian-reference/ch05.en.html) — consulté le 24/07/2026
- [Debian Wiki — systemd-networkd](https://wiki.debian.org/SystemdNetworkd) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (ici : fichier de conf + commandes) · [x] versions datées · [x] rien d'obsolète (`ip` vs `ifconfig`) · [x] **commandes `ip`/route testées dans le bac à sable** · [x] méthode conforme doc Debian · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
