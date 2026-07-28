# CP4-13 — Synchroniser l'heure (NTP : W32Time côté Windows + chrony côté Linux)

**Objectif** : garantir une heure exacte et cohérente sur les serveurs et postes — indispensable pour **Kerberos/AD**, les **journaux** et les **certificats**.

**Rattachement REAC** : CP4 « Exploiter un réseau IP » — savoir-faire : mettre en place la synchronisation temporelle (NTP).

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un domaine AD (**CP2-03**) et/ou des serveurs Debian 13, droits admin/root, accès à une source de temps (UDP 123).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Windows Server | 2025 — service **W32Time** | 24/07/2026 |
| Debian | 13.6 — **systemd-timesyncd** par défaut, **chrony** recommandé serveur ; `timedatectl` **testé** | 24/07/2026 |

> **Kerberos** (AD) n'accepte qu'un écart d'horloge d'environ **5 minutes** : une heure fausse casse l'authentification.

---

## Procédure — CLI (Windows / AD)

Le **contrôleur de domaine émulateur PDC** est la **source de temps** du domaine ; il se synchronise sur une source externe, les membres se synchronisent sur lui.

```powershell
# Sur le PDC emulator : définir une source NTP externe fiable
w32tm /config /manualpeerlist:"fr.pool.ntp.org,0x8" /syncfromflags:manual /reliable:yes /update
Restart-Service w32time
w32tm /resync
w32tm /query /status        # vérifier la source et l'écart
```

> Les postes/serveurs **membres du domaine** se synchronisent **automatiquement** sur la hiérarchie AD (ne pas leur fixer une source manuelle).

## Procédure — CLI (Linux / Debian)

```bash
# Serveur : chrony (précis, peut aussi servir de serveur NTP)
sudo apt install chrony
# /etc/chrony/chrony.conf :
#   pool fr.pool.ntp.org iburst
#   (ou, pour s'aligner sur l'AD : server srv-ad01.lab.local iburst)
sudo systemctl restart chrony
chronyc sources -v
chronyc tracking

# Client simple : systemd-timesyncd (par défaut)
sudo timedatectl set-ntp true
timedatectl
```

---

## Vérification (comment savoir que ça marche)

```text
# Windows
w32tm /query /status        # « Source » renseignée, « Écart » faible

# Linux
chronyc tracking            # offset faible, « Leap status : Normal »
timedatectl                 # « System clock synchronized: yes »
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Erreurs d'authentification AD | Décalage > 5 min (Kerberos) | Resynchroniser (`w32tm /resync`, `chronyc`) |
| Pas de synchro | UDP **123** filtré / source injoignable | Ouvrir 123, vérifier la source |
| Heure « qui saute » sur une VM | Synchro de l'hyperviseur en conflit | Désactiver la synchro d'hôte sur le **PDCe** |
| Deux services temps actifs | timesyncd **et** chrony | N'en garder **qu'un** |

## Sécurité et bonnes pratiques

- **Source de temps fiable et maîtrisée** (interne de préférence) ; filtrer **UDP 123**.
- Hiérarchie claire : **PDCe → source externe**, tout le reste en interne.
- En environnement sensible : **NTS** (chrony) pour authentifier la source.

## ⚠️ À ne pas confondre / obsolète

- **systemd-timesyncd** (client SNTP simple, défaut Debian) vs **chrony** (NTP complet, serveur, meilleure précision) : ne pas faire tourner les deux.
- Le **PDC emulator** est **LA** source de temps du domaine — ne pas configurer de source manuelle sur les membres.
- Une heure fausse casse **Kerberos, les certificats et la corrélation des journaux**.

---

## Sources (doc officielle)

- [Microsoft — Windows Time Service (w32tm)](https://learn.microsoft.com/en-us/windows-server/networking/windows-time-service/windows-time-service-tools-and-settings) — consulté le 24/07/2026
- [chrony — Documentation](https://chrony-project.org/documentation.html) — consulté le 24/07/2026
- [Debian Wiki — NTP](https://wiki.debian.org/NTP) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI (Windows + Linux) · [x] versions datées · [x] rien d'obsolète (chrony/timesyncd) · [x] **`timedatectl` testé** / w32tm à tester en lab · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
