# CP4-17 — Installer une solution de supervision (Zabbix/Centreon)

**Objectif** : installer un serveur de supervision **Zabbix 7.0 LTS**, superviser un hôte via agent, et créer tableaux de bord et alertes.

**Rattachement REAC** : CP4 « Exploiter un réseau IP » — savoir-faire : mettre en place la supervision de l'infrastructure.

**Durée** : ~40 min · **Niveau** : avancé.

---

## Prérequis

- Un serveur Debian 13 dédié, droits root/sudo, accès Internet.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Zabbix | **7.0 LTS** (dépôt Debian 13 confirmé) | 24/07/2026 |
| Agent | **zabbix-agent2** (recommandé) | 24/07/2026 |
| Install | **à tester en lab** | 24/07/2026 |

---

## Procédure — CLI (installation du serveur)

```bash
# 1. Dépôt officiel Zabbix 7.0 pour Debian 13
wget https://repo.zabbix.com/zabbix/7.0/debian/pool/main/z/zabbix-release/zabbix-release_latest_7.0+debian13_all.deb
sudo dpkg -i zabbix-release_latest_7.0+debian13_all.deb
sudo apt update

# 2. Serveur + frontend + agent2 + scripts SQL
sudo apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf \
  zabbix-sql-scripts zabbix-agent2 mariadb-server

# 3. Base de données
sudo mysql -e "CREATE DATABASE zabbix CHARACTER SET utf8mb4 COLLATE utf8mb4_bin;"
sudo mysql -e "CREATE USER zabbix@localhost IDENTIFIED BY 'MotDePasseFort';"
sudo mysql -e "GRANT ALL ON zabbix.* TO zabbix@localhost; SET GLOBAL log_bin_trust_function_creators=1;"
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix

# 4. Renseigner le mot de passe BDD dans /etc/zabbix/zabbix_server.conf → DBPassword=MotDePasseFort
sudo systemctl restart zabbix-server zabbix-agent2 apache2
sudo systemctl enable  zabbix-server zabbix-agent2 apache2
```

## Procédure — GUI (frontend web)

1. Ouvrir `http://<serveur>/zabbix` → **assistant** : base de données, paramètres, création.
2. Se connecter (**Admin / zabbix**) → **changer immédiatement le mot de passe**.
3. **Superviser un hôte** : installer `zabbix-agent2` sur la cible (`Server=<IP du serveur>` dans `/etc/zabbix/zabbix_agent2.conf`), puis **Data collection → Hosts → Create host** → lier le template **Linux by Zabbix agent**.
4. **Tableaux de bord** : *Dashboards* (graphes CPU/RAM/réseau).
5. **Alertes** : *Alerts → Media types* (e-mail) + *Actions* (déclencheur → notification).

---

## Vérification (comment savoir que ça marche)

- Le **frontend** s'ouvre et l'authentification aboutit.
- L'hôte supervisé passe en **vert** (Availability) et les **Latest data** se remplissent.
- Un déclencheur de test (ex. charge élevée) génère bien une **alerte**.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Frontend : erreur de connexion BDD | `DBPassword` non renseigné | Compléter `zabbix_server.conf`, redémarrer |
| « Zabbix server is not running » | Service arrêté / mal configuré | `systemctl status zabbix-server` ; corriger |
| Hôte injoignable | Port **10050**, `Server=` de l'agent, pare-feu | Vérifier l'agent2 et ouvrir 10050 |

## Sécurité et bonnes pratiques

- **Changer le mot de passe Admin** par défaut ; publier le frontend en **HTTPS**.
- **Restreindre l'accès** au frontend (réseau de gestion).
- Chiffrer la liaison **agent ↔ serveur** (PSK/certificat).

## ⚠️ À ne pas confondre / obsolète

- **zabbix-agent2** (Go, greffons intégrés) est **recommandé** face à l'ancien `zabbix-agent`.
- **7.0 LTS** (support long) vs **7.5** (standard, support court) : choisir **LTS** en production.
- **Centreon** est une alternative équivalente (moteur historiquement Nagios) — même logique agents/templates/alertes.

---

## Sources (doc officielle)

- [Zabbix — Download / Install (Debian, 7.0 LTS)](https://www.zabbix.com/download?zabbix=7.0&os_distribution=debian) — consulté le 24/07/2026
- [Dépôt officiel Zabbix 7.0 (Debian trixie)](https://repo.zabbix.com/zabbix/7.0/debian/dists/trixie/) — consulté le 24/07/2026
- [Zabbix — Documentation 7.0](https://www.zabbix.com/documentation/7.0/en/manual) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI (install) puis GUI (frontend) · [x] versions datées (7.0 LTS) · [x] rien d'obsolète (agent2, LTS) · [x] install à tester en lab · [x] conforme doc Zabbix · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
