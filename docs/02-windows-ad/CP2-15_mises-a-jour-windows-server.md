# CP2-15 — Gérer les mises à jour de Windows Server

**Objectif** : rechercher, installer et suivre les mises à jour de Windows Server, en interactif (GUI) et en ligne de commande.

**Rattachement REAC** : CP2 — savoir-faire : « suivre les mises à jour » ; veille technologique et sécurité.

**Durée** : ~10 min · **Niveau** : débutant.

---

## Prérequis

- Serveur Windows Server 2025, accès à Windows Update (ou à un serveur **WSUS**, voir **CP9-01**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (Desktop Experience) | 23/07/2026 |

---

## Procédure — GUI (méthode prioritaire)

1. **Paramètres** → **Windows Update** → **Rechercher des mises à jour** → **Installer** → **Redémarrer** si demandé.
2. En Server Core ou pour aller vite : `sconfig` → option **6 (Télécharger et installer les mises à jour)**.

## Procédure — CLI (module PSWindowsUpdate)

> ⚠️ **`PSWindowsUpdate` est un module communautaire** (PowerShell Gallery), **non intégré** à Windows. Il reste la voie pratique en ligne de commande ; validez-le selon la politique de votre entreprise.

```powershell
# 1. Installer le module (une seule fois)
Install-Module -Name PSWindowsUpdate -Scope AllUsers -Force

# 2. Lister les mises à jour disponibles
Get-WindowsUpdate -MicrosoftUpdate          # alias historique : Get-WUList

# 3. Installer toutes les mises à jour (+ redémarrage auto si nécessaire)
Install-WindowsUpdate -MicrosoftUpdate -AcceptAll -AutoReboot   # alias : Get-WUInstall

# Historique
Get-WUHistory
```

> Pour un **parc**, préférer la gestion **centralisée** (WSUS + GPO) — voir **CP9-01** — plutôt que serveur par serveur.

*(À exécuter en lab — non testable dans le bac à sable Linux.)*

---

## Vérification

```powershell
Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 5
Get-WURebootStatus     # indique si un redémarrage est requis
```

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| Erreur `0x80070422` | Service Windows Update arrêté | Démarrer le service `wuauserv` |
| Téléchargement bloqué | Proxy / pare-feu / WSUS injoignable | Vérifier l'accès à Windows Update ou au WSUS |
| Boucle de redémarrage | Mise à jour défaillante | Démarrage sans échec → `wusa /uninstall /kb:xxxxxxx` |

## Sécurité et bonnes pratiques

- Appliquer **rapidement** les mises à jour de sécurité.
- **Tester** sur un serveur pilote avant un déploiement de masse (anneaux de déploiement).
- Maîtriser le **calendrier** via WSUS/GPO (fenêtres de maintenance).

## ⚠️ À ne pas confondre / obsolète

- `wuauclt /detectnow` est **déprécié** sur Windows récents → utiliser `UsoClient` (intégré, limité) ou **PSWindowsUpdate**.
- Ne pas confondre mise à jour **qualité** (mensuelle) et mise à jour **de fonctionnalité** (nouvelle version d'OS).

---

## Sources (doc officielle / référence)

- [Configurer Windows Update / sconfig — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/administration/server-core/server-core-sconfig) — consulté le 23/07/2026
- [Module PSWindowsUpdate — PowerShell Gallery](https://www.powershellgallery.com/packages/PSWindowsUpdate) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
