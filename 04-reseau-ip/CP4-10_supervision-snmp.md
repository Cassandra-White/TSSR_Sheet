# CP4-10 — Superviser le réseau via SNMP (avec Zabbix/Centreon)

**Objectif** : activer SNMP sur les équipements et remonter leur état dans un superviseur (Zabbix/Centreon).

**Rattachement REAC** : CP4 « Exploiter un réseau IP » — savoir-faire : superviser l'état des équipements réseau.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Équipement à superviser (serveur Debian, switch, routeur) et un superviseur **Zabbix/Centreon** (installation : **CP4-17**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Superviseur | **Zabbix 7.0 LTS** (8.0 LTS attendue en 2026) | 24/07/2026 |
| Agent SNMP (net-snmp) | config **à tester en lab** (paquets absents du bac à sable) | 24/07/2026 |

> **SNMP** : un **agent** (`snmpd`) sur l'équipement répond aux requêtes du superviseur (OID/MIB). **v2c** = *community* en clair ; **v3** = authentification + chiffrement (recommandé).

---

## Procédure — CLI (agent SNMP v3 sur Debian)

```bash
sudo apt install snmpd snmp
# Créer un utilisateur SNMPv3 en lecture seule (snmpd arrêté) :
sudo systemctl stop snmpd
sudo net-snmp-create-v3-user -ro -A "AuthPass123!" -a SHA -X "PrivPass123!" -x AES monitor
sudo systemctl start snmpd
```

Compléter `/etc/snmp/snmpd.conf` (identité + écoute) :

```text
sysLocation  Salle serveurs - Baie A
sysContact   admin@lab.local
agentaddress udp:161
```

Interroger depuis le superviseur :

```bash
# SNMPv3
snmpwalk -v3 -l authPriv -u monitor -a SHA -A "AuthPass123!" -x AES -X "PrivPass123!" 192.168.10.5 system
# SNMPv2c (labo isolé uniquement)
snmpwalk -v2c -c public 192.168.10.5 system
```

## Procédure — GUI (ajout de l'hôte dans Zabbix)

1. **Data collection → Hosts → Create host** : nom, groupe.
2. **Interfaces → Add → SNMP** : IP de l'équipement, port **161**.
3. Renseigner les paramètres **SNMPv3** (macros `{$SNMP_*}` : utilisateur, auth SHA, priv AES).
4. **Templates** : lier un modèle (ex. *Network Generic Device by SNMP*).
5. Contrôler dans **Monitoring → Latest data** que les valeurs remontent.

---

## Vérification (comment savoir que ça marche)

```bash
snmpget -v3 -l authPriv -u monitor -a SHA -A "AuthPass123!" -x AES -X "PrivPass123!" \
  192.168.10.5 sysUpTime.0        # doit renvoyer l'uptime
```

Dans Zabbix, l'hôte passe **vert** (disponibilité SNMP) et les graphes se remplissent.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Aucune réponse SNMP | Pare-feu **UDP 161** / `agentaddress` | Ouvrir le port ; vérifier l'écoute de `snmpd` |
| Échec auth v3 | Algos/mots de passe incohérents | Aligner SHA/AES + mots de passe des deux côtés |
| OID « No Such Object » | MIB absente | `snmp-mibs-downloader` ; utiliser l'OID numérique |
| Hôte rouge dans Zabbix | Mauvaises macros SNMP | Corriger `{$SNMP_*}` sur l'hôte |

## Sécurité et bonnes pratiques

- **SNMPv3** (auth + chiffrement) ; éviter **v1/v2c** sauf réseau de gestion **isolé**.
- **Changer** les *communities* par défaut (`public`/`private`), rester en **lecture seule**.
- **Filtrer UDP 161** par IP (seul le superviseur interroge) ; superviser via le **VLAN de gestion**.

## ⚠️ À ne pas confondre / obsolète

- **v2c** transporte la *community* **en clair** → à proscrire hors réseau isolé ; préférer **v3**.
- **Poll** (superviseur interroge, UDP 161) ≠ **Trap** (l'agent notifie, UDP 162).
- SNMP en **écriture** (rw) est rarement utile et dangereux → **read-only**.

---

## Sources (doc officielle)

- [Net-SNMP — snmpd.conf](http://www.net-snmp.org/docs/man/snmpd.conf.html) — consulté le 24/07/2026
- [Zabbix — SNMP monitoring](https://www.zabbix.com/documentation/current/en/manual/config/items/itemtypes/snmp) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI (agent) puis GUI (Zabbix) · [x] versions datées (Zabbix 7.0 LTS) · [x] rien d'obsolète (SNMPv3) · [x] config à tester en lab · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
