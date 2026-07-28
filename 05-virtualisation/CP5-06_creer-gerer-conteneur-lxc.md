# CP5-06 — Créer et gérer un conteneur LXC

**Objectif** : créer un **conteneur LXC** sous Proxmox à partir d'un template, en comprenant la différence **non privilégié / privilégié** et **conteneur vs VM**.

**Rattachement REAC** : CP5 « Maintenir des serveurs dans une infrastructure virtualisée » — savoir-faire : exploiter des conteneurs système.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un hôte **Proxmox VE 9** (**CP5-01**) et un **template de conteneur** (CT) téléchargé.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Hyperviseur | **Proxmox VE 9.2** (LXC 6.0) | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> Un **conteneur LXC** partage le **noyau de l'hôte** : très **léger** et **rapide** (démarrage en secondes), mais **moins isolé** qu'une VM. **Non privilégié = par défaut et recommandé** (le root du conteneur est mappé sur un utilisateur **non privilégié** de l'hôte).

---

## Procédure — GUI

1. **Télécharger un template** : *Nœud ▸ Storage (local) ▸ CT Templates ▸ Templates* → choisir (ex. `debian-13-standard`).
2. **Create CT** :
   - **hostname**, **mot de passe** ou **clé SSH**,
   - **Template**, **disque** (rootfs), **CPU/RAM**,
   - **réseau** (bridge `vmbr0`, éventuel **VLAN tag** — **CP5-05**),
   - laisser **Unprivileged container** **coché** (défaut).
3. **Démarrer** → **Console**.

## Procédure — CLI (`pct`)

```bash
pveam update && pveam available | grep debian-13         # templates disponibles
pveam download local debian-13-standard_*_amd64.tar.zst

pct create 200 local:vztmpl/debian-13-standard_amd64.tar.zst \
  --hostname ct-web --cores 1 --memory 1024 \
  --rootfs local-lvm:8 \
  --net0 name=eth0,bridge=vmbr0,ip=dhcp \
  --unprivileged 1 --password

pct start 200 && pct enter 200                            # démarrer + entrer
```

---

## Vérification (comment savoir que ça marche)

- Le conteneur **démarre en quelques secondes**, consomme **peu** de ressources.
- `pct list` le montre **running** ; `pct enter 200` donne un shell fonctionnel.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Fonction bloquée (montage, Docker imbriqué) | Conteneur **non privilégié** limité | Activer une *feature* (nesting) ou utiliser une **VM** |
| Pas de réseau | Bridge/VLAN mal réglé | Vérifier `net0` (bridge, tag — **CP5-05**) |
| Template introuvable | Liste non à jour | `pveam update` puis `pveam available` |
| Besoin d'un autre noyau | LXC partage celui de l'hôte | Utiliser une **VM** (noyau propre) |

## Sécurité et bonnes pratiques

- **Non privilégié par défaut** : les évasions de conteneur y sont bien moins critiques.
- **Privilégié** = **environnement de confiance uniquement** (l'équipe LXC ne traite pas les évasions comme des CVE).
- **Limiter** les ressources (CPU/RAM/disque) et **isoler** le réseau (VLAN).

## ⚠️ À ne pas confondre / obsolète

- **Non privilégié** (sûr, défaut) ≠ **privilégié** (root hôte, risqué).
- **Conteneur LXC** (**système**, partage le noyau) ≠ **VM** (noyau propre, isolation forte).
- **LXC** (conteneur système) ≠ **Docker** (conteneur **applicatif** — **CP5-13**).

---

## Sources (doc officielle)

- [Proxmox VE — Linux Container (LXC)](https://pve.proxmox.com/wiki/Linux_Container) — consulté le 24/07/2026
- [Proxmox VE — Container Toolkit (`pct`)](https://pve.proxmox.com/pve-docs/chapter-pct.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (`pct`) · [x] version datée (PVE 9.2 / LXC 6.0) · [x] rien d'obsolète (non privilégié par défaut) · [x] procédure **à tester en lab** · [x] conforme doc Proxmox · [x] vérification présente (`pct list`) · [x] sécurité (non privilégié, ressources) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
