# 🔄 Gestion des changements — Logisud

| Élément | Valeur |
|---|---|
| Version | 1.0 |
| Outil | GLPI (*Assistance > Changements*) |
| Références | [SLA](sla.md) · [Procédure incident](procedure-incident.md) · [PRA](../07-sauvegarde-pra/pra.md) |

## 1. Types de changements

| Type | Définition | Validation | Exemples |
|---|---|---|---|
| **Standard** | Changement courant, à faible risque, suivant une procédure connue | Pré-approuvé | Création d'un compte AD, ajout d'un poste au domaine, mises à jour Windows mensuelles |
| **Normal** | Changement planifié comportant un risque | Analyse de risque et validation par la direction | Règle pfSense, configuration du switch, mise à jour de pfSense ou de Proxmox, nouvelle GPO |
| **Urgent** | Changement requis pour résoudre un incident P1 ou corriger une faille critique | Validation rapide par la direction, documentation a posteriori | Correctif de sécurité, correction pendant un incident P1 |

## 2. Processus d'un changement normal

```
Demande → Analyse de risque → Validation → Planification → Point de restauration
   → Exécution → Tests → Clôture
                    ↓ échec
              Retour arrière
```

1. **Demande** : création du changement dans GLPI selon le [modèle](#3-modèle-de-demande-de-changement).
2. **Analyse** : services, VLAN et flux impactés, mise à jour éventuelle de la [matrice des flux](../01-architecture/matrice-flux.md), conséquences d'un échec.
3. **Validation** par la direction.
4. **Planification** dans la fenêtre de maintenance (mercredi 18h – 20h), salariés prévenus 48 h à l'avance.
5. **Point de restauration** (voir le [§5](#5-points-de-restauration)).
6. **Exécution** depuis le bastion `NCY-BAS-01` avec un compte `adm-…`.
7. **Tests** : fonctionnement du changement et absence de problème actif dans Zabbix.
8. **Clôture** : mise à jour du journal et de la documentation du dépôt.

Aucun changement normal n'est exécuté sans plan de retour arrière.

## 3. Modèle de demande de changement

```markdown
## Changement CHG-AAAA-NNN
- Demandeur :
- Type : Standard / Normal / Urgent
- Ticket ou problème lié :
- Description et justification :
- Équipements impactés :
- VLAN / flux impactés :
- Risque : Faible / Moyen / Élevé — justification :
- Interruption de service : Oui (durée) / Non
- Date et créneau :
- Validé par / le :

### Plan d'exécution
1.

### Tests de validation
- [ ] Changement fonctionnel
- [ ] Services impactés fonctionnels
- [ ] Aucun problème actif dans Zabbix

### Plan de retour arrière
- Point de restauration :
- Étapes :

### Bilan
- Résultat : Réussi / Échec (retour arrière effectué)
- Documentation mise à jour :
```

## 4. Journal des changements

Un changement par ligne, le plus récent en haut.

| ID | Date | Type | Description | Équipement | Résultat |
|---|---|---|---|---|---|
| | | | | | |

## 5. Points de restauration

| Équipement | Méthode |
|---|---|
| VM Proxmox | Snapshot (*VM > Snapshots > Take Snapshot*), supprimé après validation du changement |
| pfSense | Export de la configuration (*Diagnostics > Backup & Restore*) |
| `NCY-SW-01` | `copy running-config startup-config` et export de la `running-config` |
| Active Directory / GPO | Snapshot de la VM et sauvegarde de la GPO concernée |

Les exports de configuration contiennent des secrets : ils sont conservés hors du dépôt (`secrets/`, exclu par le `.gitignore`).

Un snapshot ne constitue pas une sauvegarde : il réside sur le même stockage que la VM. Les sauvegardes sont décrites dans le [PRA](../07-sauvegarde-pra/pra.md).
