# CP9-04 — Automatiser l'installation Windows (fichier de réponses / autounattend)

**Objectif** : installer Windows **sans intervention** grâce à un fichier de réponses **`autounattend.xml`** (partitionnement, comptes, région, OOBE).

**Rattachement REAC** : CP9 « Exploiter et maintenir les services de déploiement des postes » — savoir-faire : automatiser l'installation des postes.

**Durée** : ~30 min · **Niveau** : avancé.

---

## Prérequis

- Un média d'installation **Windows 11 24H2** (ISO/USB).
- L'**ADK** Windows avec **Deployment Tools** → **WSIM** (Windows System Image Manager).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| OS cible | **Windows 11 24H2** (et 25H2) | 24/07/2026 |
| Outil | **WSIM** (ADK) | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **`autounattend.xml`** répond à la place de l'installateur, étape par étape, via des **passes de configuration** : `windowsPE` (langue, disque, clé), `specialize` (nom, domaine), `oobeSystem` (comptes, OOBE). On l'**autorise** rarement à la main : on le **génère avec WSIM**.

---

## Procédure — étapes

### 1. Créer le fichier avec WSIM

1. Installer l'**ADK** (+ Deployment Tools) → ouvrir **WSIM**.
2. **Select a Windows Image** : ouvrir l'`install.wim` du média (WSIM lit tous les composants configurables).
3. **File ▸ New Answer File** → ajouter les composants voulus dans les bonnes passes :
   - **Disk Configuration** (partitionnement UEFI/GPT),
   - **User Accounts** (admin local), **Product Key**,
   - **Regional/Language**, **OOBE** (bypass compte Microsoft, EULA).
4. **Tools ▸ Validate Answer File**, puis **enregistrer** sous `autounattend.xml`.

### 2. Déployer

5. Copier `autounattend.xml` **à la racine** de la clé USB bootable → démarrer dessus : l'installation se déroule **sans question**.

> Depuis **24H2**, un `autounattend.xml` à la racine de l'ISO est **automatiquement copié** dans `C:\Windows\Panther`. ⚠️ Certaines builds 24H2 **sautent** les passes `specialize`/`oobeSystem` (nouveau setup « ConX ») → **tester en VM** et adapter.

### 3. Alternative — Provisioning package

- Pour **configurer** une installation **existante** (pas réinstaller), utiliser un **package d'approvisionnement** créé avec **Windows Configuration Designer**.

---

## Vérification (comment savoir que ça marche)

- L'installation se termine **sans aucune invite** jusqu'au bureau/OOBE attendu.
- Journaux : `C:\Windows\Panther\setupact.log` / `setuperr.log` (erreurs de passe).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Des invites subsistent | Passe **sautée** (quirk 24H2) / mauvaise architecture | Tester en VM ; réordonner ; vérifier x64/ARM |
| Mauvais disque effacé | Disk Configuration ambiguë | **Tester en VM** ; cibler par ID de disque |
| Compte Microsoft imposé | Bypass OOBE absent | Ajouter le contournement dans `oobeSystem` |
| Fichier ignoré | Mauvais nom/emplacement | `autounattend.xml` **à la racine** du média |

## Sécurité et bonnes pratiques

- Le fichier contient des **identifiants en clair** (ou base64) → **protéger le média**, ne pas y laisser de **compte de jonction au domaine** privilégié (rappel **CVE-2026-0386**, **CP9-03**).
- **Supprimer/sécuriser** l'`autounattend.xml` après déploiement.
- **Tester systématiquement en VM** avant un déploiement de masse (risque d'effacement disque).

## ⚠️ À ne pas confondre / obsolète

- **`autounattend.xml`** (média/`windowsPE`, installation) ≠ **`unattend.xml`** (Sysprep/`specialize`, **CP9-02**).
- Éditer l'XML **à la main** est source d'erreurs → **WSIM** valide les composants.
- 24H2 change le setup : ne pas réutiliser aveuglément un fichier ancien.

---

## Sources (doc officielle)

- [Microsoft Learn — Answer files (unattended installation)](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/update-windows-settings-and-scripts-create-your-own-answer-file-sxs) — consulté le 24/07/2026
- [Microsoft Learn — Windows System Image Manager (WSIM)](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/windows-system-image-manager-technical-reference) — consulté le 24/07/2026
- [Microsoft Learn — Provisioning packages](https://learn.microsoft.com/en-us/windows/configuration/provisioning-packages/provisioning-packages) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI/fichier + WSIM · [x] versions datées (24H2/25H2) · [x] rien d'obsolète (quirk 24H2 signalé) · [x] procédure **à tester en lab (VM)** · [x] conforme doc Microsoft · [x] vérification présente (Panther logs) · [x] sécurité (identifiants, CVE) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
