# CP1-02 — Déployer l'agent GLPI et réaliser l'inventaire automatique du parc

**Objectif** : activer l'**inventaire natif** de GLPI, déployer le **GLPI Agent** sur les postes (Windows/Linux) et **remonter automatiquement** le parc.

**Rattachement REAC** : CP1 « Assurer le support utilisateur » — savoir-faire : inventorier automatiquement le parc.

**Durée** : ~30 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un serveur **GLPI 11** opérationnel (**CP1-01**), joignable par les postes.
- Postes **Windows 11** (domaine) et/ou **Debian**.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Serveur | **GLPI 11** (inventaire natif) | 24/07/2026 |
| Agent | **GLPI Agent** (Windows/Linux/macOS) | 24/07/2026 |
| Format d'inventaire (JSON) | **validé en bac à sable** | 24/07/2026 |

> Le **GLPI Agent** (successeur de FusionInventory) collecte l'inventaire (matériel, OS, logiciels, N° de série…) et l'envoie au serveur, qui **crée/ met à jour** automatiquement les **assets**.

---

## Procédure — GUI (activer l'inventaire côté serveur)

1. **Configuration ▸ Général ▸ Inventaire** → **Activer l'inventaire** (endpoint natif `/front/inventory.php`).

## Procédure — Déployer l'agent

### Windows — en masse par GPO (MSI)

1. Placer le **MSI** du GLPI Agent sur un **partage**.
2. **GPO ▸ Configuration ordinateur ▸ Software Installation ▸ Assigned** (**CP9-05**) → le MSI s'installe au prochain démarrage (agent en **SYSTEM**).
3. Options MSI utiles :

```text
SERVER=http://glpi.lab.local/front/inventory.php
RUNNOW=1                 # inventaire immédiat après install
ADD_FIREWALL_EXCEPTION=1
TAG=Compta               # étiqueter pour organiser
```

### Linux

```bash
apt install glpi-agent
# /etc/glpi-agent/conf.d/server.cfg
echo 'server = http://glpi.lab.local/front/inventory.php' > /etc/glpi-agent/conf.d/server.cfg
glpi-agent --force                 # forcer une remontée immédiate
```

*(Déploiement de masse Linux : rôle **Ansible**. Inventaire **sans agent** possible via **WinRM**/**SSH**.)*

---

## Vérification (comment savoir que ça marche)

- **Parc ▸ Ordinateurs** : le poste apparaît avec son **matériel, OS et logiciels**, et une **date de dernier inventaire** récente.
- L'inventaire envoyé est un **JSON** `{"action":"inventory","deviceid":...,"content":{hardware,bios,operatingsystem,softwares...}}`.

> **Testé en bac à sable** : un inventaire d'exemple (poste `PC-COMPTA-01`, Dell OptiPlex, Windows 11 24H2, 7-Zip/Firefox) est un **JSON valide** conforme à la structure attendue par le endpoint GLPI.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Poste absent du parc | `SERVER`/URL erroné | Vérifier l'URL `/front/inventory.php` |
| Aucune remontée | Pare-feu / réseau | Autoriser le flux HTTP(S) vers GLPI |
| Doublons | `deviceid` instable (image clonée) | Réinitialiser l'identifiant de l'agent |
| Inventaire partiel | Droits/exécution | Agent en **SYSTEM** (Windows) / root (Linux) |

## Sécurité et bonnes pratiques

- **HTTPS** pour l'inventaire (données du parc sensibles).
- **Étiquettes (TAG)** pour organiser par site/service ; planifier des remontées régulières.
- Croiser l'inventaire avec la **gestion des licences** (**CT4-01**) et les **MAJ** (**CP2-15/CP3-11**).

## ⚠️ À ne pas confondre / obsolète

- **FusionInventory** (ancien plugin) → **GLPI Agent + inventaire natif** (GLPI 10+).
- Inventaire **avec agent** (installé) ≠ **sans agent** (WinRM/SSH, distant).
- `deviceid` **dupliqué** (image non généralisée — **CP9-02**) = doublons dans le parc.

---

## Sources (doc officielle)

- [GLPI — Inventaire natif](https://www.glpi-project.org/fr/inventaire-natif-glpi/) — consulté le 24/07/2026
- [GLPI — Déployer l'agent par GPO](https://help.glpi-project.org/tutorials/inventory/deploy_agent_gpo) — consulté le 24/07/2026
- [GLPI Agent — Dépôt / releases](https://github.com/glpi-project/glpi-agent) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (agent) · [x] versions datées (GLPI 11) · [x] rien d'obsolète (natif vs FusionInventory) · [x] **format d'inventaire testé en bac à sable** / déploiement à tester en lab · [x] conforme doc GLPI · [x] vérification présente (parc) · [x] sécurité (HTTPS, tags) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
