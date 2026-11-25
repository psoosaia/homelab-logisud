# 📜 Accord de niveau de service (SLA) — Logisud

| Élément | Valeur |
|---|---|
| Version | 1.0 |
| Fournisseur | Service informatique de Logisud (1 administrateur systèmes et réseaux) |
| Client | Direction et salariés de Logisud (40 personnes, site de Nancy) |
| Références | [Cahier des charges](../00-cahier-des-charges/README.md) · [Plan d'adressage](../01-architecture/plan-adressage.md) · [Procédure incident](procedure-incident.md) · [PRA](../07-sauvegarde-pra/pra.md) |

## 1. Plage de service

| Élément | Valeur |
|---|---|
| Heures ouvrées | Lundi au vendredi, 8h00 – 18h00 |
| Support utilisateurs | Pendant les heures ouvrées, par ticket GLPI (`NCY-GLPI-01`) |
| Astreinte | Aucune : un incident P1 survenu hors heures ouvrées est traité à la reprise |
| Fenêtre de maintenance | Mercredi 18h00 – 20h00, salariés prévenus 48 h à l'avance |

## 2. Services couverts

| Service | Composants | Criticité | Disponibilité cible |
|---|---|---|---|
| Réseau local | `NCY-SW-01`, VLAN 10 / 20 / 30 / 40 | Critique | 99,5 % |
| Accès Internet et filtrage | `NCY-FW-01` (pfSense) | Critique | 99,5 % |
| Authentification, DNS, DHCP | `NCY-DC-01`, `NCY-DC-02` (`logisud.lan`) | Critique | 99,5 % |
| Partages de fichiers | `NCY-FS-01` | Élevée | 99 % |
| Accès à distance (commerciaux) | VPN pfSense (`10.10.100.0/24`) | Élevée | 99 % |
| Assistance et inventaire | `NCY-GLPI-01` | Moyenne | 98 % |
| Site web | `NCY-WEB-01` (DMZ) | Moyenne | 98 % |
| Supervision et journalisation | `NCY-SUP-01` (Zabbix), `NCY-SIEM-01` (Wazuh) | Moyenne | 98 % |
| Sauvegardes | `NCY-PBS-01` (Proxmox Backup Server) | Élevée | 100 % des sauvegardes réussies ou relancées sous 24 h |

**Hors SLA** (hors périmètre du cahier des charges) : messagerie (gérée par le prestataire), téléphonie, Wi-Fi, impression.

### Calcul de la disponibilité

`Disponibilité = (temps prévu – temps d'indisponibilité) / temps prévu × 100`

La disponibilité est mesurée sur les heures ouvrées, soit environ 220 h par mois.

| Disponibilité | Indisponibilité maximale par mois |
|---|---|
| 99,5 % | ≈ 66 minutes |
| 99 % | ≈ 2 h 12 |
| 98 % | ≈ 4 h 24 |

**Exclusions** : maintenances planifiées annoncées, pannes du fournisseur d'accès Internet, coupures électriques (absence d'onduleur, risque accepté dans le cahier des charges).

## 3. Priorités et délais d'intervention

La priorité est déterminée par l'impact et l'urgence. GLPI la calcule automatiquement à partir de la matrice suivante.

|  | **Urgence haute** (travail bloqué) | **Urgence moyenne** (travail dégradé) | **Urgence basse** (gêne) |
|---|---|---|---|
| **Impact fort** (toute l'entreprise) | **P1** | P2 | P3 |
| **Impact moyen** (un service, plusieurs personnes) | P2 | P3 | P4 |
| **Impact faible** (une personne) | P3 | P4 | P4 |

| Priorité | Exemples | Prise en charge (TTO) | Résolution ou contournement (TTR) |
|---|---|---|---|
| **P1 — Critique** | Plus d'accès Internet, contrôleurs de domaine indisponibles, panne de `NCY-PVE-01` | 15 min | 4 h |
| **P2 — Haute** | Partage d'un service inaccessible, VPN indisponible | 30 min | 8 h (1 jour ouvré) |
| **P3 — Moyenne** | Un poste ne se connecte plus au réseau | 2 h | 2 jours ouvrés |
| **P4 — Basse** | Installation d'un logiciel, demande d'information | 1 jour ouvré | 5 jours ouvrés |

Les délais TTO et TTR sont configurés dans GLPI (*Configuration > Niveaux de services*) et appliqués automatiquement aux tickets.

## 4. Indicateurs

| Indicateur | Source | Objectif |
|---|---|---|
| Disponibilité par service | Rapports SLA Zabbix | Voir §2 |
| Tickets résolus dans les délais | Statistiques GLPI | ≥ 90 % |
| Délai moyen de prise en charge | Statistiques GLPI | Conforme au §3 |
| Réussite des sauvegardes | Tâches `NCY-PBS-01` | 100 % |
| Incidents récurrents | Problèmes GLPI | En baisse chaque mois |

## 5. Revue

- **Rapport mensuel** à la direction : disponibilité par service, incidents P1 / P2, respect des délais, actions correctives.
- **Revue annuelle** du présent document avec la direction.
- Les engagements de ce SLA tiennent compte des contrats des fournisseurs externes (fournisseur d'accès Internet, prestataire de messagerie).
