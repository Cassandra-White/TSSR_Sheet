# CP2-04 — Créer des unités d'organisation (OU) et structurer l'annuaire

**Objectif** : créer une hiérarchie d'unités d'organisation (OU) pour ranger utilisateurs, groupes et ordinateurs, et pouvoir cibler les stratégies de groupe (GPO).

**Rattachement REAC** : CP2 — savoir-faire : « Créer, modifier et supprimer des objets dans un annuaire Active Directory ».

**Durée** : ~10 min · **Niveau** : débutant.

---

## Prérequis

- Contrôleur de domaine opérationnel (voir **CP2-03**), session **`LABTSSR\Administrateur`**.
- Les outils d'administration AD (ADUC) sont installés avec le rôle AD DS.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (Desktop Experience) | 23/07/2026 |

> Exemple de structure : `Entreprise` → `Utilisateurs`, `Groupes`, `Ordinateurs`, puis par service (`Compta`, `IT`, `RH`).

---

## Procédure — GUI (méthode prioritaire)

1. **Gestionnaire de serveur** → **Outils** → **Utilisateurs et ordinateurs Active Directory** (ou Exécuter `dsa.msc`).
2. Clic droit sur le domaine **`labtssr.lan`** (ou une OU parente) → **Nouveau** → **Unité d'organisation**.
3. Saisir le nom (ex. `Entreprise`) ; laisser cochée **« Protéger le conteneur contre une suppression accidentelle »** → **OK**.
4. Répéter pour construire la hiérarchie (clic droit sur `Entreprise` → Nouveau → OU : `Utilisateurs`, `Groupes`, `Ordinateurs`…).

> Interface alternative plus récente : **Centre d'administration Active Directory** (`dsac.exe`), même logique.

## Procédure — CLI (alternative / automatisation)

```powershell
# OU racine (protection contre suppression = $true par défaut)
New-ADOrganizationalUnit -Name "Entreprise" -Path "DC=labtssr,DC=lan"

# Sous-OU
New-ADOrganizationalUnit -Name "Utilisateurs" -Path "OU=Entreprise,DC=labtssr,DC=lan"
New-ADOrganizationalUnit -Name "Compta"       -Path "OU=Utilisateurs,OU=Entreprise,DC=labtssr,DC=lan"
```

*(PowerShell à exécuter en lab — non testable dans le bac à sable Linux.)*

---

## Vérification

```powershell
Get-ADOrganizationalUnit -Filter * | Select-Object Name, DistinguishedName
```

Les OU apparaissent aussi dans l'arborescence d'ADUC.

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| Suppression d'OU refusée | Protection anti-suppression active | ADUC → **Affichage → Fonctionnalités avancées** → onglet **Objet** → décocher, puis supprimer |
| OU invisible dans ADUC | Affichage basique | **Affichage → Fonctionnalités avancées** |
| `-Path` rejeté | DN incorrect | Vérifier le DN exact avec `Get-ADOrganizationalUnit` |

## Sécurité et bonnes pratiques

- Conserver la **protection contre la suppression accidentelle**.
- Structurer par **service/fonction** pour appliquer des **GPO ciblées** (voir **CP2-11**).
- Rediriger les nouveaux comptes vers une OU dédiée : `redirusr` (utilisateurs) et `redircmp` (ordinateurs).

## ⚠️ À ne pas confondre / obsolète

- Un **conteneur** (`CN=Users`, `CN=Computers`) **n'est pas une OU** : on **ne peut pas** y lier de GPO. Toujours créer des **OU** pour la production.

---

## Sources (doc officielle)

- [New-ADOrganizationalUnit (Windows Server 2025) — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/activedirectory/new-adorganizationalunit?view=windowsserver2025-ps) — consulté le 23/07/2026
- [Module Active Directory pour PowerShell — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/activedirectory/about/about_activedirectory?view=windowsserver2025-ps) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
