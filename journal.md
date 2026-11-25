# 📓 Journal des incidents — Préproduction

Incidents rencontrés lors du déploiement de l'infrastructure en préproduction (VMware Workstation), avec leur diagnostic et leur résolution.

| Date | Composant | Incident | Statut |
|---|---|---|---|
| 2026-11-20 | `NCY-FW-01` | [Conflit d'adresses entre le WAN et le LAN](#2026-11-20--conflit-dadresses-entre-le-wan-et-le-lan) | Résolu |
| 2026-11-20 | `NCY-FW-01` | [Version de pfSense obsolète](#2026-11-20--version-de-pfsense-obsolète) | En cours |
| 2026-11-20 | `NCY-FW-01` | [Disposition de clavier de la console](#2026-11-20--disposition-de-clavier-de-la-console) | Contourné |
| 2026-11-20 | VMware | [Carte physique du mode Bridged non définie](#2026-11-20--carte-physique-du-mode-bridged-non-définie) | Résolu |
| 2026-11-11 | VMware | [Démarrage impossible sur l'image d'installation](#2026-11-11--démarrage-impossible-sur-limage-dinstallation) | Résolu |

---

## 2026-11-20 — Conflit d'adresses entre le WAN et le LAN

- **Contexte** : premier démarrage de pfSense, installé avec les paramètres par défaut.
- **Symptôme** : la console affiche le WAN (em0) en `192.168.1.22/24`, attribué par la box, et le LAN (em1) en `192.168.1.1/24`.
- **Diagnostic** : les deux interfaces appartiennent au même réseau `192.168.1.0/24`. Le pare-feu dispose alors de deux routes vers le même réseau et ne peut pas déterminer de quel côté acheminer le trafic.
- **Cause** : l'adresse LAN par défaut de pfSense (`192.168.1.1/24`) correspond au réseau de la plupart des box Internet, dont celle du site.
- **Solution** : console, option 2 (*Set interface(s) IP address*) : LAN en `10.10.10.1/24` (passerelle du VLAN 10 selon le [plan d'adressage](01-architecture/plan-adressage.md)), sans passerelle amont, sans IPv6, sans serveur DHCP, interface web conservée en HTTPS.
- **Vérification** : WAN en `192.168.1.22/24` et LAN en `10.10.10.1/24`, dans deux réseaux distincts.
- **Leçon retenue** : vérifier systématiquement l'absence de chevauchement entre le réseau interne et le réseau amont. C'est la raison du choix de la plage `10.0.0.0/8` pour l'ensemble du site.

## 2026-11-20 — Version de pfSense obsolète

- **Contexte** : pfSense déjà installé sur la VM lors d'une première tentative.
- **Symptôme** : la console affiche pfSense 2.6.0, version de 2022 qui ne reçoit plus de correctifs de sécurité.
- **Solution** : snapshot de la VM, puis mise à jour depuis la console (option 13), après correction de l'adresse LAN pour éviter tout problème de routage pendant le téléchargement. Première mise à jour vers 2.7.0 réussie, configuration conservée.
- **Reste à faire** : poursuivre les mises à jour successives jusqu'à la dernière version stable.
- **Leçon retenue** : un pare-feu doit fonctionner avec une version maintenue. La mise à jour se fait par étapes, et toujours après un point de restauration.

## 2026-11-20 — Disposition de clavier de la console

- **Contexte** : saisie d'adresses IP dans la console pfSense.
- **Symptôme** : les caractères saisis ne correspondent pas aux touches du clavier AZERTY.
- **Cause** : la console utilise une disposition de clavier américaine (QWERTY).
- **Contournement** : saisie des chiffres et du point sur le pavé numérique ; tests de résolution DNS avec des noms identiques dans les deux dispositions (`free.fr`).
- **Leçon retenue** : limiter l'usage de la console à la configuration initiale et au dépannage ; l'administration courante passe par l'interface web.

## 2026-11-20 — Carte physique du mode Bridged non définie

- **Contexte** : configuration de la carte WAN de la VM pfSense en mode Bridged.
- **Risque identifié** : le réseau VMnet0 était en mode *Automatic*. L'hôte dispose de plusieurs cartes réseau, dont une carte virtuelle Tailscale : VMware pouvait relier le WAN à une mauvaise carte, privant pfSense d'accès à la box.
- **Solution** : *Virtual Network Editor* : VMnet0 relié explicitement à la carte Ethernet physique (Realtek PCIe 2.5GbE).
- **Vérification** : le WAN obtient une adresse de la box (`192.168.1.22`) et accède à Internet.
- **Leçon retenue** : ne jamais laisser le choix automatique de l'interface physique sur un hôte équipé de cartes réseau virtuelles.

## 2026-11-11 — Démarrage impossible sur l'image d'installation

- **Contexte** : premier démarrage de la VM pfSense sous VMware Workstation.
- **Symptôme** : message « Cannot connect the virtual device ide1:0 because no corresponding device is available on the host ».
- **Diagnostic** : vérification des paramètres du lecteur CD/DVD de la VM.
- **Cause** : le lecteur virtuel pointait vers un lecteur physique inexistant au lieu du fichier ISO.
- **Solution** : *CD/DVD (IDE)* : *Use ISO image file* avec l'image décompressée, *Connect at power on* coché.
- **Leçon retenue** : vérifier le support de démarrage avant le premier lancement d'une VM.
