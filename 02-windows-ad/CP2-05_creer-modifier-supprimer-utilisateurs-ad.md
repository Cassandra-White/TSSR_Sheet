# CP2-05 — Créer, modifier et supprimer des utilisateurs Active Directory

**Objectif** : gérer le cycle de vie d'un compte utilisateur AD — création, modification, désactivation, suppression.

**Rattachement REAC** : CP2 — savoir-faire : « Créer, modifier et supprimer des objets dans un annuaire Active Directory » ; « modifier les autorisations et les droits selon les prescriptions des administrateurs ».

**Durée** : ~15 min · **Niveau** : débutant.

---

## Prérequis

- Contrôleur de domaine + OU créées (voir **CP2-03**, **CP2-04**).
- Session **`LABTSSR\Administrateur`** (ou compte délégué).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (Desktop Experience) | 23/07/2026 |

---

## Procédure — GUI (méthode prioritaire)

**Créer**
1. `dsa.msc` → ouvrir l'OU cible (ex. `Entreprise → Utilisateurs → Compta`).
2. Clic droit → **Nouveau** → **Utilisateur**.
3. Prénom, Nom, **nom d'ouverture de session** (UPN, ex. `jdupont@labtssr.lan`) → **Suivant**.
4. Mot de passe + options (**« L'utilisateur doit changer le mot de passe à la prochaine ouverture »**) → **Suivant** → **Terminer**.

**Modifier / désactiver / supprimer**
- **Modifier** : double-clic sur le compte → onglets *Général, Compte, Membre de, Profil*.
- **Désactiver** : clic droit → **Désactiver le compte**.
- **Réinitialiser le mot de passe** : clic droit → **Réinitialiser le mot de passe** (détaillé en **CP2-10**).
- **Supprimer** : clic droit → **Supprimer**.

## Procédure — CLI (alternative / automatisation)

```powershell
# Créer un compte activé
$pw = Read-Host "Mot de passe initial" -AsSecureString
New-ADUser -Name "Jean Dupont" -GivenName "Jean" -Surname "Dupont" `
  -SamAccountName "jdupont" -UserPrincipalName "jdupont@labtssr.lan" `
  -Path "OU=Compta,OU=Utilisateurs,OU=Entreprise,DC=labtssr,DC=lan" `
  -AccountPassword $pw -ChangePasswordAtLogon $true -Enabled $true

# Modifier des attributs
Set-ADUser -Identity jdupont -Title "Comptable" -Department "Compta"

# Désactiver puis réactiver
Disable-ADAccount -Identity jdupont
Enable-ADAccount  -Identity jdupont

# Supprimer
Remove-ADUser -Identity jdupont -Confirm:$false
```

> Création **en masse** depuis un fichier CSV : voir **CP6-02**.

*(PowerShell à exécuter en lab — non testable dans le bac à sable Linux.)*

---

## Vérification

```powershell
Get-ADUser jdupont -Properties Department,Enabled | Select-Object Name,Enabled,Department
```

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| « Le mot de passe ne remplit pas les exigences » | Politique de complexité | Fournir un mot de passe fort (voir **CP2-17**) |
| Compte créé mais **désactivé** | `-AccountPassword`/`-Enabled` manquants | Fournir le mot de passe et `-Enabled $true` |
| `SamAccountName` déjà utilisé | Doublon | Choisir un identifiant unique |

## Sécurité et bonnes pratiques

- **Moindre privilège** : ne jamais placer un utilisateur standard dans *Admins du domaine*.
- Forcer le **changement de mot de passe** à la première connexion.
- Pour un départ : **désactiver** d'abord le compte, le supprimer après un délai de conservation.

## ⚠️ À ne pas confondre / obsolète

- `net user /add` crée un compte **local**, **pas** un compte de domaine.
- Les outils `dsadd/dsmod user` fonctionnent encore mais **PowerShell AD** est la voie recommandée.

---

## Sources (doc officielle)

- [New-ADUser (Windows Server 2025) — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/activedirectory/new-aduser?view=windowsserver2025-ps) — consulté le 23/07/2026
- [Set-ADUser — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/activedirectory/set-aduser?view=windowsserver2025-ps) — consulté le 23/07/2026
- [Remove-ADUser — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/activedirectory/remove-aduser?view=windowsserver2025-ps) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
