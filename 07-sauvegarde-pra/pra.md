# 🛟 Plan de reprise et de continuité d'activité — Logisud

| Élément | Valeur |
|---|---|
| Version | 1.1 |
| Statut | Rédigé — à mettre en œuvre et à tester lors de la phase sauvegarde et PRA |
| Références | [Cahier des charges](../00-cahier-des-charges/README.md#4-contraintes) · [Plan d'adressage](../01-architecture/plan-adressage.md) · [SLA](../08-procedures/sla.md) · [Procédure incident](../08-procedures/procedure-incident.md) |

## 1. Contexte

L'infrastructure repose sur un serveur de production unique (`NCY-PVE-01`). Ce point de défaillance unique est accepté par la direction pour des raisons budgétaires. Il est compensé par :
- des sauvegardes quotidiennes sur un équipement dédié (`NCY-PBS-01`) ;
- une copie externalisée ;
- le présent plan de reprise, testé périodiquement.

| Objectif | Définition | Valeur |
|---|---|---|
| **RTO** | Durée maximale d'interruption | 4 h |
| **RPO** | Perte de données maximale | 24 h |

Le RPO de 24 h s'applique aux sauvegardes quotidiennes sur `NCY-PBS-01`. La copie hors site, synchronisée chaque semaine, offre un RPO d'une semaine : elle n'est utilisée que si `NCY-PBS-01` est perdu ou compromis.

## 2. Analyse d'impact

| Service | Machines | Impact | RTO | RPO |
|---|---|---|---|---|
| Pare-feu, Internet, VPN | `NCY-FW-01` | Plus d'accès Internet, de VPN ni de routage inter-VLAN | 4 h | Dernier export de configuration |
| Annuaire, DNS, DHCP | `NCY-DC-01`, `NCY-DC-02` | Authentification impossible | 4 h | 24 h |
| Fichiers | `NCY-FS-01` | Données de l'ensemble des services inaccessibles | 4 h | 24 h |
| Réseau local | `NCY-SW-01` | Réseau interne coupé | 4 h | Dernier export de configuration |
| Assistance et inventaire | `NCY-GLPI-01` | Suivi des demandes interrompu | 1 jour ouvré | 24 h |
| Supervision, journalisation | `NCY-SUP-01`, `NCY-SIEM-01` | Perte des alertes et des traces de sécurité | 1 jour ouvré | 24 h |
| Site web | `NCY-WEB-01` | Site indisponible | 1 jour ouvré | 24 h |
| Bastion | `NCY-BAS-01` | Administration dégradée | 1 jour ouvré | 1 semaine |

## 3. Stratégie de sauvegarde (règle 3-2-1)

| Règle | Mise en œuvre |
|---|---|
| 3 copies | Production sur `NCY-PVE-01`, sauvegarde sur `NCY-PBS-01`, copie externalisée |
| 2 supports distincts | `NCY-PVE-01` et `NCY-PBS-01` sont deux machines physiques distinctes : une panne du serveur de production n'affecte pas les sauvegardes |
| 1 copie hors site | Disque USB chiffré alimenté par PBS, conservé hors des locaux |

| Élément | Méthode | Fréquence | Rétention |
|---|---|---|---|
| Ensemble des VM | Tâche de sauvegarde Proxmox vers `NCY-PBS-01`, mode *snapshot* | Chaque nuit à 22h00 | 7 quotidiennes, 4 hebdomadaires, 3 mensuelles |
| Vérification | *Verify Job* PBS | Chaque nuit, après les sauvegardes | — |
| Configuration pfSense | Export de configuration | Après chaque changement et chaque lundi | 10 versions, hors dépôt |
| Configuration `NCY-SW-01` | Export de la `running-config` | Après chaque changement | Hors dépôt |
| Copie hors site | *Sync Job* PBS vers un datastore sur disque USB amovible | Chaque vendredi | 4 semaines |

**Récupération de fichiers supprimés** : restauration unitaire depuis PBS (*File Restore*) et clichés instantanés (*Shadow Copies*) sur `NCY-FS-01`.

## 4. Scénarios de reprise

### Scénario A — VM corrompue ou supprimée
1. Ticket GLPI (P2).
2. Proxmox : stockage `NCY-PBS-01` > *Backups*, sélection de la dernière sauvegarde valide, **Restore**.
3. Démarrage de la VM et vérification du service.
4. Information des utilisateurs sur la perte des données postérieures à la sauvegarde.

Durée estimée : 1 h.

### Scénario B — Panne matérielle de `NCY-PVE-01`
1. Incident P1, information de la direction.
2. Installation de Proxmox VE sur la machine de remplacement (`NCY-PVE-01`, `10.10.10.10`).
3. Ajout de `NCY-PBS-01` comme stockage (*Datacenter > Storage > Add > Proxmox Backup Server*).
4. Restauration dans l'ordre des dépendances :
   1. `NCY-FW-01` : routage inter-VLAN et accès Internet
   2. `NCY-DC-01`, puis `NCY-DC-02` : authentification, DNS, DHCP
   3. `NCY-FS-01`
   4. `NCY-BAS-01`, `NCY-SUP-01`, `NCY-SIEM-01`, `NCY-GLPI-01`
   5. `NCY-WEB-01`
5. Vérification de chaque service avant de passer au suivant.

Objectif : 4 h au maximum (RTO). Mesures pour le tenir : support d'installation Proxmox prêt à l'emploi, machine de remplacement identifiée, sauvegardes disponibles sur `NCY-PBS-01` sans dépendance au serveur en panne.

La durée réelle est mesurée lors de l'exercice annuel (§5). Si elle dépasse le RTO, la stratégie de reprise ou le RTO est révisé avec la direction.

### Scénario C — Attaque par rançongiciel
1. Isolement immédiat des machines touchées (déconnexion réseau dans Proxmox, débranchement des postes), sans extinction afin de préserver les traces.
2. Incident P1, information de la direction. En cas d'atteinte aux données personnelles, notification à la CNIL sous 72 h (RGPD).
3. Analyse dans Wazuh (`NCY-SIEM-01`) : point d'entrée et date de l'infection.
4. Sélection d'une sauvegarde antérieure à l'infection, sur la copie hors site si `NCY-PBS-01` est compromis.
5. Restauration sur des VM saines, renouvellement de l'ensemble des mots de passe (comptes `adm-…` et `svc-…` en priorité).
6. Ouverture d'un problème GLPI : analyse du vecteur d'attaque et mesures correctives.

### Scénario D — Perte de la configuration pfSense
1. Recréation de la VM `NCY-FW-01` avec les mêmes interfaces réseau.
2. Installation de pfSense et import du dernier export de configuration.
3. Vérification : accès Internet, routage inter-VLAN, relais DHCP, VPN, règles de filtrage.

Durée estimée : 30 min.

### Scénario E — Panne d'un contrôleur de domaine
1. Les postes utilisent le second contrôleur, déclaré comme serveur DNS : pas d'interruption de l'authentification ni de la résolution de noms (P3).
2. Restauration depuis PBS, ou réinstallation et promotion du contrôleur avec réplication de l'annuaire.
3. Le DHCP étant porté uniquement par `NCY-DC-01`, sa panne empêche l'attribution d'adresses aux nouveaux postes ; les postes déjà connectés conservent leur adresse jusqu'à expiration du bail. La restauration de `NCY-DC-01` est donc prioritaire (P2). Le basculement DHCP entre les deux contrôleurs supprimerait ce risque (voir §6).

## 5. Tests

| Test | Fréquence | Méthode |
|---|---|---|
| Restauration d'une VM | Mensuelle | Restauration sous un autre identifiant, interface réseau déconnectée |
| Restauration d'un fichier | Mensuelle | *File Restore* PBS et contrôle du contenu |
| Restauration de la configuration pfSense | Trimestrielle | Import dans une VM pfSense isolée |
| Exercice complet (scénario B) | Annuelle | Reconstruction à partir des sauvegardes, durée chronométrée |

### Registre des tests

| Date | Test | Durée | Résultat | Remarques |
|---|---|---|---|---|
| | | | | |

*Aucun test réalisé à ce jour : le registre sera complété à la mise en œuvre des sauvegardes.*

## 6. Mesures de continuité

| Risque | Mesure | Statut |
|---|---|---|
| Panne d'un contrôleur de domaine | Deux contrôleurs, tous deux déclarés comme DNS sur les postes | Prévu |
| Panne du service DHCP | Basculement DHCP entre les deux contrôleurs | Évolution possible |
| Panne du lien Internet | Connexion 4G/5G de secours | Procédure manuelle |
| Coupure électrique | Onduleur | Hors périmètre (risque accepté) |
| Panne de l'hôte unique | Second nœud Proxmox | Hors budget (risque accepté, couvert par le PRA) |
