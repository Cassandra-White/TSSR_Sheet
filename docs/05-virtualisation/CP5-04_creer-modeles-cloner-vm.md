# CP5-04 — Créer des modèles (templates) et cloner des VM

**Objectif** : transformer une VM en **modèle** (template) et déployer rapidement des VM par **clonage** (lié/complet), personnalisées au premier démarrage via **cloud-init**.

**Rattachement REAC** : CP5 « Maintenir des serveurs dans une infrastructure virtualisée » — savoir-faire : industrialiser le déploiement de VM.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un hôte **Proxmox VE 9** (**CP5-01**), une VM « modèle » propre (**CP5-02**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Hyperviseur | **Proxmox VE 9.2** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> Un **template** est une **image de base en lecture seule** que l'on **clone** en quelques secondes. **Clone lié** = rapide et **économe** (partage la base) ; **clone complet** = **copie indépendante**. **Cloud-init** configure automatiquement chaque clone au **1ᵉʳ boot** (nom d'hôte, réseau, clé SSH).

---

## Procédure — GUI

1. Préparer la VM modèle (OS + **guest agent**, **nettoyée** — pas de secret ni d'identité unique — cf. **CP9-02**).
2. **(Cloud-init)** *Hardware ▸ Add ▸ CloudInit Drive*, puis onglet **Cloud-Init** : utilisateur, **clé SSH**, IP.
3. **VM ▸ More ▸ Convert to template** → la VM devient **lecture seule**.
4. **Clic droit sur le template ▸ Clone** : **Full** ou **Linked**, nouvel **ID/nom**.

## Procédure — CLI (`qm`)

```bash
qm template 9000                                   # convertir la VM 9000 en template

qm clone 9000 101 --name web1 --full 0             # clone LIÉ (--full 1 = complet)
qm set 101 --ciuser admin --sshkeys ~/.ssh/id_ed25519.pub \
  --ipconfig0 ip=192.168.10.11/24,gw=192.168.10.1  # cloud-init : compte, clé, IP
qm start 101
```

---

## Vérification (comment savoir que ça marche)

- Le **clone lié** est créé en **quelques secondes** (peu d'espace consommé).
- Au 1ᵉʳ démarrage, **cloud-init** applique le **nom d'hôte**, l'**IP** et la **clé SSH** sans intervention.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Clone lié cassé | **Template supprimé** | Ne pas supprimer un template avec des clones liés |
| Cloud-init non appliqué | Drive absent / OS sans cloud-init | Ajouter le **CloudInit Drive** ; image *cloud* |
| Doublons d'identité | Modèle **non nettoyé** | Généraliser/nettoyer avant conversion |
| Template modifiable ? | Il est **en lecture seule** | Cloner, modifier le clone, refaire un template |

## Sécurité et bonnes pratiques

- **Nettoyer/généraliser** le modèle (pas de clés/secrets/identifiants uniques dupliqués — **CP9-02**).
- Diffuser l'accès via **clé SSH** (cloud-init), pas de mot de passe en dur.
- Standardiser (mêmes réglages/sécurité) → déploiements **homogènes** et rapides.

## ⚠️ À ne pas confondre / obsolète

- **Clone lié** (économe, dépend du template) ≠ **clone complet** (autonome).
- **Template** (lecture seule) ≠ **VM** modifiable.
- Cloner **sans nettoyer** = doublons (SID/clés) — **généraliser** d'abord.

---

## Sources (doc officielle)

- [Proxmox VE — VM Templates & Clones (chapter-qm)](https://pve.proxmox.com/pve-docs/chapter-qm.html) — consulté le 24/07/2026
- [Proxmox VE — Cloud-Init Support](https://pve.proxmox.com/wiki/Cloud-Init_Support) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (`qm`) · [x] version datée (PVE 9.2) · [x] rien d'obsolète (cloud-init, clé SSH) · [x] procédure **à tester en lab** · [x] conforme doc Proxmox · [x] vérification présente (clone + cloud-init) · [x] sécurité (nettoyage, clé SSH) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
