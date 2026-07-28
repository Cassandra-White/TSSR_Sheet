# CP7-12 — Appliquer les recommandations ANSSI (durcissement réseau)

**Objectif** : disposer d'une **démarche et d'une checklist** pour durcir un réseau d'entreprise selon les recommandations de l'**ANSSI**, et savoir où trouver les guides de référence.

**Rattachement REAC** : CP7 « Maintenir et sécuriser les accès Internet et les interconnexions » — savoir-faire : appliquer une politique de sécurité et durcir les équipements.

**Durée** : ~20 min · **Niveau** : intermédiaire · **Type** : Méthode.

---

## Prérequis

- Une cartographie à jour du réseau (**CP4-11**) : équipements, VLAN, flux, accès Internet.
- Accès administrateur aux commutateurs, pare-feu et serveurs.

## Cadre de référence (ANSSI)

| Guide ANSSI | À quoi il sert |
|---|---|
| **Guide d'hygiène informatique (42 mesures)** | Socle : les bonnes pratiques indispensables, du poste au réseau. Référence 2026. |
| **Recommandations pour l'administration sécurisée des SI** (PA-022) | Isoler et protéger l'administration (postes et réseau d'admin dédiés). |
| **Recommandations sécurité commutateurs / architecture réseau** | Durcir switches, VLAN, filtrage, cloisonnement. |

> **Démarche en 4 temps (ordre 2026)** : **1. Connaître** (cartographier) → **2. Contrôler les accès** (comptes, admin) → **3. Durcir** (services, protocoles, segmentation) → **4. Superviser & journaliser** (**CP4-17**, **CP7-10**). On durcit ce qu'on connaît, on surveille ce qu'on a durci.

---

## Procédure — Checklist de durcissement réseau

### 1. Connaître (cartographier)

1. Inventaire des équipements, firmwares et **versions** (mises à jour à jour ? — **CP2-15/CP3-11**).
2. Schémas **physique et logique**, plan d'adressage, matrice des flux (**CP4-01/CP4-11**).

### 2. Contrôler les accès d'administration

3. **Supprimer/renommer** les comptes par défaut, **mots de passe forts et uniques** (**CP2-17**), **MFA** quand c'est possible (**CP7-18**).
4. **Administration isolée** : réseau/VLAN d'admin dédié, hors du réseau bureautique (principe PA-022).
5. **Chiffrer l'administration** : **SSHv2** et **HTTPS** uniquement — **désactiver Telnet et HTTP**.
6. **SNMPv3** (auth + chiffrement) ; proscrire **SNMP v1/v2c** (**CP4-10**).

### 3. Durcir équipements et protocoles

7. **Désactiver les services et ports inutiles** (CDP/LLDP hors besoin, serveurs web/console non utilisés).
8. **Segmenter en VLAN** par usage (utilisateurs, serveurs, admin, DMZ, IoT) et **filtrer** entre segments (**CP4-04**, **CP7-04**).
9. **Sécuriser les ports d'accès** : **port security** (limite d'@MAC), **BPDU Guard**, **DHCP snooping**, **Dynamic ARP Inspection** (**CP4-15**).
10. **Désactiver les ports non utilisés** et les placer dans un VLAN « poubelle » isolé.
11. **Filtrage périmétrique** : politique **« tout ce qui n'est pas autorisé est interdit »** (deny by default) sur le pare-feu (**CP7-02**).
12. **Authentifier les protocoles** dynamiques (OSPF/routing avec authentification — **CP7-09**).

### 4. Superviser et journaliser

13. **Journalisation centralisée** des accès et des événements (**CP7-10**), horloge **NTP** commune (**CP4-13**).
14. **Supervision** disponibilité/anomalies (**CP4-17**), alertes.
15. **Sauvegarde des configurations** des équipements (**CP8-08**) et revue périodique.

---

## Vérification (comment savoir que ça marche)

- **Audit de configuration** ligne à ligne contre la checklist (Telnet/HTTP off ? SNMPv3 ? VLAN d'admin ? deny by default ?).
- **Scan de découverte** interne (**nmap**, **CP4-08**) : aucun service d'administration en clair ne doit répondre.
- **Test de segmentation** : depuis le VLAN utilisateurs, les interfaces d'admin des équipements doivent être **injoignables**.

## Dépannage (pièges fréquents)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Perte d'accès après durcissement | Telnet/HTTP coupé sans SSH/HTTPS prêt | Toujours **valider le nouvel accès avant** de couper l'ancien |
| Durcissement partiel/oublié | Pas de référentiel | S'appuyer sur les **42 mesures** + suivi écrit |
| Blocage de flux légitime | Segmentation trop stricte | Ajuster à partir de la **matrice des flux** |

## Sécurité et bonnes pratiques

- **Défense en profondeur** : empiler les mesures (périmètre + segmentation + durcissement des hôtes + supervision), ne pas tout miser sur le pare-feu.
- **Moindre privilège** partout (comptes, ACL, flux).
- **Sauvegarder la config avant** toute modification et **procéder progressivement**, un équipement à la fois.
- **Réversibilité** : garder un accès de secours (console série) pendant les opérations.

## ⚠️ À ne pas confondre / obsolète

- **Telnet, HTTP, SNMP v1/v2c, SSHv1** : à proscrire → **SSHv2 / HTTPS / SNMPv3**.
- « Durcir » ≠ « installer un pare-feu » : c'est une **démarche globale** (les 4 temps), pas un seul produit.
- Le **guide d'hygiène** n'est pas une norme opposable mais **la** base de bonnes pratiques reconnue en France.

---

## Sources (doc officielle)

- [ANSSI — Guide d'hygiène informatique (42 mesures)](https://cyber.gouv.fr/publications/guide-dhygiene-informatique) — consulté le 24/07/2026
- [ANSSI — Recommandations relatives à l'administration sécurisée des SI (PA-022)](https://cyber.gouv.fr/publications/recommandations-relatives-ladministration-securisee-des-si) — consulté le 24/07/2026
- [ANSSI — MesServicesCyber (guides et outils)](https://messervices.cyber.gouv.fr/guides/guide-dhygiene-informatique) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] Type Méthode (checklist + démarche) · [x] réf datée (guides ANSSI 2026) · [x] rien d'obsolète (SSHv2/HTTPS/SNMPv3) · [x] non applicable au bac à sable (méthode) — vérif par audit/nmap en lab · [x] conforme aux guides ANSSI · [x] vérification présente · [x] sécurité (défense en profondeur, moindre privilège) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
