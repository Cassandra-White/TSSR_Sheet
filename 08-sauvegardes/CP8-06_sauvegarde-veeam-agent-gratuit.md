# CP8-06 — Sauvegarder postes/données avec Veeam Agent (gratuit)

**Objectif** : sauvegarder un poste/serveur avec **Veeam Agent** (édition gratuite), créer un **média de récupération** et restaurer (fichier, volume, **bare-metal**).

**Rattachement REAC** : CP8 « Sauvegardes et restaurations des éléments de l'infrastructure » — savoir-faire : sauvegarder/restaurer un poste.

**Durée** : ~25 min · **Niveau** : débutant/intermédiaire.

---

## Prérequis

- Un poste **Windows 11** / serveur Windows (ou Linux) et une **cible** (disque USB, dossier partagé/NAS).
- Une **clé USB** vierge pour le média de récupération.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Agent Windows (gratuit) | **Veeam Agent for Windows 13.0.3** | 24/07/2026 |
| Agent Linux (gratuit) | **Veeam Agent for Linux 13.0.2** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **Veeam Agent gratuit** sauvegarde des **postes/serveurs autonomes** (hors console centrale) vers un disque local/USB/NAS, avec restauration **fichier, volume ou machine complète**. Le **média de récupération** permet la restauration **bare-metal** sur un matériel neuf.

---

## Procédure — GUI (Veeam Agent for Windows)

1. **Installer** Veeam Agent for Windows (édition **Free**).
2. **Configure Backup** → choisir le **mode** :
   - **Entire computer** (recommandé, permet le bare-metal),
   - **Volume level** (volumes choisis),
   - **File level** (dossiers ciblés).
3. **Destination** : disque **local/USB**, **dossier partagé/NAS** (SMB) — idéalement **externe et tournant** (3-2-1).
4. **Schedule** : fréquence (ex. quotidienne) ; activer le **chiffrement** (mot de passe AES) si la cible sort du site.
5. **Créer le média de récupération** : *Recovery Media* → graver une **clé USB bootable** (à conserver en lieu sûr).

### Restaurer

6. **Fichier** : *Restore ▸ Individual files* → monte la sauvegarde → copier le fichier voulu.
7. **Volume** : *Restore ▸ Volumes*.
8. **Bare-metal** (machine HS) : démarrer sur le **média de récupération** → **Bare Metal Recovery** → choisir la sauvegarde → restaurer.

## Procédure — CLI (Veeam Agent for Linux)

```bash
veeamconfig ui                      # assistant texte de configuration
# Créer un job vers un dépôt local :
veeamconfig job create --name job1 --backupvolume --volume /dev/sda2 \
  --repoName Repo1
veeamconfig job start --name job1   # lancer la sauvegarde
veeamconfig session list            # suivre l'état des jobs
```

> Sous Linux, le média de récupération se génère aussi (`veeam` recovery ISO) pour la restauration bare-metal.

---

## Vérification (comment savoir que ça marche)

- Le job affiche **Success** ; la sauvegarde est visible sur la cible (fichiers `.vbk`/`.vib`).
- **Test** : restaurer un fichier dans un dossier temporaire et comparer.
- Le **média de récupération** démarre bien et **voit** la sauvegarde.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Job échoue vers le NAS | Droits SMB / chemin | Vérifier compte et partage (**CP2-08**) |
| Média ne trouve pas la sauvegarde | Pilotes stockage/réseau absents | Régénérer le média avec les pilotes du matériel |
| Cible pleine | Rétention trop longue | Ajuster la rétention / cible plus grande |
| Sauvegarde incohérente | VSS/instantané | Vérifier VSS (Windows) ; fermer les applis sensibles |

## Sécurité et bonnes pratiques

- **Chiffrer** la sauvegarde (mot de passe AES) si la cible est **externe/hors site**.
- Conserver le **média de récupération** et le mot de passe de chiffrement **en lieu sûr et séparé**.
- Respecter **3-2-1** (**CP8-01**) : au moins une copie **hors site**.
- **Tester** une restauration complète (**CP8-07**) — un média non testé peut manquer un pilote.

## ⚠️ À ne pas confondre / obsolète

- Édition **gratuite** (poste autonome) ≠ **Veeam Backup & Replication** (console centralisée, planification de parc).
- **File level** ne permet **pas** le bare-metal : choisir **Entire computer** pour une reprise complète.
- Un média de récupération **jamais testé** peut échouer le jour J (pilotes manquants).

---

## Sources (doc officielle)

- [Veeam — Agent for Microsoft Windows (User Guide)](https://helpcenter.veeam.com/docs/agentforwindows/userguide/overview.html) — consulté le 24/07/2026
- [Veeam — Agent for Linux (User Guide)](https://helpcenter.veeam.com/docs/agentforlinux/userguide/overview.html) — consulté le 24/07/2026
- [Veeam — Build Numbers (KB2683)](https://www.veeam.com/kb2683) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (Linux) · [x] versions datées (13.0.3 / 13.0.2) · [x] rien d'obsolète (Free vs B&R) · [x] procédure **à tester en lab** · [x] GUI conforme doc Veeam · [x] vérification présente · [x] sécurité (chiffrement, média sûr, 3-2-1) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
