# CP5-13 — Découvrir Docker et les conteneurs applicatifs (notion de container)

**Objectif** : comprendre les notions d'**image** et de **conteneur** Docker, déployer une pile avec **docker compose**, et situer Docker face à **LXC** et aux **VM**.

**Rattachement REAC** : CP5 « Maintenir des serveurs dans une infrastructure virtualisée » — savoir-faire : exploiter des conteneurs applicatifs.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un serveur/VM **Debian** (**CP3-01**), accès **root**. *(Bonne pratique : Docker **dans une VM** sur Proxmox.)*

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Moteur | **Docker** (Debian 13) | 24/07/2026 |
| `docker-compose.yml` | **structure validée en bac à sable** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> Une **image** = un modèle **immuable** (application + dépendances) ; un **conteneur** = une **instance en exécution** de cette image, **éphémère** et **reproductible**. Docker isole une **application** ; **LXC** (**CP5-06**) isole un **système** ; une **VM** a son **propre noyau**.

---

## Procédure — CLI

```bash
# Installer Docker + le plugin compose
apt install docker.io docker-compose-plugin

# Lancer un conteneur (image officielle nginx)
docker run -d --name web -p 8080:80 nginx:1.27-alpine
docker ps                     # conteneurs en cours
docker logs web ; docker exec -it web sh   # journaux / shell dans le conteneur
```

Déployer une **pile multi-conteneurs** (`docker-compose.yml`) :

```yaml
services:
  web:
    image: nginx:1.27-alpine
    ports: ["8080:80"]
    volumes: ["./html:/usr/share/nginx/html:ro"]
    restart: unless-stopped
  db:
    image: mariadb:11
    environment: { MARIADB_ROOT_PASSWORD: motdepassefort }
    volumes: ["db-data:/var/lib/mysql"]
volumes:
  db-data:
```

```bash
docker compose up -d          # démarre toute la pile
docker compose ps ; docker compose down
```

> **Testé en bac à sable** : le `docker-compose.yml` ci-dessus est **valide** (services `web`+`db`, image, ports, **volume nommé** pour la persistance) et déployable via `docker compose up -d`.

---

## Vérification (comment savoir que ça marche)

```bash
docker ps                     # le conteneur "web" est "Up"
curl http://localhost:8080    # le service répond
```

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Image introuvable | Tag/registry faux | Vérifier `image:tag` (Docker Hub) |
| Port déjà utilisé | Conflit `-p` | Changer le port hôte |
| Données perdues au redémarrage | Pas de **volume** | Monter un **volume** pour la persistance |
| Permissions | Docker en root | Envisager le mode **rootless** |

## Sécurité et bonnes pratiques

- **Images officielles/de confiance** et **épinglées** (tag précis) ; **scanner** les images (chaîne d'appro — **CP6-10**).
- **Volumes** pour les données ; **réseaux** isolés ; **secrets** hors du code (pas en clair).
- Privilégier **Docker dans une VM** (isolation du noyau) plutôt que dans un LXC.

## ⚠️ À ne pas confondre / obsolète

- **Docker** (conteneur **applicatif**) ≠ **LXC** (conteneur **système**, **CP5-06**) ≠ **VM** (noyau propre).
- Conteneur **éphémère** : les données vivent dans des **volumes**, pas dans le conteneur.
- `docker-compose` (v1, ancien binaire) → **`docker compose`** (plugin v2).

---

## Sources (doc officielle)

- [Docker — Documentation](https://docs.docker.com/) — consulté le 24/07/2026
- [Docker — Compose (compose file)](https://docs.docker.com/compose/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] CLI · [x] daté 24/07/2026 · [x] rien d'obsolète (`docker compose` v2) · [x] **compose validé en bac à sable** / exécution à tester en lab · [x] conforme doc Docker · [x] vérification présente (`docker ps`/curl) · [x] sécurité (images de confiance, volumes, secrets) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
