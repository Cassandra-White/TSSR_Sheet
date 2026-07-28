# CP9-10 — Déployer des logiciels avec winget / Chocolatey

**Objectif** : installer, mettre à jour et déployer des logiciels **en ligne de commande** avec **winget** et **Chocolatey**, en lot (**export/import**) et en entreprise (dépôt interne).

**Rattachement REAC** : CP9 « Exploiter et maintenir les services de déploiement des postes » — savoir-faire : automatiser l'installation d'applications.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- **Windows 11 24H2** (**winget** est intégré via *App Installer*).
- Accès Internet (ou un **dépôt interne** en entreprise), droits admin pour les installations machine.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Gestionnaire intégré | **winget** (Windows Package Manager) | 24/07/2026 |
| Gestionnaire tiers | **Chocolatey** | 24/07/2026 |
| Manifeste `winget export` (JSON) | **validé en bac à sable** (schéma 2.0) | 24/07/2026 |

> **Gestionnaires de paquets Windows** (comme `apt` sous Linux) : installer/mettre à jour depuis un dépôt **de confiance**, en une commande, scriptable. **winget** = intégré Windows ; **Chocolatey** = plus mature en **entreprise** (dépôts privés, internalisation hors-ligne).

---

## Procédure — CLI

### winget

```powershell
winget search firefox
winget install --id Mozilla.Firefox -e --silent          # -e = correspondance exacte
winget upgrade --all                                      # tout mettre à jour
winget list                                               # inventaire installé

# Déploiement en LOT (poste de référence -> autres postes)
winget export -o apps.json                                # exporter la liste
winget import apps.json --accept-package-agreements       # réinstaller en lot

# État désiré (Desired State Configuration)
winget configure config.dsc.yaml
```

### Chocolatey

```powershell
# Installation de Chocolatey (script officiel, à récupérer depuis chocolatey.org)
choco install firefox -y
choco upgrade all -y
choco list

# Entreprise : pointer un dépôt PRIVÉ (Nexus/ProGet) et internaliser les paquets
choco source add -n interne -s "https://repo.lab.local/choco"
```

> **Exemple testé** : un fichier `winget export` (schéma `winget-packages.schema.2.0.json`) listant `7zip.7zip`, `Mozilla.Firefox`, `Notepad++.Notepad++` est un **JSON valide réimportable** par `winget import` (vérifié en bac à sable).

---

## Vérification (comment savoir que ça marche)

```powershell
winget list --id Mozilla.Firefox     # l'application apparaît installée
choco list | findstr firefox
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| `winget` introuvable | *App Installer* absent/ancien | Installer/mettre à jour *App Installer* (Store) |
| Invite d'acceptation de source | 1ʳᵉ utilisation | `--accept-source-agreements` / `--accept-package-agreements` |
| Installation non silencieuse | Switch manquant | `--silent` (winget) / `-y` (choco) |
| Paquet Store absent de l'export | Limite `winget export` | Compléter manuellement / Intune |

## Sécurité et bonnes pratiques

- **Sources de confiance uniquement** ; en entreprise, **dépôt interne** (Chocolatey internalisé / winget via **Intune** — **CP9-09**) pour ne pas tirer d'Internet sans contrôle (rappel anti `curl|bash` — **CP6-10**).
- **Épingler/valider les versions** et **auditer** les paquets déployés.
- Réserver les installations **machine** aux comptes admin ; journaliser.

## ⚠️ À ne pas confondre / obsolète

- **winget** (intégré, grand public/moderne) vs **Chocolatey** (entreprise, dépôts privés) vs **Scoop** (portable, utilisateur).
- `winget import` **réinstalle** une liste ; il ne **synchronise pas** les versions comme une vraie GPO/Intune.
- Ne pas **internaliser** en environnement sensible = risque d'approvisionnement non maîtrisé.

---

## Sources (doc officielle)

- [Microsoft Learn — Windows Package Manager (winget)](https://learn.microsoft.com/en-us/windows/package-manager/winget/) — consulté le 24/07/2026
- [Microsoft Learn — winget export / import](https://learn.microsoft.com/en-us/windows/package-manager/winget/export) — consulté le 24/07/2026
- [Chocolatey — Documentation](https://docs.chocolatey.org/en-us/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI (winget + Chocolatey) · [x] versions datées · [x] rien d'obsolète (dépôt interne, comparaison) · [x] **manifeste export testé en bac à sable** / installs à tester en lab · [x] conforme doc Microsoft/Chocolatey · [x] vérification présente (`winget list`) · [x] sécurité (sources de confiance, internalisation) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
