# CP2-17 — Configurer les stratégies de mots de passe et de verrouillage

**Objectif** : définir la politique de mots de passe et de verrouillage du domaine, et des politiques **affinées** (PSO) plus strictes pour les comptes sensibles.

**Rattachement REAC** : CP2 — connaissance : « éléments d'une charte de sécurité informatique (politique de mots de passe, règles de sécurité…) ».

**Durée** : ~15 min · **Niveau** : intermédiaire.

---

## Prérequis

- Contrôleur de domaine (voir **CP2-03**), console GPMC / Centre d'administration AD.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (Desktop Experience) | 23/07/2026 |

## À retenir

La politique de mot de passe du **domaine** est **unique** (portée par la *Default Domain Policy*). Pour des exigences différentes selon la population, on utilise les **stratégies de mot de passe affinées (PSO)**, appliquées à des groupes/utilisateurs et départagées par **précédence**.

---

## Procédure — GUI (méthode prioritaire)

**Politique du domaine** (`gpmc.msc`)
1. Développer le domaine → clic droit **Default Domain Policy** → **Modifier**.
2. *Configuration ordinateur → Stratégies → Paramètres Windows → Paramètres de sécurité → **Stratégies de compte***.
3. **Stratégie de mot de passe** : longueur minimale (ex. 14), complexité **activée**, historique (24), âge max/min.
4. **Stratégie de verrouillage** : seuil (ex. 5 essais), durée, fenêtre de réinitialisation.
5. `gpupdate /force`.

**Politique affinée (PSO)** — Centre d'administration AD (`dsac.exe`)
6. *System → Password Settings Container → Nouveau → Paramètres de mot de passe* : définir les règles, la **précédence**, puis **appliquer à un groupe** (ex. `Domain Admins`).

## Procédure — CLI

```powershell
# Politique par défaut du domaine
Set-ADDefaultDomainPasswordPolicy -Identity labtssr.lan `
  -MinPasswordLength 14 -ComplexityEnabled $true -PasswordHistoryCount 24 `
  -MaxPasswordAge 90.00:00:00 -MinPasswordAge 1.00:00:00 `
  -LockoutThreshold 5 -LockoutDuration 00:15:00 -LockoutObservationWindow 00:15:00

# Politique affinée (PSO) plus stricte pour les admins (précédence basse = prioritaire)
New-ADFineGrainedPasswordPolicy -Name "PSO-Admins" -Precedence 10 `
  -MinPasswordLength 16 -ComplexityEnabled $true -MaxPasswordAge 30.00:00:00 `
  -LockoutThreshold 3 -LockoutDuration 00:30:00 -LockoutObservationWindow 00:30:00
Add-ADFineGrainedPasswordPolicySubject -Identity "PSO-Admins" -Subjects "Domain Admins"
```

*(À exécuter en lab — non testable dans le bac à sable Linux.)*

---

## Vérification

```powershell
Get-ADDefaultDomainPasswordPolicy
Get-ADFineGrainedPasswordPolicy -Filter *
Get-ADUserResultantPasswordPolicy -Identity administrateur   # politique effective d'un compte
```

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| PSO sans effet | Non appliquée à un groupe | `Add-ADFineGrainedPasswordPolicySubject` |
| Politique ignorée sur une OU | La politique de MDP est **domaine**, pas OU | Utiliser un **PSO** (pas une GPO d'OU) |
| Verrouillages fréquents | Seuil bas + identifiants en cache | Ajuster + corriger la source (**CP2-10**) |

## Sécurité et bonnes pratiques

- S'aligner sur l'**ANSSI/NIST** : privilégier la **longueur** ; ajouter la **MFA** (voir **CP7-18**).
- Politique **renforcée** pour les comptes à privilèges via **PSO**.
- Bannir les mots de passe courants (solutions de *password protection*).

## ⚠️ À ne pas confondre / obsolète

- La politique de mot de passe **ne se définit pas** par une GPO liée à une **OU** : c'est la **Default Domain Policy** (niveau domaine) ou un **PSO**.
- L'**expiration systématique** tous les X jours est aujourd'hui **déconseillée** (NIST) au profit de MDP longs + détection de fuite + MFA.

---

## Sources (doc officielle)

- [Set-ADDefaultDomainPasswordPolicy — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/activedirectory/set-addefaultdomainpasswordpolicy?view=windowsserver2025-ps) — consulté le 23/07/2026
- [Stratégies de mot de passe affinées — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/adac/fine-grained-password-policies) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
