# CP1-15 — Maintenance matérielle d'un poste (diagnostic panne, remplacement de composants)

**Objectif** : diagnostiquer une **panne matérielle** et **remplacer** les composants d'un poste (RAM, disque, alimentation, ventilation) en respectant les précautions **ESD**.

**Rattachement REAC** : CP1 « Assurer le support utilisateur » — savoir-faire : maintenir le matériel.

**Durée** : ~25 min · **Niveau** : intermédiaire · **Type** : Méthode.

---

## Prérequis

- Un poste à intervenir, l'**outillage** (tournevis, bracelet antistatique), et les **pièces** de rechange compatibles.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Démarche / sécurité | Méthode | 24/07/2026 |

> ⚡ **ESD (décharge électrostatique)** : une simple étincelle **invisible** peut **détruire** un composant. **Bracelet antistatique**, se **décharger**, **débrancher** l'alimentation (et la batterie) **avant** toute manipulation.

---

## Procédure — Méthode

### 1. Sécurité avant d'ouvrir

- **Débrancher** le secteur, retirer la **batterie** (portable), appuyer sur le bouton d'alimentation (décharge résiduelle).
- **Bracelet antistatique** relié à une masse ; surface **antistatique**.
- ⚠️ Ne pas ouvrir un **bloc d'alimentation** ni un **écran** (condensateurs — risque électrique).

### 2. Diagnostiquer le composant (symptôme → pièce)

| Symptôme | Composant probable | Vérification |
|---|---|---|
| Pas de POST / **bips** | RAM / carte mère | Réinsérer la RAM ; **memtest** (**CP1-13**) |
| Plantages aléatoires | RAM | `mdsched`/**memtest** |
| Lenteurs, secteurs | Disque/SSD | **SMART** (**STO-09**) |
| Ne s'allume pas | Alimentation | Testeur d'alim / remplacement |
| Surchauffe, arrêts | Ventilation / **pâte thermique** | Nettoyer, refaire la pâte |
| Heure qui se perd | **Pile CMOS** | Remplacer la pile |

### 3. Remplacer le composant

- **RAM** : ouvrir les clips, respecter le **détrompeur**, enclencher jusqu'au clic.
- **Disque/SSD** : SATA (câble + alim) ou **NVMe M.2** (vis) ; **sauvegarder les données avant** (**CP8**).
- **Alimentation / ventilateurs** : repérer le câblage avant de débrancher.
- **Pâte thermique** : nettoyer l'ancienne, appliquer une **fine** couche, remonter le dissipateur.

### 4. Tester après remontage

- **POST** OK, **memtest**, **températures** normales, stabilité.

---

## Vérification (comment savoir que c'est réparé)

- Le poste **démarre** et reste **stable** ; les diagnostics (memtest/SMART/températures) sont **bons**.
- Le symptôme initial a **disparu**.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Ne boote pas après montage | RAM mal enclenchée / câble oublié | Réinsérer, vérifier tous les connecteurs |
| Composant « grillé » | **ESD** / mauvais branchement | Prévention ESD ; vérifier polarité/détrompeurs |
| Surchauffe persistante | Pâte mal appliquée / ventilo | Refaire la pâte, vérifier la rotation du ventilo |
| Incompatibilité | Pièce non adaptée | Vérifier références (RAM/format M.2, watts alim) |

## Sécurité et bonnes pratiques

- **ESD obligatoire** ; **débrancher** avant d'ouvrir ; prudence **alim/écran** (condensateurs).
- **Sauvegarder les données** avant de toucher au disque (**CP8**), et **effacer** un disque réformé (**STO-08/CT4-01**).
- **Pièces compatibles** (références, capacités, connectique) ; documenter l'intervention (**CP1-09**).

## ⚠️ À ne pas confondre / obsolète

- **Nettoyer** ≠ **remplacer** : parfois une surchauffe = simple **poussière** à retirer.
- **SATA** (câble) ≠ **NVMe M.2** (port dédié) : formats de stockage différents.
- Manipuler **sans ESD** = risque de casse **invisible**.

---

## Sources (référence)

- [ANSSI/ADEME — réemploi & maintenance responsable](https://www.ademe.fr/) — consulté le 24/07/2026
- [Documentation constructeur (Dell/HP/Lenovo — guides de maintenance)](https://www.dell.com/support/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] Type Méthode · [x] daté 24/07/2026 · [x] rien d'obsolète (NVMe, ESD) · [x] non applicable au bac à sable (méthode) · [x] cohérent guides constructeurs · [x] vérification présente (POST/memtest/températures) · [x] sécurité (ESD, débrancher, données) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
