# CP4-08 — Utiliser les outils d'analyse réseau (Wireshark, nmap)

**Objectif** : capturer et analyser du trafic (Wireshark / tcpdump / tshark) et cartographier un réseau (nmap), pour le diagnostic et l'audit.

**Rattachement REAC** : CP4 « Exploiter un réseau IP » — savoir-faire : analyser le trafic et inventorier un réseau.

**Durée** : ~25 min · **Niveau** : intermédiaire.

> ⚠️ **Usage responsable** : n'analyser et ne scanner **que** des réseaux dont on est **responsable ou explicitement autorisé** à auditer. Un scan non autorisé peut être **illégal**.

---

## Prérequis

- Un poste (Windows/Linux), droits admin/root. Paquets : `wireshark`, `tcpdump`, `nmap`.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| `tcpdump` | présent dans le bac à sable (capture **à tester en lab** : privilèges) | 24/07/2026 |
| Wireshark / nmap | **à tester en lab** (paquets absents du bac à sable) | 24/07/2026 |

---

## Procédure — GUI (Wireshark)

1. Lancer **Wireshark**, choisir l'**interface** à écouter, démarrer la capture.
2. **Filtre d'affichage** (barre verte) : `ip.addr == 192.168.10.5`, `tcp.port == 443`, `dns`, `icmp`, `http`.
3. Clic droit sur un paquet → **Follow → TCP Stream** pour reconstituer un échange.
4. Arrêter la capture ; **Fichier → Enregistrer** (`.pcapng`) pour analyse ultérieure.

## Procédure — CLI

### Capture (tcpdump / tshark)

```bash
sudo tcpdump -i eth0 -n icmp                 # pings
sudo tcpdump -i eth0 -n host 192.168.10.5    # un hôte
sudo tcpdump -i eth0 -n port 80 -w capture.pcap   # écrire pour Wireshark
sudo tshark  -i eth0 -f "tcp port 443"       # capture filtrée en CLI
```

### Découverte / audit (nmap)

```bash
nmap -sn 192.168.10.0/24            # hôtes actifs (ping scan, pas de ports)
sudo nmap -sS -p 1-1000 192.168.10.5   # ports TCP (SYN scan, root)
nmap -sV 192.168.10.5               # versions des services
sudo nmap -O 192.168.10.5           # empreinte du système d'exploitation
```

---

## Vérification (comment savoir que ça marche)

- Wireshark/tcpdump affichent les **paquets attendus** (ex. l'echo-request/reply d'un `ping`).
- `nmap -sn` liste les **hôtes actifs** ; `nmap -sV` renvoie l'état des **ports** et les services — à comparer à l'inventaire attendu.

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| « No interfaces / permission denied » | Droits insuffisants | Root/administrateur ; groupe `wireshark` (dumpcap) |
| Aucune trame capturée | Mauvaise interface / pas de trafic | Choisir la bonne interface ; générer du trafic |
| nmap ne voit « rien » | Pare-feu bloquant / hôtes filtrés | `-Pn` (sauter la découverte), ajuster les options |
| Trop de bruit | Absence de filtre | Filtres de capture (`-f`) et d'affichage |

## Sécurité et bonnes pratiques

- **Autorisation obligatoire** avant tout scan/capture (cadre légal, charte).
- Les captures `.pcap` contiennent des **données sensibles** (identifiants, RGPD) : les **protéger** et les purger.
- Réaliser ces analyses **en lab** ou lors d'audits cadrés.

## ⚠️ À ne pas confondre / obsolète

- **Capture** (écoute passive : Wireshark/tcpdump) ≠ **scan** (sonde active : nmap).
- **Wireshark** (GUI) et **tshark/tcpdump** (CLI) partagent le même moteur (libpcap).
- Le **SYN scan** (`-sS`) nécessite les droits **root** ; sinon nmap bascule en `-sT` (connect).

---

## Sources (doc officielle)

- [Wireshark — User's Guide](https://www.wireshark.org/docs/wsug_html_chunked/) — consulté le 24/07/2026
- [Nmap — Reference Guide](https://nmap.org/book/man.html) — consulté le 24/07/2026
- [tcpdump(1) — Manuel officiel](https://www.tcpdump.org/manpages/tcpdump.1.html) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI (Wireshark) puis CLI (tcpdump/tshark/nmap) · [x] versions/ref datées · [x] rien d'obsolète · [x] tcpdump présent / capture-scan à tester en lab · [x] conforme doc · [x] vérification présente · [x] **sécurité + cadre légal** · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
