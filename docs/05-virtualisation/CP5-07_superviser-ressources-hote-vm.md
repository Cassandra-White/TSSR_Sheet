# CP5-07 — Superviser les ressources de l'hôte et des VM

**Objectif** : suivre l'utilisation des ressources (CPU, RAM, disque, réseau) de l'hôte Proxmox et des VM — graphes intégrés, **serveur de métriques externe** et intégration **Zabbix**.

**Rattachement REAC** : CP5 « Maintenir des serveurs dans une infrastructure virtualisée » — savoir-faire : superviser une infrastructure virtualisée.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un hôte **Proxmox VE 9** (**CP5-01**) avec des VM.
- Optionnel : un **InfluxDB/Graphite** + **Grafana**, ou un serveur **Zabbix** (**CP4-17**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Hyperviseur | **Proxmox VE 9.2** | 24/07/2026 |
| Métriques externes | **InfluxDB / Graphite** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> Proxmox fournit des **graphes intégrés** (RRD) par nœud et par VM — pratiques pour le **temps réel** mais à **historique limité**. Pour du **long terme** et des **tableaux de bord**, on pousse les métriques vers un **serveur externe** (InfluxDB + Grafana) et/ou on supervise via **Zabbix/Centreon**.

---

## Procédure — GUI

### Graphes intégrés

1. **Nœud (ou VM) ▸ Summary** : CPU, RAM, disque, réseau, en temps réel + historique (heure/jour/semaine/mois/an).

### Serveur de métriques externe (InfluxDB/Graphite)

2. **Datacenter ▸ Metric Server ▸ Add** : type **InfluxDB** (ou Graphite), hôte, port, base/token → Proxmox **pousse** ses métriques.
3. Visualiser dans **Grafana** (tableaux de bord, historique long).

### Supervision centralisée (Zabbix — **CP4-17**)

4. Superviser l'hôte via **SNMP/agent** (seuils, alertes) pour intégrer Proxmox au **NOC** global.

---

## Vérification (comment savoir que ça marche)

- Les **graphes** s'affichent et évoluent ; les métriques **arrivent** dans InfluxDB/Grafana.
- Une **alerte** se déclenche bien lors d'un dépassement de **seuil** (ex. RAM > 90 %).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Historique trop court | RRD **local** limité | Serveur de métriques **externe** (InfluxDB) |
| Aucune métrique poussée | Metric Server mal configuré | Vérifier hôte/port/token InfluxDB |
| Pas d'alerte | Pas de supervision active | Ajouter **Zabbix** + seuils (**CP4-17**) |
| Graphes vides | Service `rrdcached`/pvestatd | Vérifier les services Proxmox |

## Sécurité et bonnes pratiques

- **Superviser pour anticiper** : saturation disque, RAM, panne matérielle (**STO-09**) avant l'incident.
- **Alertes** sur seuils + **capacité** (prévoir la croissance).
- Réseau de supervision **maîtrisé** ; ne pas exposer les endpoints de métriques.

## ⚠️ À ne pas confondre / obsolète

- **Graphes RRD locaux** (court terme) ≠ **InfluxDB+Grafana** (long terme, tableaux de bord).
- **Supervision** (voir l'état) ≠ **alerting** (être prévenu) : mettre en place **les deux**.
- Métriques **hôte** ≠ métriques **par VM** : suivre les deux niveaux.

---

## Sources (doc officielle)

- [Proxmox VE — External Metric Server](https://pve.proxmox.com/wiki/External_Metric_Server) — consulté le 24/07/2026
- [Proxmox VE — Host System Administration (monitoring)](https://pve.proxmox.com/pve-docs/chapter-sysadmin.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI · [x] version datée (PVE 9.2) · [x] rien d'obsolète (métriques externes, Grafana) · [x] procédure **à tester en lab** · [x] conforme doc Proxmox · [x] vérification présente (graphes/alertes) · [x] sécurité (anticipation, alertes) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
