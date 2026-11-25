# 📚 Procédures d'exploitation — Logisud

Documents de gestion des services informatiques, inspirés d'ITIL. Ils répondent à l'exigence du [cahier des charges](../00-cahier-des-charges/README.md) : un seul administrateur, une exploitation centralisée, supervisée et documentée.

| Document | Rôle |
|---|---|
| [SLA](sla.md) | Services couverts, disponibilité visée, priorités et délais d'intervention |
| [Procédure de traitement des incidents](procedure-incident.md) | De l'alerte ou du ticket jusqu'à la clôture |
| [Démarche de diagnostic](diagnostic.md) | Méthode et commandes de diagnostic |
| [Gestion des changements](changements.md) | Processus de modification de l'infrastructure et journal des changements |
| [PRA / PCA](../07-sauvegarde-pra/pra.md) | Sauvegardes, objectifs de reprise et procédures en cas de sinistre |

**Enchaînement :** une alerte **Zabbix** ou un ticket **GLPI** déclenche un **incident**, priorisé selon le **SLA** et analysé avec la **démarche de diagnostic**. Toute correction qui modifie la configuration suit la **gestion des changements**. En cas de sinistre, le **PRA** s'applique.
