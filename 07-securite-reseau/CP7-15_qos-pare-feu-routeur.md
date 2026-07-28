# CP7-15 — Configurer la QoS (qualité de service) sur le pare-feu/routeur

**Objectif** : prioriser le trafic sensible (VoIP, visio) et **lisser la latence** (anti-bufferbloat) sur la liaison Internet, via le *traffic shaper* de pfSense.

**Rattachement REAC** : CP7 « Maintenir et sécuriser les accès Internet et les interconnexions » — savoir-faire : garantir la qualité de service.

**Durée** : ~25 min · **Niveau** : intermédiaire.

---

## Prérequis

- Pare-feu **pfSense** en coupure Internet (**CP7-01**).
- **Débits réels** montant/descendant de la liaison (mesurés, pas les débits commerciaux).
- Trafic à prioriser identifié (ex. **VoIP** — **CP1-11**).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Pare-feu | pfSense CE **2.8.1** | 24/07/2026 |
| Mécanismes | **Limiters (dummynet) + FQ_CoDel**, ALTQ (HFSC) | 24/07/2026 |
| Config appliance | **à tester en lab** | 24/07/2026 |

> **Bufferbloat** = mise en mémoire tampon excessive : quand le lien sature, les tampons débordent et la latence explose (VoIP hachée, navigation lente). **FQ_CoDel** répartit les flux et contrôle le délai de file → c'est l'algorithme **recommandé** contre le bufferbloat.

---

## Procédure — GUI (pfSense)

### Méthode A — Limiters + FQ_CoDel (anti-bufferbloat, recommandé)

1. **Firewall ▸ Traffic Shaper ▸ Limiters ▸ New Limiter**.
2. Créer **WAN-Down** : *Bandwidth* ≈ **90 %** du débit **descendant** réel, *Queue Management Algorithm* = **FQ_CoDel**, *Scheduler* = **FQ_CoDel**. (Régler ~90 % pour que **pfSense** garde la maîtrise de la file, pas la box FAI.)
3. Créer **WAN-Up** de la même façon avec le débit **montant**.
4. **Firewall ▸ Rules ▸ LAN** : sur la règle *pass* générale, section *In/Out pipe* → **In = WAN-Up**, **Out = WAN-Down** (les limiters s'appliquent au trafic traversant).

### Méthode B — File prioritaire pour la VoIP (ALTQ / assistant)

1. **Firewall ▸ Traffic Shaper ▸ Wizards ▸ Multiple LAN/WAN**.
2. Renseigner les interfaces et les **débits WAN**.
3. Étape **Voice over IP** : activer, choisir le fournisseur/plage — l'assistant crée une **file prioritaire** (HFSC) pour la signalisation SIP et la voix (RTP).
4. Ajuster le paramètre **quantum ≈ 300** pour favoriser les petits paquets VoIP. **Save**, puis vérifier les files dans **Firewall ▸ Traffic Shaper ▸ Queues**.

## Procédure — CLI (équivalent routeur Cisco IOS — MQC)

```text
class-map match-any VOIP
 match ip dscp ef                         ! marquage EF (Expedited Forwarding) = voix
policy-map WAN-OUT
 class VOIP
  priority percent 30                      ! file à priorité stricte (LLQ) pour la voix
 class class-default
  fair-queue                               ! équité pour le reste
interface GigabitEthernet0/0
 service-policy output WAN-OUT
```

> **Marquage DSCP** : marquer **EF** la voix au plus près de la source (téléphone/switch) puis **faire confiance** au marquage sur le routeur/pare-feu.

---

## Vérification (comment savoir que ça marche)

- **Test bufferbloat** : lancer un test de charge (ex. *Waveform Bufferbloat* / DSLReports) **pendant** un gros téléchargement : la latence doit rester basse (note **A**) au lieu de grimper.
- pfSense **Status ▸ Queues** (ou **Diagnostics ▸ Limiter Info**) : le trafic passe bien dans les files, peu ou pas de *drops* anormaux.
- Un **appel VoIP** reste net même quand le lien est saturé par un transfert.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| QoS sans effet | Débits WAN surestimés | Baisser à **~90 %** du débit **mesuré** |
| VoIP toujours hachée | Trafic non classé / non marqué | Vérifier la correspondance de règle et le **DSCP EF** |
| Limiter dans un seul sens | *In/Out pipe* incomplet | Affecter **In = up** et **Out = down** sur la règle |
| Latence encore haute | Mauvais algorithme | Utiliser **FQ_CoDel** (pas de simple FIFO) |

## Sécurité et bonnes pratiques

- La QoS relève de la **disponibilité** : elle garantit les services critiques quand le lien est chargé — la documenter dans la politique réseau (**CP4-11**).
- **Marquage cohérent** de bout en bout (téléphone → switch → pare-feu) ; ne pas faire confiance au DSCP venant d'Internet.
- **Ne pas sur-réserver** : la somme des files prioritaires doit laisser de la bande passante au trafic ordinaire.

## ⚠️ À ne pas confondre / obsolète

- **Limiters (dummynet)** = plafond de débit **par IP/réseau** ; **files ALTQ** = **priorité** relative entre classes — usages différents, parfois combinés.
- **FQ_CoDel** (moderne, anti-bufferbloat) ≫ simples files **FIFO/PRIQ** héritées.
- QoS ≠ « plus de débit » : elle **arbitre** la bande passante existante, elle ne l'augmente pas.

---

## Sources (doc officielle)

- [pfSense — Traffic Shaper](https://docs.netgate.com/pfsense/en/latest/trafficshaper/index.html) — consulté le 24/07/2026
- [pfSense — Limiters](https://docs.netgate.com/pfsense/en/latest/trafficshaper/limiters.html) — consulté le 24/07/2026
- [pfSense — CoDel Limiters (bufferbloat)](https://docs.netgate.com/pfsense/en/latest/recipes/codel-limiters.html) — consulté le 24/07/2026
- [Cisco — QoS: Modular QoS Command-Line Interface (MQC)](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/qos_mqc/configuration/15-mt/qos-mqc-15-mt-book.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (Cisco MQC) · [x] versions datées · [x] rien d'obsolète (FQ_CoDel vs FIFO/PRIQ) · [x] config **à tester en lab** · [x] GUI conforme doc Netgate · [x] vérification présente (test bufferbloat) · [x] sécurité (disponibilité, marquage) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
