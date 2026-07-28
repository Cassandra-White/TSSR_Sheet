# CP2-02 — Configurer IP fixe, nom d'hôte et fuseau horaire

**Objectif** : attribuer au serveur une adresse IP fixe (passerelle + DNS), le renommer et régler son fuseau horaire — préalable indispensable avant tout rôle.

**Rattachement REAC** : CP2 « Exploiter des serveurs Windows… » — savoir-faire : « Exploiter un serveur Windows », préparation à l'intégration au domaine.

**Durée** : ~15 min · **Niveau** : débutant.

---

## Prérequis

- Windows Server 2025 installé (voir **CP2-01**), session **Administrateur** ouverte.
- Le plan d'adressage du lab (voir **LAB-02**) : IP, masque, passerelle, DNS à utiliser.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (Desktop Experience) | 23/07/2026 |

> Exemple utilisé : IP `192.168.10.10/24`, passerelle `192.168.10.254`, nom `SRV-AD01`.

---

## Procédure — GUI (méthode prioritaire)

**Adresse IP fixe**
1. Menu Exécuter (`Win+R`) → `ncpa.cpl` → clic droit sur la carte réseau → **Propriétés**.
2. Sélectionner **Protocole Internet version 4 (TCP/IPv4)** → **Propriétés**.
3. Cocher **Utiliser l'adresse IP suivante** :
   - Adresse IP : `192.168.10.10`
   - Masque : `255.255.255.0`
   - Passerelle : `192.168.10.254`
4. **Serveur DNS préféré** : pour un futur contrôleur de domaine, indiquer **sa propre adresse** (ou `127.0.0.1`) ; sinon, le DNS interne de l'entreprise.
5. **OK** pour valider.

**Nom d'hôte et fuseau** (Gestionnaire de serveur → **Serveur local**)
6. Cliquer le nom d'ordinateur (ex. `WIN-XXXX`) → **Modifier** → saisir `SRV-AD01` → **OK** → **Redémarrer**.
7. Après redémarrage : **Serveur local** → **Fuseau horaire** → régler sur *(UTC+01:00) Bruxelles, Paris…*.

## Procédure — CLI (alternative / automatisation)

```powershell
# 1. Repérer l'index de la carte (colonne ifIndex)
Get-NetAdapter

# 2. Adresse IP fixe + passerelle (adapter -InterfaceIndex)
New-NetIPAddress -InterfaceIndex 5 -IPAddress 192.168.10.10 -PrefixLength 24 -DefaultGateway 192.168.10.254

# 3. Serveur DNS
Set-DnsClientServerAddress -InterfaceIndex 5 -ServerAddresses 127.0.0.1

# 4. Fuseau horaire (France)
Set-TimeZone -Name "Romance Standard Time"

# 5. Renommer puis redémarrer
Rename-Computer -NewName "SRV-AD01" -Restart
```

Alternative menu texte : `sconfig` → **8) Paramètres réseau**, **2) Nom de l'ordinateur**, **9) Date et heure**.

*(PowerShell à exécuter en lab — non testable dans le bac à sable Linux.)*

---

## Vérification

```powershell
Get-NetIPConfiguration          # IP, passerelle et DNS attendus
Test-Connection 192.168.10.254  # ping passerelle
Resolve-DnsName microsoft.com   # résolution DNS
hostname                        # doit renvoyer SRV-AD01
```

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| « Adresse IP en conflit » | IP déjà utilisée | Choisir une IP libre / vérifier les baux DHCP |
| Pas d'accès réseau | Passerelle ou DNS erronés | Corriger via `Get-NetIPConfiguration` |
| `Rename-Computer` échoue | Serveur déjà membre d'un domaine | Renommer **avant** la jonction au domaine |

## Sécurité et bonnes pratiques

- Un serveur (surtout un **DC**) doit avoir une **IP fixe**.
- Le DNS d'un futur DC doit pointer vers **lui-même**, jamais vers la box du FAI.
- Nom d'hôte explicite et normalisé (ex. `SRV-AD01`) — le fixer **avant** la promotion.

## ⚠️ À ne pas confondre / obsolète

- Préférer les cmdlets **NetTCPIP** (`New-NetIPAddress`, `Set-DnsClientServerAddress`) à l'ancien `netsh interface ip set address` (encore fonctionnel mais non recommandé).
- `ipconfig /all` reste utile pour l'**affichage** et le diagnostic.

---

## Sources (doc officielle)

- [New-NetIPAddress — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/nettcpip/new-netipaddress) — consulté le 23/07/2026
- [Set-DnsClientServerAddress — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/dnsclient/set-dnsclientserveraddress) — consulté le 23/07/2026
- [Rename-Computer — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/rename-computer) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
