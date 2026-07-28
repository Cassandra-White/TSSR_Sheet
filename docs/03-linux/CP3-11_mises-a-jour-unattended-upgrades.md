# CP3-11 — Gérer les mises à jour et correctifs (apt, unattended-upgrades)

**Objectif** : mettre à jour un serveur Debian manuellement, puis **automatiser les correctifs de sécurité** avec `unattended-upgrades` et `needrestart`.

**Rattachement REAC** : CP3 « Exploiter des serveurs Linux » — savoir-faire : maintenir un serveur Linux à jour et sécurisé.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Debian 13 (**CP3-01**), accès réseau, droits root/sudo. APT/dépôts opérationnels (**CP3-05**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian / APT | 13.6 « trixie » — APT 3.0 | 24/07/2026 |
| `unattended-upgrades` / `needrestart` | présence vérifiée dans le bac à sable ; config **à tester en lab** | 24/07/2026 |

---

## Procédure — CLI

### Mise à jour manuelle

```bash
sudo apt update              # rafraîchir l'index
apt list --upgradable        # voir ce qui peut être mis à jour
sudo apt upgrade             # appliquer (sans retirer de paquets)
sudo apt full-upgrade        # + gérer les changements de dépendances
sudo apt autoremove --purge  # nettoyer les paquets/noyaux orphelins
```

### Correctifs de sécurité automatiques

```bash
sudo apt install unattended-upgrades apt-listchanges needrestart
sudo dpkg-reconfigure -plow unattended-upgrades   # crée /etc/apt/apt.conf.d/20auto-upgrades
```

`/etc/apt/apt.conf.d/20auto-upgrades` :

```text
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

Options utiles dans `/etc/apt/apt.conf.d/50unattended-upgrades` (limiter aux **origines de sécurité**) :

```text
Unattended-Upgrade::Mail "admin@exemple.fr";
Unattended-Upgrade::MailReport "only-on-error";
Unattended-Upgrade::Automatic-Reboot "false";
Unattended-Upgrade::Automatic-Reboot-Time "03:30";
```

Redémarrage des services impactés — `needrestart` (`/etc/needrestart/needrestart.conf`) :

```perl
# 'i' = interactif (demande) ; 'a' = redémarrage automatique des services
$nrconf{restart} = 'a';
```

Tester sans rien installer :

```bash
sudo unattended-upgrade --dry-run --debug
```

---

## Vérification (comment savoir que ça marche)

```bash
systemctl status unattended-upgrades
apt-config dump | grep -i periodic          # doit montrer Update-Package-Lists "1" / Unattended-Upgrade "1"
sudo cat /var/log/unattended-upgrades/unattended-upgrades.log
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Aucune MAJ auto appliquée | `20auto-upgrades` absent ou à `"0"` | Le (re)créer via `dpkg-reconfigure` |
| Reboot intempestif la nuit | `Automatic-Reboot "true"` | Passer à `"false"` (ou fixer une heure maîtrisée) |
| Services non redémarrés après MAJ de lib | `needrestart` en interactif | Mode `'a'` dans `needrestart.conf` |
| `/boot` plein (anciens noyaux) | Noyaux non purgés | `sudo apt autoremove --purge` |

## Sécurité et bonnes pratiques

- **Limiter l'automatique aux correctifs de sécurité** ; garder les montées de version fonctionnelles sous contrôle.
- Activer les **notifications mail** (`MailReport "only-on-error"`).
- Maîtriser la **fenêtre de redémarrage** ; tester d'abord sur un serveur non critique.

## ⚠️ À ne pas confondre / obsolète

- `apt upgrade` **n'installe/ne retire pas** de paquets pour satisfaire les dépendances → utiliser **`full-upgrade`** au besoin.
- Ne pas confondre une **mise à jour de paquets** (courante) avec une **montée de version majeure** de Debian (12→13, procédure dédiée).
- `needrestart` en mode interactif peut **bloquer un script/automatisation** → mode `'a'` sur un serveur autonome.

---

## Sources (doc officielle)

- [Debian Wiki — UnattendedUpgrades](https://wiki.debian.org/UnattendedUpgrades) — consulté le 24/07/2026
- [Debian Wiki — PeriodicUpdates](https://wiki.debian.org/PeriodicUpdates) — consulté le 24/07/2026
- [unattended-upgrade(8) — Debian Manpages (trixie)](https://manpages.debian.org/trixie/unattended-upgrades/unattended-upgrade.8.en.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] versions datées · [x] rien d'obsolète (full-upgrade, needrestart) · [x] présence vérifiée, config à tester en lab · [x] conforme doc Debian · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
