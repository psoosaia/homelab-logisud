# 🏷️ Conventions de nommage — Logisud

## 1. Domaine Active Directory

| Élément | Valeur |
|---|---|
| Nom de domaine (DNS) | `logisud.lan` |
| Nom NetBIOS | `LOGISUD` |

**Justification :**
- L'extension `.lan` n'existe pas sur Internet : aucun risque de conflit avec un domaine public.
- L'extension `.local` est écartée, car elle est réservée au protocole mDNS (appareils Apple, imprimantes, Linux) et provoque des problèmes de résolution de noms.
- L'option recommandée par Microsoft, un sous-domaine du domaine public (`ad.logisud.fr`), est écartée car Logisud ne possède pas de nom de domaine public.
- Le nom NetBIOS respecte la limite de 15 caractères.

## 2. Machines

**Format :** `NCY-<RÔLE>-<NUMÉRO>`

- `NCY` : code du site de Nancy. En cas d'ouverture d'un autre site, il aura son propre code (ex. `MTZ` pour Metz).
- Numéro sur 2 chiffres pour les serveurs et équipements, sur 3 chiffres pour les postes.
- **15 caractères maximum** (limite des noms d'ordinateurs Windows).
- Lettres majuscules, chiffres et tirets uniquement.

| Code | Rôle | Exemple |
|---|---|---|
| `FW` | Pare-feu | `NCY-FW-01` |
| `SW` | Switch | `NCY-SW-01` |
| `PVE` | Hyperviseur Proxmox | `NCY-PVE-01` |
| `DC` | Contrôleur de domaine | `NCY-DC-01` |
| `FS` | Serveur de fichiers | `NCY-FS-01` |
| `GLPI` | Gestion de parc et assistance | `NCY-GLPI-01` |
| `SUP` | Supervision (Zabbix) | `NCY-SUP-01` |
| `PBS` | Serveur de sauvegarde | `NCY-PBS-01` |
| `WEB` | Serveur web | `NCY-WEB-01` |
| `PC` | Poste fixe | `NCY-PC-001` |
| `LT` | Ordinateur portable | `NCY-LT-001` |

## 3. Comptes

| Type | Format | Exemple | Usage |
|---|---|---|---|
| Utilisateur | `prenom.nom` | `marie.martin` | Travail quotidien |
| Administrateur | `adm-prenom.nom` | `adm-jean.dupont` | Administration uniquement |
| Service | `svc-application` | `svc-glpi` | Utilisé par une application, jamais par une personne |

**Règles :**
- Minuscules, sans accents ni espaces : `Élodie Lefèvre` → `elodie.lefevre`.
- Noms composés avec un tiret : `jean-pierre.martin`.
- Homonymes : ajout d'un chiffre au second compte : `jean.dupont2`.
- 20 caractères maximum (limite Windows) : au-delà, le nom de famille est tronqué.

**Justification des comptes d'administration séparés :** principe du moindre privilège. Le compte quotidien sert à la messagerie et à la navigation, qui sont les principales portes d'entrée des logiciels malveillants. S'il est compromis, l'attaquant n'obtient aucun droit d'administration.

## 4. Groupes — méthode AGDLP

Les **comptes** (A) sont placés dans des **groupes globaux** (G) par service. Ces groupes globaux sont placés dans des **groupes de domaine local** (DL), qui seuls reçoivent les **permissions** (P) sur les ressources.

| Type | Format | Exemple |
|---|---|---|
| Groupe global (personnes) | `GG_<Service>` | `GG_Comptabilite` |
| Groupe de domaine local (droit) | `DL_<Ressource>_<Droit>` | `DL_Compta_RW` |

Droits : `RW` = lecture et écriture (Modifier), `RO` = lecture seule.

**Exemple complet — partage de la comptabilité :**

| Compte | → Groupe global | → Groupe de domaine local | → Permission sur `\\NCY-FS-01\Comptabilite` |
|---|---|---|---|
| `marie.martin` | `GG_Comptabilite` | `DL_Compta_RW` | Modifier |
| `paul.bernard` (direction) | `GG_Direction` | `DL_Compta_RO` | Lecture |

**Justification :** quand un salarié change de service, on modifie uniquement son groupe global. Pour donner un accès à tout un service, on ajoute son groupe global dans le bon groupe de domaine local, sans jamais modifier les droits du dossier.