# CP4-02 — Configurer une interface et tester la connectivité

**Objectif** : attribuer une adresse IP à une interface et diagnostiquer la connectivité, côté **Windows** et côté **Linux**.

**Rattachement REAC** : CP4 « Exploiter un réseau IP » — savoir-faire : configurer et vérifier la connectivité d'un poste/serveur.

**Durée** : ~15 min · **Niveau** : débutant.

---

## Prérequis

- Un poste Windows 11 / Windows Server 2025 et/ou un serveur Debian 13. Plan d'adressage connu (**CP4-01**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows | 11 24H2 / Server 2025 | 24/07/2026 |
| Debian (`ip`, `ping`) | 13.6 — **commandes `ip` validées** (voir CP3-02) | 24/07/2026 |

---

## Procédure — GUI (Windows, méthode prioritaire)

1. **Paramètres → Réseau et Internet → Ethernet → Modifier** l'attribution IP (ou `ncpa.cpl` → *Propriétés* → **Protocole Internet version 4 (TCP/IPv4)**).
2. Choisir **Manuel/Utiliser l'adresse suivante** : IP, masque, passerelle, DNS.
3. Valider.

## Procédure — CLI

### Windows

```powershell
ipconfig /all                                   # afficher la configuration complète
# Configuration moderne (PowerShell) :
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.1.10 `
  -PrefixLength 24 -DefaultGateway 192.168.1.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.1.1
Get-NetIPConfiguration

# En DHCP :
ipconfig /release ; ipconfig /renew ; ipconfig /flushdns
```

### Linux (Debian)

```bash
ip -c addr            # afficher les adresses
ip route             # table de routage
# Config permanente : voir CP3-02 (/etc/network/interfaces)
```

## Tester la connectivité

```bash
ping 192.168.1.1                 # passerelle (Windows : 4 pings ; Linux : ping -c4)
ping 1.1.1.1                     # accès IP externe
ping debian.org                  # + résolution DNS
```

```powershell
tracert 8.8.8.8                  # (Windows) chemin routé   |  Linux : traceroute / tracepath / mtr
pathping 8.8.8.8                 # (Windows) pertes par saut
Test-NetConnection intranet.lab.local -Port 443   # (Windows) test d'un port TCP
```

---

## Vérification (comment savoir que ça marche)

- `ipconfig` / `ip addr` affichent l'IP attendue (pas d'adresse **169.254.x.x**).
- `ping` de la passerelle **répond**, puis d'une IP externe, puis d'un nom (DNS).

## Dépannage (démarche couche par couche)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Adresse **169.254.x.x** (APIPA) | Pas de bail DHCP | Vérifier le lien, le serveur DHCP, `ipconfig /renew` |
| Ping passerelle KO | IP/masque/câble | Vérifier l'adressage et le lien physique |
| Ping IP OK mais nom KO | DNS | Corriger le serveur DNS ; `ipconfig /flushdns` |
| « Destination host unreachable » | Pas de route/passerelle | Vérifier la passerelle par défaut |

## Sécurité et bonnes pratiques

- **Documenter** l'adressage (cohérence avec le plan **CP4-01**).
- Désactiver les interfaces réseau **inutilisées**.

## ⚠️ À ne pas confondre / obsolète

- **Windows** : `tracert` ; **Linux** : `traceroute`/`tracepath`/`mtr`.
- **Linux** : `ifconfig`/`netstat` (net-tools) sont **obsolètes** → `ip`/`ss`.
- **Windows** : `netsh interface ip …` fonctionne encore, mais **PowerShell** (`New-NetIPAddress`) est recommandé.
- **169.254.x.x** n'est pas une « vraie » IP : c'est le signe d'un **DHCP injoignable**.

---

## Sources (doc officielle)

- [Test-NetConnection — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/nettcpip/test-netconnection) — consulté le 24/07/2026
- [New-NetIPAddress — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/nettcpip/new-netipaddress) — consulté le 24/07/2026
- [ip(8) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/iproute2/ip.8.en.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI (Windows) puis CLI (Win + Linux) · [x] versions datées · [x] rien d'obsolète (`ip` vs `ifconfig`) · [x] commandes `ip` validées / autres à tester en lab · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
