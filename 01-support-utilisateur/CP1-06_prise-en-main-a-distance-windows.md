# CP1-06 — Prise de main à distance Windows (Assistance rapide + Bureau à distance/RDP)

**Objectif** : dépanner un poste Windows à distance avec **Assistance rapide** (Quick Assist, avec l'utilisateur) et **Bureau à distance** (RDP), en respectant la sécurité.

**Rattachement REAC** : CP1 « Assurer le support utilisateur » — savoir-faire : prendre la main à distance.

**Durée** : ~20 min · **Niveau** : débutant/intermédiaire.

---

## Prérequis

- Postes **Windows 11 24H2** / serveurs **WS 2025**, connectivité réseau.
- Pour RDP : droits admin sur la cible ou appartenance au groupe **Utilisateurs du Bureau à distance**.

## Environnement testé

| Élément | Version validée | Date |
|---|---|---|
| Assistance | **Assistance rapide** (Quick Assist) | 24/07/2026 |
| Bureau à distance | **RDP** (`mstsc.exe`), **NLA** | 24/07/2026 |
| Procédure appliance | **à tester en lab** | 24/07/2026 |

> **Deux usages** : **Assistance rapide** = dépannage **ponctuel avec l'utilisateur présent** (il voit et autorise). **Bureau à distance (RDP)** = session à distance (souvent serveur/admin), avec ou sans l'utilisateur.

---

## Procédure — GUI

### Assistance rapide (Quick Assist)

1. Les deux ouvrent **Assistance rapide** (intégrée à Windows 11 / Microsoft Store).
2. L'**aidant** clique *Aider quelqu'un* → obtient un **code de sécurité** ; l'**utilisateur** le saisit et **autorise** le partage/contrôle.
3. La session passe par le service Microsoft en **HTTPS (443) / TLS** (pas de port à ouvrir).

### Bureau à distance (RDP)

1. Sur la **cible** : **Paramètres ▸ Système ▸ Bureau à distance** → **Activer** (ou par **GPO**), laisser **NLA** activé.
2. Depuis le poste technicien : **`mstsc.exe`** → adresse de la cible → identifiants.

## Procédure — CLI (activer RDP sur la cible)

```powershell
# Activer le Bureau à distance + NLA + règle pare-feu
Set-ItemProperty 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name fDenyTSConnections -Value 0
Set-ItemProperty 'HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp' -Name UserAuthentication -Value 1  # NLA
Enable-NetFirewallRule -DisplayGroup "Bureau à distance"
```

---

## Vérification (comment savoir que ça marche)

- **Assistance rapide** : l'aidant voit/contrôle l'écran **après autorisation** de l'utilisateur.
- **RDP** : la mire d'authentification apparaît, puis le **bureau distant** s'affiche (`mstsc` connecté sur le **port 3389**).

## Dépannage (erreurs fréquentes)

| Symptôme | Cause probable | Solution |
|---|---|---|
| RDP refusé | Pare-feu **3389** / droits | Autoriser la règle ; ajouter au groupe *Bureau à distance* |
| « NLA requis » | Client trop ancien / creds | Utiliser des identifiants valides, NLA des deux côtés |
| Quick Assist ne se lance pas | App/Store/Internet | Installer/mettre à jour l'app ; accès Internet |
| Fichier `.rdp` douteux | **Hameçonnage** (CVE-2026-26151) | Ne pas ouvrir un `.rdp` non sollicité |

## Sécurité et bonnes pratiques

- **NLA obligatoire** ; **ne jamais exposer RDP directement sur Internet** → **VPN** (**CP7-07**) ou **passerelle RDS**, idéalement **MFA** (**CP7-18**).
- **Restreindre** le groupe *Utilisateurs du Bureau à distance* (moindre privilège).
- **Assistance rapide** : toujours **avec le consentement** de l'utilisateur ; se méfier des **arnaques au support** (ne jamais donner le contrôle à un inconnu).
- Se méfier des **fichiers `.rdp`** reçus (spoofing — **CVE-2026-26151**, correctif avril 2026).

## ⚠️ À ne pas confondre / obsolète

- **Assistance à distance** historique (`msra`) **dépréciée** → **Assistance rapide** (Quick Assist).
- L'**app « Bureau à distance »** autonome (Store/MSI) est **retirée (27/03/2026)** → **Windows App** ; mais l'utilitaire intégré **`mstsc.exe`** **reste supporté**.
- **RDP** (contrôle total, admin/serveur) ≠ **Assistance rapide** (accompagnement ponctuel).

---

## Sources (doc officielle)

- [Microsoft — Assistance rapide (Quick Assist)](https://support.microsoft.com/windows/résoudre-les-problèmes-avec-l-assistance-rapide) — consulté le 24/07/2026
- [Microsoft Learn — Bureau à distance : activer/se connecter](https://learn.microsoft.com/fr-fr/windows-server/remote/remote-desktop-services/clients/remote-desktop-allow-access) — consulté le 24/07/2026
- [Microsoft — Retrait du client Bureau à distance (Windows App)](https://learn.microsoft.com/en-us/windows-app/) — consulté le 24/07/2026

## Validation (checklist §7 du plan)

- [x] GUI puis CLI (PowerShell) · [x] versions datées (Win 11 24H2) · [x] rien d'obsolète (msra déprécié, app RD retirée, mstsc OK) · [x] procédure **à tester en lab** · [x] conforme doc Microsoft · [x] vérification présente · [x] sécurité (NLA, pas de RDP exposé, .rdp) · [x] sources · [x] rattachement REAC · [x] statut mis à jour dans le plan
