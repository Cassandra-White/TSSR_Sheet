# CP5-02 — Créer et configurer une machine virtuelle

**Objectif** : créer une **VM** performante et moderne sous Proxmox VE (q35/UEFI, **VirtIO**, **QEMU Guest Agent**) et y installer un OS.

**Rattachement REAC** : CP5 « Maintenir des serveurs dans une infrastructure virtualisée » — savoir-faire : créer/configurer des VM.

**Durée** : ~30 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un hôte **Proxmox VE 9** (**CP5-01**), une **ISO** d'installation téléversée (stockage *ISO images*).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Hyperviseur | **Proxmox VE 9.2** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> Pour de bonnes **performances** et une gestion propre : machine **q35** + **UEFI (OVMF)**, périphériques **VirtIO** (paravirtualisés) et **QEMU Guest Agent** (arrêt propre + remontée d'IP).

---

## Procédure — GUI (assistant *Create VM*)

1. **General** : nom, VMID.
2. **OS** : sélectionner l'**ISO** et le type d'OS.
3. **System** : **Machine = q35**, **BIOS = OVMF (UEFI)** + **Add EFI Disk**, cocher **QEMU Agent**.
4. **Disks** : contrôleur **VirtIO SCSI**, taille, cocher **Discard** (TRIM sur SSD).
5. **CPU** : cœurs, *type* **host** (perf) ; **Memory** : RAM (ballooning possible).
6. **Network** : pont **vmbr0**, modèle **VirtIO (paravirtualized)**.
7. **Démarrer** la VM → installer l'OS via la **console noVNC**.
8. Dans l'invité : installer **`qemu-guest-agent`** (et les **pilotes VirtIO** pour Windows).

## Procédure — CLI (`qm`)

```bash
qm create 100 --name vm-debian --memory 4096 --cores 2 \
  --machine q35 --bios ovmf --scsihw virtio-scsi-single \
  --net0 virtio,bridge=vmbr0 --agent enabled=1

qm set 100 --efidisk0 local-lvm:1 \
  --scsi0 local-lvm:32,discard=on \
  --ide2 local:iso/debian-13.iso,media=cdrom \
  --boot order=scsi0

qm start 100
```

---

## Vérification (comment savoir que ça marche)

- La VM **démarre** et l'OS s'installe via la console.
- Après installation du **guest agent** : l'**adresse IP** de la VM s'affiche dans Proxmox, et l'**arrêt propre** (*Shutdown*) fonctionne.

> Le guest agent communique via un **canal virtio-serial** (pas le réseau) : il fonctionne **même sans réseau** dans la VM.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Windows ne voit pas le disque | Pilote **VirtIO** absent à l'install | Monter l'ISO **virtio-win** et charger le pilote |
| Pas d'IP affichée | **Guest agent** non installé | Installer `qemu-guest-agent` dans l'invité |
| Ne boote pas (UEFI) | **EFI Disk** oublié | Ajouter un disque EFI (OVMF) |
| Perfs faibles | Périphériques émulés | Utiliser **VirtIO** (disque/réseau) |

## Sécurité et bonnes pratiques

- **QEMU Guest Agent** pour des **arrêts propres** (cohérence des sauvegardes — **CP5-03/CP8-05**).
- **Isoler** la VM sur le bon **VLAN/pont** (**CP5-05/CP4-04**) ; ressources dimensionnées.
- Créer des **modèles** (*templates*) pour standardiser (**CP5-04**).

## ⚠️ À ne pas confondre / obsolète

- **q35 + OVMF (UEFI)** (moderne) vs **440FX + SeaBIOS** (hérité) : privilégier q35/UEFI pour les invités récents.
- **VirtIO** (paravirtualisé, rapide) ≠ périphériques **émulés** (compatibles mais lents).
- **Guest agent** (virtio-serial) ≠ accès **réseau** : indépendants.

---

## Sources (doc officielle)

- [Proxmox VE — QEMU/KVM Virtual Machines](https://pve.proxmox.com/pve-docs/chapter-qm.html) — consulté le 24/07/2026
- [Proxmox VE — QEMU Guest Agent](https://pve.proxmox.com/wiki/Qemu-guest-agent) — consulté le 24/07/2026
- [Proxmox VE — Manuel `qm.conf`](https://pve.proxmox.com/wiki/Manual:_qm.conf) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (`qm`) · [x] version datée (PVE 9.2) · [x] rien d'obsolète (q35/OVMF, VirtIO) · [x] procédure **à tester en lab** · [x] conforme doc Proxmox · [x] vérification présente (IP guest agent) · [x] sécurité (arrêt propre, isolation) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
