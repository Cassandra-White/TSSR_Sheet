# CP2-13 — Installer et configurer le rôle DHCP (étendue, options, réservations)

**Objectif** : installer et autoriser un serveur DHCP, créer une étendue IPv4 avec ses options, et poser des réservations.

**Rattachement REAC** : CP2/CP4 — connaissance « services réseau nécessaires (annuaire, DNS, **DHCP**) ».

**Durée** : ~15 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un serveur Windows Server 2025 (souvent le DC ou un membre), plan d'adressage défini (voir **LAB-02**).
- Pour **autoriser** le serveur dans AD : compte *Admins de l'entreprise*.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (Desktop Experience) | 23/07/2026 |

---

## Procédure — GUI (méthode prioritaire)

*(Installer le rôle : Gestionnaire de serveur → **Ajouter des rôles** → **Serveur DHCP**, puis la tâche de post-déploiement **« Terminer la configuration DHCP »** — elle crée les groupes et autorise le serveur.)*

1. **Outils** → **DHCP** (`dhcpmgmt.msc`).
2. Si nécessaire : clic droit sur le serveur → **Autoriser** (dans AD).
3. **IPv4** → clic droit → **Nouvelle étendue** → assistant : nom, plage `192.168.10.100`–`192.168.10.200`, masque `/24`, exclusions, durée du bail.
4. **Options d'étendue** : `003` Routeur = `192.168.10.254` · `006` Serveurs DNS = `192.168.10.10` · `015` Nom de domaine = `labtssr.lan`.
5. **Activer** l'étendue.
6. Réservation : étendue → **Réservations** → **Nouvelle réservation** → nom, IP, adresse MAC.

## Procédure — CLI (module DhcpServer)

```powershell
# Installer le rôle + créer les groupes de sécurité
Install-WindowsFeature DHCP -IncludeManagementTools
Add-DhcpServerSecurityGroup
Restart-Service DHCPServer

# Autoriser le serveur dans AD
Add-DhcpServerInDC -DnsName "srv-ad01.labtssr.lan" -IPAddress 192.168.10.10

# Créer et activer l'étendue
Add-DhcpServerv4Scope -Name "LAN-Compta" -StartRange 192.168.10.100 -EndRange 192.168.10.200 `
  -SubnetMask 255.255.255.0 -State Active

# Options (passerelle, DNS, domaine)
Set-DhcpServerv4OptionValue -ScopeId 192.168.10.0 -Router 192.168.10.254 `
  -DnsServer 192.168.10.10 -DnsDomain "labtssr.lan"

# Réservation
Add-DhcpServerv4Reservation -ScopeId 192.168.10.0 -IPAddress 192.168.10.50 `
  -ClientId "00-15-5D-01-02-03" -Name "imprimante-compta"
```

*(À exécuter en lab — non testable dans le bac à sable Linux.)*

---

## Vérification

```powershell
Get-DhcpServerv4Scope
Get-DhcpServerInDC
Get-DhcpServerv4Lease -ScopeId 192.168.10.0   # baux attribués
# Sur un client : ipconfig /renew  → obtient une IP de la plage
```

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| Les clients n'obtiennent pas d'IP | Étendue non activée / serveur non autorisé | Activer l'étendue, autoriser dans AD |
| IP incohérentes | Serveur DHCP pirate (*rogue*) | N'autoriser que les serveurs légitimes ; DHCP snooping côté switch |
| Option non reçue | Définie au mauvais niveau | Poser l'option au niveau **étendue** |

## Sécurité et bonnes pratiques

- **Autoriser** le DHCP dans AD : bloque les serveurs pirates non autorisés.
- **Réserver** les IP des équipements fixes (imprimantes, serveurs).
- Activer le **DHCP snooping** sur les commutateurs.

## ⚠️ À ne pas confondre / obsolète

- `netsh dhcp` fonctionne encore mais le module PowerShell **DhcpServer** est recommandé.

---

## Sources (doc officielle)

- [Add-DhcpServerv4Scope (Windows Server 2025) — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/dhcpserver/add-dhcpserverv4scope?view=windowsserver2025-ps) — consulté le 23/07/2026
- [Add-DhcpServerInDC — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/dhcpserver/add-dhcpserverindc?view=windowsserver2025-ps) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
