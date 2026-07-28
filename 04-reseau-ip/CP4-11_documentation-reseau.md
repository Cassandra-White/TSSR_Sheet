# CP4-11 — Documenter le réseau (schémas physique et logique)

**Objectif** : produire et maintenir une documentation réseau exploitable : schéma **physique**, schéma **logique**, plan d'adressage et inventaire.

**Rattachement REAC** : CP4 « Exploiter un réseau IP » — savoir-faire (méthode) : documenter une infrastructure réseau.

**Durée** : ~20 min · **Niveau** : débutant.

---

## Prérequis

- Connaissance du réseau à documenter (équipements, VLAN, adressage — **CP4-01/04**).

## Que documenter ?

| Document | Contenu | Outils |
|---|---|---|
| **Schéma physique** | équipements, baies, ports, câblage, liens, modèles/n° série | draw.io / diagrams.net |
| **Schéma logique** | VLAN, sous-réseaux, passerelles, routage, flux | draw.io / diagrams.net |
| **Plan d'adressage (IPAM)** | attribution des IP, réservations DHCP, DNS | tableur, **NetBox**, phpIPAM |
| **Inventaire du parc** | matériel, garanties, localisation | **GLPI** (**CP1-01**) |
| **Configurations** | sauvegardes des équipements | **CP8-08**, dépôt **Git** |

---

## Méthode

1. **Conventions de nommage** cohérentes : `SW-CoeurA`, `SRV-AD01`, `R-WAN01`, `FW-01`… (site-rôle-numéro).
2. **Schéma logique** — exemple minimal :

   ```
   [Internet]—[R-WAN01]—[FW-01]—[SW-CoeurA]
                                    ├─ VLAN10 POSTES    192.168.10.0/24 (gw .1)
                                    ├─ VLAN20 SERVEURS  192.168.20.0/24 (gw .1)
                                    └─ VLAN99 GESTION   192.168.99.0/24 (gw .1)
   ```

3. **Schéma physique** : qui est branché où (switch/port ↔ équipement), étiquetage des câbles (**CP4-12**).
4. **Tenir à jour** : mettre à jour **à chaque changement** (une doc périmée est pire qu'une absence de doc).
5. **Versionner** (Git) et **restreindre l'accès**.

---

## Vérification (comment savoir que la doc est bonne)

- Un collègue peut **comprendre et dépanner** le réseau **sans** le concepteur.
- Le schéma correspond à la réalité (VLAN, adressage, liens) **après** le dernier changement.

## Dépannage (pièges fréquents)

| Symptôme | Cause | Solution |
|---|---|---|
| Doc trompeuse | Non mise à jour | Intégrer la MAJ doc au processus de changement |
| Incohérences de noms | Pas de convention | Définir et appliquer une nomenclature |
| Fuite d'informations | Mots de passe dans la doc | Gestionnaire de secrets, jamais en clair |

## Sécurité et bonnes pratiques

- La **cartographie réseau est sensible** : accès **restreint** (elle facilite aussi les attaques).
- **Aucun secret en clair** dans la documentation.
- **Versionner** la doc et la conserver **hors de la seule tête** de l'administrateur.

## ⚠️ À ne pas confondre / obsolète

- **Schéma physique** (câblage, ports) ≠ **schéma logique** (VLAN, IP, flux) : les **deux** sont nécessaires.
- La documentation est **vivante** : elle se met à jour, elle n'est jamais « finie ».

---

## Sources (référence)

- [ANSSI — Cartographie du système d'information](https://cyber.gouv.fr/publications/cartographie-du-systeme-dinformation) — consulté le 24/07/2026
- [NetBox — Documentation (DCIM/IPAM)](https://netboxlabs.com/docs/netbox/) — consulté le 24/07/2026
- [diagrams.net (draw.io)](https://www.drawio.com/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] Méthode · [x] repères datés · [x] rien d'obsolète · [x] exemple concret fourni · [x] conforme bonnes pratiques (ANSSI) · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
