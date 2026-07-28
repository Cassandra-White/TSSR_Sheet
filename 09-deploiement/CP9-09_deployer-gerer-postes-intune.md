# CP9-09 — Déployer et gérer les postes avec Intune (MDM moderne, BYOD)

**Objectif** : gérer les postes et mobiles avec **Microsoft Intune** — enrôler, appliquer conformité et configuration, déployer des apps, sécuriser via **Accès conditionnel**, gérer le **BYOD** (MAM).

**Rattachement REAC** : CP9 « Exploiter et maintenir les services de déploiement des postes » — savoir-faire : administrer un parc via MDM moderne.

**Durée** : ~35 min · **Niveau** : avancé.

---

## Prérequis

- Un tenant **Microsoft Entra ID** + licences **Intune**.
- Des appareils **Windows 11 / iOS / Android** à enrôler.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Console | **Microsoft Intune admin center** | 24/07/2026 |
| Identité | **Microsoft Entra ID** | 24/07/2026 |
| Procédure appliance | **à tester en lab (tenant)** | 24/07/2026 |

> **Intune** = **MDM/MAM dans le Cloud**. L'ancienne marque « **Microsoft Endpoint Manager (MEM)** » est **retirée** au profit de « **Microsoft Intune** » ; MEM désignait l'ensemble **Intune + Configuration Manager + Autopilot + Defender**.

---

## Procédure — GUI (Intune admin center)

### 1. Enrôler les appareils

- **Windows d'entreprise** : **Autopilot** (préconfiguré, l'utilisateur déballe et se connecte) ou **jonction Entra + enrôlement automatique**.
- **iOS/Android** : application **Company Portal**.
- **BYOD** : **MAM sans enrôlement** (protection des apps pro uniquement — **CP9-08**).

### 2. Profils de configuration

- **Devices ▸ Configuration** : Wi-Fi, VPN, **BitLocker**, restrictions, mises à jour.

### 3. Politiques de conformité

- **Devices ▸ Compliance** : exiger chiffrement, version d'OS minimale, code PIN → l'appareil devient **conforme / non conforme**.

### 4. Accès conditionnel (Entra)

- Créer une règle **exigeant un appareil conforme + MFA** pour accéder à M365.
- ⚠️ **Déployer d'abord en *Report-Only*** (journalise sans bloquer), analyser 1–2 semaines, puis **activer**.

### 5. Déployer des applications

- **Apps ▸ Add** : Store/**winget** (**CP9-10**), MSI/Win32, et **App Protection Policies** pour le MAM (BYOD).

---

## Vérification (comment savoir que ça marche)

- L'appareil apparaît **Managed** et **Compliant** dans la console.
- Une app assignée s'installe ; un appareil **non conforme** est **bloqué** par l'Accès conditionnel.
- Côté utilisateur : **Company Portal** liste les apps/état.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Enrôlement échoue | Licence/Entra/MDM authority | Vérifier les licences et l'auto-enrollment Entra |
| Blocage général d'accès | Règle CA trop large | **Report-Only** d'abord ; **compte break-glass** exclu |
| Autopilot ne démarre pas | **Hash matériel** non importé | Importer le hash de l'appareil |
| App non installée | Assignation/ciblage | Vérifier le groupe d'affectation |

## Sécurité et bonnes pratiques

- **Accès conditionnel + MFA** (**CP7-18**) : socle de la sécurité Intune.
- **Compte « break-glass »** exclu des règles CA (éviter de se verrouiller dehors).
- **Report-Only** avant application ; **moindre privilège** sur les rôles Intune.
- **Effacement sélectif** pour le BYOD (retire le pro sans toucher au perso).

## ⚠️ À ne pas confondre / obsolète

- **Microsoft Endpoint Manager (MEM)** : **marque retirée** → « **Microsoft Intune** ».
- **MDM** (appareil enrôlé) ≠ **MAM** (apps protégées, BYOD).
- **Autopilot** = standard pour le Windows **d'entreprise** ; le BYOD passe plutôt par **MAM**.

---

## Sources (doc officielle)

- [Microsoft Learn — Microsoft Intune (documentation)](https://learn.microsoft.com/en-us/intune/intune-service/fundamentals/) — consulté le 24/07/2026
- [Microsoft Learn — Accès conditionnel + conformité Intune](https://learn.microsoft.com/en-us/intune/device-security/conditional-access-integration/overview) — consulté le 24/07/2026
- [Microsoft Learn — Windows Autopilot](https://learn.microsoft.com/en-us/autopilot/overview) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI (Intune) · [x] daté 24/07/2026 · [x] rien d'obsolète (rebrand MEM→Intune, Report-Only) · [x] procédure **à tester en lab (tenant)** · [x] conforme doc Microsoft · [x] vérification présente (Compliant) · [x] sécurité (CA/MFA, break-glass) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
