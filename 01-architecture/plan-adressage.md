# 🌐 Plan d'adressage — Logisud

## 1. Découpage en VLANs

Le réseau est découpé **par niveau de confiance**, et non par service de l'entreprise. Les machines qui ont le même rôle et le même niveau de risque sont regroupées dans un même VLAN. Tout trafic entre deux VLANs traverse le pare-feu pfSense, qui applique les règles de la [matrice des flux](matrice-flux.md).

L'accès aux données entre services (comptabilité, RH…) n'est pas géré par le réseau, mais par les droits NTFS selon la méthode AGDLP (voir les [conventions de nommage](conventions-nommage.md)).

| ID | Nom | Contenu | Justification |
|---|---|---|---|
| 10 | ADMIN | Switch, hyperviseur Proxmox, bastion | Zone la plus critique : son contrôle donne accès à toute l'infrastructure. Accessible uniquement via le bastion |
| 20 | SERVEURS | Contrôleurs de domaine, fichiers, GLPI, supervision, SIEM, sauvegarde | Contient les données de l'entreprise. Accès limité aux services strictement nécessaires |
| 30 | UTILISATEURS | Postes des salariés | Zone la plus exposée aux logiciels malveillants (messagerie, navigation, clés USB). N'accède jamais à l'administration |
| 40 | DMZ | Serveur web | Seule zone joignable depuis Internet. Isolée du réseau interne : même compromise, elle ne peut rien atteindre |
| 999 | PARKING | Ports inutilisés du switch | Ports désactivés : une prise libre ne donne accès à aucun réseau |

**Bonnes pratiques appliquées :**
- Le VLAN 1 (VLAN par défaut des switchs) n'est utilisé pour aucun usage.
- Les numéros 50 à 90 sont réservés pour de futurs VLANs (Wi-Fi, imprimantes), actuellement hors périmètre.
- Le trafic entre machines d'un même VLAN ne passe pas par le pare-feu : la propagation entre postes est limitée par le pare-feu Windows, configuré par GPO.

## 2. Réseaux

| VLAN | Nom | Réseau | Passerelle | Adresses fixes | Plage DHCP |
|---|---|---|---|---|---|
| 10 | ADMIN | `10.10.10.0/24` | `10.10.10.1` | `.2` à `.99` | Aucune |
| 20 | SERVEURS | `10.10.20.0/24` | `10.10.20.1` | `.2` à `.99` | Aucune |
| 30 | UTILISATEURS | `10.10.30.0/24` | `10.10.30.1` | `.2` à `.99` (réserve) | `.100` à `.199` |
| 40 | DMZ | `10.10.40.0/24` | `10.10.40.1` | `.2` à `.99` | Aucune |
| — | VPN nomades | `10.10.100.0/24` | `10.10.100.1` | — | Attribution par le serveur VPN |

**Choix techniques :**
- **Plage `10.0.0.0/8`** : les plages `192.168.0.0/24` et `192.168.1.0/24`, utilisées par la plupart des box Internet, sont évitées. Un commercial connecté en VPN depuis son domicile ne subit ainsi aucun conflit d'adresses.
- **Un `/24` par VLAN** (254 adresses) : en adressage privé, la place n'est pas limitée. Ce choix privilégie la lisibilité et anticipe la croissance de l'entreprise.
- **Le 3ᵉ octet correspond au numéro de VLAN** : l'adresse `10.10.30.x` appartient au VLAN 30. Le diagnostic est immédiat.
- **La passerelle est toujours en `.1`.**
- **Le VPN dispose de son propre réseau** (`10.10.100.0/24`), éloigné des VLANs, pour être filtré séparément par pfSense et sans risque de collision avec un futur VLAN.
- **Seuls les postes utilisateurs reçoivent une adresse par DHCP.** Les serveurs et équipements ont une adresse fixe, toujours en dehors de la plage DHCP.

## 3. Adresses fixes

### VLAN 10 — ADMIN

| Tranche | Usage | Machine | Adresse |
|---|---|---|---|
| `.1` | Passerelle | pfSense | `10.10.10.1` |
| `.2` à `.9` | Équipements réseau | NCY-SW-01 | `10.10.10.2` |
| `.10` à `.19` | Hyperviseurs | NCY-PVE-01 | `10.10.10.10` |
| `.20` à `.29` | Accès d'administration | NCY-BAS-01 | `10.10.10.20` |

### VLAN 20 — SERVEURS

| Tranche | Usage | Machine | Adresse |
|---|---|---|---|
| `.1` | Passerelle | pfSense | `10.10.20.1` |
| `.10` à `.19` | Annuaire et DNS | NCY-DC-01 | `10.10.20.10` |
| | | NCY-DC-02 | `10.10.20.11` |
| `.20` à `.29` | Fichiers | NCY-FS-01 | `10.10.20.20` |
| `.30` à `.39` | Exploitation et sécurité | NCY-GLPI-01 | `10.10.20.30` |
| | | NCY-SUP-01 | `10.10.20.31` |
| | | NCY-SIEM-01 | `10.10.20.32` |
| `.40` à `.49` | Sauvegarde | NCY-PBS-01 | `10.10.20.40` |

`NCY-PBS-01` est une **machine physique distincte** de l'hyperviseur `NCY-PVE-01`, raccordée au switch dans le VLAN 20 : les sauvegardes restent disponibles en cas de panne du serveur de production. Tous les autres serveurs sont des VM hébergées sur `NCY-PVE-01`.

### VLAN 40 — DMZ

| Tranche | Usage | Machine | Adresse |
|---|---|---|---|
| `.1` | Passerelle | pfSense | `10.10.40.1` |
| `.10` à `.19` | Services web | NCY-WEB-01 | `10.10.40.10` |

Les adresses sont regroupées par tranches de dix selon la fonction : un nouveau serveur prend la première adresse libre de sa tranche (par exemple, un troisième contrôleur de domaine prendrait `10.10.20.12`).

## 4. Services réseau

| Service | Configuration |
|---|---|
| DNS | `10.10.20.10` (NCY-DC-01) et `10.10.20.11` (NCY-DC-02). Les deux contrôleurs de domaine sont déclarés sur chaque poste pour assurer la continuité en cas de panne de l'un d'eux |
| Redirection DNS | Les contrôleurs de domaine transmettent les requêtes Internet à pfSense |
| DHCP | Assuré par NCY-DC-01 pour le VLAN 30 |
| Relais DHCP | Configuré sur pfSense : les demandes DHCP sont diffusées en broadcast et ne franchissent pas les VLANs, pfSense les relaie donc vers NCY-DC-01 |