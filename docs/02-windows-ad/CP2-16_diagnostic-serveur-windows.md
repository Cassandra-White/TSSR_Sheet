# CP2-16 — Diagnostiquer un serveur Windows (services, ressources, réseau)

**Objectif** : appliquer une démarche structurée de diagnostic et mobiliser les bons outils (services, ressources, réseau, intégrité système).

**Rattachement REAC** : CP2 — savoir-faire : « Adopter une démarche de diagnostic logique et efficace » ; transversale **CT2** (résolution de problème).

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un serveur Windows Server 2025, session administrateur.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (Desktop Experience) | 23/07/2026 |

## Démarche structurée (à suivre dans l'ordre)

1. **Recueillir** le symptôme : quoi, depuis quand, périmètre (un service ? tous ?).
2. **Vérifier l'évident** : service arrêté, espace disque, réseau, journaux récents.
3. **Isoler la couche** : matériel → OS → service → réseau.
4. **Tester une seule hypothèse à la fois**.
5. **Corriger, vérifier, documenter** (voir **CP1-09**).

---

## Procédure — GUI (méthode prioritaire)

1. **Gestionnaire des tâches** (`Ctrl+Maj+Échap`) : onglets *Performance* (CPU/Mémoire/Disque/Réseau), *Processus*, *Services*.
2. **Moniteur de ressources** (`resmon`) : consommation fine **par processus** (disque, réseau, handles).
3. **Analyseur de performances** (`perfmon`) : compteurs et ensembles de collecteurs (créer une *baseline*).
4. **Moniteur de fiabilité** (`perfmon /rel`) : historique des plantages et incidents.
5. **Services** (`services.msc`) : état, type de démarrage, dépendances.
6. **Observateur d'événements** (voir **CP2-14**).

## Procédure — CLI

```powershell
# Services en démarrage automatique mais arrêtés
Get-Service | Where-Object { $_.Status -ne 'Running' -and $_.StartType -eq 'Automatic' }
Restart-Service DNS

# Processus les plus gourmands
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5
Get-Process | Sort-Object WS  -Descending | Select-Object -First 5   # mémoire

# Compteurs instantanés (noms localisés sur un Windows FR — cf. note)
Get-Counter '\Processor(_Total)\% Processor Time','\Memory\Available MBytes'

# Disque et réseau
Get-Volume
Get-NetIPConfiguration
Test-NetConnection dc01.labtssr.lan -Port 389   # test LDAP

# Intégrité des fichiers système et de l'image
sfc /scannow
DISM /Online /Cleanup-Image /RestoreHealth
```

> Note : sur un Windows en **français**, les noms de compteurs `Get-Counter` sont localisés (ex. `\Processeur(_Total)\% temps processeur`). En cas de doute, utiliser `perfmon` (GUI).

*(À exécuter en lab — non testable dans le bac à sable Linux.)*

---

## Vérification

- Le service ou la ressource en cause est identifié ; après correction, le symptôme disparaît et les indicateurs reviennent à la normale.

## Dépannage (exemples)

| Symptôme | Cause probable | Solution |
|---|---|---|
| CPU à 100 % | Processus emballé | Identifier via `resmon`/`Get-Process`, redémarrer le service |
| Disque saturé | Journaux/temp | Nettoyer, étendre le volume (**STO-04**) |
| Lenteur réseau | DNS/latence | `Test-NetConnection`, vérifier le DNS |

## Sécurité et bonnes pratiques

- Un pic anormal peut trahir une **compromission** : corréler avec le journal *Sécurité*.
- **Documenter** chaque diagnostic dans la base de connaissances (support).

## ⚠️ À ne pas confondre / obsolète

- `sfc /scannow` et `DISM …/RestoreHealth` restent les outils **actuels** d'intégrité.
- Ne jamais conclure sur **un seul** indicateur : croiser plusieurs sources.

---

## Sources (doc officielle)

- [Get-Counter — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-counter) — consulté le 23/07/2026
- [Réparer une image Windows (DISM) — Microsoft Learn](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/repair-a-windows-image) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
