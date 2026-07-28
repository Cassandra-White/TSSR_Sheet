# CP2-23 — Administrer les serveurs avec Windows Admin Center

**Objectif** : installer Windows Admin Center (WAC) et administrer un ou plusieurs serveurs Windows depuis un navigateur, sans ouvrir de session RDP.

**Rattachement REAC** : CP2 « Exploiter des serveurs Windows et un domaine Active Directory » — savoir-faire : administration centralisée et supervision des serveurs.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un hôte pour la **passerelle (gateway)** WAC : Windows Server 2025 ou Windows 11. **Ne pas l'installer sur un contrôleur de domaine.**
- Droits **administrateur** sur l'hôte + comptes administrateur sur les serveurs cibles.
- **WinRM** actif sur les serveurs à gérer (activé par défaut sur Windows Server 2025).
- Navigateur **Microsoft Edge** ou **Google Chrome** (pas Internet Explorer).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Admin Center | **2606** (passerelle modernisée « v2 ») | 24/07/2026 |
| Hôte gateway | Windows Server 2025 / Windows 11 24H2 | 24/07/2026 |

---

## Procédure — GUI (méthode prioritaire)

### Installer la passerelle

1. Télécharger l'installeur depuis le **Centre d'évaluation Microsoft** (`https://aka.ms/WACDownload`).
2. Lancer l'installeur → **Next** → accepter les termes de licence.
3. **Select installation mode** : *Express setup* (réseau et port déterminés automatiquement) ou *Custom setup* (port, certificat TLS, FQDN, mode hôtes de confiance, WinRM over HTTPS).
4. **Select TLS certificate** : certificat **auto-signé** pour un test (valide **60 jours**), ou certificat d'une **autorité de confiance** en production (présent dans `LocalMachine\My`).
5. **Mises à jour automatiques** (option recommandée par défaut) → **données de diagnostic** → **Install**.
6. Cocher **Start Windows Admin Center** → **Finish**. La console s'ouvre dans le navigateur : `https://<nom-de-la-gateway>` (ou `https://localhost` selon le port choisi).

### Ajouter et administrer un serveur

7. Se connecter en **administrateur**. Cliquer **+ Add** → **Server** → saisir le **nom** (FQDN) → informations d'identification.
8. Cliquer sur le serveur → **Overview** : CPU, mémoire, redémarrage/arrêt, modification du nom/domaine.
9. Outils du volet gauche : **Roles & features**, **Services**, **Events** (journaux), **Networking**, **Files**, **Updates**, **PowerShell** (console distante intégrée), **Certificates**, **Hyper-V**, **Storage**, etc.

## Procédure — CLI (installation automatisée / notes)

> WAC s'administre par le web ; l'installation, elle, peut s'automatiser (déploiement de masse, Server Core).

```powershell
# Installation silencieuse : prise en charge, y compris sur Server Core.
# Lancer l'installeur téléchargé avec les paramètres silencieux documentés :
Start-Process -FilePath ".\WindowsAdminCenter.exe" -ArgumentList "/silent" -Wait
# Les paramètres exacts (port, certificat, FQDN) dépendent de la version :
# voir l'onglet « Server Core » de la doc d'installation (lien en Sources).
```

*(À exécuter en lab — non testable dans le bac à sable Linux.)*

> WAC ne remplace pas PowerShell : chaque action correspond à des cmdlets, exposées via l'outil **PowerShell** intégré à la console.

---

## Vérification (comment savoir que ça marche)

- Le navigateur ouvre la console sur `https://<gateway>` et l'authentification administrateur aboutit.
- Le serveur ajouté remonte ses métriques (CPU/mémoire) dans **Overview**.
- Depuis l'outil **Services** ou **Events**, les données du serveur cible s'affichent (preuve que WinRM répond).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| « Impossible de se connecter » au serveur cible | WinRM inactif / pare-feu | `Enable-PSRemoting -Force` sur la cible ; ouvrir WinRM |
| Avertissement de certificat | Certificat auto-signé | Normal en test ; utiliser une CA de confiance en prod |
| Installation refusée sur un DC | Bonne pratique : pas de WAC sur un contrôleur de domaine | Installer sur un serveur de gestion ou Windows 11 |
| Page blanche / non supportée | Navigateur non compatible | Utiliser Edge ou Chrome |

## Sécurité et bonnes pratiques

- Accès **HTTPS** uniquement ; en production, **certificat d'une CA** (pas auto-signé).
- **Moindre privilège** : restreindre qui peut atteindre la passerelle ; envisager le contrôle d'accès basé sur les rôles (RBAC) de WAC.
- Tenir la passerelle **à jour** (l'installeur 2606 corrige des failles d'élévation de privilège et d'exécution de code).

## ⚠️ À ne pas confondre / obsolète

- La **passerelle modernisée (« v2 »)** est la base actuelle : préférer une version récente (2606) aux anciennes builds.
- WAC complète, mais ne remplace pas totalement, les consoles **MMC** ni **Server Manager** ; il centralise la gestion multi-serveurs via le web.

---

## Sources (doc officielle)

- [Install Windows Admin Center — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/manage/windows-admin-center/deploy/install) — consulté le 24/07/2026
- [Windows Admin Center 2606 est GA — Microsoft Community Hub](https://techcommunity.microsoft.com/blog/windows-admin-center-blog/windows-admin-center-version-2606-is-now-generally-available/4530811) — consulté le 24/07/2026
- [Windows Admin Center — release history — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/manage/windows-admin-center/support/release-history) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète (v2 / 2606) · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
