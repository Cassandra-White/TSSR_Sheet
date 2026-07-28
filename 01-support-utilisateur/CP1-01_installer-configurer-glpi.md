# CP1-01 — Installer et configurer GLPI (gestion de parc + helpdesk)

**Objectif** : installer **GLPI** sur un serveur Debian (pile LAMP) et le configurer via l'assistant web pour gérer le **parc** et le **helpdesk**.

**Rattachement REAC** : CP1 « Assurer le support utilisateur en centre de services » — savoir-faire : mettre en place l'outil de gestion de parc et d'assistance.

**Durée** : ~35 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un serveur **Debian 13** (**CP3-01**), accès **root**, résolution DNS/nom d'hôte (**CP3-02**).
- Une pile **LAMP** : Apache, **PHP 8.2+**, **MariaDB 10.5+**.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Logiciel | **GLPI 11.0.x** | 24/07/2026 |
| Serveur | Debian 13 + Apache 2.4 + **PHP 8.3** + MariaDB | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **GLPI** = **ITAM** (inventaire/parc) **+** **ITSM/helpdesk** (tickets). GLPI 11 tourne sur une **LAMP** classique ; Debian 13 fournit **PHP 8.3** (compatible).

---

## Procédure — CLI (préparer la pile)

```bash
apt install apache2 mariadb-server \
  php php-mysql php-gd php-intl php-curl php-xml php-mbstring php-zip php-bz2 php-ldap

mysql_secure_installation           # sécuriser MariaDB

# Base + utilisateur dédiés
mysql -e "CREATE DATABASE glpidb CHARACTER SET utf8mb4;"
mysql -e "CREATE USER 'glpi'@'localhost' IDENTIFIED BY 'MotDePasseFort';"
mysql -e "GRANT ALL ON glpidb.* TO 'glpi'@'localhost'; FLUSH PRIVILEGES;"

# Récupérer GLPI 11 (release officielle) et le déployer
tar xzf glpi-11.*.tgz -C /var/www/html/
chown -R www-data:www-data /var/www/html/glpi
systemctl restart apache2
```

## Procédure — GUI (assistant web)

1. Ouvrir **`http://<serveur>/glpi`** → l'assistant se lance.
2. Choisir la **langue**, accepter la **licence**, vérifier les **prérequis** (extensions PHP en vert).
3. **Connexion base** : hôte `localhost`, base `glpidb`, utilisateur `glpi` + mot de passe → **initialiser** les tables.
4. Terminer : GLPI crée **4 comptes par défaut** (`glpi/glpi`, `tech/tech`, `normal/normal`, `post-only/postonly`).

### Post-installation (obligatoire)

```bash
rm -f /var/www/html/glpi/install/install.php     # retirer l'installeur
```

- **Changer les 4 mots de passe par défaut** ; configurer le **cron** GLPI et le **fuseau horaire** PHP/MariaDB.

---

## Vérification (comment savoir que ça marche)

- Connexion au tableau de bord GLPI OK ; **Administration ▸ Vérification** ne signale pas d'erreur.
- Les comptes par défaut ont été **modifiés** ; `install.php` a disparu.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Prérequis PHP en rouge | Extension manquante | Installer `php-gd`/`php-intl`/`php-xml`… puis relancer |
| Erreur de connexion BdD | Droits/utilisateur | Vérifier le `GRANT` et le mot de passe |
| Avertissement fuseau horaire | TZ non chargée | `mariadb-tzinfo-to-sql` + timezone PHP |
| Page blanche | Droits `www-data` | `chown -R www-data:www-data /var/www/html/glpi` |

## Sécurité et bonnes pratiques

- **Changer immédiatement** les 4 comptes par défaut et **supprimer `install.php`**.
- Servir en **HTTPS** (**CP3-13/CP7-11**), restreindre l'accès (réseau d'admin).
- **Sauvegarder** la base (dump SQL — **CP8-09**) et le dossier de fichiers.

## ⚠️ À ne pas confondre / obsolète

- Comptes **par défaut** = **faille** connue → à changer sans délai.
- **Inventaire natif GLPI** (intégré, GLPI 10+) a **remplacé** le plugin **FusionInventory** (**CP1-02**).
- **GLPI 11** exige **PHP 8.2+** (ne pas viser une vieille PHP 7).

---

## Sources (doc officielle)

- [GLPI — Prérequis d'installation](https://glpi-install.readthedocs.io/en/latest/prerequisites.html) — consulté le 24/07/2026
- [GLPI — Documentation d'installation](https://glpi-install.readthedocs.io/en/latest/) — consulté le 24/07/2026
- [GLPI Project — Site officiel](https://glpi-project.org/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI (LAMP) puis GUI (assistant) · [x] versions datées (GLPI 11 / Debian 13) · [x] rien d'obsolète (PHP 8.2+, inventaire natif) · [x] procédure **à tester en lab** · [x] conforme doc GLPI · [x] vérification présente · [x] sécurité (comptes défaut, install.php, HTTPS) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
