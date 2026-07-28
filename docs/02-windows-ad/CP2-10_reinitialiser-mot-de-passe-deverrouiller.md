# CP2-10 — Réinitialiser un mot de passe / déverrouiller un compte AD

**Objectif** : réinitialiser le mot de passe d'un utilisateur et/ou déverrouiller son compte (opération de support très courante).

**Rattachement REAC** : CP2 « Exploiter… un domaine Active Directory » (modifier les comptes) ; appui au support **CP1**.

**Durée** : ~5 min · **Niveau** : débutant.

---

## Prérequis

- Contrôleur de domaine (voir **CP2-03**), compte disposant du droit délégué *Réinitialiser le mot de passe*.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (Desktop Experience) | 23/07/2026 |

---

## Procédure — GUI (méthode prioritaire)

1. `dsa.msc` → localiser l'utilisateur (clic droit sur le domaine → **Rechercher**).
2. Clic droit sur le compte → **Réinitialiser le mot de passe**.
3. Saisir le nouveau mot de passe ; cocher **« L'utilisateur doit changer le mot de passe à la prochaine ouverture de session »** et **« Déverrouiller le compte de l'utilisateur »** → **OK**.

> Déverrouillage seul : clic droit → **Propriétés** → onglet **Compte** → cocher **« Déverrouiller le compte »** (case visible uniquement si le compte est verrouillé).

## Procédure — CLI (alternative / automatisation)

```powershell
# Réinitialiser le mot de passe
$pw = Read-Host "Nouveau mot de passe" -AsSecureString
Set-ADAccountPassword -Identity jdupont -Reset -NewPassword $pw

# Forcer le changement à la prochaine connexion
Set-ADUser -Identity jdupont -ChangePasswordAtLogon $true

# Déverrouiller
Unlock-ADAccount -Identity jdupont
```

*(À exécuter en lab — non testable dans le bac à sable Linux.)*

---

## Vérification

```powershell
Get-ADUser jdupont -Properties LockedOut,PasswordLastSet | Select-Object Name,LockedOut,PasswordLastSet
Search-ADAccount -LockedOut   # liste tous les comptes verrouillés
```

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| Le compte se **reverrouille** aussitôt | Ancien mot de passe en cache (lecteur mappé, tâche planifiée, mobile) | Identifier la source via l'événement **4740**, corriger |
| « Le mot de passe ne remplit pas les exigences » | Complexité / historique | Mot de passe fort et non réutilisé |
| Impossible de réinitialiser | Droits insuffisants | Déléguer *Réinitialiser le mot de passe* sur l'OU |

## Sécurité et bonnes pratiques

- Toujours **vérifier l'identité** du demandeur avant de réinitialiser (support).
- Mot de passe **temporaire** + changement obligatoire à la première connexion.
- **Auditer** les verrouillages (événement 4740) : ils peuvent signaler une attaque.

## ⚠️ À ne pas confondre / obsolète

- Éviter `net user /domain` pour la gestion des comptes AD → utiliser **PowerShell AD**.
- *Réinitialiser* (l'admin fixe un nouveau MDP) ≠ *Modifier* (l'utilisateur change le sien en connaissant l'ancien).

---

## Sources (doc officielle)

- [Set-ADAccountPassword — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/activedirectory/set-adaccountpassword?view=windowsserver2025-ps) — consulté le 23/07/2026
- [Unlock-ADAccount — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/activedirectory/unlock-adaccount?view=windowsserver2025-ps) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
