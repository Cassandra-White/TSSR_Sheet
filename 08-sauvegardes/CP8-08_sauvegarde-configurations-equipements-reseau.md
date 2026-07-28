# CP8-08 — Sauvegarder les configurations des équipements réseau (switch/firewall)

**Objectif** : exporter et **versionner** les configurations des équipements réseau (commutateurs, pare-feu), manuellement puis **automatiquement**, pour pouvoir les restaurer vite.

**Rattachement REAC** : CP8 « Sauvegardes et restaurations des éléments de l'infrastructure » — savoir-faire : sauvegarder les configurations des équipements.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Accès administrateur aux équipements (**CP4-03**, **CP7-01**).
- Un dépôt sûr pour les fichiers de configuration (serveur SCP + **Git** conseillé).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Commutateur | syntaxe **Cisco IOS** (référence) | 24/07/2026 |
| Pare-feu | pfSense CE **2.8.1** | 24/07/2026 |
| Automatisation | **Oxidized** (backend Git) | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **La config est aussi une donnée à sauvegarder.** Un switch/pare-feu qui meurt se remplace en minutes **si** on a sa configuration ; sinon, c'est une reconstruction manuelle longue et risquée.

---

## Procédure — GUI (pfSense)

1. **Diagnostics ▸ Backup & Restore** → **Download configuration as XML** : télécharge le **`config.xml`** (toute la configuration).
2. **AutoConfigBackup (ACB)** : **Services ▸ Auto Config Backup** → activer → à chaque changement, pfSense **chiffre** la config (passphrase) et l'envoie **hors site** (Netgate) en HTTPS.
3. **Restauration** : **Backup & Restore ▸ Restore** → sélectionner un `config.xml` → reboot.

## Procédure — CLI (Cisco IOS) et automatisation

### Export manuel Cisco

```text
show running-config                     ! afficher la config active
copy running-config startup-config      ! (write memory) la rendre persistante
copy running-config scp:                ! exporter vers un serveur SCP (chiffré)
```

### Restauration Cisco

```text
copy scp: running-config                ! réinjecter une config sauvegardée
configure replace scp:<fichier>         ! remplacement atomique (rollback possible)
```

### Automatisation — Oxidized + Git (recommandé)

- **Oxidized** (remplaçant moderne de **RANCID**, 130+ OS dont **pfSense**) se connecte périodiquement en SSH, récupère la config et **commit dans Git** à **chaque changement** → historique complet + **diff** entre versions.

```yaml
# extrait oxidized config : un nœud à sauvegarder
nodes:
  - name: sw-core-01
    model: ios
    username: backup
    password: <secret>
output:
  default: git          # backend Git recommandé (versioning)
```

---

## Vérification (comment savoir que ça marche)

- Le fichier de configuration est présent et **complet** (taille cohérente) dans le dépôt.
- Sous Git/Oxidized : `git log` montre un **commit à chaque changement** ; `git diff` montre les modifications.
- **Test** : restaurer la config sur un équipement de lab → il redémarre avec les mêmes VLAN/règles.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Transfert refusé | **TFTP** bloqué / non chiffré | Utiliser **SCP** (chiffré) plutôt que TFTP |
| Config restaurée incomplète | `startup-config` non sauvegardée | `copy run start` avant export |
| Secrets en clair dans le dépôt | Config contient mots de passe/clés | Dépôt **chiffré/restreint**, ACB avec passphrase |
| Oxidized ne collecte pas | Modèle/identifiants faux | Vérifier `model` et le compte de collecte |

## Sécurité et bonnes pratiques

- Les configs contiennent des **secrets** (mots de passe, clés VPN) → **accès restreint**, **chiffrement**, dépôt **hors site** (3-2-1 — **CP8-01**).
- **Compte de collecte dédié** en lecture, journalisé (**CP7-10**).
- **Versionner** (Git) : traçabilité des changements + **rollback** rapide.
- Sauvegarder **après chaque modification** validée (ou automatiser).

## ⚠️ À ne pas confondre / obsolète

- **TFTP/HTTP** (clair) → **SCP/HTTPS** (chiffré).
- **`running-config`** (active, volatile) ≠ **`startup-config`** (persistante) : sauvegarder la bonne.
- **RANCID** (ancien) → **Oxidized** (Git, moderne) pour l'automatisation.

---

## Sources (doc officielle)

- [pfSense — Backup and Recovery / AutoConfigBackup](https://docs.netgate.com/pfsense/en/latest/backup/index.html) — consulté le 24/07/2026
- [Oxidized — Projet (sauvegarde config réseau, Git)](https://github.com/ytti/oxidized) — consulté le 24/07/2026
- [Cisco — Managing Configuration Files](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/fundamentals/configuration/15mt/fundamentals-15-mt-book/cf-config-files.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI (pfSense) puis CLI (IOS/Oxidized) · [x] versions datées · [x] rien d'obsolète (SCP vs TFTP, Oxidized vs RANCID) · [x] procédure **à tester en lab** · [x] GUI conforme doc Netgate · [x] vérification présente (git diff) · [x] sécurité (secrets, hors site) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
