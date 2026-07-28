# CP2-22 — Configurer l'agrégation de cartes réseau (NIC teaming)

**Objectif** : regrouper au moins deux cartes réseau physiques du serveur en une seule interface logique, pour obtenir la **redondance** (tolérance de panne d'une carte, d'un câble ou d'un commutateur) et l'**agrégation de bande passante**.

**Rattachement REAC** : CP2 « Exploiter des serveurs Windows et un domaine Active Directory » — savoir-faire : assurer la disponibilité et la continuité de service réseau du serveur.

**Durée** : ~15 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un serveur **Windows Server 2025** avec **au moins deux cartes réseau** de même vitesse (idéalement même modèle) — le teaming ne prend pas en compte des débits différents.
- Droits **administrateur local**.
- Côté commutateur : rien à faire en mode *Switch Independent* ; en mode *Static* ou *LACP*, l'agrégation doit être configurée sur le switch (voir **CP4-14**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (Desktop Experience) | 24/07/2026 |

---

## Deux technologies — laquelle choisir ?

| Besoin | Technologie | Interface |
|---|---|---|
| Teaming du **trafic de l'hôte** (serveur physique, pas de Hyper-V) | **LBFO** (*Load Balancing / Failover*) | **GUI** (Gestionnaire de serveur) **+ PowerShell** |
| Teaming rattaché à un **commutateur virtuel Hyper-V** | **SET** (*Switch Embedded Teaming*) | **PowerShell / Windows Admin Center** (pas de GUI) |

> ⚠️ **Point d'actualité (WS 2025)** : rattacher un commutateur virtuel Hyper-V à une équipe **LBFO** est désormais **bloqué**. Pour un hôte Hyper-V, on utilise **obligatoirement SET** (voir la section CLI et l'encart « obsolète » plus bas).

---

## Procédure — GUI (méthode prioritaire) — équipe LBFO

1. **Gestionnaire de serveur** → **Serveur local**. En face de **Association de cartes réseau (NIC Teaming)**, cliquer sur **Désactivé** : la console `lbfoadmin` s'ouvre.
2. Volet **Équipes** → menu **TÂCHES** → **Nouvelle équipe**.
3. **Nommer** l'équipe (ex. `Team-LAN`) et **cocher** au moins deux cartes membres (ex. `Ethernet`, `Ethernet 2`).
4. Déplier **Propriétés supplémentaires** :
   - **Mode d'association** : *Indépendant du commutateur* (défaut, aucune config switch, permet de répartir les cartes sur **deux switches** distincts) · *Statique* · *LACP* (négociation dynamique, exige un switch configuré).
   - **Mode d'équilibrage de charge** : **Dynamique** (recommandé).
   - **Carte de secours** : laisser *Aucune* pour agréger les débits, ou désigner une carte *Veille* pour du pur failover actif/passif.
5. **OK**. Une **interface d'équipe** virtuelle apparaît (ex. `Team-LAN`).
6. Ouvrir **Connexions réseau** (`ncpa.cpl`) et **affecter l'adresse IP fixe sur l'interface d'équipe** (comme dans **CP2-02**) — **jamais** sur les cartes physiques membres.

## Procédure — CLI (alternative / automatisation)

### A. Équipe LBFO en PowerShell (teaming d'hôte)

```powershell
# 1. Repérer les cartes physiques
Get-NetAdapter

# 2. Créer l'équipe : indépendant du commutateur + équilibrage dynamique
New-NetLbfoTeam -Name "Team-LAN" -TeamMembers "Ethernet","Ethernet 2" `
  -TeamingMode SwitchIndependent -LoadBalancingAlgorithm Dynamic -Confirm:$false

# Variante LACP (switch configuré en 802.1ax) :
# New-NetLbfoTeam -Name "Team-LAN" -TeamMembers "Ethernet","Ethernet 2" `
#   -TeamingMode LACP -LoadBalancingAlgorithm Dynamic -Confirm:$false

# 3. Affecter l'IP sur l'interface d'équipe (et non sur les membres)
New-NetIPAddress -InterfaceAlias "Team-LAN" -IPAddress 192.168.10.10 `
  -PrefixLength 24 -DefaultGateway 192.168.10.1
Set-DnsClientServerAddress -InterfaceAlias "Team-LAN" -ServerAddresses 192.168.10.10
```

