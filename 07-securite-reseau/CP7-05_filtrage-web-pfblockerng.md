# CP7-05 — Configurer le filtrage web (pfBlockerNG ; Squid déprécié)

**Objectif** : filtrer l'accès web des postes (publicités, domaines malveillants, catégories) sur pfSense.

**Rattachement REAC** : CP7 « Maintenir et sécuriser les accès Internet et les interconnexions » — savoir-faire : filtrer et sécuriser les accès web.

**Durée** : ~30 min · **Niveau** : intermédiaire.

---

## Prérequis

- pfSense opérationnel (**CP7-01**), le **DNS Resolver** actif, accès Internet pour les listes.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| pfSense CE | 2.8.1 — paquet **pfBlockerNG** | 24/07/2026 |
| Configuration | **à tester en lab** | 24/07/2026 |

> ⚠️ **Actualité importante** : **Squid, SquidGuard et Lightsquid sont DÉPRÉCIÉS** sur pfSense CE/Plus (failles amont non corrigées) — Netgate **recommande de les désinstaller** et ils cesseront de fonctionner dans les prochaines versions. Le filtrage web moderne se fait avec **pfBlockerNG (DNSBL)**.

---

## Procédure — GUI (pfBlockerNG / DNSBL)

1. **System → Package Manager → Available Packages** : installer **pfBlockerNG**.
2. Lancer l'**assistant** ; activer **DNSBL** (filtrage par résolution DNS).
3. **Feeds / catégories** : activer des listes de blocage (publicité, malware, phishing, catégories) ; planifier leur **mise à jour** (cron).
4. **Forcer le DNS interne** : les clients doivent utiliser le **DNS Resolver de pfSense** ; **rediriger/bloquer** le port **53** sortant pour empêcher de contourner le filtre.
5. *(Option)* **IP filtering** pfBlockerNG : bloquer des plages d'IP / pays malveillants en entrée/sortie.

> Le filtrage **par DNS** fonctionne aussi pour le **HTTPS** (blocage à la résolution du nom, **sans** déchiffrer le trafic).

---

## Vérification (comment savoir que ça marche)

- Un domaine présent dans une liste renvoie la **page de blocage / NXDOMAIN**.
- **Reports** de pfBlockerNG : compteurs de requêtes bloquées (hits).
- Un poste qui tente un **autre DNS** (8.8.8.8) est **redirigé/bloqué** (port 53 forcé).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Aucun filtrage | Clients n'utilisent pas le DNS pfSense | Distribuer/forcer le DNS interne, bloquer le 53 sortant |
| Listes vides | Mise à jour non effectuée | Forcer *Update* ; vérifier l'accès Internet |
| Faux positifs | Domaine légitime en liste | Ajouter une **whitelist** |
| HTTPS « non filtré » | Attente d'un blocage par proxy | Le DNSBL bloque **au niveau du nom**, c'est normal |

## Sécurité et bonnes pratiques

- **Forcer le DNS interne** (sinon le filtrage est contournable).
- Bloquer **malware/phishing** en amont via les feeds ; tenir les listes **à jour**.
- Le filtrage implique une **traçabilité** : informer les utilisateurs (charte — **CP1-10**, RGPD — **CT4-01**).

## ⚠️ À ne pas confondre / obsolète

- **Squid / SquidGuard / Lightsquid = dépréciés** (à désinstaller) → **pfBlockerNG (DNSBL)**.
- Le **filtrage DNS** (simple, respectueux) est préférable au **proxy HTTPS avec SSL bump** (intrusif, complexe, casse la confidentialité).
- Un **proxy cache** (Squid) n'est plus la voie recommandée sur pfSense.

---

## Sources (doc officielle)

- [Netgate — pfBlockerNG](https://docs.netgate.com/pfsense/en/latest/packages/pfblocker.html) — consulté le 24/07/2026
- [Netgate — Deprecated packages (Squid/SquidGuard/Lightsquid)](https://docs.netgate.com/pfsense/en/latest/releases/index.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI · [x] version datée · [x] **rien d'obsolète (Squid déprécié → pfBlockerNG)** · [x] config à tester en lab · [x] conforme doc Netgate · [x] vérification présente · [x] sécurité · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
