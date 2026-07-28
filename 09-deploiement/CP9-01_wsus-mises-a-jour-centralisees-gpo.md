# CP9-01 — Installer et configurer WSUS (mises à jour centralisées) + GPO

**Objectif** : centraliser les mises à jour Windows avec **WSUS**, approuver les correctifs, et diriger les postes vers le serveur via **GPO**.

**Rattachement REAC** : CP9 « Exploiter et maintenir les services de déploiement des postes » — savoir-faire : gérer les mises à jour du parc.

**Durée** : ~35 min · **Niveau** : intermédiaire.

---

## Prérequis

- **Windows Server 2025** membre du domaine (**CP2-03**), disque avec espace pour le contenu des mises à jour.
- Des clients **Windows 11 24H2** dans le domaine.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Serveur | **Windows Server 2025** — rôle WSUS | 24/07/2026 |
| Clients | **Windows 11 24H2** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> ⚠️ **Statut à connaître (examen + réalité)** : **WSUS est déprécié depuis le 20/09/2024** (la **synchronisation des pilotes** est arrêtée depuis avril 2025). Il **reste présent et supporté** dans Windows Server 2025 (jusqu'à la fin de vie de l'OS), mais Microsoft oriente vers **Windows Autopatch + Intune** (postes) et **Azure Update Manager** (serveurs). On l'apprend car il est **encore très déployé**.

---

## Procédure — GUI

### Installer WSUS

1. **Server Manager ▸ Add Roles and Features ▸ Windows Server Update Services** → base **WID** (ou SQL) + dossier de contenu (ex. `E:\WSUS`).
2. Lancer le **post-install** (roue crantée / *Launch Post-Installation tasks*).

### Configurer la synchronisation

3. Console **Update Services ▸ Options ▸ Configuration Wizard** : source = **Microsoft Update**, choisir **langues**, **produits** (Windows 11, Server 2025…), **classifications** (*Security*, *Critical*, *Definition Updates*), et une **synchronisation planifiée**.

### Approuver et cibler

4. **Computers** : créer des **groupes** (ex. *Pilotes*, *Production*).
5. **Updates** : **approuver** les correctifs (manuellement ou via **règles d'auto-approbation**) pour chaque groupe ; **refuser** (Decline) les mises à jour remplacées.

## Procédure — CLI (PowerShell)

```powershell
Install-WindowsFeature -Name UpdateServices -IncludeManagementTools

# Post-install (dossier de contenu)
& 'C:\Program Files\Update Services\Tools\wsusutil.exe' postinstall CONTENT_DIR=E:\WSUS

# Module WSUS : cibler et approuver
$w = Get-WsusServer
Get-WsusUpdate -Approval Unapproved -Classification Security |
  Approve-WsusUpdate -Action Install -TargetGroupName "Pilotes"
```

## Procédure — GPO (diriger les clients)

**Computer Configuration ▸ Policies ▸ Administrative Templates ▸ Windows Components ▸ Windows Update** :

- **Specify intranet Microsoft update service location** = `http://SRV-WSUS:8530` (détection **et** rapport).
- **Configure Automatic Updates** (ex. téléchargement + installation planifiée).
- **Enable client-side targeting** = nom du groupe WSUS (ex. `Production`).

---

## Vérification (comment savoir que ça marche)

```powershell
gpupdate /force                          # appliquer la GPO sur le client
# Forcer la détection WSUS (remplace l'ancien wuauclt /detectnow)
(New-Object -ComObject Microsoft.Update.AutoUpdate).DetectNow()
```

- Le poste **apparaît** dans la console WSUS (bon groupe) et remonte son état de conformité.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Client absent de la console | **SusClientID dupliqué** (image clonée) | Réinitialiser l'ID (`wuauclt /resetauthorization` / clés `SoftwareDistribution`) |
| Pas de contact WSUS | GPO/URL/port | Vérifier `http://serveur:8530`, la GPO appliquée |
| Rien à approuver | Produits/classifications non cochés | Revoir **Options ▸ Products and Classifications** |
| Erreur pilotes | **Sync pilotes dépréciée** | Ne plus compter sur WSUS pour les pilotes |

## Sécurité et bonnes pratiques

- **Approche par anneaux** : approuver d'abord pour un groupe **Pilotes**, puis **Production** après validation.
- **HTTPS** (port **8531**) pour la connexion cliente si possible.
- **Tester** les correctifs avant approbation large ; **refuser** les mises à jour superflues.
- Anticiper la **bascule** vers Intune/Azure Update Manager (WSUS déprécié).

## ⚠️ À ne pas confondre / obsolète

- **WSUS déprécié** (2024) mais **encore fonctionnel** ≠ supprimé : alternatives cloud recommandées.
- Port par défaut **8530/8531** (et non 80/443) depuis WS 2012.
- **Synchronisation des pilotes** WSUS **arrêtée** → passer par d'autres canaux.

---

## Sources (doc officielle)

- [Microsoft — WSUS deprecation (Windows IT Pro Blog)](https://techcommunity.microsoft.com/blog/windows-itpro-blog/windows-server-update-services-wsus-deprecation/4250436) — consulté le 24/07/2026
- [Microsoft Learn — Deploy Windows Server Update Services](https://learn.microsoft.com/en-us/windows-server/administration/windows-server-update-services/deploy/deploy-windows-server-update-services) — consulté le 24/07/2026
- [Microsoft Learn — Configure Automatic Updates par GPO](https://learn.microsoft.com/en-us/windows-server/administration/windows-server-update-services/deploy/4-configure-group-policy-settings-for-automatic-updates) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI + GPO · [x] versions datées (WS 2025 / Win 11 24H2) · [x] **statut déprécié signalé** + alternatives · [x] procédure **à tester en lab** · [x] GUI conforme doc Microsoft · [x] vérification présente · [x] sécurité (anneaux, HTTPS) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
