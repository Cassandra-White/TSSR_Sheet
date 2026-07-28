# STO-10 — Configurer un stockage réseau iSCSI (cible + initiateur)

**Objectif** : exposer un stockage **bloc** sur le réseau IP (**cible** iSCSI) et le connecter depuis un client (**initiateur**), avec **IQN**, **LUN** et **CHAP**.

**Rattachement REAC** : CP5 (baies de stockage) / STO — savoir-faire : mettre en œuvre un stockage réseau.

**Durée** : ~30 min · **Niveau** : avancé.

---

## Prérequis

- Un serveur pour la **cible** (Windows Server 2025 **ou** Debian) et un **client initiateur**.
- Un **réseau de stockage** dédié recommandé (VLAN isolé).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Cible Windows | **iSCSI Target Server** (WS 2025) | 24/07/2026 |
| Cible/initiateur Linux | **targetcli** (LIO) / **open-iscsi** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **iSCSI = SCSI sur IP.** La **cible** (target) expose des **LUN** (disques logiques) ; l'**initiateur** (client) s'y connecte et voit un **disque local**. Chaque extrémité a un **IQN** (nom unique). L'authentification se fait en **CHAP**.

---

## Procédure — GUI (Windows)

### Côté cible

1. **Server Manager ▸ Add Roles ▸ File and Storage Services ▸ iSCSI Target Server**.
2. **File and Storage Services ▸ iSCSI ▸ New iSCSI Virtual Disk** : créer le disque (VHDX), définir la **cible**, autoriser les **initiateurs** (par **IQN**), activer **CHAP**.

### Côté initiateur

3. Ouvrir l'applet **Initiateur iSCSI** → onglet **Cibles** → saisir l'**IP de la cible** → **Connexion**.
4. **Gestion des disques** : initialiser/formater le nouveau disque (**STO-04**).

## Procédure — CLI (Linux)

### Cible (`targetcli` / LIO)

```bash
targetcli
/backstores/block create disk1 /dev/sdb                     # LUN à partir d'un disque
/iscsi create iqn.2026-07.local.lab:target1                  # créer la cible (IQN)
/iscsi/iqn.2026-07.local.lab:target1/tpg1/luns create /backstores/block/disk1
/iscsi/iqn.2026-07.local.lab:target1/tpg1/acls create iqn.2026-07.local.lab:client1
# activer/renseigner CHAP dans les attributs du TPG, puis :
saveconfig
```

### Initiateur (`open-iscsi`)

```bash
iscsiadm -m discovery -t st -p 192.168.99.20                 # découvrir les cibles
iscsiadm -m node -T iqn.2026-07.local.lab:target1 -p 192.168.99.20 -l   # se connecter
lsblk                                                        # le nouveau disque apparaît
```

---

## Vérification (comment savoir que ça marche)

- Sur l'initiateur, un **nouveau disque** apparaît (`lsblk` / Gestion des disques) → on peut le **partitionner/formater** (**STO-04/05**).
- `iscsiadm -m session` (Linux) liste la session active.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Cible injoignable | Port **3260** bloqué | Ouvrir 3260/TCP ; vérifier la route/VLAN |
| Connexion refusée | **ACL/IQN** ou **CHAP** faux | Vérifier l'IQN autorisé et les identifiants CHAP |
| Disque non persistant | Session non automatique | `iscsiadm ... --op update -n node.startup -v automatic` |
| Corruption si double accès | LUN monté par **2 clients** | Un LUN = **1 initiateur** (sauf FS cluster) |

## Sécurité et bonnes pratiques

- **CHAP** (idéalement mutuel) + **VLAN de stockage isolé** : sans auth, n'importe qui peut se connecter.
- Trafic iSCSI **non chiffré** → réseau **dédié**/isolé (ou IPsec).
- **Un LUN par initiateur** (sauf système de fichiers de cluster) pour éviter la corruption.

## ⚠️ À ne pas confondre / obsolète

- **Cible** (serveur, expose) ≠ **initiateur** (client, consomme).
- **iSCSI** (bloc sur IP Ethernet, économique) vs **Fibre Channel** (SAN dédié).
- **iSCSI** (bloc, comme un disque) ≠ **NFS/SMB** (partage de **fichiers**).

---

## Sources (doc officielle)

- [Red Hat — Configuring an iSCSI target (targetcli)](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/9/html/managing_storage_devices/configuring-an-iscsi-target_managing-storage-devices) — consulté le 24/07/2026
- [Microsoft Learn — iSCSI Target Server](https://learn.microsoft.com/en-us/windows-server/storage/iscsi/iscsi-target-server) — consulté le 24/07/2026
- [open-iscsi — Documentation](https://github.com/open-iscsi/open-iscsi) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (targetcli/open-iscsi) · [x] versions datées · [x] rien d'obsolète (CHAP, VLAN dédié) · [x] procédure **à tester en lab** · [x] conforme doc RedHat/Microsoft · [x] vérification présente (`lsblk`/session) · [x] sécurité (CHAP, isolation) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
