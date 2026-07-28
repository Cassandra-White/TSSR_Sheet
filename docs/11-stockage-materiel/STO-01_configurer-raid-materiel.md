# STO-01 — Configurer un RAID matériel (contrôleur ; niveaux 0/1/5/6/10)

**Objectif** : créer et gérer une grappe **RAID** sur un **contrôleur matériel**, choisir le bon **niveau** (0/1/5/6/10) et prévoir un **disque de secours** (hot spare).

**Rattachement REAC** : CP5 (équipements matériels : serveurs, baies) / CP8 (supports de sauvegarde) — savoir-faire : mettre en œuvre le stockage résilient.

**Durée** : ~30 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un serveur avec **contrôleur RAID matériel** (Dell **PERC**, HPE **Smart Array**, Broadcom/LSI **MegaRAID**…) et plusieurs disques identiques.
- Accès à l'utilitaire du contrôleur (au boot ou via **iDRAC/iLO**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Contrôleur | RAID matériel (référence : **MegaRAID / storcli**) | 24/07/2026 |
| Démo parité/capacité | **testée en bac à sable** (XOR + table) | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **RAID matériel** = un **contrôleur dédié** (avec cache, souvent batterie **BBU**) gère la grappe **indépendamment de l'OS**, qui ne voit qu'**un seul volume**. Cela décharge le CPU et permet le **cache d'écriture**.

---

## Niveaux RAID — capacité utile et tolérance (ex. 4 disques de 4 To)

| Niveau | Principe | Capacité utile | Tolérance de panne |
|---|---|---|---|
| **RAID 0** | Agrégation par bandes | **16 To** | **aucune** (risqué) |
| **RAID 1** | Miroir | 4 To | 1 (n-1) |
| **RAID 5** | Bandes + **1 parité** | **12 To** | **1 disque** |
| **RAID 6** | Bandes + **2 parités** | 8 To | **2 disques** |
| **RAID 10** | Miroir **+** bandes | 8 To | 1 par sous-miroir |

> **Parité (RAID 5) démontrée** en bac à sable : `P = d1 XOR d2 XOR d3`. Après perte de `d2`, reconstruction `d2 = P XOR d1 XOR d3` → **valeur identique**. C'est ainsi que la grappe **survit à un disque perdu**.

---

## Procédure — GUI (utilitaire du contrôleur / iDRAC-iLO)

1. Au démarrage, entrer dans l'utilitaire RAID (ex. **Ctrl+R** MegaRAID) **ou** via **iDRAC/iLO** (Lifecycle/Web).
2. **Create Virtual Disk** : sélectionner les disques physiques, choisir le **niveau RAID**, la **taille de bande** (stripe) et la **politique de cache** (Write-Back si BBU saine).
3. Définir un **Hot Spare** (disque de secours global ou dédié) pour la reconstruction automatique.
4. **Initialiser** la grappe → l'OS voit **un seul** volume.

## Procédure — CLI (ex. Broadcom `storcli`)

```bash
storcli /c0 add vd type=raid5 drives=252:0-3 spare=252:4   # créer une grappe RAID5 + spare
storcli /c0/vall show                                       # état des volumes
storcli /c0/eall/sall show                                  # état des disques physiques
```

---

## Vérification (comment savoir que ça marche)

- Le contrôleur affiche la grappe **Optimal** ; l'OS voit **un** disque de la bonne taille.
- Test : **retirer un disque** → état **Degraded** → **reconstruction automatique** sur le hot spare → retour **Optimal**.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Grappe **Degraded** | Disque défaillant | Remplacer (**STO-07**) ; reconstruction sur spare |
| Cache write-back désactivé | **BBU** déchargée/absente | Vérifier/remplacer la batterie du contrôleur |
| Disques ignorés | **Foreign config** | Importer/effacer la config étrangère |
| 2ᵉ panne pendant rebuild | **URE** sur gros disques | Préférer **RAID 6 / RAID 10** |

## Sécurité et bonnes pratiques

- **Hot spare** + **supervision** (alertes contrôleur, **SMART** — **STO-09**).
- **Gros disques (>8 To)** : privilégier **RAID 6** ou **RAID 10** (fenêtre de reconstruction longue = risque de 2ᵉ panne).
- **Write-back** seulement avec **BBU** saine (sinon risque de perte au coupure).
- **RAID ≠ sauvegarde** : il protège d'une **panne disque**, pas d'une suppression/corruption/rançongiciel (**CP8-01**).

## ⚠️ À ne pas confondre / obsolète

- **RAID matériel** (contrôleur dédié) ≠ **RAID logiciel** (mdadm — **STO-02**).
- **RAID 5** sur **gros disques** = risqué (URE au rebuild) → **RAID 6/10**.
- **Parité** (5/6, capacité) ≠ **miroir** (1/10, performance/redondance).

---

## Sources (doc officielle)

- [Broadcom — storcli (MegaRAID) Reference](https://techdocs.broadcom.com/us/en/storage-and-ethernet-connectivity/enterprise-storage-solutions/megaraid-12gb-s.html) — consulté le 24/07/2026
- [Dell — PERC (RAID) Documentation](https://www.dell.com/support/kbdoc/en-us/000133678/) — consulté le 24/07/2026
- [Linux RAID Wiki — niveaux RAID](https://raid.wiki.kernel.org/index.php/Introduction) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (`storcli`) · [x] daté 24/07/2026 · [x] rien d'obsolète (RAID6/10 sur gros disques) · [x] **parité/capacité testées en bac à sable** / contrôleur à tester en lab · [x] conforme doc constructeurs · [x] vérification présente (Optimal/rebuild) · [x] sécurité (spare, BBU, RAID≠backup) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
