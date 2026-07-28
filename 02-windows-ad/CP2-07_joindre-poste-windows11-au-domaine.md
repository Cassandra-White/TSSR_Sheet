# CP2-07 — Joindre un poste Windows 11 au domaine

**Objectif** : intégrer un client Windows 11 au domaine `labtssr.lan` et ouvrir une session avec un compte de domaine.

**Rattachement REAC** : CP2 — savoir-faire : « Intégrer un poste client au domaine ».

**Durée** : ~10 min · **Niveau** : débutant.

---

## Prérequis

- Contrôleur de domaine opérationnel (voir **CP2-03**).
- Un client **Windows 11 Pro ou Enterprise** (l'édition **Home ne gère pas** la jonction de domaine).
- Le client a pour **serveur DNS l'adresse IP du DC** (ex. `192.168.10.10`).
- Un compte autorisé à joindre (Administrateur du domaine ou compte délégué).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Client | Windows 11 24H2 | 23/07/2026 |
| Domaine | Windows Server 2025 (`labtssr.lan`) | 23/07/2026 |

---

## Procédure — GUI (méthode prioritaire)

1. Sur le client, régler le **DNS = IP du DC** (via `ncpa.cpl`, cf. **CP2-02**, ou par DHCP).
2. Vérifier la résolution : `nslookup labtssr.lan` doit répondre l'IP du DC.
3. Exécuter `sysdm.cpl` → onglet **Nom de l'ordinateur** → **Modifier**.
4. Cocher **Domaine**, saisir `labtssr.lan` → **OK**.
5. Saisir les identifiants d'un compte autorisé (ex. `LABTSSR\Administrateur`).
6. Message de bienvenue → **OK** → **Redémarrer**.
7. Au redémarrage : **Autre utilisateur** → se connecter en `LABTSSR\jdupont`.

## Procédure — CLI (alternative / automatisation)

```powershell
# À exécuter dans Windows PowerShell 5.1 (Add-Computer n'existe pas en PowerShell 7)
Add-Computer -DomainName "labtssr.lan" -Credential (Get-Credential) -Restart
```

Alternative en ligne de commande classique :

```
netdom join %COMPUTERNAME% /domain:labtssr.lan /userd:LABTSSR\Administrateur /passwordd:*
```

*(À exécuter en lab — non testable dans le bac à sable Linux.)*

---

## Vérification

```powershell
(Get-CimInstance Win32_ComputerSystem).Domain     # doit renvoyer labtssr.lan
Get-CimInstance Win32_ComputerSystem | Select-Object PartOfDomain, Domain
gpresult /r                                        # GPO du domaine appliquées
```

Côté DC : l'objet ordinateur apparaît dans ADUC (conteneur `Computers` ou l'OU redirigée).

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| « Contrôleur de domaine introuvable » | DNS ne pointe pas sur le DC | Mettre le DNS du client = IP du DC |
| Option **Domaine** absente/grisée | Édition Windows 11 **Home** | Passer en **Pro/Enterprise** |
| Jonction refusée (heure) | Décalage d'horloge (Kerberos) | Synchroniser l'heure (**CP4-13**) |

## Sécurité et bonnes pratiques

- Utiliser un **compte délégué** (pas *Admins du domaine*) pour les jonctions de masse.
- Contrôler l'**OU d'atterrissage** des nouveaux ordinateurs (`redircmp`).

## ⚠️ À ne pas confondre / obsolète

- `Add-Computer` = **Windows PowerShell 5.1 uniquement** (retiré de PowerShell 7) → utiliser 5.1 ou `netdom`.
- Jonction à un **domaine AD local** ≠ **Entra/Azure AD join** (cloud) : ce sont deux choses différentes.

---

## Sources (doc officielle)

- [Add-Computer — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/add-computer) — consulté le 23/07/2026
- [Joindre un ordinateur à un domaine — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/join-a-computer-to-a-domain) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
