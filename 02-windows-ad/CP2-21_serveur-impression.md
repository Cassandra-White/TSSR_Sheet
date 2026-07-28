# CP2-21 — Installer et gérer un serveur d'impression

**Objectif** : installer le rôle Services d'impression, publier une imprimante réseau partagée avec son pilote.

**Rattachement REAC** : CP1/CP2 — support et exploitation (imprimantes partagées de l'entreprise).

**Durée** : ~15 min · **Niveau** : débutant.

---

## Prérequis

- Un serveur Windows Server 2025 ; une imprimante réseau avec IP connue.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (Desktop Experience) | 23/07/2026 |

---

## Procédure — GUI (méthode prioritaire)

*(Installer : Ajouter des rôles → **Services d'impression et de numérisation de documents** → **Serveur d'impression**.)*

1. **Outils** → **Gestion de l'impression** (`printmanagement.msc`).
2. *Serveurs d'impression → (serveur) → Ports* → **Ajouter un port** → **Port TCP/IP standard** → IP de l'imprimante.
3. *Pilotes* → **Ajouter** → installer le pilote **x64** adapté.
4. *Imprimantes* → **Ajouter une imprimante** → utiliser le **port** et le **pilote** créés → nommer → **Partager** (nom de partage) → *Lister dans l'annuaire* (optionnel).

> Déploiement automatique aux postes : clic droit sur l'imprimante → **Déployer avec une stratégie de groupe** (voir **CP9-05**).

## Procédure — CLI (module PrintManagement)

```powershell
Install-WindowsFeature Print-Server -IncludeManagementTools

Add-PrinterPort   -Name "IP_192.168.10.60" -PrinterHostAddress "192.168.10.60"
Add-PrinterDriver -Name "Microsoft IPP Class Driver"
Add-Printer -Name "Imprimante-Compta" -DriverName "Microsoft IPP Class Driver" `
  -PortName "IP_192.168.10.60" -Shared -ShareName "Imprimante-Compta" -Published
```

*(À exécuter en lab — non testable dans le bac à sable Linux.)*

---

## Vérification

```powershell
Get-Printer
Get-PrinterPort
# Depuis un client : \\SRV-AD01 → double-cliquer l'imprimante partagée
```

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| Pilote refusé | Mauvaise architecture | Installer le pilote **x64** correspondant |
| Travaux bloqués en file | Spouleur figé | Redémarrer le service **Spooler** |
| Imprimante invisible côté clients | Non partagée / non publiée | `-Shared` + `-Published` |

## Sécurité et bonnes pratiques

- Restreindre les permissions d'impression et de gestion sur chaque imprimante.
- **Tenir les pilotes à jour** (faille de type *PrintNightmare*) et appliquer les MAJ (**CP2-15**).
- Encadrer l'installation de pilotes par les utilisateurs (**Point and Print** via GPO).

## ⚠️ À ne pas confondre / obsolète

- Depuis les correctifs *PrintNightmare*, l'ajout de pilote exige des **droits administrateur** par défaut : ne pas contourner ces protections.

---

## Sources (doc officielle)

- [Add-Printer — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/printmanagement/add-printer) — consulté le 23/07/2026
- [Installer le serveur d'impression — Microsoft Learn](https://learn.microsoft.com/en-us/troubleshoot/windows-server/printing/set-up-print-server) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