- `-TeamingMode` : `SwitchIndependent` (défaut) · `Static` · `Lacp`.
- `-LoadBalancingAlgorithm` : `Dynamic` (recommandé) · `HyperVPort` · `TransportPorts` / `IPAddresses` / `MacAddresses`.

### B. Équipe SET en PowerShell (hôte Hyper-V — pas de GUI)

```powershell
# Rôle Hyper-V requis. SET s'intègre AU commutateur virtuel.
New-VMSwitch -Name "SET-Switch" -NetAdapterName "Ethernet","Ethernet 2" `
  -EnableEmbeddedTeaming $true -AllowManagementOS $true

# Algorithme d'équilibrage : Dynamic ou HyperVPort (SET = toujours Switch Independent)
Set-VMSwitchTeam -Name "SET-Switch" -LoadBalancingAlgorithm Dynamic
```

*(Commandes non testables dans le bac à sable Linux — **à exécuter en lab**. Syntaxe vérifiée sur la doc Microsoft Learn, vue Windows Server 2025.)*

---

## Vérification (comment savoir que ça marche)

```powershell
# LBFO : l'équipe doit être « Up » et les membres « Active »
Get-NetLbfoTeam
Get-NetLbfoTeamMember

# SET :
Get-VMSwitchTeam
```

- Test de **redondance** : lancer un `ping -t` vers le serveur, **débrancher un câble** d'une carte membre → le ping ne doit **pas** s'interrompre.
- Dans la console NIC Teaming (GUI), l'équipe et chaque membre s'affichent en vert (**Actif**).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Équipe/membre en **Faulted** ou *Disconnected* | Câble, ou mode incompatible avec le switch | Vérifier le câblage ; en LACP/Static, configurer l'agrégation côté switch (**CP4-14**) |
| Le débit n'augmente pas sur **un seul transfert** | Un flux unique reste sur **une** carte | Normal : le gain vient de **flux multiples** ; le failover, lui, fonctionne dès 2 cartes |
| **Impossible de rattacher un vSwitch Hyper-V** à l'équipe | LBFO **bloqué** pour Hyper-V en WS 2025 | Recréer en **SET** (`New-VMSwitch -EnableEmbeddedTeaming`) |
| LACP « ne monte pas » | EtherChannel/LACP absent côté switch | Configurer le port-channel LACP sur le commutateur |

## Sécurité et bonnes pratiques

- **Cartes identiques** (même vitesse/modèle) ; ne pas mélanger 1 Gb/s et 10 Gb/s.
- **Redondance de commutateur** : mode *Indépendant du commutateur* + cartes réparties sur **deux switches** → survit à la panne d'un switch entier.
- **IP uniquement sur l'interface d'équipe** (LBFO) ou sur le vSwitch (SET), jamais sur les cartes physiques membres.
- **Documenter** le mappage carte ↔ port de switch (câblage) pour faciliter le diagnostic.

## ⚠️ À ne pas confondre / obsolète

- **LBFO est déprécié comme *uplink* d'un commutateur virtuel Hyper-V** (depuis WS 2022) et **bloqué en WS 2025**. Pour Hyper-V → **SET** uniquement.
- **SET n'a pas d'interface graphique** dans le Gestionnaire de serveur : se configure en **PowerShell** ou via **Windows Admin Center** (**CP2-23**). SET n'accepte que le mode **Switch Independent** (2 à 8 cartes identiques).
- Ne pas confondre le **teaming côté hôte (Windows)** avec l'**agrégation côté commutateur (LACP / EtherChannel = CP4-14)** : en mode LACP, les deux moitiés doivent être configurées de façon cohérente.

---

## Sources (doc officielle)

- [New-NetLbfoTeam — Microsoft Learn (Windows Server 2025)](https://learn.microsoft.com/en-us/powershell/module/netlbfo/new-netlbfoteam?view=windowsserver2025-ps) — consulté le 24/07/2026
- [New-VMSwitch — Microsoft Learn (Windows Server 2025)](https://learn.microsoft.com/en-us/powershell/module/hyper-v/new-vmswitch?view=windowsserver2025-ps) — consulté le 24/07/2026
- [Enable NIC Teaming (LBFO) and Switch Embedded Teaming (SET) in Windows Server 2025 — 4sysops](https://4sysops.com/archives/enable-nic-teaming-lbfo-and-switch-embedded-teaming-set-in-windows-server-2025/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète (LBFO vs SET signalé) · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
