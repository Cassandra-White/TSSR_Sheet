# CP7-02 — Créer et tester des règles de filtrage (firewall)

**Objectif** : écrire, ordonner et tester des règles de filtrage sur pfSense.

**Rattachement REAC** : CP7 « Maintenir et sécuriser les accès Internet et les interconnexions » — savoir-faire : filtrer le trafic réseau.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- pfSense installé et opérationnel (**CP7-01**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| pfSense CE | 2.8.1 | 24/07/2026 |
| Configuration | **à tester en lab** | 24/07/2026 |

## Principes des règles pfSense

- Les règles sont définies **par interface** et évaluées **de haut en bas** : **la première qui correspond gagne** (les suivantes sont ignorées).
- Une règle s'applique au trafic **entrant sur l'interface** (les règles **LAN** filtrent le trafic **venant du LAN**).
- Par défaut : **WAN** bloque tout entrant · **LAN** = « autoriser vers n'importe où ».
- pfSense est **stateful** : le trafic **retour** est autorisé automatiquement (pas de règle inverse à créer).

---

## Procédure — GUI

### Créer un alias (regrouper des objets)

**Firewall → Aliases → Add** : ex. `Serveurs_Web` = 192.168.20.10, 192.168.20.11.

### Créer une règle

1. **Firewall → Rules →** onglet de l'interface (ex. **LAN**) → **Add**.
2. **Action** : *Pass* / *Block* / *Reject*.
3. **Protocol**, **Source**, **Destination** (ou l'alias), **Destination port**.
4. Cocher **Log** si l'on veut tracer les correspondances ; **Description** claire.
5. **Save** puis **Apply Changes**. **Ordonner** les règles (les plus spécifiques en haut).

### Exemple

- Autoriser **LAN → `Serveurs_Web` : 443** (Pass, TCP).
- **Bloquer** un poste vers un réseau sensible (Block, placé **au-dessus** de la règle « allow any »).

---

## Vérification (comment savoir que ça marche)

- Depuis un poste : la connexion **autorisée** aboutit, la **bloquée** échoue.
- **Diagnostics → States** : les connexions actives.
- **Status → System Logs → Firewall** : les paquets passés/bloqués (si *Log* activé).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Règle sans effet | Une règle **au-dessus** correspond déjà | Revoir l'**ordre** (première correspondance gagne) |
| Rien dans les logs | *Log* non coché | Activer *Log* sur la règle |
| Blocage inattendu | Trafic retour ? | Non : pfSense est **stateful**, le retour est géré |
| Verrouillage admin | Règle trop restrictive | Anti-lockout ; corriger via console |

## Sécurité et bonnes pratiques

- **Moindre autorisation** : bloquer par défaut, **n'ouvrir que le nécessaire**.
- **Aliases** pour des règles lisibles et maintenables.
- **Journaliser** les blocages ; documenter chaque règle (Description).

## ⚠️ À ne pas confondre / obsolète

- Les règles filtrent le trafic **entrant sur l'interface** : bien raisonner « d'où vient le paquet ».
- **Block** (rejet **silencieux**, drop) ≠ **Reject** (renvoie une erreur à l'émetteur).
- Inutile de créer une règle « retour » : le **stateful** s'en charge.

---

## Sources (doc officielle)

- [Netgate — Firewall Rule Basics](https://docs.netgate.com/pfsense/en/latest/firewall/index.html) — consulté le 24/07/2026
- [Netgate — Firewall Rule Processing Order](https://docs.netgate.com/pfsense/en/latest/firewall/rule-methodology.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI · [x] version datée · [x] rien d'obsolète (stateful, Block/Reject) · [x] config à tester en lab · [x] conforme doc Netgate · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
