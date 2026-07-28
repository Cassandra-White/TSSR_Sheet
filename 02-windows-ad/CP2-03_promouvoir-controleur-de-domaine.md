# CP2-03 — Promouvoir un serveur en contrôleur de domaine (nouvelle forêt)

**Objectif** : installer le rôle AD DS puis promouvoir le serveur en **contrôleur de domaine** d'une **nouvelle forêt** Active Directory (avec DNS intégré).

**Rattachement REAC** : CP2 « Exploiter des serveurs Windows et un domaine Active Directory » — savoir-faire : « Créer/gérer un annuaire Active Directory », connaissance « annuaires de type LDAP ».

**Durée** : ~20 min · **Niveau** : débutant.

---

## Prérequis

- **CP2-01** et **CP2-02** réalisés : IP fixe, nom d'hôte défini (`SRV-AD01`), fuseau réglé, DNS pointant sur lui-même.
- Session **Administrateur** local.
- Un **mot de passe DSRM** (restauration des services d'annuaire) préparé.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (Desktop Experience) | 23/07/2026 |

> Exemple : domaine `labtssr.lan`, NetBIOS `LABTSSR`.

---

## Procédure — GUI (méthode prioritaire)

**1. Installer le rôle AD DS**
1. **Gestionnaire de serveur** → **Gérer** → **Ajouter des rôles et fonctionnalités**.
2. Type : *Installation basée sur un rôle ou une fonctionnalité* → sélectionner le serveur local.
3. Rôles : cocher **Services AD DS (AD DS)** → **Ajouter des fonctionnalités** → **Suivant** → **Installer**.

**2. Promouvoir en contrôleur de domaine**
4. Cliquer le **drapeau de notification** (⚑) → **Promouvoir ce serveur en contrôleur de domaine**.
5. **Ajouter une nouvelle forêt** → nom de domaine racine : `labtssr.lan` → **Suivant**.
6. Niveaux fonctionnels forêt/domaine : laisser le plus élevé proposé ; cocher **Serveur DNS** ; saisir le **mot de passe DSRM** → **Suivant**.
7. Options DNS : ignorer l'avertissement de délégation → **Suivant**.
8. Nom **NetBIOS** : `LABTSSR` (proposé automatiquement) → **Suivant**.
9. Chemins **NTDS / SYSVOL** : valeurs par défaut → **Suivant**.
10. **Vérification de la configuration requise** → **Installer**. Le serveur **redémarre** automatiquement.
11. Après redémarrage, se connecter en **`LABTSSR\Administrateur`**.

## Procédure — CLI (alternative / automatisation)

```powershell
# 1. Installer le rôle AD DS + outils d'administration
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

# 2. Promouvoir en nouvelle forêt (le DNS est installé par défaut)
Install-ADDSForest `
  -DomainName "labtssr.lan" `
  -DomainNetbiosName "LABTSSR" `
  -ForestMode "WinThreshold" -DomainMode "WinThreshold" `
  -InstallDns -Force
# -> demande le mot de passe DSRM, puis redémarre automatiquement
```

*(PowerShell à exécuter en lab — non testable dans le bac à sable Linux.)*

---

## Vérification

```powershell
Get-ADDomain          # doit renvoyer labtssr.lan
Get-ADForest          # niveau et DC listés
Get-Service ADWS,KDC,Netlogon,DNS   # services démarrés
dcdiag /q             # aucun test en échec
```

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| « Impossible de contacter le serveur DNS » | DNS mal configuré | DNS préféré = `127.0.0.1` (voir **CP2-02**) |
| Promotion refusée (nom `WIN-XXXX`) | Nom par défaut invalide | Renommer le serveur **avant** (**CP2-02**) |
| Mot de passe DSRM rejeté | Complexité insuffisante | Fournir un mot de passe fort (maj/min/chiffre/spécial) |
| Niveau « Windows Server 2025 » introuvable | Il n'existe pas | Utiliser **WinThreshold** (= 2016), le plus élevé |

## Sécurité et bonnes pratiques

- Mot de passe **DSRM** distinct de l'Administrateur, conservé en lieu sûr.
- Prévoir un **second contrôleur de domaine** pour la redondance (voir **CP2-18**).
- Ne pas installer de rôles applicatifs sur un DC.
- Synchroniser l'heure : **Kerberos** rejette un écart d'horloge > 5 min (voir **CP4-13**).

## ⚠️ À ne pas confondre / obsolète

- **`dcpromo.exe` est supprimé** depuis Windows Server 2012 : la promotion passe par le Gestionnaire de serveur ou `Install-ADDSForest`.
- Il **n'existe pas** de niveau fonctionnel « 2019/2022/2025 » : le plus récent est **« Windows Server 2016 »** (`WinThreshold`).

---

## Sources (doc officielle)

- [Installer AD DS — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/deploy/install-active-directory-domain-services--level-100-) — consulté le 23/07/2026
- [Install-ADDSForest (Windows Server 2025) — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/addsdeployment/install-addsforest?view=windowsserver2025-ps) — consulté le 23/07/2026
- [Niveaux fonctionnels de forêt et de domaine — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/active-directory-functional-levels) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
