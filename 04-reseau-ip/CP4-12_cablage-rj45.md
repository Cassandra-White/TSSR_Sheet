# CP4-12 — Câblage réseau : réaliser et tester un cordon RJ45

**Objectif** : sertir un cordon réseau RJ45 selon les normes T568A/B et le tester avant mise en service.

**Rattachement REAC** : CP4 « Exploiter un réseau IP » — savoir-faire (méthode) : réaliser et vérifier un câblage cuivre.

**Durée** : ~20 min · **Niveau** : débutant.

---

## Prérequis

- Câble à paires torsadées (Cat 5e/6/6a), connecteurs **RJ45**, **pince à sertir**, **pince à dénuder**, **testeur de câble**.

## Normes de brochage (TIA-568)

On choisit **une** norme et on s'y tient. **T568B** est le plus courant en entreprise.

| Broche | **T568B** | **T568A** |
|---|---|---|
| 1 | blanc-orange | blanc-vert |
| 2 | orange | vert |
| 3 | blanc-vert | blanc-orange |
| 4 | bleu | bleu |
| 5 | blanc-bleu | blanc-bleu |
| 6 | vert | orange |
| 7 | blanc-marron | blanc-marron |
| 8 | marron | marron |

- **Câble droit** (*straight-through*) : **même norme aux deux extrémités** (poste ↔ switch).
- **Câble croisé** (*crossover*) : **T568A d'un côté, T568B de l'autre** (switch ↔ switch, historiquement).

---

## Méthode — sertissage

1. **Dégainer** ~2,5 cm de gaine sans entailler les brins.
2. **Démêler** les 4 paires et les **ordonner** selon la norme choisie (T568B).
3. **Aplatir/aligner** les 8 fils, puis **couper net** (~14 mm) pour qu'ils atteignent le fond du connecteur.
4. **Insérer** dans le RJ45, fils jusqu'au bout, **gaine engagée** sous la zone de sertissage.
5. **Sertir** à la pince (elle enfonce les contacts et bloque la gaine). Répéter à l'autre extrémité.

## Vérification — test au testeur de câble

1. Brancher les deux bouts sur le testeur.
2. Les **8 voyants** doivent s'allumer **dans l'ordre 1→8** (émetteur/récepteur).
3. Une fois branché : la **LED du port** s'allume et le lien monte (`ip link` = *UP*), au débit attendu (1 Gb/s en Cat 5e+).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Pas de lien | Ordre des fils incorrect / mauvais sertissage | Refaire selon la norme, resertir |
| Négocie **100 Mb/s** au lieu de 1000 | Une paire coupée (le Gigabit utilise **4 paires**) | Vérifier au testeur, resertir |
| Voyant testeur manquant/inversé | Fil ouvert ou paires inversées | Contrôler l'ordre des brins |
| Lien intermittent | Gaine non engagée / brins mal enfoncés | Resertir proprement |

## Sécurité et bonnes pratiques

- Respecter la **longueur maximale 100 m** (cuivre) et le **rayon de courbure**.
- Éloigner les câbles des **sources d'interférences** (courants forts, néons).
- **Étiqueter** les cordons et reporter dans la **documentation** (**CP4-11**).

## ⚠️ À ne pas confondre / obsolète

- Le **câble croisé** est devenu **quasi inutile** grâce à l'**Auto-MDIX** (les équipements modernes croisent automatiquement).
- **Ne pas mélanger** T568A et T568B sur un câble censé être **droit**.
- La **catégorie** du câble (5e/6/6a) conditionne le **débit max** : la choisir selon le besoin (10 Gb/s → Cat 6a).

---

## Sources (référence)

- [ANSI/TIA-568 — Commercial Building Telecommunications Cabling (présentation)](https://en.wikipedia.org/wiki/TIA-568) — consulté le 24/07/2026
- [Fluke Networks — Wiring T568A/T568B (référence câblage)](https://www.flukenetworks.com/knowledge-base/dtx-cableanalyzer/t568a-and-t568b-wiring-schemes) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] Méthode · [x] normes datées (TIA-568) · [x] rien d'obsolète (Auto-MDIX signalé) · [x] procédure de test fournie · [x] conforme norme · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
