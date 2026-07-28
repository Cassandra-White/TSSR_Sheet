# CP7-11 — Mettre en place une PKI et gérer des certificats

**Objectif** : créer une autorité de certification (**CA**) interne, émettre un certificat serveur (clé + CSR + signature), le vérifier et savoir le **révoquer** — pour sécuriser TLS, VPN et l'accès aux consoles.

**Rattachement REAC** : CP7 « Maintenir et sécuriser les accès Internet et les interconnexions » — savoir-faire : gérer les certificats et l'authentification.

**Durée** : ~30 min · **Niveau** : avancé.

---

## Prérequis

- Un pare-feu **pfSense** (**CP7-01**) **ou** un serveur Windows **AD DS** (**CP2-03**) pour la voie GUI.
- Un poste avec **OpenSSL** pour la voie CLI.
- Une convention de nommage DNS interne (ex. `*.lab.local`).

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| PKI en ligne de commande | **OpenSSL 3.x** | 24/07/2026 |
| PKI GUI (pare-feu) | pfSense CE **2.8.1** — Cert Manager | 24/07/2026 |
| PKI GUI (Windows) | **AD CS** (rôle) — Windows Server 2025 | 24/07/2026 |

> **Principe PKI** : une **CA racine** (clé privée = secret le plus précieux) **signe** les certificats des serveurs/utilisateurs. Les clients qui font **confiance à la CA une fois** acceptent ensuite tous les certificats qu'elle émet. Racine **longue durée**, certificats feuilles **courts**.

---

## Procédure — GUI

### Option A — pfSense (System ▸ Cert Manager)

1. Onglet **CAs ▸ Add** : *Method* = **Create an internal CA**, renseigner CN (`LabTSSR-Root-CA`), clé **RSA 4096** ou **ECDSA P-256**, durée longue (ex. 3650 j). **Save**.
2. Onglet **Certificates ▸ Add** : *Method* = **Create an internal Certificate**, choisir la CA, *Type* = **Server Certificate**, CN + **Subject Alternative Names (SAN)** = tous les noms DNS/IP du service. **Save**.
3. Le certificat est utilisable pour l'**interface web** (System ▸ Advanced), **OpenVPN** (**CP7-07**) ou **IPsec** (**CP7-06**).
4. **Révocation** : onglet **Certificate Revocation ▸ Add** une CRL liée à la CA, y ajouter le certificat compromis.

### Option B — Windows Server (rôle AD CS)

1. **Server Manager ▸ Add Roles** → **Active Directory Certificate Services** → service **Certification Authority**.
2. Post-déploiement : **Enterprise CA** (intégrée à l'annuaire) → **Root CA** → nouvelle clé privée.
3. Console **certsrv.msc** : les modèles (*templates*) émettent les certificats ; les postes du domaine font **automatiquement confiance** à la CA (publiée dans AD → *Trusted Root*).
4. **Révocation** : clic droit sur le certificat ▸ **Revoke** ; la **CRL** est republiée automatiquement.

## Procédure — CLI (OpenSSL — testé en bac à sable)

```bash
# 1) CA racine : clé privée + certificat auto-signé (longue durée)
openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes \
  -keyout ca.key -out ca.crt \
  -subj "/C=FR/O=LabTSSR/CN=LabTSSR-Root-CA"

# 2) Serveur : clé privée + demande de signature (CSR)
openssl req -newkey rsa:2048 -sha256 -nodes \
  -keyout srv-web.key -out srv-web.csr \
  -subj "/C=FR/O=LabTSSR/CN=srv-web.lab.local"

# 3) Extensions — le SAN est OBLIGATOIRE (le CN seul n'est plus validé)
cat > srv-web.ext <<'EOF'
basicConstraints=CA:FALSE
keyUsage=digitalSignature,keyEncipherment
extendedKeyUsage=serverAuth
subjectAltName=DNS:srv-web.lab.local,DNS:www.lab.local,IP:192.168.10.10
EOF

# 4) La CA signe le CSR -> certificat serveur (durée <= 398 jours)
openssl x509 -req -in srv-web.csr -CA ca.crt -CAkey ca.key -CAcreateserial \
  -out srv-web.crt -days 397 -sha256 -extfile srv-web.ext
```

> **PKI à l'échelle** : pour émettre/renouveler automatiquement (courte durée, **ACME** comme Let's Encrypt mais en interne), utiliser **step-ca** (Smallstep). OpenSSL reste parfait pour comprendre et pour les cas ponctuels.

---

## Vérification (comment savoir que ça marche)

```bash
openssl verify -CAfile ca.crt srv-web.crt          # attendu : "srv-web.crt: OK"
openssl x509 -in srv-web.crt -noout -ext subjectAltName   # le SAN doit lister les noms/IP
openssl x509 -in srv-web.crt -noout -issuer -subject -dates
```

Résultats obtenus en bac à sable : `srv-web.crt: OK`, SAN = `DNS:srv-web.lab.local, DNS:www.lab.local, IP Address:192.168.10.10`, émetteur = `LabTSSR-Root-CA`.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| Navigateur : `NET::ERR_CERT_COMMON_NAME_INVALID` | **SAN absent** | Réémettre avec `subjectAltName` |
| `unable to get local issuer certificate` | CA non installée sur le client | Importer `ca.crt` dans le magasin *Autorités racines de confiance* |
| Certificat refusé pour sa durée | Validité **> 398 j** | Réémettre ≤ 397 j |
| Révocation ignorée | CRL/OCSP non publiée ou non atteignable | Publier la CRL et renseigner son URL (CDP) |

## Sécurité et bonnes pratiques

- **Protéger la clé de la CA racine** : hors-ligne si possible, accès très restreint ; sa compromission invalide toute la PKI.
- **Architecture recommandée** : CA racine **hors-ligne** + **CA intermédiaire** qui signe au quotidien.
- **Algorithmes actuels** : **RSA ≥ 2048** (ou **ECDSA P-256**), empreinte **SHA-256** minimum.
- **Cycle de vie** : certificats **courts**, renouvellement anticipé, **révocation** (CRL/OCSP) dès qu'une clé fuit ; surveiller les dates d'expiration.

## ⚠️ À ne pas confondre / obsolète

- **CN seul = obsolète** : les clients modernes valident le **SAN** — toujours le renseigner.
- **SHA-1** et **MD5** : proscrits pour les signatures.
- Certificat **auto-signé** unique (avertissement navigateur permanent) ≠ **PKI** (une CA de confiance signe des feuilles vérifiables).
- **Validité > 398 jours** pour un certificat TLS public : rejetée par les navigateurs.

---

## Sources (doc officielle)

- [pfSense — Certificate Manager](https://docs.netgate.com/pfsense/en/latest/certificates/index.html) — consulté le 24/07/2026
- [Microsoft Learn — Active Directory Certificate Services (AD CS)](https://learn.microsoft.com/en-us/windows-server/identity/ad-cs/active-directory-certificate-services-overview) — consulté le 24/07/2026
- [OpenSSL — Manuel `openssl-req` / `openssl-x509`](https://docs.openssl.org/master/man1/openssl-req/) — consulté le 24/07/2026
- [Smallstep — step-ca (PKI/ACME interne)](https://smallstep.com/docs/step-ca/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI · [x] versions datées · [x] rien d'obsolète (SAN, SHA-256, ≤398 j) · [x] CLI **testée en bac à sable** (CA→CSR→signature→`verify OK`) · [x] GUI conforme doc Netgate/Microsoft · [x] vérification présente · [x] sécurité (protection CA, révocation) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
