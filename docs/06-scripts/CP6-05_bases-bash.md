# CP6-05 — Bases du scripting Bash (variables, conditions, boucles)

**Objectif** : écrire un script Bash robuste utilisant variables, conditions, boucles et fonctions.

**Rattachement REAC** : CP6 « Automatiser des tâches à l'aide de scripts » — savoir-faire : maîtriser les bases du scripting Linux.

**Durée** : ~20 min · **Niveau** : débutant.

---

## Prérequis

- Debian 13 (**CP3-01**), un éditeur (`nano`/`vim`).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian / Bash | 13.6 « trixie » | 24/07/2026 |
| Script d'exemple | **exécuté dans le bac à sable** | 24/07/2026 |

---

## Procédure — CLI

### Squelette d'un script

```bash
#!/usr/bin/env bash
set -euo pipefail          # -e: stop si erreur · -u: variable non définie = erreur · pipefail
```

```bash
chmod +x monscript.sh      # rendre exécutable
./monscript.sh             # exécuter
```

### Script d'exemple (testé)

```bash
#!/usr/bin/env bash
set -euo pipefail
nom="Bob"; nb=3
echo "Bonjour $nom (nb=$nb)"
if [[ $nb -gt 2 ]]; then echo "nb > 2"; else echo "nb <= 2"; fi
for i in 1 2 3; do echo "  iteration $i"; done
services=("ssh" "cron" "nginx")
for s in "${services[@]}"; do echo "  service: $s"; done
saluer() { echo "Salut $1"; }
saluer "Alice"
echo "date: $(date +%F) | calcul: $((2 + 3 * 4))"
```

**Sortie obtenue (bac à sable) :**

```
Bonjour Bob (nb=3)
nb > 2
  iteration 1
  iteration 2
  iteration 3
  service: ssh
  service: cron
  service: nginx
Salut Alice
date: 2026-07-24 | calcul: 14
```

### Points de syntaxe

- **Variables** : `var="valeur"` (pas d'espaces autour du `=`), usage `"$var"` (toujours entre guillemets).
- **Tests** `[[ … ]]` : `-eq -ne -gt -lt` (nombres), `=`/`!=` (chaînes), `-z`/`-n` (vide), `-f`/`-d` (fichier/dossier).
- **Substitution** : `$(commande)` ; **arithmétique** : `$(( … ))`.
- **Arguments** : `$1 $2 …`, `$#` (nombre), `$@` (tous).

---

## Vérification (comment savoir que ça marche)

```bash
bash -n monscript.sh       # vérifier la SYNTAXE sans exécuter
shellcheck monscript.sh    # linter (bonnes pratiques) — paquet "shellcheck"
./monscript.sh             # exécuter
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| `Permission denied` | Script non exécutable | `chmod +x script.sh` |
| Comportement bizarre avec des espaces | Variable non quotée | Toujours `"$var"` |
| `bad interpreter: ^M` | Fins de ligne Windows (CRLF) | `dos2unix script.sh` |
| Erreur silencieuse ignorée | Pas de `set -e` | Ajouter `set -euo pipefail` |

## Sécurité et bonnes pratiques

- **`set -euo pipefail`** en tête : un script qui échoue **s'arrête** au lieu de continuer à l'aveugle.
- **Quoter** systématiquement les variables ; **valider** les entrées.
- Éviter **`eval`** ; ne pas exécuter d'entrée non fiable.

## ⚠️ À ne pas confondre / obsolète

- **`[[ … ]]`** (Bash, plus sûr) préférable à **`[ … ]`** (POSIX) dans un script Bash.
- **`$(…)`** remplace les **backticks** `` `…` `` (obsolètes, non imbriquables).
- Comparaison **numérique** `-eq` ≠ comparaison **de chaînes** `=`.

---

## Sources (doc officielle)

- [GNU Bash — Manuel de référence](https://www.gnu.org/software/bash/manual/bash.html) — consulté le 24/07/2026
- [ShellCheck — analyse statique de scripts shell](https://www.shellcheck.net/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées · [x] rien d'obsolète (`[[ ]]`, `$()`) · [x] **script exécuté dans le bac à sable** · [x] conforme doc · [x] vérification présente · [x] sécurité (`set -euo pipefail`) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
