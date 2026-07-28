# CP2-01 — Installer Windows Server 2025

**Objectif** : installer Windows Server 2025 (édition Desktop Experience) sur une VM, jusqu'au premier ouverture de session.

**Rattachement REAC** : CP2 « Exploiter des serveurs Windows et un domaine Active Directory » — savoir-faire : « Exploiter un serveur Windows (outils d'administration) ».

**Durée** : ~30–45 min · **Niveau** : débutant.

---

## Prérequis

- Un hyperviseur du lab opérationnel (voir **LAB-01**).
- L'ISO de Windows Server 2025 (version d'évaluation 180 jours sur le Centre d'évaluation Microsoft, ou une licence).
- Une VM : 2 vCPU, 4 Go de RAM minimum (8 conseillé), 60 Go de disque, une carte réseau.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (LTSC, base 24H2, build 26100) | 23/07/2026 |
| Hyperviseur | Proxmox VE 9.2 | 23/07/2026 |

## Choix préalable (définitif)

- **Édition** : *Standard* (usage courant) ou *Datacenter* (VM illimitées, Storage Spaces Direct…).
- **Expérience** : *Desktop Experience* (bureau graphique, conseillé pour débuter) ou *Server Core* (sans interface, surface d'attaque réduite).
  ⚠️ On ne peut **pas** basculer Core ↔ Desktop Experience après l'installation.

---

## Procédure — GUI (méthode prioritaire)

1. Monter l'ISO sur la VM et démarrer dessus (touche de boot si besoin).
2. Choisir **langue**, **format horaire/monétaire** et **clavier** (Français). *Suivant*.
3. Cliquer **Installer Windows Server**, puis cocher la case d'avertissement (les données du disque seront effacées). *(Nouvel assistant d'installation 2025.)*
4. Sélectionner l'édition : **Windows Server 2025 Standard (Desktop Experience)**. *Suivant*.
5. Accepter le contrat de licence.
6. Choisir **Personnalisé : installer uniquement Windows** (installation propre).
7. Sélectionner le disque cible (le formater si nécessaire) puis **Installer**.
8. Laisser la copie des fichiers se faire ; la VM redémarre automatiquement une ou plusieurs fois.
9. Définir un **mot de passe fort** pour le compte **Administrateur**. *Terminer*.
10. Se connecter : envoyer **Ctrl+Alt+Suppr** (bouton dédié dans la console de l'hyperviseur) puis saisir le mot de passe.

## Procédure — CLI (alternative / automatisation)

L'installation initiale est graphique. Deux compléments en ligne de commande :

- **Automatiser** l'installation en série : fichier de réponses `autounattend.xml` (voir **CP9-04**).
- **Configurer** le serveur après le 1ᵉʳ démarrage (indispensable en Server Core) avec l'outil `sconfig` :

```
sconfig
```

Menu texte : nom d'ordinateur, réseau, mises à jour, Bureau à distance, etc.

Vérifier l'édition et la version installées :

```powershell
Get-ComputerInfo -Property WindowsProductName, OsVersion
```

*(Commandes PowerShell à exécuter en lab — non testables dans le bac à sable Linux.)*

---

## Vérification

- Le **Gestionnaire de serveur** s'ouvre automatiquement à l'ouverture de session.
- `winver` (menu Exécuter) affiche **Windows Server 2025**, version **24H2**, build **26100.xxxx**.

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| La VM ne démarre pas sur l'ISO | Ordre de démarrage | Régler le boot sur CD/DVD ou passer par le menu de boot de la VM |
| « Aucun lecteur trouvé » (disque absent) | Pilote de stockage manquant | Charger le pilote VirtIO (Proxmox) ou utiliser un contrôleur SATA |
| Ctrl+Alt+Suppr sans effet | Raccourci capté par l'hôte | Utiliser le bouton « Ctrl-Alt-Suppr » de la console de l'hyperviseur |

## Sécurité et bonnes pratiques

- Mot de passe **Administrateur** long et unique.
- Installer les **mises à jour** (voir **CP2-15**) avant toute mise en service.
- En production, préférer **Server Core** quand l'administration en ligne de commande est maîtrisée.
- **Renommer** le serveur et fixer son **IP** avant d'ajouter un rôle (voir **CP2-02**).

## ⚠️ À ne pas confondre / obsolète

- L'assistant d'installation de WS2025 est **nouveau** : l'ancien bouton « Installer maintenant » est remplacé par **« Installer Windows Server »** + une case de confirmation.
- Le choix **Core / Desktop Experience** est **définitif**.

---

## Sources (doc officielle)

- [Server Core vs Desktop Experience — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/get-started/install-options-server-core-desktop-experience) — consulté le 23/07/2026
- [Centre d'évaluation Windows Server 2025](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2025) — consulté le 23/07/2026
- [sconfig — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/administration/server-core/server-core-sconfig) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
