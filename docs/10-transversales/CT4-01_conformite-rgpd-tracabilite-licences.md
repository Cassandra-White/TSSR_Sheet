# CT4-01 — Conformité : RGPD, obligations légales de traçabilité et licences logicielles

**Objectif** : connaître les obligations de **conformité** qui concernent le technicien — **RGPD** (données personnelles), **traçabilité légale** (journaux) et **licences logicielles**.

**Rattachement REAC** : Compétences transversales — « Prendre en compte le cadre légal et réglementaire ».

**Durée** : ~20 min · **Niveau** : intermédiaire · **Type** : Méthode.

> ⚖️ *Ce tuto donne des repères pratiques, ce n'est pas un avis juridique. En cas de doute, se référer au DPO/juriste.*

---

## Prérequis

- Notions de sécurité (chiffrement, accès, sauvegarde) déjà vues (**CP7/CP8**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Cadre / démarche | Méthode (droit FR/UE) | 24/07/2026 |

---

## Procédure — Méthode

### 1. RGPD (Règlement UE 2016/679)

**Principes** : licéité, **finalité**, **minimisation** (collecter le strict nécessaire), **limitation de conservation**, **sécurité**, et **droits des personnes** (accès, rectification, effacement, portabilité).

**Obligations pratiques** (depuis 2018, la **déclaration préalable CNIL est supprimée** au profit de la **responsabilité/accountability**) :

- **Registre des traitements** : **écrit**, à jour, présentable en cas de contrôle → **pierre angulaire** de la conformité.
- **DPO** (délégué) selon l'organisme ; **AIPD/DPIA** pour les traitements à risque.
- **Notification de violation** de données à la CNIL sous **72 h**.

**Rôle du TSSR** : assurer la **sécurité technique** des données personnelles — **chiffrement**, **contrôle d'accès** (moindre privilège), **sauvegardes** (**CP8**), **MFA** (**CP7-18**), **effacement sécurisé** au retrait (**STO-08**), **minimisation** des journaux.

> **Contrôles CNIL 2026** : la **cybersécurité** représente ~**50 %** des contrôles → la sécurité n'est plus optionnelle.

### 2. Traçabilité légale (journaux)

- Conserver les **données de connexion** permettant d'identifier l'origine d'un accès **jusqu'à 1 an** (LCEN, **décret 2021-1362** — mise en œuvre en **CP7-10**).
- **Équilibre RGPD** : ne pas conserver **au-delà** du nécessaire ; protéger et restreindre l'accès aux journaux.

### 3. Licences logicielles

| Type | Exemples | À respecter |
|---|---|---|
| **Libre / open source** | GPL (*copyleft*), MIT/Apache/BSD (permissives) | Attribution, obligations copyleft (redistribution du code) |
| **Propriétaire** | Windows, VMware… | **CLUF/EULA**, nombre d'installations, métrage |

- **Modes de licence** : par **utilisateur**, par **cœur/CPU**, par **appareil**, **abonnement**.
- **Conformité** : tenir un **inventaire des licences**, ne pas **sous-licencier** (illégal) ni **sur-payer** ; se préparer aux **audits éditeurs**.

---

## Vérification (comment savoir que c'est maîtrisé)

- Le **registre des traitements** existe et est à jour.
- Les **licences** sont inventoriées et **conformes** (nb d'installations = nb de licences).
- Les **journaux** sont conservés **selon la loi**, ni trop ni trop peu, et **protégés**.

## Dépannage (pièges fréquents)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Sur-collecte de données | Finalité/minimisation ignorées | Ne collecter que le **nécessaire** |
| Audit éditeur douloureux | Licences non suivies | **Inventaire** + rapprochement régulier |
| Journaux trop/pas assez gardés | Rétention mal réglée | Aligner sur la **loi** (1 an) + RGPD |
| Violation non notifiée | Procédure absente | Prévoir la **notification 72 h** |

## Sécurité et bonnes pratiques

- **RGPD = sécurité des données** côté technique : chiffrement, accès, sauvegarde testée, effacement sécurisé.
- **Documenter** (registre, procédures) : la **preuve** de conformité compte autant que la conformité.
- **Sensibiliser** les utilisateurs (**CP1-10**) : la conformité est l'affaire de tous.

## ⚠️ À ne pas confondre / obsolète

- **Déclaration préalable CNIL** = **supprimée** (depuis 2018) → **registre + accountability**.
- **Libre** (≠ gratuit) impose aussi des **obligations** (attribution, copyleft).
- **Traçabilité** (obligation de conserver les logs d'accès) vs **RGPD** (ne pas conserver au-delà du nécessaire) : trouver l'**équilibre**.

---

## Sources (doc officielle / référence)

- [CNIL — RGPD : se mettre en conformité / registre](https://www.cnil.fr/fr/rgpd-de-quoi-parle-t-on) — consulté le 24/07/2026
- [EUR-Lex — Règlement (UE) 2016/679 (RGPD)](https://eur-lex.europa.eu/legal-content/FR/TXT/?uri=CELEX%3A32016R0679) — consulté le 24/07/2026
- [Légifrance — Décret n° 2021-1362 (conservation des données de connexion)](https://www.legifrance.gouv.fr/jorf/id/JORFTEXT000044228912) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] Type Méthode · [x] daté 24/07/2026 · [x] rien d'obsolète (accountability vs déclaration) · [x] non applicable au bac à sable (méthode) · [x] cohérent CNIL/EUR-Lex/Légifrance · [x] vérification présente · [x] sécurité (chiffrement, accès, preuve) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
