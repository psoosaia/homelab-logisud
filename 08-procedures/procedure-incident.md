# 🚨 Procédure de traitement des incidents — Logisud

| Élément | Valeur |
|---|---|
| Version | 1.1 |
| Statut | Rédigé — applicable après le déploiement de GLPI, Zabbix et Wazuh |
| Outils | GLPI (`NCY-GLPI-01`) · Zabbix (`NCY-SUP-01`) · Wazuh (`NCY-SIEM-01`) |
| Références | [SLA](sla.md) · [Démarche de diagnostic](diagnostic.md) · [Gestion des changements](changements.md) · [PRA](../07-sauvegarde-pra/pra.md) |

## 1. Définitions

| Terme | Définition | Objectif |
|---|---|---|
| **Incident** | Interruption ou dégradation d'un service | Rétablir le service au plus vite, y compris par un contournement |
| **Problème** | Cause, connue ou inconnue, d'un ou plusieurs incidents | Identifier la cause racine et l'éliminer |

Les incidents sont gérés dans *Assistance > Tickets* et les problèmes dans *Assistance > Problèmes* de GLPI.

## 2. Niveaux de support

| Niveau | Intervenant | Rôle |
|---|---|---|
| **N1** | Administrateur, compte utilisateur standard | Prise en compte du ticket, qualification, application des solutions connues de la base de connaissances GLPI, sans intervention sur l'infrastructure |
| **N2** | Administrateur, compte `adm-…` via le bastion `NCY-BAS-01` | Diagnostic approfondi et intervention sur les serveurs, le réseau et le pare-feu |

Avec un administrateur unique, N1 et N2 sont assurés par la même personne : la distinction porte sur le périmètre d'intervention et le compte utilisé, conformément au principe du moindre privilège.
| **N3** | Fournisseur d'accès, prestataire de messagerie, éditeurs | Problèmes hors du périmètre interne |

## 3. Cycle de vie d'un incident

```
Détection → Enregistrement → Catégorisation → Priorisation → Diagnostic
    → (Escalade) → Résolution → Vérification → Clôture
```

### 3.1 Détection
- Alerte **Zabbix** (déclencheur en état *Problème*). Les déclencheurs sont évalués sur une durée (ex. `min(/NCY-FS-01/system.cpu.util,5m)>90`) pour ignorer les pics ponctuels. Des dépendances entre déclencheurs suppriment les alertes en cascade, par exemple pour les équipements situés derrière `NCY-SW-01` lorsqu'il est injoignable.
- Alerte **Wazuh** (événement de sécurité).
- Ticket **GLPI** ouvert par un salarié.
- Constatation de l'administrateur.

### 3.2 Enregistrement
Tout incident fait l'objet d'un ticket GLPI, quelle que soit sa durée.

### 3.3 Catégorisation
Réseau · Pare-feu / Internet · VPN · Annuaire (AD / DNS / DHCP) · Fichiers · Poste de travail · Sauvegarde · Supervision · Sécurité.

### 3.4 Priorisation
Impact et urgence renseignés dans le ticket. GLPI calcule la priorité et applique les délais du [SLA](sla.md#3-priorités-et-délais-dintervention).

### 3.5 Diagnostic
1. Recherche dans la base de connaissances et les tickets précédents.
2. Application de la [démarche de diagnostic](diagnostic.md).
3. Chaque test et son résultat sont consignés en suivi du ticket.

### 3.6 Escalade
| Type | Déclencheur |
|---|---|
| **Fonctionnelle** (vers N3) | Panne d'un fournisseur, anomalie logicielle, compétence externe requise |
| **Hiérarchique** (vers la direction) | Incident P1 de plus de 2 h, incident P2 dépassant son délai de résolution, ou risque de dépassement du SLA |

### 3.7 Résolution
- Correction ou contournement.
- Toute modification de configuration suit la [gestion des changements](changements.md), en changement urgent pour un P1.
- **Incident de sécurité** : isolement de la machine concernée, puis application du [PRA, scénario C](../07-sauvegarde-pra/pra.md#scénario-c--attaque-par-rançongiciel).

### 3.8 Vérification et clôture
- Confirmation du rétablissement auprès de l'utilisateur.
- Retour à l'état normal dans Zabbix.
- Solution documentée dans le ticket (cause, action, durée).
- Passage en *Résolu*, puis en *Clos* après validation de l'utilisateur ou automatiquement après 2 jours ouvrés.
- Création d'un article dans la base de connaissances si la solution est réutilisable.

### 3.9 Ouverture d'un problème
Un problème GLPI est ouvert et lié aux tickets concernés lorsque :
- l'incident était de priorité P1 ;
- ou le même incident s'est produit au moins 3 fois dans le mois ;
- ou la cause reste inconnue après résolution.

Le problème comprend une analyse de cause racine (méthode des 5 pourquoi) et l'action corrective définitive.

## 4. Communication

| Moment | Communication |
|---|---|
| Prise en charge | Notification automatique GLPI (priorité attribuée) |
| P1 / P2 en cours | Point d'avancement toutes les heures (suivi public du ticket, information de la direction pour un P1) |
| P3 / P4 en cours | Suivi public du ticket à chaque changement d'état (prise en charge, attente, résolution) |
| Résolution | Information du rétablissement et demande de confirmation |

## 5. Modèle de ticket

```markdown
## Ticket GLPI n° …
- Date / heure de détection :
- Source : Zabbix / Wazuh / Salarié / Administrateur
- Demandeur :
- Service touché :
- Catégorie :
- Impact : Fort / Moyen / Faible
- Urgence : Haute / Moyenne / Basse
- Priorité : P1 / P2 / P3 / P4
- Symptômes :

### Suivis
- [hh:mm] test effectué → résultat

### Solution
- Cause :
- Action corrective / contournement :
- Changement associé : CHG-…
- Date / heure de résolution :
- Durée d'indisponibilité :
- SLA respecté : Oui / Non
- Problème ouvert : Oui (n° …) / Non
```
