# CP9-03 — Déploiement réseau PXE (WDS ou équivalent)

**Objectif** : démarrer un poste **par le réseau (PXE)** pour installer un OS, via **WDS** (ou un équivalent), en connaissant son statut de sécurité actuel.

**Rattachement REAC** : CP9 « Exploiter et maintenir les services de déploiement des postes » — savoir-faire : déployer les postes par le réseau.

**Durée** : ~35 min · **Niveau** : avancé.

---

## Prérequis

- **Windows Server 2025**, un serveur **DHCP** (**CP2-13**), une **image de démarrage** WinPE (**CP9-02**).
- Postes clients avec **PXE** activé (UEFI) sur le bon VLAN.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Serveur | **Windows Server 2025** — rôle WDS | 24/07/2026 |
| Amorçage | **WinPE** (ADK) + DHCP | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> ⚠️ **Statut sécurité important (2026)** : Microsoft **bloque le déploiement WDS « hands-free » (automatisé)** à cause de **CVE-2026-0386** (identifiants exposés) — **Phase 1 (13/01/2026)** désactivable, **Phase 2 (avril 2026)** **désactivé par défaut**. Le **déploiement de `boot.wim` pour Windows 11 via WDS est déprécié**. **En revanche, le boot PXE lui-même reste pris en charge.** Alternatives Microsoft : **Configuration Manager**, **Autopilot**, méthodes **WinPE**. **MDT est retiré (janvier 2026).**

---

## Procédure — GUI (WDS)

1. **Server Manager ▸ Add Roles ▸ Windows Deployment Services** (Transport + Deployment).
2. Console **WDS ▸ Configure Server** : choix AD-intégré, dossier **RemoteInstall**.
3. **Réponse PXE** : *Respond to all client computers* (idéalement avec approbation admin).
4. **Boot Images** : ajouter un **`boot.wim`** (WinPE) ; **Install Images** : ajouter les images à déployer (**CP9-02**).
5. Côté client : démarrer en **PXE** (F12) → WinPE se charge → sélection de l'image.

### Cohabitation DHCP / PXE

- **WDS et DHCP sur le même serveur** : dans WDS ▸ **DHCP**, cocher *ne pas écouter le port 67* et *ajouter les options DHCP PXE*.
- **Serveurs séparés** : configurer les **options DHCP 66** (serveur de boot) / **67** (fichier de boot `boot\x64\wdsmgfw.efi`) ou des **IP Helpers/relais** sur le routeur.

## Procédure — CLI (wdsutil)

```cmd
wdsutil /initialize-server /reminst:"D:\RemoteInstall"
wdsutil /set-server /answerclients:all
wdsutil /add-image /imagefile:"N:\images\boot.wim" /imagetype:boot
wdsutil /add-image /imagefile:"N:\images\Win11-24H2.wim" /imagetype:install
```

> Compte tenu des restrictions WDS, un déploiement **WinPE + DISM** depuis un **partage réseau** (**CP9-02**) ou une solution **ConfigMgr/PXE tierce** est souvent préférable pour du neuf.

---

## Vérification (comment savoir que ça marche)

- Le client obtient une **adresse DHCP** puis **télécharge le NBP** (PXE) et charge **WinPE**.
- L'image d'installation est proposée et s'applique jusqu'à l'OOBE.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| **PXE-E53** (no boot filename) | Options DHCP 66/67 absentes / relais manquant | Configurer les options ou l'**IP Helper** |
| Boot échoue en UEFI | Mauvais **NBP** (BIOS vs UEFI) | Utiliser `wdsmgfw.efi` pour UEFI |
| Déploiement auto refusé | **Hands-free désactivé** (CVE-2026-0386) | Appliquer le durcissement KB ; déploiement **assisté** |
| Client non répondu | *Answer clients* / approbation | Régler la politique de réponse PXE |

## Sécurité et bonnes pratiques

- **CVE-2026-0386** : ne pas activer le déploiement **automatisé** WDS sans le durcissement Microsoft ; privilégier le déploiement **assisté**.
- **PXE n'est pas authentifié** : le confiner à un **VLAN de déploiement** isolé (**CP4-04**), activer l'**approbation admin** des machines.
- **Secure Boot** et intégrité des images de boot ; journaliser les déploiements.
- Prévoir la **bascule** vers ConfigMgr/Autopilot (WDS restreint, MDT retiré).

## ⚠️ À ne pas confondre / obsolète

- **PXE** (démarrage réseau, **OK**) ≠ **WDS boot.wim/hands-free** (déprécié/bloqué pour Win 11).
- **MDT retiré (janv. 2026)** : ne pas bâtir de nouvelle infra dessus.
- Options **DHCP 66/67** ≠ **proxyDHCP/IP Helper** : selon que DHCP et WDS sont sur le même hôte ou non.

---

## Sources (doc officielle)

- [Microsoft Learn — Fonctionnalités supprimées/dépréciées (Windows Server)](https://learn.microsoft.com/en-us/windows-server/get-started/removed-deprecated-features-windows-server) — consulté le 24/07/2026
- [Microsoft Support — WDS hands-free hardening (CVE-2026-0386)](https://support.microsoft.com/en-us/servicing/os/windows/2025/12/windows-deployment-services-wds-hands-free-deployment-hardening-guidance-related-to-cve-2026-0386) — consulté le 24/07/2026
- [Microsoft Learn — WDS boot.wim support](https://learn.microsoft.com/en-us/windows/deployment/wds-boot-support) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (`wdsutil`) · [x] versions datées · [x] **statut/dépréciation + CVE signalés** · [x] procédure **à tester en lab** · [x] conforme doc Microsoft · [x] vérification présente · [x] sécurité (CVE, VLAN isolé, approbation) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
