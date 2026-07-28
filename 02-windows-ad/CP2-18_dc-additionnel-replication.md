# CP2-18 — Ajouter un contrôleur de domaine supplémentaire (redondance + réplication)

**Objectif** : promouvoir un second contrôleur de domaine (réplica) dans le domaine existant, pour la tolérance de panne, et vérifier la réplication.

**Rattachement REAC** : CP2 — haute disponibilité de l'annuaire ; connaissance des annuaires et de la sécurité.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Domaine existant (voir **CP2-03**).
- Un 2ᵉ serveur Windows Server 2025 (ex. `SRV-AD02`), IP fixe, **DNS pointant sur le 1ᵉʳ DC**.
- Compte *Admins du domaine* et un mot de passe DSRM.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (Desktop Experience) | 23/07/2026 |

---

## Procédure — GUI (méthode prioritaire)

1. Sur `SRV-AD02` : DNS préféré = **IP du DC existant** (`SRV-AD01`).
2. **Gestionnaire de serveur** → **Ajouter des rôles** → **Services AD DS** → **Installer**.
3. Drapeau ⚑ → **Promouvoir ce serveur en contrôleur de domaine**.
4. **Ajouter un contrôleur de domaine à un domaine existant** → domaine `labtssr.lan` → fournir les identifiants *Admins du domaine*.
5. Options : cocher **Serveur DNS** + **Catalogue global (GC)**, choisir le **site** AD, saisir le **mot de passe DSRM** → **Suivant**.
6. **Installer** → le serveur redémarre.

## Procédure — CLI (alternative / automatisation)

```powershell
# Sur le futur DC (accès au domaine requis)
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

Install-ADDSDomainController -DomainName "labtssr.lan" `
  -InstallDns -Credential (Get-Credential "LABTSSR\Administrateur") -Force
# -> demande le mot de passe DSRM, puis redémarre automatiquement
```

*(À exécuter en lab — non testable dans le bac à sable Linux.)*

---

## Vérification

```powershell
Get-ADDomainController -Filter * | Select-Object HostName, Site, IsGlobalCatalog
repadmin /replsummary      # réplication saine, 0 échec
repadmin /showrepl
dcdiag /q
```

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| Promotion échoue (DNS) | DNS ne pointe pas sur le DC existant | Corriger le DNS préféré |
| Réplication en erreur | Pare-feu / ports AD / horloge | Ouvrir les ports AD entre DC, synchroniser l'heure |
| Pas de Catalogue global | Option non cochée | Activer le GC (propriétés *NTDS Settings*) |

## Sécurité et bonnes pratiques

- **Au moins deux DC** pour la tolérance de panne.
- Connaître l'emplacement des **rôles FSMO** (voir **CP2-19**).
- **Sauvegarder l'état système** d'un DC régulièrement (voir **CP8-03**).

## ⚠️ À ne pas confondre / obsolète

- `dcpromo` est supprimé → utiliser `Install-ADDSDomainController` (ou l'assistant du Gestionnaire de serveur).
- Un **réplica** partage la même base AD ; ce n'est pas un domaine séparé.

---

## Sources (doc officielle)

- [Install-ADDSDomainController (Windows Server 2025) — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/addsdeployment/install-addsdomaincontroller?view=windowsserver2025-ps) — consulté le 23/07/2026
- [repadmin — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/repadmin) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
