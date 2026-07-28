# CP9-05 — Déployer des logiciels via GPO

**Objectif** : installer automatiquement un logiciel **MSI** sur les postes/utilisateurs du domaine via une **stratégie de groupe** (assigner / publier).

**Rattachement REAC** : CP9 « Exploiter et maintenir les services de déploiement des postes » — savoir-faire : déployer des applications sur le parc.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Domaine **AD** (**CP2-03**), un **partage réseau** de dépôt logiciel, un paquet **`.msi`**.
- Postes **Windows 11** dans le domaine.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Serveur | **Windows Server 2025** — GPMC | 24/07/2026 |
| Clients | **Windows 11 24H2** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **Deux modes** : **Assigner** (installation forcée — *ordinateur* : au démarrage, pour tous ; *utilisateur* : à l'ouverture de session) ou **Publier** (proposé à l'utilisateur via *Ajout/Suppression de programmes*, ou installé à la demande). La GPO ne gère **nativement** que le **MSI**.

---

## Procédure — GUI

1. **Déposer le MSI** sur un **partage** en **lecture** pour les objets concernés (chemin **UNC** : `\\SRV\Deploy$\7zip.msi`). Pour un déploiement **ordinateur**, autoriser la lecture aux **comptes d'ordinateur** (`Domain Computers`).
2. **Group Policy Management** → créer/éditer une GPO liée à l'OU cible.
3. **Computer Configuration** (ou *User*) **▸ Policies ▸ Software Settings ▸ Software Installation** → clic droit **New ▸ Package**.
4. Choisir le MSI **via le chemin UNC** (pas un chemin local !) → **Assigned** (ou *Published* côté utilisateur).
5. Sur le client : `gpupdate /force` puis **redémarrer** (déploiement *ordinateur* s'applique au boot).

## Procédure — mise à jour / retrait

- **Mettre à jour** : ajouter le nouveau package et le définir comme **upgrade** de l'ancien.
- **Retirer** : clic droit sur le package ▸ **All Tasks ▸ Remove** → *Immediately uninstall*.

---

## Vérification (comment savoir que ça marche)

```powershell
gpresult /h C:\gpresult.html      # la GPO d'installation logicielle est appliquée
Get-Package -Name "7-Zip*"        # le logiciel est présent
```

- Journal **Event Viewer ▸ Application** (source *Application Management / MsiInstaller*).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Rien ne s'installe | Partage non lisible par le **compte d'ordinateur** | Donner *Read* à `Domain Computers` |
| « chemin introuvable » | Chemin **local** au lieu d'**UNC** | Toujours pointer en **`\\serveur\partage`** |
| Échoue au boot | MSI non silencieux / dépendances | Vérifier le MSI ; installer les prérequis |
| Ouverture de session lente | Trop d'applis *assignées utilisateur* | Préférer *assigné ordinateur* / échelonner |

## Sécurité et bonnes pratiques

- **Partage en lecture seule**, paquets **MSI signés** et validés.
- **Tester** sur une OU pilote avant le parc entier.
- Documenter les versions déployées ; prévoir la **désinstallation** propre.

## ⚠️ À ne pas confondre / obsolète

- **Assigner** (forcé) ≠ **Publier** (à la demande, utilisateur seulement).
- GPO Software Installation = **MSI uniquement** (EXE/MSIX → repackager) ; **pas de reporting** ni de gestion fine des versions.
- **Méthode héritée** mais fonctionnelle : pour du moderne, **Intune + winget** (**CP9-10**) offrent plus de souplesse (EXE/MSIX, suivi).

---

## Sources (doc officielle)

- [Microsoft — Use Group Policy to install software (MSI)](https://learn.microsoft.com/en-us/troubleshoot/windows-server/group-policy/use-group-policy-to-install-software) — consulté le 24/07/2026
- [Microsoft Learn — Software installation extension (Group Policy)](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh147307(v=ws.11)) — consulté le 24/07/2026
- [Microsoft Learn — Add and manage apps with Intune (winget)](https://learn.microsoft.com/en-us/mem/intune/apps/apps-add) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI (GPMC) · [x] versions datées (WS 2025 / Win 11 24H2) · [x] rien d'obsolète (limites GPSI + alternative Intune/winget) · [x] procédure **à tester en lab** · [x] conforme doc Microsoft · [x] vérification présente (`gpresult`) · [x] sécurité (partage RO, MSI signé) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
