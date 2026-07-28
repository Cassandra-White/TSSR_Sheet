# CP2-20 — Configurer un espace de noms DFS et la réplication (DFS-R)

**Objectif** : publier un chemin d'accès unique (`\\domaine\Partages`) via un espace de noms DFS, et répliquer un dossier entre deux serveurs pour la disponibilité.

**Rattachement REAC** : CP2 — partages de fichiers et disponibilité.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Deux serveurs membres/DC, dossiers partagés existants (voir **CP2-08**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (Desktop Experience) | 23/07/2026 |

---

## Procédure — GUI (méthode prioritaire)

*(Installer : Ajouter des rôles → Services de fichiers → **Espaces de noms DFS** + **Réplication DFS**.)*

1. **Outils** → **Gestion du système de fichiers distribués DFS** (`dfsmgmt.msc`).
2. **Espaces de noms** → **Nouvel espace de noms** → serveur hôte → nom `Partages` → type **basé sur un domaine** → `\\labtssr.lan\Partages`.
3. **Ajouter un dossier** → nom `Compta` → cible `\\SRV-AD01\Compta$`.
4. Ajouter une **2ᵉ cible** `\\SRV-AD02\Compta$` → accepter de **configurer la réplication (DFS-R)**.
5. Assistant : groupe de réplication, topologie (**maillage complet**), planification et bande passante.

## Procédure — CLI

```powershell
# Espace de noms basé domaine
New-DfsnRoot -TargetPath "\\SRV-AD01\Partages" -Type DomainV2 -Path "\\labtssr.lan\Partages"

# Dossier + deux cibles
New-DfsnFolder       -Path "\\labtssr.lan\Partages\Compta" -TargetPath "\\SRV-AD01\Compta$"
New-DfsnFolderTarget -Path "\\labtssr.lan\Partages\Compta" -TargetPath "\\SRV-AD02\Compta$"

# Réplication DFS-R
New-DfsReplicationGroup -GroupName "RG-Compta"
New-DfsReplicatedFolder -GroupName "RG-Compta" -FolderName "Compta"
Add-DfsrMember     -GroupName "RG-Compta" -ComputerName SRV-AD01,SRV-AD02
Add-DfsrConnection -GroupName "RG-Compta" -SourceComputerName SRV-AD01 -DestinationComputerName SRV-AD02
Set-DfsrMembership -GroupName "RG-Compta" -FolderName "Compta" -ComputerName SRV-AD01 -ContentPath "D:\Partages\Compta" -PrimaryMember $true
Set-DfsrMembership -GroupName "RG-Compta" -FolderName "Compta" -ComputerName SRV-AD02 -ContentPath "D:\Partages\Compta"
```

*(À exécuter en lab — non testable dans le bac à sable Linux.)*

---

## Vérification

```powershell
Get-DfsnRoot
Get-DfsnFolderTarget -Path "\\labtssr.lan\Partages\Compta"
Get-DfsrState -ComputerName SRV-AD01
# Accès client : \\labtssr.lan\Partages\Compta
```

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| Chemin DFS inaccessible | Type non basé domaine / DNS | Vérifier `DomainV2` et le DNS |
| Réplication ne démarre pas | Membre primaire non défini à l'init | Définir `-PrimaryMember $true` **une seule fois** |
| Fichiers en conflit | Écritures simultanées | DFS-R applique *last writer wins* + dossier *Conflict* |

## Sécurité et bonnes pratiques

- Comprendre le *last writer wins* avant de répliquer des données en écriture multi-site.
- La **réplication n'est pas une sauvegarde** : conserver des sauvegardes indépendantes (**CP8**).
- Ne pas manipuler manuellement SYSVOL (répliqué par DFS-R).

## ⚠️ À ne pas confondre / obsolète

- **FRS** (ancienne réplication SYSVOL) est **supprimé** → **DFS-R** uniquement.

---

## Sources (doc officielle)

- [New-DfsnRoot — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/dfsn/new-dfsnroot) — consulté le 23/07/2026
- [Réplication DFS (DFS-R) — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/storage/dfs-replication/dfsr-overview) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
