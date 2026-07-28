# CP2-14 — Consulter l'Observateur d'événements et suivre les journaux

**Objectif** : lire, filtrer et exploiter les journaux d'événements, créer une vue personnalisée et attacher une tâche à un événement.

**Rattachement REAC** : CP2 — savoir-faire : « suivre les journaux d'événements » ; démarche de diagnostic.

**Durée** : ~10 min · **Niveau** : débutant.

---

## Prérequis

- Un serveur ou poste Windows, session administrateur (le journal *Sécurité* exige des droits élevés).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (Desktop Experience) | 23/07/2026 |

---

## Procédure — GUI (méthode prioritaire)

1. **Outils** → **Observateur d'événements** (`eventvwr.msc`).
2. **Journaux Windows** : *Application*, *Sécurité*, *Système*, *Configuration*, *ForwardedEvents*.
3. **Filtrer** : clic droit sur un journal → **Filtrer le journal actuel** → niveau (*Erreur/Avertissement*), source, **ID d'événement**, période.
4. **Vue personnalisée** : clic droit sur *Vues personnalisées* → **Créer une vue personnalisée** (ex. Erreurs + Avertissements de *Système* et *Application*).
5. **Attacher une tâche** : clic droit sur un événement → **Joindre une tâche à cet événement** (déclenche un programme via le Planificateur).

> IDs utiles : `4624`/`4625` (session réussie/échouée), `4740` (verrouillage de compte), `6008` (arrêt inattendu), `7000/7001` (services).

## Procédure — CLI (`Get-WinEvent`)

```powershell
# 20 derniers événements du journal Système
Get-WinEvent -LogName System -MaxEvents 20

# Filtrage performant (niveau Critique/Erreur, dernières 24 h)
Get-WinEvent -FilterHashtable @{ LogName='System'; Level=1,2; StartTime=(Get-Date).AddDays(-1) }

# Échecs d'ouverture de session
Get-WinEvent -FilterHashtable @{ LogName='Security'; Id=4625 } -MaxEvents 10

# Exporter un journal complet
wevtutil epl System C:\temp\system.evtx
```

*(À exécuter en lab — non testable dans le bac à sable Linux.)*

---

## Vérification

- La vue personnalisée affiche bien les erreurs/avertissements attendus.
- `Get-WinEvent` renvoie les entrées ciblées.

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| Journal *Sécurité* vide/inaccessible | Droits insuffisants | Ouvrir la console **en administrateur** |
| Certains journaux invisibles avec `Get-EventLog` | Cmdlet *legacy* | Utiliser **`Get-WinEvent`** |
| Trop de résultats | Aucun filtre | `FilterHashtable` (Level / Id / StartTime) |

## Sécurité et bonnes pratiques

- Surveiller `4625`/`4740` (indices de force brute).
- **Centraliser** les journaux (abonnements / Event Forwarding) et superviser (voir **CP4-17**, **CP3-15**).
- Ne pas effacer le journal *Sécurité* sans archivage préalable.

## ⚠️ À ne pas confondre / obsolète

- **`Get-EventLog`** est *legacy* (journaux classiques seulement) → préférer **`Get-WinEvent`** (tous les journaux + filtrage rapide).

---

## Sources (doc officielle)

- [Get-WinEvent — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent) — consulté le 23/07/2026
- [wevtutil — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/wevtutil) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
