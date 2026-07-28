# CP1-13 — Diagnostiquer un poste utilisateur (matériel/logiciel) — démarche structurée

**Objectif** : diagnostiquer un poste Windows avec une **démarche structurée** et les **outils intégrés**, en isolant **matériel** vs **logiciel**.

**Rattachement REAC** : CP1 « Assurer le support utilisateur » — savoir-faire : diagnostiquer un poste.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un poste **Windows 11 24H2** et des droits **administrateur**.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| OS | **Windows 11 24H2** | 24/07/2026 |
| Outils | Observateur d'événements, `sfc`/`DISM`, diag. mémoire | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **Démarche** (rappel **CT2-01 / CP1-08**) : **cadrer** (QQOQCP) → **isoler** matériel vs logiciel → tester **une hypothèse à la fois**, du plus **simple/probable** au plus complexe.

---

## Procédure — GUI (outils Windows)

1. **Moniteur de fiabilité** + **Observateur d'événements** (`eventvwr`) : **par où commencer** — historique des plantages, erreurs pilotes/disque/service.
2. **Gestionnaire des tâches** (CPU/RAM/disque), **Gestionnaire de périphériques** (pilotes/matériel en défaut), **`msinfo32`** (configuration).
3. **Sécurité Windows ▸ Performances et intégrité de l'appareil**.
4. **Diagnostic de mémoire Windows** (`mdsched`) si suspicion de RAM.

## Procédure — CLI (réparer le système)

```cmd
sfc /scannow                                    :: vérifie/répare les fichiers système
DISM /Online /Cleanup-Image /RestoreHealth      :: répare l'image Windows (si SFC échoue)
chkdsk C: /f                                     :: vérifie/répare le disque (au reboot)
```

> **Ordre conseillé** : Fiabilité/Événements → **SFC** → **DISM** → **mémoire** → **disque** → **batterie** (portable).

## Isoler une panne matérielle

- **RAM** : `mdsched` / **memtest** ; **disque** : **SMART** (**STO-09**) ; **surchauffe** : températures/ventilation.
- **Démarrage minimal** / **mode sans échec** pour écarter un logiciel/pilote.

---

## Vérification (comment savoir que c'est résolu)

- La **cause** est identifiée (matériel ou logiciel), le **correctif** est appliqué, le poste est **stable** dans le temps.
- Le journal (Fiabilité/Événements) ne remonte plus l'erreur.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| `sfc` ne corrige pas tout | Image Windows altérée | Lancer **DISM** puis re-`sfc` |
| Redémarrages en boucle | Pilote/logiciel | **Mode sans échec**, désinstaller/rollback |
| Plantages aléatoires | **RAM** défectueuse | `mdsched`/memtest → remplacer le module |
| Lenteurs/à-coups | Disque en fin de vie | **SMART** (**STO-09**) → remplacer |

## Sécurité et bonnes pratiques

- **Sauvegarder** les données avant toute manipulation à risque (**CP8**).
- **Un changement à la fois** et le **documenter** (**CP1-09**) pour tracer ce qui a résolu.
- Écarter d'abord les causes **simples** (câble, alim, MAJ) avant de conclure à une panne lourde.

## ⚠️ À ne pas confondre / obsolète

- **Réinstaller d'emblée** ≠ **diagnostiquer** : on cherche la **cause** d'abord.
- **SFC** (fichiers système) puis **DISM** (image) : ordre logique.
- **Symptôme** (ce qu'on voit) ≠ **cause** (**CT2-01**).

---

## Sources (doc officielle)

- [Microsoft Learn — sfc / DISM (réparation)](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/repair-a-windows-image) — consulté le 24/07/2026
- [Microsoft — Observateur d'événements / Moniteur de fiabilité](https://support.microsoft.com/windows) — consulté le 24/07/2026
- [Microsoft — Diagnostic de mémoire Windows](https://support.microsoft.com/windows) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (`sfc`/`DISM`) · [x] version datée (Win 11 24H2) · [x] rien d'obsolète (diagnostiquer avant réinstaller) · [x] procédure **à tester en lab** · [x] conforme doc Microsoft · [x] vérification présente · [x] sécurité (sauvegarde, un changement à la fois) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
