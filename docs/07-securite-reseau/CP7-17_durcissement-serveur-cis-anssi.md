# CP7-17 — Durcir un serveur (hardening Windows/Linux selon CIS/ANSSI)

**Objectif** : durcir un serveur en s'appuyant sur un **référentiel reconnu** (CIS Benchmarks, baselines Microsoft, ANSSI) : **auditer l'écart**, **appliquer**, **re-auditer**, documenter.

**Rattachement REAC** : CP7 « Maintenir et sécuriser les accès Internet et les interconnexions » (+ CP2/CP3) — savoir-faire : durcir les systèmes.

**Durée** : ~35 min · **Niveau** : avancé.

---

## Prérequis

- Un serveur **Windows Server 2025** (**CP2**) et/ou **Debian 13** (**CP3**), de préférence en **préproduction**.
- Un **snapshot**/sauvegarde préalable (**CP8**) et un **accès de secours** (console).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Référentiel Windows | **CIS Benchmark WS 2025** + **Microsoft Security Baseline** | 24/07/2026 |
| Référentiel Linux | **CIS Benchmark Debian** / **SCAP Security Guide** | 24/07/2026 |
| Outil d'audit Linux | **Lynis 3.1.8** (obtenu ; audit complet **root** → lab) | 24/07/2026 |

> **Démarche (4 temps)** : **1. Choisir un référentiel** (CIS **Niveau 1** = sûr et large ; **Niveau 2** = strict) → **2. Auditer** l'écart → **3. Appliquer** (progressivement) → **4. Re-auditer** et **documenter les exceptions**.

---

## Procédure — GUI (Windows Server)

1. Télécharger le **Microsoft Security Compliance Toolkit** + la **Security Baseline** de Windows Server 2025 (et/ou le **CIS Benchmark** correspondant).
2. Sur une machine **du domaine** : **importer** la baseline comme **GPO** (via *Group Policy Management*) et la lier à l'OU des serveurs (**CP2-11**).
3. Sur une machine **hors domaine** : appliquer avec **`LGPO.exe`** :

```powershell
LGPO.exe /g .\GPOs\{baseline-guid}    # importe la stratégie locale depuis la baseline
gpupdate /force
```

4. Ajuster les réglages incompatibles avec les applications, **documenter** chaque exception.

## Procédure — CLI (Linux : audit puis application)

### Auditer l'écart

```bash
# Lynis : score de durcissement (hardening index) + suggestions
sudo lynis audit system            # rapport : "Hardening index : NN"

# OpenSCAP : évaluation contre un profil CIS (SCAP Security Guide)
sudo apt install libopenscap8 ssg-debderived
sudo oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis_level1_server \
     --results scan.xml --report rapport.html \
     /usr/share/xml/scap/ssg/content/ssg-debian13-ds.xml
```

### Appliquer les mesures

- Points clés récurrents (recoupent CP3) : **SSH durci** (**CP3-09**), **pare-feu** (**CP3-14**), **MAJ auto** (**CP3-11**), services inutiles désactivés, `sysctl` réseau, permissions/`umask` (**CP3-04**), journalisation (**CP3-15**).
- À l'échelle : rejouer un **rôle Ansible de durcissement CIS** pour une application reproductible.

> **Testé en bac à sable** : **Lynis 3.1.8** récupéré et exécutable (`lynis show version`). L'audit complet nécessitant **root**, le score de durcissement est **à produire en lab**.

---

## Vérification (comment savoir que ça marche)

- **Re-scan** après application : le **hardening index** Lynis **remonte**, les contrôles OpenSCAP passent de *fail* à *pass*.
- Windows : `gpresult /h rapport.html` confirme l'application de la baseline.
- Les services **métier fonctionnent toujours** (pas de régression).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Service cassé après durcissement | Réglage **Niveau 2** trop strict | Revenir/exception **documentée**, rester en L1 |
| Baseline « incompatible » | Mauvaise version d'OS | Prendre le benchmark **de la version exacte** |
| Perte d'accès admin | SSH/RDP durci sans nouvel accès prêt | **Valider le nouvel accès avant** ; garder une console |
| Score qui ne monte pas | Mesures non appliquées/non persistées | Vérifier l'application réelle, re-scanner |

## Sécurité et bonnes pratiques

- **Tester en préproduction**, **snapshot avant**, appliquer **progressivement** (L1 puis L2 ciblé).
- **Moindre privilège**, désactiver comptes/services par défaut, **journaliser** (**CP7-10**).
- **Documenter** le référentiel retenu, la version, et **chaque exception** (traçabilité audit).
- Durcir n'est pas ponctuel : **re-auditer** après chaque changement majeur.

## ⚠️ À ne pas confondre / obsolète

- Durcissement **manuel non tracé** ≠ **référentiel + audit reproductible** (CIS/SCAP).
- **Baseline Microsoft** (préliminaire) et **CIS** (exhaustif) sont **complémentaires**, pas exclusifs.
- **CIS Niveau 1** (sûr) vs **Niveau 2** (haute sécurité, impact fonctionnel possible).

---

## Sources (doc officielle)

- [CIS — Benchmarks](https://www.cisecurity.org/cis-benchmarks) — consulté le 24/07/2026
- [Microsoft Learn — Security baselines & Security Compliance Toolkit](https://learn.microsoft.com/en-us/windows/security/operating-system-security/device-management/windows-security-configuration-framework/security-compliance-toolkit-10) — consulté le 24/07/2026
- [CISOfy — Lynis (audit de durcissement)](https://cisofy.com/lynis/) — consulté le 24/07/2026
- [ANSSI — Recommandations de configuration système (GNU/Linux, Windows)](https://cyber.gouv.fr/publications) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI (Windows/GPO) puis CLI (Lynis/OpenSCAP) · [x] versions datées · [x] rien d'obsolète (audit reproductible) · [x] **Lynis testé** (présent, v3.1.8) / audit complet à faire en lab · [x] conforme CIS/Microsoft · [x] vérification présente (re-scan) · [x] sécurité (préprod, snapshot, exceptions) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
