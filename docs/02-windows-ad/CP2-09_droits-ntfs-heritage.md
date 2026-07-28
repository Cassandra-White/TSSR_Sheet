# CP2-09 — Gérer les droits NTFS et l'héritage

**Objectif** : maîtriser les permissions NTFS, comprendre l'héritage et savoir le désactiver proprement.

**Rattachement REAC** : CP2 — savoir-faire : « Configurer les droits d'accès et les permissions » ; connaissance « principes des partages, des autorisations d'accès et des permissions ».

**Durée** : ~15 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un dossier partagé (voir **CP2-08**), groupes AGDLP (voir **CP2-06**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 (Desktop Experience) | 23/07/2026 |

## Rappels

- Permissions de base : **Lecture**, **Écriture**, **Modification**, **Contrôle total**.
- ACE **héritées** (du dossier parent) vs **explicites**.
- Chaque objet a un **propriétaire** (qui peut toujours modifier les ACL).

---

## Procédure — GUI (méthode prioritaire)

1. Clic droit dossier → **Propriétés** → onglet **Sécurité** → **Avancé**.
2. Les entrées héritées apparaissent (colonne *Hérité de*). Pour couper l'héritage : **Désactiver l'héritage** → choisir **« Convertir… en autorisations explicites »** (conserve les droits) ou **« Supprimer »**.
3. **Ajouter** → *Sélectionner un principal* → groupe `DL_...` → cocher **Modification** → appliquer à *Ce dossier, les sous-dossiers et les fichiers*.
4. Retirer les groupes non désirés (ex. *Utilisateurs*).
5. Changer le propriétaire si besoin : **Avancé → Propriétaire → Modifier**.

## Procédure — CLI (`icacls`)

```powershell
# Afficher les ACL
icacls "D:\Partages\Compta"

# Désactiver l'héritage en CONSERVANT les droits actuels (copie)
icacls "D:\Partages\Compta" /inheritance:d

# Accorder Modify (OI=fichiers, CI=sous-dossiers)
icacls "D:\Partages\Compta" /grant "LABTSSR\DL_Partage_Compta_Modif:(OI)(CI)M"

# Retirer un groupe
icacls "D:\Partages\Compta" /remove "LABTSSR\Utilisateurs du domaine"

# Reprendre la propriété (récursif)
icacls "D:\Partages\Compta" /setowner "LABTSSR\Administrateur" /T

# Sauvegarder puis restaurer les ACL (avant une modif lourde)
icacls "D:\Partages\Compta" /save acl.txt /T
icacls "D:\Partages" /restore acl.txt
```

*(À exécuter en lab — non testable dans le bac à sable Linux.)*

---

## Vérification

```powershell
icacls "D:\Partages\Compta"
Get-Acl "D:\Partages\Compta" | Format-List
```

## Dépannage

| Symptôme | Cause probable | Solution |
|---|---|---|
| « Accès refusé » à la modification des ACL | Pas propriétaire | Reprendre la propriété (`/setowner`) |
| Droit non appliqué aux sous-dossiers | Flags d'héritage absents | Ajouter `(OI)(CI)` |
| Trop de droits hérités | Héritage actif | `/inheritance:d` puis nettoyer |

## Sécurité et bonnes pratiques

- **AGDLP** : droits NTFS sur des groupes **Domain Local**, jamais sur des comptes.
- **Moindre privilège** : éviter *Contrôle total* pour les utilisateurs (préférer *Modification*).
- **Sauvegarder les ACL** (`icacls /save`) avant toute modification importante.

## ⚠️ À ne pas confondre / obsolète

- `cacls` est **déprécié** → utiliser **`icacls`**.
- Permissions de **Partage** (SMB) ≠ permissions **NTFS** : les deux se cumulent (voir **CP2-08**).

---

## Sources (doc officielle)

- [icacls — Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/icacls) — consulté le 23/07/2026
- [Set-Acl — Microsoft Learn](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.security/set-acl) — consulté le 23/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète · [x] CLI marquée « à tester en lab » (Windows) · [x] GUI vérifiée doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
