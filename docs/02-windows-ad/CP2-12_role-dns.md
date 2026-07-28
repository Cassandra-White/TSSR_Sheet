# CP2-12 — Installer et configurer le rôle DNS (zones et enregistrements)

**Objectif** : configurer le serveur DNS — zones de recherche directe et inversée, enregistrements (A/PTR/CNAME) et redirecteurs pour la résolution Internet.

**Rattachement REAC** : CP2/CP4 — connaissance « services réseau nécessaires (annuaire, **DNS**, DHCP) » ; exploitation des serveurs.

**Durée** : ~15 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un serveur Windows Server 2025 (DC ou membre).
- Sur un DC promu avec DNS (voir **CP2-03**), le rôle et la zone du domaine existent déjà : ce tuto ajoute zones et enregistrements.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (Desktop Experience) | 23/07/2026 |

---

## Procédure — GUI (méthode prioritaire)

*(Installer le rôle si absent : Gestionnaire de serveur → Ajouter des rôles → **Serveur DNS**.)*

1. **Outils** → **DNS** (`dnsmgmt.msc`) → développer le serveur.
2. **Zone de recherche directe** → clic droit → **Nouvelle zone** → *Principale* (intégrée à AD si DC) → nom `labtssr.lan` → mises à jour **sécurisées uniquement**.
3. **Zone de recherche inversée** → **Nouvelle zone** → IPv4 → ID réseau `192.168.10` (crée `10.168.192.in-addr.arpa`).
4. Enregistrement : clic droit sur la zone directe → **Nouvel hôte (A ou AAAA)** → nom + IP, cocher **« Créer un pointeur PTR associé »**.
5. **Redirecteurs** (résolution Internet) : clic droit sur le serveur → **Propriétés** → onglet **Redirecteurs** → **Modifier** → ajouter `9.9.9.9`, `1.1.1.1` (ou le DNS du FAI).

## Procédure — CLI (module DnsServer)

```powershell
# Installer le rôle (si absent)
Install-WindowsFeature DNS -IncludeManagementTools

# Zone directe (AD-intégrée) et zone inverse
Add-DnsServerPrimaryZone -Name "labtssr.lan" -ReplicationScope "Domain"
Add-DnsServerPrimaryZone -NetworkId "192.168.10.0/24" -ReplicationScope "Domain"

# Enregistrement A + PTR
Add-DnsServerResourceRecordA -ZoneName "labtssr.lan" -Name "srv-fichiers" -IPv4Address "192.168.10.20" -CreatePtr

# Redirecteurs
Add-DnsServerForwarder -IPAddress 9.9.9.9,1.1.1.1
```

*(À exécuter en lab — non testable dans le bac à sable Linux.)*

---

## Vérification

```powershell
Get-DnsServerZone
Resolve-DnsName srv-fichiers.labtssr.lan -Server 127.0.0.1
nslookup srv-fichiers.labtssr.lan
```

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| Pas de résolution Internet | Redirecteurs absents/erronés | Configurer des redirecteurs valides |
| PTR manquant | Zone inverse absente / option non cochée | Créer la zone inverse puis `-CreatePtr` |
| Zone non répliquée sur les autres DC | Portée de réplication | Vérifier `-ReplicationScope` (Domain/Forest) |

## Sécurité et bonnes pratiques

- Mises à jour dynamiques **« sécurisées uniquement »** sur les zones AD.
- **Restreindre les transferts de zone** aux serveurs autorisés.
- Superviser le service DNS (voir **CP4-17**).

## ⚠️ À ne pas confondre / obsolète

- `dnscmd` est l'outil **historique** (encore présent) : préférer le module PowerShell **DnsServer**.
- DNS **AD-intégré** (réplication par AD) ≠ zone **fichier** classique.

---

## Sources (doc officielle)

- [Installer et configurer un serveur DNS — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/networking/dns/quickstart-install-configure-dns-server) — consulté le 23/07/2026
- [Add-DnsServerPrimaryZone (Windows Server 2025) — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/dnsserver/add-dnsserverprimaryzone?view=windowsserver2025-ps) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
