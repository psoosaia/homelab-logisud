# 📋 Cahier des charges — Logisud

## 1. Présentation de l'entreprise

| | |
|---|---|
| **Raison sociale** | Logisud (entreprise fictive) |
| **Activité** | Logistique et distribution de fournitures pour les professionnels |
| **Effectif** | 40 salariés |
| **Localisation** | Nancy (54), un site unique : bureaux + entrepôt |
| **Création** | 2009 |
| **Équipe informatique** | 1 administrateur systèmes et réseaux |

## 2. Organisation

| Service | Effectif | Besoins particuliers |
|---|---|---|
| Direction | 3 | Accès à l'ensemble des documents de l'entreprise |
| Comptabilité | 5 | Données financières confidentielles |
| Ressources humaines | 3 | Données personnelles des salariés (RGPD) |
| Commercial | 10 | Travail en déplacement, accès à distance |
| Logistique | 17 | Postes partagés dans l'entrepôt |
| Informatique | 2 | Administration de l'ensemble du système d'information |

## 3. Besoins

### Utilisateurs et postes
- [ ] Chaque salarié dispose d'un identifiant unique pour se connecter à n'importe quel poste
- [ ] Les postes sont uniformes (fond d'écran, paramètres) et ceux d'un même service disposent des mêmes logiciels
- [ ] Les mots de passe respectent une politique de sécurité commune

### Fichiers
- [ ] Chaque service dispose d'un espace de fichiers partagé
- [ ] Un service ne peut pas accéder aux fichiers d'un autre service, sauf dans un espace commun à toute l'entreprise
- [ ] Les fichiers supprimés par erreur peuvent être récupérés

### Accès à distance
- [ ] Les commerciaux accèdent à leurs fichiers de manière sécurisée depuis l'extérieur

### Exploitation
- [ ] Le parc informatique est inventorié
- [ ] Les salariés peuvent signaler un problème informatique et en suivre le traitement
- [ ] L'administrateur est alerté automatiquement en cas de panne

### Sécurité
- [ ] Les services sont cloisonnés sur le réseau
- [ ] Les connexions et événements de sécurité sont journalisés
- [ ] L'administration des serveurs est réservée à des comptes dédiés

### Continuité d'activité
- [ ] Les données et les serveurs sont sauvegardés quotidiennement
- [ ] Les sauvegardes sont conservées sur un équipement distinct du serveur de production
- [ ] Une copie des sauvegardes est conservée hors site
- [ ] Une procédure de reprise après sinistre est documentée et testée

## 4. Contraintes

| Contrainte | Détail |
|---|---|
| **Budget** | Limité : un serveur de production unique et un équipement de sauvegarde dédié, en matériel reconditionné |
| **Disponibilité (RTO)** | Le système doit être rétabli en moins de 4 h après une panne |
| **Perte de données (RPO)** | 24 h de données au maximum |
| **Réglementation** | Protection des données personnelles (RGPD), notamment pour les RH |
| **Ressources humaines** | Un seul administrateur : l'exploitation doit être centralisée, supervisée et documentée |

> ⚠️ **Risque accepté** : le serveur unique constitue un point de défaillance unique. Ce risque est accepté par la direction pour des raisons budgétaires et compensé par des sauvegardes quotidiennes sur un équipement dédié, une copie externalisée et un plan de reprise d'activité testé.

## 5. Hors périmètre

- Téléphonie
- Wi-Fi (bureaux et invités)
- Messagerie, externalisée chez un prestataire
- Alimentation secourue (onduleur)
- Impression