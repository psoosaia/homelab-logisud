# 🔀 Matrice des flux — Logisud

## Principe

Politique par défaut : **tout flux entre deux zones est refusé**. Seuls les flux listés ici sont autorisés par le pare-feu pfSense.

Le pare-feu ne filtre que le trafic qui le traverse : les flux internes à un même VLAN ne sont pas concernés par cette matrice.

## Vue d'ensemble entre zones

| Source ↓ / Destination → | ADMIN | SERVEURS | UTILISATEURS | DMZ | VPN | Internet |
|---|---|---|---|---|---|---|
| **ADMIN** | — | ✅ Administration | ✅ Administration | ✅ Administration | ❌ | ✅ Mises à jour |
| **SERVEURS** | ❌ | — | ❌ | ❌ | ❌ | ✅ Mises à jour |
| **UTILISATEURS** | ✅ Bastion uniquement (HTTPS) | ✅ AD, DNS, fichiers, GLPI | — | ❌ | ❌ | ✅ Navigation |
| **DMZ** | ❌ | ❌ | ❌ | — | ❌ | ✅ Mises à jour |
| **VPN** | ❌ | ✅ AD, DNS, fichiers | ❌ | ❌ | — | ❌ |
| **Internet** | ❌ | ❌ | ❌ | ✅ Site web (HTTPS) | ✅ Connexion VPN | — |

## Justification des choix

- **ADMIN** peut joindre toutes les zones pour les administrer, mais aucune zone ne peut joindre ADMIN, à l'exception de l'accès HTTPS au bastion depuis les postes des administrateurs.
- **SERVEURS** ne peut pas initier de connexion vers les autres zones internes : un serveur compromis ne peut pas attaquer les postes.
- **UTILISATEURS** accède uniquement aux services dont les salariés ont besoin : authentification, résolution de noms, partages de fichiers et demandes d'assistance.
- **DMZ** est totalement isolée du réseau interne : le serveur web, exposé à Internet, est considéré comme potentiellement compromis.
- **VPN** donne aux commerciaux un accès limité à l'authentification et à leurs fichiers, sans accès à Internet via l'entreprise ni aux autres zones.
- **Internet** ne peut joindre que le site web en DMZ et le point d'entrée du VPN.

## Détail des flux

*À compléter en phase 1, lors de la configuration de pfSense : protocoles, ports et machines précises pour chaque flux autorisé.*