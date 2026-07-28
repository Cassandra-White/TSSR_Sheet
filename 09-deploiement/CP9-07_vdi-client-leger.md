# CP9-07 — Mettre en place une infrastructure VDI / client léger (bases)

**Objectif** : comprendre et déployer les bases d'un poste **virtualisé (VDI)** accessible depuis un **client léger**, en local (RDS) ou dans le Cloud (Azure Virtual Desktop / Windows 365).

**Rattachement REAC** : CP9 « Exploiter et maintenir les services de déploiement des postes » — savoir-faire : mettre en œuvre la virtualisation de postes.

**Durée** : ~30 min · **Niveau** : intermédiaire.

---

## Prérequis

- Pour le **on-prem** : un domaine AD (**CP2-03**), un hôte de virtualisation (**CP5**), des licences RDS (CAL).
- Pour le **cloud** : un abonnement **Azure** + **Entra ID**.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| VDI on-prem | **Remote Desktop Services** (WS 2025) | 24/07/2026 |
| VDI cloud | **Azure Virtual Desktop** / **Windows 365** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **VDI** = bureaux **centralisés** sur des serveurs ; l'utilisateur s'y connecte via un **protocole d'affichage** (RDP) depuis un **client léger** (peu de ressources locales). Deux familles : **session-based (RDSH)** — plusieurs sessions sur un serveur partagé — et **VDI** — une **VM par utilisateur** (*pooled* non-persistant ou *personal* persistant).

---

## Procédure — GUI

### On-prem — Remote Desktop Services

1. **Server Manager ▸ Add Roles ▸ Remote Desktop Services** → scénario *Session-based* ou *VDI*.
2. Déployer les rôles : **Connection Broker**, **Web Access**, **Session/Virtualization Host**.
3. Créer une **collection** (*pooled*/*personal*), publier un **bureau** ou des **RemoteApp**.
4. L'utilisateur se connecte via **RD Web** ou l'app **Bureau à distance**.

### Cloud — Azure Virtual Desktop / Windows 365

- **AVD** (portail Azure) : créer un **host pool**, un **workspace**, un **app group**, affecter les utilisateurs (Entra).
- **Windows 365** : **Cloud PC** à **coût fixe**, sans infrastructure à gérer (plus simple).

### Clients légers

- Matériel léger (IGEL, NComputing, ThinOS…) ou **reconversion** de PC anciens (ex. **LTSP** sous Linux) — le calcul se fait côté serveur.

---

## Vérification (comment savoir que ça marche)

- L'utilisateur ouvre une **session distante** et retrouve son bureau/ses apps.
- Sur client léger, l'**audio/vidéo Teams** est bien **déchargé** (offload) côté terminal.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Connexion refusée | **Licences RDS (CAL)** manquantes | Configurer le serveur de licences RDS |
| Session lente | Protocole/GPU / profils | GPU, réglages RDP, **FSLogix** pour les profils |
| Apps Office KO sur RDS | Support **M365 Apps** WS 2016/2019 **terminé (14/10/2025)** | Utiliser **WS 2022/2025** |
| Bureaux non persistants perdent les données | Collection *pooled* | Choisir *personal* ou rediriger les profils |

## Sécurité et bonnes pratiques

- **MFA / Accès conditionnel** (**CP7-18**, **CP9-09**) sur l'accès aux bureaux distants.
- **Aucune donnée locale** sur le client léger (tout reste au centre) → vol de terminal sans impact données.
- **Patcher** les hôtes (**CP2-15**) et isoler le réseau de la ferme VDI.

## ⚠️ À ne pas confondre / obsolète

- **RDSH** (sessions partagées, 1 serveur) ≠ **VDI** (1 VM par utilisateur).
- **VMware Horizon → Omnissa** (indépendant depuis la cession Broadcom) ; **Citrix** impose un **nouveau modèle de licence (15/04/2026)**.
- **AVD** (flexible, à l'usage) ≠ **Windows 365** (Cloud PC, coût fixe).

---

## Sources (doc officielle)

- [Microsoft Learn — Remote Desktop Services](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/rds-overview) — consulté le 24/07/2026
- [Microsoft — Azure Virtual Desktop](https://learn.microsoft.com/en-us/azure/virtual-desktop/overview) — consulté le 24/07/2026
- [Microsoft — Windows 365 (Cloud PC)](https://learn.microsoft.com/en-us/windows-365/overview) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI (on-prem + cloud) · [x] versions datées (WS 2025 / AVD) · [x] rien d'obsolète (Omnissa, fin M365 Apps WS2019) · [x] procédure **à tester en lab** · [x] conforme doc Microsoft · [x] vérification présente · [x] sécurité (MFA, pas de données locales) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
