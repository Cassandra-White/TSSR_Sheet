# 🛠️  Docs & Fiches Mémoires

[![MkDocs Material](https://img.shields.io/badge/Docs-MkDocs_Material-blue?style=for-the-badge&logo=materialfordocs)](https://cassandra-white.github.io/TSSR_Sheet/)
[![GitHub Pages](https://img.shields.io/badge/GitHub_Pages-Deployed-success?style=for-the-badge&logo=github)](https://cassandra-white.github.io/TSSR_Sheet/)

Bienvenue sur **TSSR_Sheet** ! Ce dépôt regroupe un ensemble de **fiches synthétiques, tutoriels et aides-mémoires** dédiés à l'administration systèmes, réseaux et à la préparation du titre professionnel **Technicien Supérieur Systèmes et Réseaux (TSSR)**.

👉 **[Accéder au site de documentation interactif (avec recherche intégrée)](https://cassandra-white.github.io/TSSR_Sheet/)**

---

## 📚 Sommaire des Catégories

### 🌐 Réseaux & Services
* **Adressage & Routage** : Subnetting, VLANs, NAT/PAT, Modèle OSI & TCP/IP.
* **Services Réseau** : Configuration & dépannage DHCP, DNS, IPAM, NTP.
* **Équipements & Sécurité** : Switchs/Routeurs (Cisco, PfSense), VPN (OpenVPN, WireGuard), Firewall.

### 🐧 Administration Linux
* **Bases & Commandes** : Gestion des utilisateurs, permissions (`chmod`/`chown`), processus, SSH.
* **Services & Serveurs** : Apache2, Nginx, Bind9, Samba, NFS.
* **Scripting** : Automatisation en Bash & Shell scripting.

### 🪟 Windows Server & Active Directory
* **Active Directory (AD DS)** : Gestion des OU, utilisateurs, groupes et GPO (Group Policy Objects).
* **Services Windows** : DNS, DHCP, WSUS, WDS / MDT (Déploiement), FSRM & Partages réseau.
* **PowerShell** : Scripts d'administration et automatisation Active Directory.

### ☁️ Virtualisation, Container & Cloud
* **Hyperviseurs** : Proxmox VE, VMware ESXi / vSphere, Hyper-V.
* **Containers & Outils** : Docker, Docker Compose, Git & gestion de versions.

### 🔒 Sécurité, Sauvegardes & Supervision
* **Sauvegardes** : Stratégie 3-2-1, Veeam Backup & Replication, Rsync / Borg.
* **Sécurité & Supervision** : Hardening, PKI / Certificats SSL/TLS, Zabbix / Nagios / PRTG.

---

## 🔍 Comment rechercher un tutoriel ?

### 1. Via le site web MkDocs (Recommandé)
Consulte la documentation en ligne : **[cassandra-white.github.io/TSSR_Sheet](https://cassandra-white.github.io/TSSR_Sheet/)**  
Utilise la barre de recherche en haut du site pour trouver immédiatement une commande ou un concept dans toutes les fiches.

### 2. Recherche rapide sur GitHub (Raccourci VS Code)
Depuis la page de ce dépôt sur GitHub :
1. Appuie sur la touche **`.`** (point) de ton clavier pour ouvrir l'éditeur **VS Code Web**.
2. Fais **`Ctrl + Shift + F`** (ou `Cmd + Shift + F` sur Mac) pour rechercher un terme dans tout le contenu du repo.

### 3. Tester / Consulter en local
```bash
# Cloner le dépôt
git clone [https://github.com/Cassandra-White/TSSR_Sheet.git](https://github.com/Cassandra-White/TSSR_Sheet.git)
cd TSSR_Sheet

# Installer MkDocs (si besoin)
pip install mkdocs-material

# Lancer le serveur local avec rechargement automatique
mkdocs serve