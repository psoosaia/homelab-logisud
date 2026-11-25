# 🔍 Démarche de diagnostic — Logisud

| Élément | Valeur |
|---|---|
| Version | 1.1 |
| Statut | Rédigé — commandes à valider lors du déploiement de chaque équipement |
| Utilisation | Étape « Diagnostic » de la [procédure de traitement des incidents](procedure-incident.md) |
| Références | [Plan d'adressage](../01-architecture/plan-adressage.md) · [Matrice des flux](../01-architecture/matrice-flux.md) · [Journal des changements](changements.md#4-journal-des-changements) |

## 1. Étapes

| # | Étape | Points à établir |
|---|---|---|
| 1 | Recueil des symptômes | Message d'erreur, heure d'apparition, changement récent, reproductibilité |
| 2 | Délimitation du périmètre | Utilisateur, poste, VLAN ou ensemble de l'infrastructure |
| 3 | Hypothèses | Causes possibles, classées de la plus probable à la moins probable |
| 4 | Tests | Une hypothèse testée à la fois |
| 5 | Correction et vérification | Service rétabli pour l'ensemble des utilisateurs concernés |
| 6 | Documentation | Ticket GLPI complété, base de connaissances mise à jour |

Le [journal des changements](changements.md#4-journal-des-changements) est consulté dès l'étape 1.

## 2. Délimitation du périmètre

Le troisième octet de l'adresse IP correspond au numéro de VLAN (`10.10.30.x` = VLAN 30 UTILISATEURS).

| Constat | Zone à investiguer |
|---|---|
| Un seul utilisateur, sur tous les postes | Compte AD (verrouillage, expiration du mot de passe, appartenance aux groupes `GG_…`) |
| Un seul poste, pour tous les utilisateurs | Poste : carte réseau, configuration IP, câble, port du switch |
| Tous les postes du VLAN 30 | DHCP (`NCY-DC-01`), relais DHCP de pfSense, passerelle `10.10.30.1`, trunk du switch |
| Toute l'infrastructure | `NCY-FW-01`, `NCY-SW-01`, `NCY-PVE-01`, lien Internet |
| Utilisateurs VPN uniquement | Serveur VPN de pfSense, règles du réseau `10.10.100.0/24` |

## 3. Méthodes

| Méthode | Principe | Usage |
|---|---|---|
| Ascendante | De la couche 1 vers la couche 7 | Panne réseau générale, cause inconnue |
| Descendante | De la couche 7 vers la couche 1 | Dysfonctionnement d'une seule application |
| Diviser pour régner | Test en couche 3 (`ping`), puis progression vers le haut ou vers le bas | Cas général |

Un échec de `ping` est interprété au regard de la [matrice des flux](../01-architecture/matrice-flux.md) : l'ICMP n'est pas autorisé entre toutes les zones.

## 4. Vérifications par couche

| Couche | Vérifications | Switch Cisco (`NCY-SW-01`) | Poste ou serveur Windows | Serveur Linux | pfSense (`NCY-FW-01`) |
|---|---|---|---|---|---|
| 1 — Physique | Câble, voyants, état du port | `show interfaces status` | `Get-NetAdapter` | `ip link` | *Status > Interfaces* |
| 2 — Liaison | VLAN du port, trunk vers pfSense, table MAC | `show vlan brief` · `show interfaces trunk` · `show mac address-table` | `arp -a` | `ip neigh` | *Diagnostics > ARP Table* |
| 3 — Réseau | Adresse IP, masque, passerelle, routage | `show ip interface brief` | `ipconfig /all` · `ping` · `tracert` | `ip a` · `ip r` · `ping` · `traceroute` | *Diagnostics > Ping* · *Diagnostics > Routes* |
| 4 — Transport | Port du service, filtrage pfSense | — | `Test-NetConnection <IP> -Port <port>` | `ss -tulpn` · `nc -zv <IP> <port>` | *Status > System Logs > Firewall* |
| 7 — Application | Service, résolution DNS, compte AD, droits NTFS | — | `nslookup <nom>.logisud.lan` · `services.msc` · Observateur d'événements | `systemctl status <service>` · `journalctl -u <service>` | *Status > Services* |

Les couches 5 et 6 ne font pas l'objet de vérifications spécifiques : elles sont couvertes par les tests de la couche 7.

## 5. Cas fréquents

| Symptôme | Cause probable | Vérification |
|---|---|---|
| Adresse `169.254.x.x` (APIPA) | Aucune réponse DHCP : port dans le mauvais VLAN (999), relais DHCP pfSense ou service DHCP de `NCY-DC-01` | `show vlan brief`, journaux pfSense, console DHCP |
| Accès par adresse IP mais pas par nom | Serveurs DNS du poste incorrects | `ipconfig /all` : DNS `10.10.20.10` et `10.10.20.11` |
| Serveur joignable mais service inaccessible | Flux bloqué par pfSense ou service arrêté | `Test-NetConnection`, journaux du pare-feu, état du service |
| Accès refusé à un partage | Groupes AD ou droits NTFS (AGDLP) | Appartenance aux groupes `GG_…` et `DL_…`, onglet Sécurité du dossier |
