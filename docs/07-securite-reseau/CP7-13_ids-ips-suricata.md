# CP7-13 — Mettre en place un IDS/IPS (Suricata) sur le pare-feu

**Objectif** : installer **Suricata** sur pfSense, charger les règles **Emerging Threats**, détecter (IDS) puis **bloquer** (IPS inline) le trafic malveillant.

**Rattachement REAC** : CP7 « Maintenir et sécuriser les accès Internet et les interconnexions » — savoir-faire : détecter et prévenir les intrusions.

**Durée** : ~35 min · **Niveau** : avancé.

---

## Prérequis

- Pare-feu **pfSense** opérationnel (**CP7-01**) avec accès Internet (téléchargement des règles).
- Journalisation/horloge en place (**CP7-10**, **CP4-13**) pour exploiter les alertes.
- Carte réseau compatible **netmap** pour le mode **IPS inline**.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Pare-feu | pfSense CE **2.8.1** | 24/07/2026 |
| Moteur | **Suricata 7.x** (paquet pfSense) | 24/07/2026 |
| Règles | **ET Open** (Emerging Threats, gratuit) | 24/07/2026 |
| Config appliance | **à tester en lab** | 24/07/2026 |

> **IDS ≠ IPS.** En **IDS** (détection) Suricata **observe** une copie du trafic et **alerte**. En **IPS inline** il s'insère dans le chemin des paquets (**netmap**) : chaque paquet passe par le moteur avant décision **pass/drop** → il **bloque** en temps réel.

---

## Procédure — GUI (pfSense)

1. **System ▸ Package Manager ▸ Available Packages** → installer **Suricata**.
2. **Services ▸ Suricata ▸ Global Settings** :
   - cocher **ET Open Emerging Threats** (et **Snort GPLv2** / **ET Pro** si `oinkcode`),
   - définir l'**intervalle de mise à jour** des règles (ex. quotidien).
3. Onglet **Updates ▸ Update** : télécharger les jeux de règles.
4. Onglet **Interfaces ▸ Add** : choisir l'interface (**WAN** pour voir les attaques venant d'Internet), **Enable**.
   - **Block Offenders** = coché, **IPS Mode = Inline** (blocage) — ou laisser en **Legacy/détection** pour n'alerter d'abord.
5. Onglet **Categories** de l'interface : sélectionner les catégories de règles pertinentes.
6. Onglet **Interfaces** : **démarrer** Suricata (flèche verte) sur l'interface.

> **Bonne pratique** : démarrer en **IDS (détection seule)** quelques jours, repérer les **faux positifs**, les **supprimer** (onglet *Suppress*), **puis** basculer en **IPS inline**.

## Procédure — CLI (console pfSense / FreeBSD)

```sh
# Mettre à jour les règles
suricata-update

# Vérifier la configuration AVANT de (re)démarrer  (équivaut au -T)
suricata -T -c /usr/local/etc/suricata/suricata.yaml -v

# Suivre les alertes en direct (journal EVE au format JSON)
tail -f /var/log/suricata/suricata_*/eve.json
```

> Sur pfSense, la gestion se fait surtout via la **GUI** ; la CLI sert au **diagnostic** et à automatiser `suricata-update` (cron nocturne + rechargement des règles).

---

## Vérification (comment savoir que ça marche)

- Depuis un poste **derrière** le pare-feu, générer un trafic de test connu des règles ET :

```sh
curl http://testmynids.org/uid/index.html   # déclenche une alerte ET "id check returned root"
```

- **Services ▸ Suricata ▸ Alerts** : l'alerte correspondante doit apparaître (signature, IP source/dest).
- En mode **IPS inline**, la connexion de test doit en plus être **bloquée** (IP dans la liste des *Blocked*).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Aucune alerte | Règles non téléchargées / interface non démarrée | Onglet **Updates**, puis démarrer Suricata sur l'interface |
| IPS inline indisponible | NIC **non compatible netmap** | Utiliser le mode **Legacy** (détection + blocage par table) |
| Blocage de trafic légitime | Faux positif | **Supprimer** la règle (Suppress List) ou l'affiner |
| CPU/RAM élevés | Trop de catégories actives | Ne garder que les catégories utiles ; matériel adapté |

## Sécurité et bonnes pratiques

- **Mettre à jour les règles automatiquement** (menace en évolution constante) et **surveiller** les alertes (**CP4-17/CP7-10**).
- **Positionnement** : WAN pour les attaques externes, **LAN/DMZ** pour détecter les mouvements internes.
- **Défense en profondeur** : l'IDS/IPS **complète** le filtrage (**CP7-02**), il ne le remplace pas.
- Basculer en IPS **après** réglage des faux positifs pour éviter de couper un service légitime.

## ⚠️ À ne pas confondre / obsolète

- **IDS** (détecte/alerte) ≠ **IPS** (bloque en ligne).
- **Suricata** (multi-thread, natif ET Open) vs **Snort 2.9** (mono-thread) : Suricata recommandé pour les nouveaux déploiements sur pfSense.
- Un IDS/IPS **ne déchiffre pas** le TLS par défaut : il voit les métadonnées, pas le contenu chiffré.

---

## Sources (doc officielle)

- [Netgate — Suricata (paquet pfSense)](https://docs.netgate.com/pfsense/en/latest/packages/suricata/index.html) — consulté le 24/07/2026
- [Suricata — Documentation officielle](https://docs.suricata.io/en/latest/) — consulté le 24/07/2026
- [Emerging Threats — Règles ET Open](https://rules.emergingthreats.net/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète (Suricata vs Snort mono-thread) · [x] CLI de contrôle fournie / config **à tester en lab** · [x] GUI conforme doc Netgate · [x] vérification présente (testmynids) · [x] sécurité (MàJ règles, faux positifs) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
