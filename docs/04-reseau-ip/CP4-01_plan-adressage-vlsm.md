# CP4-01 — Élaborer un plan d'adressage IP et découper en sous-réseaux (VLSM)

**Objectif** : concevoir un plan d'adressage et découper un bloc IP en sous-réseaux de tailles adaptées aux besoins (VLSM — *Variable Length Subnet Mask*).

**Rattachement REAC** : CP4 « Exploiter un réseau IP » — savoir-faire (méthode) : concevoir et documenter un plan d'adressage.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Notions d'IPv4 : adresse, masque, notation **CIDR** (`/n`), adresses privées **RFC 1918** (`10/8`, `172.16/12`, `192.168/16`).

## Rappels utiles

- Hôtes utilisables dans un sous-réseau = **2^h − 2** (h = bits d'hôte ; on retire l'**adresse réseau** et le **broadcast**).

| Masque | CIDR | Hôtes utilisables |
|---|---|---|
| 255.255.255.0 | /24 | 254 |
| 255.255.255.128 | /25 | 126 |
| 255.255.255.192 | /26 | 62 |
| 255.255.255.224 | /27 | 30 |
| 255.255.255.240 | /28 | 14 |
| 255.255.255.248 | /29 | 6 |
| 255.255.255.252 | /30 | 2 |

---

## Méthode (5 étapes)

1. **Recenser les besoins** : pour chaque segment, le nombre d'hôtes **+ une marge de croissance**.
2. **Trier du plus grand au plus petit** besoin.
3. **Choisir le plus petit masque** qui couvre chaque besoin (2^h − 2 ≥ besoin).
4. **Allouer en partant du début du bloc**, du plus grand au plus petit : c'est le principe **VLSM** — il évite le gaspillage **et** garantit l'alignement des sous-réseaux.
5. **Documenter** : réseau, masque, plage d'hôtes, passerelle (souvent la 1ʳᵉ adresse utilisable), broadcast.

---

## Exemple complet — découpe de `192.168.1.0/24`

Besoins : LAN A = 100 hôtes · LAN B = 50 · LAN C = 25 · LAN D = 10 · 2 liens WAN = 2 hôtes chacun.

> Table **vérifiée programmatiquement** (module Python `ipaddress`, contrôle d'alignement strict).

| Segment | Besoin | Réseau / masque | Plage d'hôtes | Broadcast | Dispo |
|---|---|---|---|---|---|
| LAN A | 100 | `192.168.1.0/25` | .1 → .126 | .127 | 126 |
| LAN B | 50 | `192.168.1.128/26` | .129 → .190 | .191 | 62 |
| LAN C | 25 | `192.168.1.192/27` | .193 → .222 | .223 | 30 |
| LAN D | 10 | `192.168.1.224/28` | .225 → .238 | .239 | 14 |
| WAN 1 | 2 | `192.168.1.240/30` | .241 → .242 | .243 | 2 |
| WAN 2 | 2 | `192.168.1.244/30` | .245 → .246 | .247 | 2 |

Dernière adresse consommée : `192.168.1.247` — le reste du /24 (`.248` → `.255`) reste **libre** pour la croissance.

> Outil de contrôle : `ipcalc 192.168.1.0/25` ou, en une ligne, `python3 -c "import ipaddress; n=ipaddress.ip_network('192.168.1.128/26'); print(list(n.hosts())[0], list(n.hosts())[-1], n.broadcast_address)"`.

---

## Vérification (comment savoir que le plan est bon)

- Chaque sous-réseau est **aligné** sur un multiple de sa taille (sinon chevauchement).
- `broadcast = adresse réseau + taille − 1` ; les plages **ne se recouvrent pas**.
- La somme des blocs alloués **tient** dans le bloc de départ.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Sous-réseaux qui se chevauchent | Allocation non alignée / pas du plus grand au plus petit | Reprendre l'ordre VLSM (décroissant) |
| Un hôte « manque » | Oubli du réseau et du broadcast (−2) | Recompter avec 2^h − 2 |
| Gaspillage d'adresses | Masque fixe (FLSM) au lieu de VLSM | Adapter le masque à chaque segment |

## Sécurité et bonnes pratiques

- **Segmenter par usage** (un VLAN/sous-réseau par population : serveurs, postes, gestion, DMZ).
- **Prévoir la croissance** (marge sur chaque segment).
- **Documenter** le plan (indispensable à l'exploitation et au diagnostic).
- Réserver des plages **statiques** hors de l'étendue **DHCP**.

## ⚠️ À ne pas confondre / obsolète

- **VLSM** (masques variables, sans gaspillage) ≠ **FLSM** (masque unique).
- Liens point-à-point : `/30` (2 hôtes) ou `/31` (**RFC 3021**, 2 adresses sans broadcast) pour économiser.
- Ne pas confondre **masque décimal** (255.255.255.192) et **CIDR** (/26) : ce sont deux écritures du même masque.

---

## Sources (référence)

- [RFC 1918 — Address Allocation for Private Internets](https://www.rfc-editor.org/rfc/rfc1918) — consulté le 24/07/2026
- [RFC 3021 — Using 31-Bit Prefixes on IPv4 Point-to-Point Links](https://www.rfc-editor.org/rfc/rfc3021) — consulté le 24/07/2026
- [RFC 4632 — CIDR](https://www.rfc-editor.org/rfc/rfc4632) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] Méthode (pas de GUI/CLI) · [x] rappels datés/normés · [x] rien d'obsolète (/31 RFC 3021) · [x] **table VLSM vérifiée programmatiquement** · [x] conforme RFC · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
