# CP7-03 — Configurer le NAT (redirection de ports, outbound)

**Objectif** : configurer le NAT sur pfSense — redirection de ports entrants (DNAT) et NAT sortant (SNAT).

**Rattachement REAC** : CP7 « Maintenir et sécuriser les accès Internet et les interconnexions » — savoir-faire : gérer la traduction d'adresses.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- pfSense opérationnel (**CP7-01**), notions de règles (**CP7-02**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| pfSense CE | 2.8.1 | 24/07/2026 |
| Configuration | **à tester en lab** | 24/07/2026 |

## Les types de NAT

| Type | Rôle | Où |
|---|---|---|
| **Outbound (SNAT)** | le LAN sort vers Internet avec l'IP **WAN** | Firewall → NAT → *Outbound* (**Automatic** par défaut) |
| **Port Forward (DNAT)** | exposer un **service interne** depuis Internet | Firewall → NAT → *Port Forward* |
| **1:1** | mapper **une IP publique ↔ une IP interne** | Firewall → NAT → *1:1* |

---

## Procédure — GUI

### Redirection de ports (exposer un serveur web interne)

1. **Firewall → NAT → Port Forward → Add**.
2. **Interface** : WAN · **Protocol** : TCP · **Destination port** : 443.
3. **Redirect target IP** : `192.168.20.10` (serveur) · **Redirect target port** : 443.
4. Cocher **Add associated filter rule** (pfSense crée la règle firewall WAN correspondante).
5. **Save** → **Apply**.

### NAT sortant (personnaliser)

- **Firewall → NAT → Outbound** : passer de **Automatic** à **Hybrid** pour ajouter des règles spécifiques (ex. forcer une IP source pour un sous-réseau) sans perdre les règles automatiques.

---

## Vérification (comment savoir que ça marche)

- Depuis **l'extérieur**, le service interne (443) **répond**.
- **Diagnostics → States** : la session NAT apparaît.
- **Status → System Logs → Firewall** : le trafic est bien traité.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Le port forward ne marche pas | Règle firewall WAN absente | Cocher **Add associated filter rule** |
| Toujours injoignable | Mauvaise IP/port cible ou service arrêté | Vérifier la cible et l'écoute (`ss -tulpn`) |
| Marche dehors, pas depuis le LAN | NAT reflection désactivée | Activer **NAT Reflection** (ou split-DNS) |
| Rien ne sort | Double NAT (derrière la box) | Mettre la box en *bridge* ou gérer le double NAT |

## Sécurité et bonnes pratiques

- **N'exposer que le strict nécessaire** ; restreindre la **source** quand c'est possible.
- Placer les services exposés dans une **DMZ** (**CP7-04**).
- **Journaliser** les accès entrants ; surveiller les tentatives.

## ⚠️ À ne pas confondre / obsolète

- **Port Forward** (entrant, **DNAT**) ≠ **Outbound** (sortant, **SNAT**).
- Le **NAT** (traduction) et la **règle firewall** (autorisation) sont **deux choses distinctes** : la case *associated filter rule* les relie.
- **NAT Reflection** est nécessaire pour atteindre un service exposé **depuis l'intérieur** via l'IP publique.

---

## Sources (doc officielle)

- [Netgate — Network Address Translation](https://docs.netgate.com/pfsense/en/latest/nat/index.html) — consulté le 24/07/2026
- [Netgate — Port Forwards](https://docs.netgate.com/pfsense/en/latest/nat/port-forwards.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI · [x] version datée · [x] rien d'obsolète (DNAT/SNAT, reflection) · [x] config à tester en lab · [x] conforme doc Netgate · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
