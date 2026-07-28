# CP4-07 — Diagnostiquer une panne réseau par couche (modèle OSI)

**Objectif** : appliquer une démarche structurée, couche par couche (modèle OSI), pour localiser rapidement l'origine d'une panne réseau.

**Rattachement REAC** : CP4 « Exploiter un réseau IP » — savoir-faire (méthode) : diagnostiquer méthodiquement un incident réseau.

**Durée** : ~15 min · **Niveau** : intermédiaire.

---

## Prérequis

- Bases IP (**CP4-01/02**) et accès aux postes/équipements concernés.

## Principe

On isole la panne en testant les couches **de bas en haut** (physique → application). Chaque couche a ses symptômes et ses commandes.

| Couche OSI | On vérifie… | Outils |
|---|---|---|
| 1 — Physique | câble, port, LED de lien, SFP | voyants ; `ip link` (state UP/DOWN) |
| 2 — Liaison | VLAN, MAC, ARP, duplex | `ip neigh` ; switch : `show mac address-table`, `show vlan` |
| 3 — Réseau | IP, masque, passerelle, routage | `ip addr`, `ip route`, `ping`, `traceroute` |
| 4 — Transport | ports, pare-feu | `ss -tulpn`, `Test-NetConnection -Port`, `nc` |
| 5-7 — Session→Appli | DNS, service, application | `dig`/`nslookup`, `curl`, journaux (`journalctl`) |

---

## Démarche structurée (méthode)

1. **Cadrer** l'incident : qui, quoi, depuis quand, périmètre (QQOQCP — **CP1-08**). Un seul poste ? tout un VLAN ? tout le site ?
2. **Reproduire** et **isoler** : comparer un poste qui marche / un poste en panne.
3. **Tester de bas en haut** :
   - Lien physique **UP** ? (`ip link`)
   - IP correcte (pas de **169.254.x.x**) ? (`ip addr`)
   - Passerelle joignable ? (`ping` passerelle)
   - IP distante joignable ? (`ping` serveur, `traceroute`)
   - Nom résolu ? (`dig`/`nslookup`)
   - Port du service ouvert ? (`Test-NetConnection -Port`, `ss`)
   - Service applicatif **up** ? (journaux)
4. **Un changement à la fois**, en notant chaque test.
5. **Corriger**, **vérifier**, puis **documenter** le compte rendu (**CP1-09**).

## Exemple — « je n'accède pas à l'intranet »

Lien UP → IP OK (pas APIPA) → `ping` passerelle OK → `ping` IP serveur OK → **`nslookup intranet.lab.local` échoue** ⇒ panne **DNS** (couche 7 côté résolution) : corriger le serveur DNS distribué, `ipconfig /flushdns`.

## Vérification

- À chaque couche franchie, **un test** confirme (lien up, ping, résolution, port, réponse applicative).

## Dépannage (repères rapides)

| Symptôme | Couche probable | Premier réflexe |
|---|---|---|
| Pas de lien / port down | 1 | Câble, port, voyants |
| `169.254.x.x` | 3 (DHCP) | Vérifier DHCP/lien |
| Ping IP OK, nom KO | 7 (DNS) | Serveur DNS, `flushdns` |
| Ping OK mais appli KO | 4-7 | Port/pare-feu, service, journaux |

## Sécurité et bonnes pratiques

- **Documenter** le diagnostic et la résolution (traçabilité, base de connaissances — **CP1-05**).
- Ne pas modifier la production **à l'aveugle** : un changement testé à la fois.

## ⚠️ À ne pas confondre / obsolète

- **Modèle OSI** (7 couches, pédagogique) ≠ **modèle TCP/IP** (4 couches, réel) : le premier aide à raisonner.
- **169.254.x.x** = problème **couche 3/DHCP**, pas « pas de réseau du tout ».
- « Ça ne marche pas » n'est pas un diagnostic : **isoler la couche** d'abord.

---

## Sources (référence)

- [Cloudflare — Le modèle OSI](https://www.cloudflare.com/fr-fr/learning/ddos/glossary/open-systems-interconnection-model-osi/) — consulté le 24/07/2026
- [Cisco — Troubleshooting methodology (OSI)](https://www.cisco.com/c/en/us/support/docs/ip/routing-information-protocol-rip/13730-ext-ping-trace.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] Méthode (pas de GUI/CLI dédiés) · [x] repères datés · [x] rien d'obsolète · [x] commandes de test issues de tutos validés (CP4-02, CP3-06) · [x] conforme modèle OSI · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
