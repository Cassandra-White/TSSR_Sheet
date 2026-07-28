# CP6-07 — Script Bash : gestion d'utilisateurs / rapport système

**Objectif** : écrire des scripts qui génèrent un rapport d'état du serveur et créent des utilisateurs en masse.

**Rattachement REAC** : CP6 « Automatiser des tâches à l'aide de scripts » — savoir-faire : automatiser l'exploitation Linux.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Debian 13 (**CP3-01**), bases Bash (**CP6-05**), droits root/sudo pour la création de comptes.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian / Bash | 13.6 « trixie » | 24/07/2026 |
| Script de rapport | **exécuté dans le bac à sable** | 24/07/2026 |

---

## Procédure — CLI

### 1. Rapport système (testé)

```bash
#!/usr/bin/env bash
set -euo pipefail
echo "=== Rapport $(date +%F_%T) — hôte $(hostname) ==="
echo "Uptime : $(uptime -p 2>/dev/null || uptime)"
echo "Disque / : $(df -h / | awk 'NR==2{print $3" / "$2" ("$5")"}')"
echo "Mémoire : $(free -h | awk '/Mem:/{print $3" / "$2}')"
echo "Comptes (uid>=1000) :"
getent passwd | awk -F: '$3>=1000 && $3<65000 {print "  "$1" (uid "$3")"}'
```

**Sortie obtenue (bac à sable) :**

```
=== Rapport 2026-07-24_18:12:16 — hôte claude ===
Uptime : up 42 minutes
Disque / : 5.6G / 9.6G (59%)
Mémoire : 195Mi / 3.8Gi
Comptes (uid>=1000) :
  ubuntu (uid 1000)
  ...
```

### 2. Création d'utilisateurs en masse (depuis un fichier)

Fichier `utilisateurs.txt` (`identifiant;groupe`) :

```text
bmartin;compta
adurand;it
```

Script (à lancer en **root**) :

```bash
#!/usr/bin/env bash
set -euo pipefail
while IFS=';' read -r user groupe; do
    [[ -z "$user" ]] && continue
    getent group "$groupe" >/dev/null || groupadd "$groupe"
    if id "$user" &>/dev/null; then
        echo "existe déjà : $user"
    else
        useradd -m -s /bin/bash -G "$groupe" "$user"
        echo "$user:ChangeMe!2026" | chpasswd
        chage -d 0 "$user"          # forcer le changement au 1er login
        echo "créé : $user ($groupe)"
    fi
done < utilisateurs.txt
```

---

## Vérification (comment savoir que ça marche)

```bash
./rapport.sh                        # le rapport s'affiche (testé)
getent passwd bmartin               # le compte existe
id bmartin                          # groupes de l'utilisateur
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| `useradd: Permission denied` | Pas root | Lancer en `sudo`/root |
| La dernière ligne du fichier est ignorée | Pas de saut de ligne final | Ajouter une ligne vide ou gérer `read` |
| Champs mal découpés | Mauvais séparateur | `IFS=';'` conforme au fichier |
| Comptes AD/LDAP absents de `/etc/passwd` | Source distante | Utiliser **`getent`** (pas `cat /etc/passwd`) |

## Sécurité et bonnes pratiques

- **Mot de passe temporaire + `chage -d 0`** (changement au 1er login) ; ne jamais laisser de mot de passe en clair durable.
- Le **rapport peut être sensible** : le stocker/transmettre de façon protégée.
- Créer les comptes avec le **minimum de privilèges** ; journaliser.

## ⚠️ À ne pas confondre / obsolète

- **`getent passwd`** interroge **toutes** les sources (locale **+ LDAP/AD**) ; **`cat /etc/passwd`** = local uniquement.
- `chpasswd` lit `user:motdepasse` sur l'entrée standard — pratique pour scripter, mais attention au **clair**.
- `IFS` contrôle le découpage de `read` : l'ajuster au format du fichier.

---

## Sources (doc officielle)

- [getent(1) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/manpages/getent.1.en.html) — consulté le 24/07/2026
- [useradd(8) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/passwd/useradd.8.en.html) — consulté le 24/07/2026
- [chpasswd(8) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/passwd/chpasswd.8.en.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées · [x] rien d'obsolète (`getent` vs `/etc/passwd`) · [x] **rapport exécuté dans le bac à sable** ; création à valider en lab (root) · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
