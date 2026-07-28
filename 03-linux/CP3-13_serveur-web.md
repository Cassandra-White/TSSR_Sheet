# CP3-13 — Installer un serveur web de base (Apache/Nginx) pour un service interne

**Objectif** : héberger une page/un service interne sur un serveur Debian avec Nginx **ou** Apache.

**Rattachement REAC** : CP3 « Exploiter des serveurs Linux » — savoir-faire : mettre en service une application web interne.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Debian 13 (**CP3-01**), IP fixe (**CP3-02**), droits root/sudo.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Debian | 13.6 « trixie » | 24/07/2026 |
| Nginx / Apache | **1.26** / **2.4.64** — config **à tester en lab** | 24/07/2026 |

> Configuration en **CLI** ; consultation en **GUI** (navigateur).

---

## Procédure — CLI

### Contenu à publier

```bash
sudo mkdir -p /var/www/intranet
echo "<h1>Intranet - OK</h1>" | sudo tee /var/www/intranet/index.html
```

### Option A — Nginx

```bash
sudo apt install nginx
sudo tee /etc/nginx/sites-available/intranet.conf >/dev/null <<'EOF'
server {
    listen 80;
    server_name intranet.lab.local;
    root  /var/www/intranet;
    index index.html;
}
EOF
sudo ln -s /etc/nginx/sites-available/intranet.conf /etc/nginx/sites-enabled/
sudo rm -f /etc/nginx/sites-enabled/default      # éviter le conflit sur le port 80
sudo nginx -t                                    # tester la configuration
sudo systemctl reload nginx
```

### Option B — Apache

```bash
sudo apt install apache2
sudo tee /etc/apache2/sites-available/intranet.conf >/dev/null <<'EOF'
<VirtualHost *:80>
    ServerName intranet.lab.local
    DocumentRoot /var/www/intranet
</VirtualHost>
EOF
sudo a2ensite intranet.conf
sudo a2dissite 000-default.conf
sudo apache2ctl configtest                       # tester la configuration
sudo systemctl reload apache2
```

> Ouvrir les ports **80/443** au pare-feu (**CP3-14**).

## Procédure — GUI (accès)

Depuis un poste du réseau : navigateur → `http://<IP-du-serveur>` (ou `http://intranet.lab.local` si le DNS/hosts est configuré). La page « Intranet - OK » s'affiche.

---

## Vérification (comment savoir que ça marche)

```bash
curl -I http://localhost            # attendu : HTTP/1.1 200 OK
sudo nginx -t                       # ou : sudo apache2ctl configtest
ss -tulpn | grep ':80'              # le serveur écoute sur le port 80
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| **403 Forbidden** | Droits / `index` absent | Vérifier droits de `/var/www/intranet` et présence d'`index.html` |
| **404 Not Found** | Mauvais `root`/`DocumentRoot` | Corriger le chemin |
| Le service ne démarre pas (port 80 pris) | Nginx **et** Apache installés | N'en garder **qu'un** sur le port 80 |
| Accessible en local mais pas à distance | Pare-feu | Ouvrir le port 80 (**CP3-14**) |

## Sécurité et bonnes pratiques

- Service **interne** : restreindre l'accès (pare-feu, `listen` sur l'IP interne).
- Passer en **HTTPS** (Let's Encrypt ou PKI interne — **CP7-11**).
- Masquer la version du serveur (`server_tokens off;` sous Nginx) ; tenir à jour (**CP3-11**).

## ⚠️ À ne pas confondre / obsolète

- **Ne pas** faire tourner **Nginx et Apache** simultanément sur le **port 80**.
- Sous Nginx, l'activation d'un site = **lien symbolique** `sites-available → sites-enabled` (pas de `a2ensite`, qui est propre à Apache).
- Toujours **tester la config** (`nginx -t` / `apache2ctl configtest`) **avant** de recharger.

---

## Sources (doc officielle)

- [Debian Wiki — Nginx](https://wiki.debian.org/Nginx) — consulté le 24/07/2026
- [Debian Wiki — Apache](https://wiki.debian.org/Apache) — consulté le 24/07/2026
- [Nginx — Beginner's Guide](https://nginx.org/en/docs/beginners_guide.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI puis GUI (accès navigateur) · [x] versions datées (1.26 / 2.4.64) · [x] rien d'obsolète · [x] config à tester en lab · [x] conforme doc · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
