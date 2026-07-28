# STO-14 — Installer et gérer un onduleur (UPS) + arrêt propre automatique

**Objectif** : raccorder un **onduleur (UPS)**, superviser son état et déclencher un **arrêt propre automatique** des serveurs **avant** la fin de batterie.

**Rattachement REAC** : CP5 (data center / équipements) / STO — savoir-faire : sécuriser l'alimentation.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un **onduleur** (USB, série ou réseau/SNMP) relié au(x) serveur(s).
- Linux : paquet **nut**. Windows : **PowerChute** (APC) / **WinNUT**.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Supervision UPS | **NUT 2.8.5** (avril 2026) | 24/07/2026 |
| Windows | **APC PowerChute** / **WinNUT** (client) | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> Un **onduleur** maintient l'alimentation sur **batterie** le temps de **couper proprement** les serveurs → évite la corruption de données/FS. Objectif : **arrêt automatique** déclenché **avant** l'épuisement de la batterie.

---

## Types d'onduleurs (choisir selon la criticité)

| Type | Principe | Usage |
|---|---|---|
| **Offline / standby** | Bascule sur batterie en cas de coupure | Postes, faible criticité |
| **Line-interactive** | + régulation de tension (**AVR**) | PME, petits serveurs |
| **Online / double conversion** | Alimentation **toujours** via l'onduleur | **Serveurs/baies critiques** |

---

## Procédure — CLI (Linux, NUT)

```bash
apt install nut
```

Configurer (fichiers `/etc/nut/`) :

```ini
# ups.conf : déclarer l'onduleur et son pilote
[myups]
  driver = usbhid-ups          # USB HID (ou snmp-ups, apcsmart…)
  port = auto

# nut.conf : rôle du serveur
MODE=standalone                # (ou 'netserver' pour partager l'UPS à d'autres serveurs)

# upsmon.conf : surveillance + arrêt propre
MONITOR myups@localhost 1 upsmon <motdepasse> master
SHUTDOWNCMD "/sbin/shutdown -h +0"
```

```bash
systemctl restart nut-server nut-monitor
upsc myups                     # état : charge, batterie %, autonomie estimée
upsmon -c fsd                  # TEST : forcer l'arrêt "final shutdown"
```

> **Plusieurs serveurs sur un même onduleur** : un serveur en **`netserver`** (master, relié à l'UPS) ; les autres en **`netclient`** (slaves) s'arrêtent sur ordre du master.

## Procédure — GUI (Windows)

- **APC PowerChute** (ou logiciel constructeur) : détecte l'UPS en USB, règle les **seuils** (arrêt à X % / Y minutes d'autonomie) et l'**arrêt automatique**.
- **WinNUT** : client Windows se connectant à un **serveur NUT** (master) pour s'arrêter sur alerte batterie basse.

---

## Vérification (comment savoir que ça marche)

```bash
upsc myups ups.status          # "OL" = sur secteur ; "OB" = sur batterie ; "LB" = batterie basse
upsc myups battery.runtime     # autonomie estimée (secondes)
```

- **Test réel** : couper l'alimentation d'entrée de l'UPS → les serveurs doivent **s'arrêter proprement** avant la fin de batterie.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| « Driver not connected » | Port/permissions USB | Vérifier le câble, `port=auto`, droits `/dev` |
| Arrêt trop tardif | Seuils mal réglés | Régler `LB`/marge de temps **avec sécurité** |
| Slaves non arrêtés | Mode/master-slave | `netserver` sur le master, `netclient` sur les slaves |
| Pas d'alerte | upsmon inactif | `systemctl status nut-monitor` |

## Sécurité et bonnes pratiques

- **Tester l'arrêt propre** régulièrement (une coupure réelle simulée), pas seulement la config.
- Régler les **seuils avec marge** (déclencher l'arrêt **avant** la batterie critique).
- **Remplacer les batteries** périodiquement (vieillissement) ; ne pas **surcharger** l'onduleur.
- **Superviser** (NUT + Zabbix — **CP4-17**) et **journaliser** les événements d'alimentation.

## ⚠️ À ne pas confondre / obsolète

- **Onduleur** (batterie + arrêt propre) ≠ **multiprise parafoudre** (protection surtension seule).
- **Online double-conversion** (critique) ≠ **offline/standby** (basique).
- Un onduleur **sans arrêt automatique configuré** ne protège pas les **données** (juste quelques minutes de courant).

---

## Sources (doc officielle)

- [Network UPS Tools (NUT) — Documentation](https://networkupstools.org/documentation.html) — consulté le 24/07/2026
- [NUT — upsmon (arrêt propre)](https://networkupstools.org/docs/man/upsmon.html) — consulté le 24/07/2026
- [Schneider Electric — APC PowerChute](https://www.apc.com/us/en/product-category/88972-powerchute-software/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI (NUT) puis GUI (PowerChute) · [x] versions datées (NUT 2.8.5) · [x] rien d'obsolète (online vs offline) · [x] procédure **à tester en lab** · [x] conforme doc NUT/APC · [x] vérification présente (`upsc`/test coupure) · [x] sécurité (test arrêt, seuils, batteries) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
