# CT3-01 — Mettre en place une veille technologique et s'auto-former

**Objectif** : organiser une **veille** technologique et sécurité efficace (sources fiables, alertes, rythme) et **s'auto-former** en continu.

**Rattachement REAC** : Compétences transversales — « Actualiser ses compétences / apprendre en continu ».

**Durée** : ~15 min · **Niveau** : débutant · **Type** : Méthode.

---

## Prérequis

- Un accès Internet et de quoi centraliser ses sources (agrégateur RSS, favoris, notes).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Sources / démarche | Méthode | 24/07/2026 |

> **L'IT évolue vite** : nouvelles vulnérabilités, versions, **dépréciations** (ex. WSUS/WDS/**MDT** vus en **CP9**). La veille = **sécurité** (corriger à temps) **et** employabilité (rester à jour).

---

## Procédure — Méthode

### 1. Les trois veilles

| Veille | But | Exemples de sources |
|---|---|---|
| **Sécurité / vulnérabilités** | Corriger avant l'exploitation | **CERT-FR/ANSSI**, **NVD** (CVE/CVSS), CISA, OpenCVE, bulletins éditeurs |
| **Technologique** | Suivre produits/versions | Docs officielles, blogs, RSS, newsletters, Reddit r/sysadmin |
| **Réglementaire** | Rester conforme | CNIL, ANSSI (**CT4-01**) |

### 2. Organiser sa veille

- **Agrégateur RSS** (ex. Feedly) pour centraliser blogs/bulletins.
- **Alertes CVE ciblées** sur les produits **de son parc** (pas tout le NVD).
- **Rythme** : coup d'œil **quotidien** sécurité, **hebdomadaire** techno ; **curation** (garder l'essentiel, pas tout lire).

### 3. Vérifier l'information

- **Recouper au moins 2 sources indépendantes** + le **bulletin officiel** **avant** d'agir (patch, changement).

### 4. S'auto-former

- **MOOC/plateformes** : Microsoft Learn, OpenClassrooms, Coursera.
- **Lab personnel** (**LAB-01**) : le meilleur terrain d'entraînement (tester avant la prod).
- **Pratique guidée** (labs de type CTF/exercices), **certifications** pour structurer un parcours.

---

## Vérification (comment savoir que c'est en place)

- Vos **flux/alertes** sont actifs et **filtrés** sur votre périmètre.
- Face à une alerte (ex. CVE critique sur un produit du parc), vous savez **où vérifier** et **quoi appliquer** (**CP2-15/CP3-11**).

## Dépannage (pièges fréquents)

| Symptôme | Cause probable | Solution |
|---|---|---|
| **Infobésité** | Trop de sources | **Curer** ; cibler son parc |
| Fausse alerte / rumeur | Source unique | **Recouper** + bulletin officiel |
| Veille **passive** | Pas d'action | Transformer l'info en **décision** (patch, test) |
| Retard sur les dépréciations | Pas de veille produit | Suivre les *lifecycle/roadmap* éditeurs |

## Sécurité et bonnes pratiques

- **Alertes CVE** sur le parc + **application des correctifs** (chaînage MàJ **CP2-15/CP3-11**, durcissement **CP7-17**).
- **Prudence** sur les sources (désinformation) ; ne pas exécuter une commande non comprise (**CT1-02**, **CP6-10**).
- **Partager** la veille à l'équipe (capitalisation).

## ⚠️ À ne pas confondre / obsolète

- **Veille** (suivre) ≠ **auto-formation** (monter en compétence) : complémentaires.
- **CVE** (identifiant de vulnérabilité) / **CVSS** (score de gravité) : deux choses distinctes.
- Une **source unique** n'est jamais une preuve → recouper.

---

## Sources (doc officielle / référence)

- [CERT-FR (ANSSI) — Alertes et bulletins](https://www.cert.ssi.gouv.fr/) — consulté le 24/07/2026
- [NVD — National Vulnerability Database (CVE/CVSS)](https://nvd.nist.gov/) — consulté le 24/07/2026
- [Microsoft Learn — auto-formation](https://learn.microsoft.com/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] Type Méthode · [x] daté 24/07/2026 · [x] rien d'obsolète (sources 2026) · [x] non applicable au bac à sable (méthode) · [x] cohérent CERT-FR/NVD · [x] vérification présente · [x] sécurité (alertes CVE, recoupement) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
