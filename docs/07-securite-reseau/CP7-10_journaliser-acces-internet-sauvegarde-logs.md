# CP7-10 — Journaliser les accès Internet et sauvegarder les logs (traçabilité)

**Objectif** : envoyer les journaux du pare-feu (accès Internet, filtrage, DNS) vers un **collecteur syslog centralisé**, les conserver et les faire tourner, dans le respect de l'obligation légale de traçabilité.

**Rattachement REAC** : CP7 « Maintenir et sécuriser les accès Internet et les interconnexions » — savoir-faire : journaliser les accès et assurer la traçabilité.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un pare-feu **pfSense** opérationnel (**CP7-01**), horloge synchronisée en **NTP** (**CP4-13** — sinon les logs sont inexploitables).
- Un serveur **Debian 13** dédié « collecteur » (**CP3-01**), atteignable depuis le pare-feu.
- Réseau d'administration/segment dédié recommandé pour le flux syslog.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Pare-feu | pfSense CE **2.8.1** | 24/07/2026 |
| Collecteur | Debian **13** + **rsyslog 8.x** | 24/07/2026 |
| Rotation | **logrotate** (motif + syntaxe validés à blanc) | 24/07/2026 |

> **Local ≠ centralisé.** pfSense journalise en local dans un **anneau circulaire** (taille fixe, en RAM) : les vieux événements sont **écrasés**. Pour la traçabilité, il **faut** exporter vers un collecteur qui, lui, conserve.

---

## Procédure — GUI (pfSense : export syslog)

1. **Status ▸ System Logs ▸ Settings**.
2. Section **Remote Logging Options** → cocher **Enable Remote Logging**.
3. **Source Address** : l'interface d'où partent les paquets syslog (idéalement l'interface de management).
4. **Remote Log Servers** : saisir l'adresse du collecteur au format `ip:port` (ex. `192.168.99.10:514`) — jusqu'à **3** serveurs.
5. **Remote Syslog Contents** : cocher ce qu'on veut exporter — au minimum **Firewall Events** (accès/blocages), et selon le besoin **DNS Resolver/Forwarder**, **DHCP**, **System Events**, **VPN**.
6. **Save**.

> **Chiffrement** : par défaut syslog part en **UDP/514 en clair**. Pour du **TCP + TLS**, installer le paquet **`syslog-ng`** (System ▸ Package Manager) et configurer une destination TLS. Sinon, cantonner le flux à un segment d'administration de confiance.

## Procédure — CLI (collecteur Debian : réception + rétention)

Sur le serveur collecteur, créer `/etc/rsyslog.d/10-remote.conf` — **un fichier par équipement émetteur** :

```rsyslog
module(load="imudp")
input(type="imudp" port="514")
module(load="imtcp")          # si réception TCP souhaitée
input(type="imtcp" port="514")

template(name="PerHostFile" type="string"
         string="/var/log/remote/%HOSTNAME%/%PROGRAMNAME%.log")

if ($fromhost-ip != '127.0.0.1') then {
    action(type="omfile" dynaFile="PerHostFile")
    stop
}
```

```bash
rsyslogd -N1                       # contrôle de syntaxe AVANT de recharger  (testé : OK)
systemctl restart rsyslog
ss -lunp | grep :514               # le collecteur écoute bien en UDP/514
```

Rotation + conservation ~**1 an** — `/etc/logrotate.d/remote` :

```conf
/var/log/remote/*/*.log {
    daily
    rotate 366          # ~1 an d'historique
    compress
    delaycompress
    missingok
    notifempty
}
```

```bash
logrotate --debug /etc/logrotate.d/remote   # simulation à blanc (testé : motif reconnu, 366 rotations)
```

> **Testé en bac à sable** : `rsyslogd -N1` valide la config sans erreur ; `logrotate --debug` reconnaît le motif et confirme *« after 1 days (366 rotations) »*. Le **flux réseau pfSense → collecteur** reste **à valider en lab**.

---

## Vérification (comment savoir que ça marche)

```bash
# Sur le collecteur : générer un message de test puis le retrouver
logger -n 127.0.0.1 -P 514 -d "test-tracabilite"
ls -R /var/log/remote/                       # un dossier par HOSTNAME émetteur
tail -f /var/log/remote/<pfsense>/filterlog.log
```

- Côté pfSense, provoquer un accès bloqué (ex. vers un port fermé) : une ligne **`filterlog`** doit apparaître sur le collecteur (règle, action `block/pass`, IP source/dest, port).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Rien n'arrive sur le collecteur | Pare-feu local / mauvais port | Autoriser **UDP/TCP 514** entrant ; vérifier `ss -lunp` |
| Horodatage incohérent | NTP absent | Synchroniser pfSense **et** collecteur (**CP4-13**) |
| Logs mélangés dans un seul fichier | Template non appliqué | Utiliser `dynaFile` par `%HOSTNAME%` |
| Disque qui sature | Rétention trop longue / pas de compression | Ajuster `rotate`, activer `compress`, disque dédié |

## Sécurité et bonnes pratiques

- **Traçabilité légale (France)** : les données de **connexion** permettant d'identifier l'origine d'un accès se conservent **jusqu'à 1 an** (art. 6 LCEN, **décret n° 2021-1362**). Adapter la rétention à cette obligation **et** au RGPD (pas de conservation au-delà du nécessaire — voir **CT4-01**).
- **Intégrité** : collecteur **dédié**, accès restreint ; idéalement stockage en **append-only**/hashé pour prouver la non-altération.
- **Confidentialité** : privilégier **TCP + TLS** (syslog-ng) ou un **segment d'administration** isolé ; le syslog en clair est interceptable.
- **Disponibilité** : surveiller l'espace disque du collecteur (**CP3-10**) et le sauvegarder (**CP8**).

## ⚠️ À ne pas confondre / obsolète

- **UDP/514 en clair** = hérité et non fiable (pertes, écoute possible) → préférer **TCP/TLS** quand c'est possible.
- Journal **local pfSense** (circulaire, volatile) ≠ **collecteur** (persistant) : seul le second assure la traçabilité.
- Ne pas confondre **logs de connexion** (qui/quand/où — soumis à obligation) et **contenu** (interdit de conserver sans base légale).

---

## Sources (doc officielle)

- [pfSense — Log Settings](https://docs.netgate.com/pfsense/en/latest/monitoring/logs/settings.html) — consulté le 24/07/2026
- [pfSense — Remote Logging with Syslog](https://docs.netgate.com/pfsense/en/latest/monitoring/logs/remote.html) — consulté le 24/07/2026
- [rsyslog — Configuration](https://www.rsyslog.com/doc/configuration/index.html) — consulté le 24/07/2026
- [Légifrance — Décret n° 2021-1362 (conservation des données de connexion, art. 6 LCEN)](https://www.legifrance.gouv.fr/jorf/id/JORFTEXT000044228912) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète (TCP/TLS vs UDP clair) · [x] CLI testée (rsyslog `-N1`, logrotate `--debug`) / flux réseau « à tester en lab » · [x] GUI conforme doc Netgate · [x] vérification présente · [x] sécurité (rétention légale, intégrité, chiffrement) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
