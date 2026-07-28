# CP1-11 — Configurer un téléphone IP / softphone (SIP)

**Objectif** : configurer un **téléphone IP** (matériel) ou un **softphone** (logiciel) et l'**enregistrer** sur un PBX via un compte **SIP**, puis tester un appel.

**Rattachement REAC** : CP1 « Assurer le support utilisateur » — savoir-faire : mettre en service la téléphonie IP.

**Durée** : ~20 min · **Niveau** : intermédiaire.

---

## Prérequis

- Un **PBX/serveur SIP** (ex. **Asterisk/FreePBX**) avec un **compte (extension)** créé.
- Un **softphone** (Linphone, Zoiper, MicroSIP) ou un **téléphone IP** matériel.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Softphones | **Linphone** (OSS, TLS/SRTP), Zoiper, MicroSIP | 24/07/2026 |
| PBX | **Asterisk / FreePBX** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **VoIP** = voix sur IP. Le **SIP** gère la **signalisation** (établir/terminer l'appel) ; la voix circule en **RTP**. Le téléphone (matériel ou **softphone**) **s'enregistre** auprès du PBX avec un **compte SIP** (serveur, utilisateur, mot de passe).

---

## Procédure — GUI (softphone, ex. Linphone/Zoiper)

1. **Installer** le softphone (multiplateforme).
2. **Ajouter un compte SIP** :
   - **Serveur/domaine** : IP ou FQDN du PBX,
   - **Utilisateur** : l'extension (ex. `4001`),
   - **Mot de passe** : le *secret* SIP,
   - **Transport** : **TLS** (recommandé) sinon UDP/TCP.
3. **Codecs** : `ulaw`/`alaw` (**G.711**) en LAN ; **G.729** en WAN faible débit.
4. Le statut passe à **« Enregistré / Registered »**.

## Téléphone IP matériel

1. Configurer le **réseau** (DHCP ou IP fixe, **VLAN voix** — **CP4-04**).
2. Saisir le **compte SIP** (mêmes champs) dans l'interface web du téléphone.

## Côté PBX (Asterisk — rappel)

```ini
[4001]
type=friend
secret=MotDePasseFortSIP
context=interne
disallow=all
allow=ulaw
allow=alaw
```

---

## Vérification (comment savoir que ça marche)

- Le compte affiche **Registered** (côté softphone) et l'extension est **en ligne** côté PBX.
- **Appel de test** : *echo test* du PBX, ou appel vers un autre poste → **audio dans les deux sens**.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Pas d'enregistrement | Identifiants/serveur | Vérifier extension, secret, adresse du PBX |
| Bloqué par le pare-feu | Ports SIP **5060/5061** | Autoriser SIP + plage **RTP** |
| Audio **unidirectionnel** | **NAT** / ports RTP | Configurer NAT/STUN ; ouvrir la plage RTP |
| Voix hachée | Débit/QoS | **VLAN voix** + **QoS** (**CP7-15**) ; codec adapté |

## Sécurité et bonnes pratiques

- **Chiffrer** : **TLS** (signalisation) + **SRTP** (voix) — le SIP en clair est écoutable.
- **VLAN voix dédié** (isolation + QoS) — **CP4-04/CP7-15**.
- **Mots de passe SIP forts** : les comptes SIP sont **très ciblés** par le bruteforce → **Fail2ban** sur le PBX (**CP3-18**).

## ⚠️ À ne pas confondre / obsolète

- **Softphone** (logiciel) ≠ **téléphone IP** (matériel) : même compte SIP.
- **SIP en clair (UDP 5060)** exposé = cible → **TLS/SRTP** + VLAN.
- **VoIP/SIP** (IP) remplace le **RTC/analogique** (obsolète, fin du RTC).

---

## Sources (doc officielle)

- [Asterisk — Configuring SIP Phones](https://docs.asterisk.org/) — consulté le 24/07/2026
- [Linphone — Documentation](https://www.linphone.org/technical-corner/linphone) — consulté le 24/07/2026
- [Zoiper — Configuration](https://www.zoiper.com/en/support) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI (+ config PBX) · [x] daté 24/07/2026 · [x] rien d'obsolète (TLS/SRTP, fin RTC) · [x] procédure **à tester en lab** · [x] conforme doc Asterisk/Linphone · [x] vérification présente (Registered + appel test) · [x] sécurité (chiffrement, VLAN, Fail2ban) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
