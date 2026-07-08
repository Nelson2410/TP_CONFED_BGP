# 🌐 Infrastructure Réseau BGP avec Confédération

[![Network](https://img.shields.io/badge/Network-BGP-blue.svg)](https://github.com) [![Cisco](https://img.shields.io/badge/Cisco-IOS-red.svg)](https://www.cisco.com) [![Documentation](https://img.shields.io/badge/Documentation-Complete-green.svg)](./1.Documentation/Doc_TP_Confed_BGP.pdf) [![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## 📖 Table des Matières

- **CONCEPTION & ARCHITECTURE**
  - [Description du Projet](#description-du-projet)
    - [Objectifs du Projet](#objectifs)
  - [Architecture & Topologie](#architecture-et-topologie)
    - [Systèmes Autonomes (AS)](#vue-densemble-des-systèmes-autonomes-as)
    - [Schéma de la Topologie](#schéma-de-la-topologie)
  - [Plan d'Adressage](#plan-dadressage)
    - [Loopbacks (Router-ID)](#interfaces-loopback-router-id)
    - [Liens Point-à-Point](#liens-point-à-point-réseaux-de-transit)
    - [Réseaux LAN Clients](#réseaux-lan-clients)
  - [Structure du Projet](#structure-du-projet)

- **CONFIGURATION & DEPLOIEMENT**
  - [Configuration BGP](#configuration-bgp)
    - [1. Backbone Principal](#1-backbone-principal-as-65000)
    - [2. Confédération](#2-confédération-as-65010)
    - [3. AS Clients](#3-as-clients)
  - [Déploiement Réseau](#procédure-de-déploiement)
    - [1. Prérequis techniques](#1-prérequis)
    - [2. Importation de Topologie](#2-importer-la-topologie)
    - [3. Push des Configurations](#3-déployer-les-configurations)
    - [4. Sauvegarde & Écritures](#4-sauvegarde)

- **TESTS & DIAGNOSTICS**
  - [Vérification & Tests](#vérification-et-tests)
    - [1. Connectivité de base](#1-connectivité-de-base)
    - [2. Validation BGP](#2-vérification-bgp)
    - [3. Etat Confédération](#3-vérification-de-la-confédération)
    - [4. Pings de bout en bout](#4-tests-de-bout-en-bout-ping--traceroute)
  - [Dépannage BGP](#dépannage-bgp)
    - [Session Idle / Active](#session-en-état-idle-ou-active)
    - [Routes non-propagées](#routes-bgp-absentes-ou-non-propagées)
    - [Boucle & Accessibilité](#boucle-de-routage-ou-reachability)
    - [Commandes Debugging](#commandes-de-debugging)

- **PEDAGOGIE & INFOS**
  - [Guide & Contribution](#guide-de-contribution-et-utilisation-pédagogique)
    - [Comment contribuer ?](#comment-contribuer)
    - [Standards Cisco IOS](#standards-de-code-cisco-ios)
    - [Signaler un Bug](#signaler-un-bug)
  - [Licence & Auteur](#licence-et-auteur)

---

## Description du Projet

Projet d'infrastructure réseau avancée implémentant une **architecture de confédération BGP** avec 13 routeurs Cisco répartis sur 7 systèmes autonomes (AS). Ce projet démontre la mise en œuvre de concepts réseau complexes incluant iBGP, eBGP, et la confédération BGP pour améliorer la scalabilité.

### Objectifs

- Démontrer l'implémentation complète d'une confédération BGP.
- Illustrer les bonnes pratiques de configuration BGP et de conception de topologie.
- Fournir une infrastructure réseau exhaustive et documentée (plan d'adressage, configurations, tests).
- Servir de référence pour l'apprentissage, les certifications (CCNP, CCIE) et le déploiement sur GNS3/EVE-NG.

---

## Architecture et Topologie

### Vue d'ensemble des Systèmes Autonomes (AS)

L'infrastructure Nelson Networking est composée de **13 routeurs**, avec une architecture de **confédération BGP** au cœur du réseau :

| AS | Type | Description | Routeurs |
|----|------|-------------|----------|
| **AS 65000** | Backbone principal | AS de transit central (Full-Mesh iBGP) | R1, R2, R3 |
| **AS 65010** | Confédération | AS confédéré (Identifier BGP) | R4 à R9 |
| **AS 65011** | Sub-AS | Membre de la confédération (Triangle iBGP) | R4, R5, R6 |
| **AS 65012** | Sub-AS | Membre de la confédération (Triangle iBGP) | R7, R8, R9 |
| **AS 65060** | Client | Héberge le serveur WEB-SRV | R10 |
| **AS 65070** | Client | Héberge le poste PC1 | R11 |
| **AS 65080** | Client | Héberge le serveur FTP-SRV | R12 |
| **AS 65090** | Client | Héberge le poste PC2 | R13 |

### Schéma de la Topologie

![Topologie Réseau](./1.Documentation/TOPOLOGIE%204.png)

#### Connexions principales :
* **Backbone AS 65000** : Connecté en triangle (R1 ↔ R2 ↔ R3).
* **Sub-AS 65011** : Connecté en triangle (R4 ↔ R5 ↔ R6). R4 est la passerelle vers AS 65000.
* **Sub-AS 65012** : Connecté en triangle (R7 ↔ R8 ↔ R9). R7 est la passerelle vers l'extérieur.
* **Interconnexions externes** :
  - **R1 (AS 65000)** ↔ **R10 (AS 65060)** - Accès WEB-SRV
  - **R1 (AS 65000)** ↔ **R4 (AS 65011)** - Connexion confédération
  - **R3 (AS 65000)** ↔ **R11 (AS 65070)** - Accès PC1
  - **R7 (AS 65012)** ↔ **R12 (AS 65080)** - Accès FTP-SRV
  - **R8 (AS 65012)** ↔ **R13 (AS 65090)** - Accès PC2

---

## Plan d'Adressage

### Interfaces Loopback (Router-ID)

Chaque routeur possède une interface Loopback0 utilisée comme Router-ID BGP.

| Routeur | Loopback0 (Router-ID) | Routeur | Loopback0 (Router-ID) |
|---------|-----------------------|---------|-----------------------|
| R1      | `1.1.1.1/32`          | R8      | `8.8.8.8/32`          |
| R2      | `2.2.2.2/32`          | R9      | `9.9.9.9/32`          |
| R3      | `3.3.3.3/32`          | R10     | `10.10.10.10/32`      |
| R4      | `4.4.4.4/32`          | R11     | `11.11.11.11/32`      |
| R5      | `5.5.5.5/32`          | R12     | `12.12.12.12/32`      |
| R6      | `6.6.6.6/32`          | R13     | `13.13.13.13/32`      |
| R7      | `7.7.7.7/32`          | -       | -                     |

### Liens Point-à-Point (Réseaux de Transit)

| Lien | Subnet | Routeur A (Interface) | Routeur B (Interface) |
|------|--------|-----------------------|-----------------------|
| **R1 ↔ R10** | `10.0.0.0/30` | R1: e0/0 (10.0.0.2) | R10: e0/0 (10.0.0.1) |
| **R1 ↔ R4** | `10.0.0.4/30` | R1: e0/3 (10.0.0.5) | R4: e0/0 (10.0.0.6) |
| **R7 ↔ R4** | `10.0.0.8/30` | R7: e0/0 (10.0.0.10) | R4: (10.0.0.9) |
| **R7 ↔ R12** | `10.0.0.12/30` | R7: (10.0.0.13) | R12: e0/0 (10.0.0.14) |
| **R1 ↔ R2** | `10.0.1.0/30` | R1: e0/1 (10.0.1.1) | R2: e0/0 (10.0.1.2) |
| **R1 ↔ R3** | `10.0.1.4/30` | R1: e0/2 (10.0.1.5) | R3: e0/0 (10.0.1.6) |
| **R2 ↔ R3** | `10.0.1.8/30` | R2: e0/1 (10.0.1.9) | R3: e0/1 (10.0.1.10) |
| **R4 ↔ R5** | `10.0.2.0/30` | R4: e0/1 (10.0.2.1) | R5: e0/1 (10.0.2.2) |
| **R4 ↔ R6** | `10.0.2.4/30` | R4: e0/2 (10.0.2.5) | R6: e0/1 (10.0.2.6) |
| **R5 ↔ R6** | `10.0.2.8/30` | R5: e0/0 (10.0.2.9) | R6: e0/0 (10.0.2.10) |
| **R7 ↔ R9** | `10.0.3.0/30` | R7: e0/1 (10.0.3.1) | R9: e0/1 (10.0.3.2) |
| **R7 ↔ R8** | `10.0.3.4/30` | R7: e0/2 (10.0.3.5) | R8: e0/0 (10.0.3.6) |
| **R8 ↔ R9** | `10.0.3.8/30` | R8: e0/1 (10.0.3.9) | R9: e0/0 (10.0.3.10) |
| **R3 ↔ R11** | `10.0.4.0/30` | R3: e0/2 (10.0.4.1) | R11: e0/0 (10.0.4.2) |
| **R8 ↔ R13** | `10.0.5.0/30` | R8: e0/2 (10.0.5.1) | R13: e0/0 (10.0.5.2) |

### Réseaux LAN Clients

| Réseau | AS | Routeur | Hôte Client |
|--------|----|---------|-------------|
| `192.168.60.0/24` | 65060 | R10 | WEB-SRV |
| `192.168.70.0/24` | 65070 | R11 | PC1 |
| `192.168.80.0/24` | 65080 | R12 | FTP-SRV |
| `192.168.90.0/24` | 65090 | R13 | PC2 |

---

## Configuration BGP

### 1. Backbone Principal (AS 65000)
- **iBGP Full-Mesh** entre R1, R2, R3.
- Peering eBGP avec les clients AS 65060 (R10) et AS 65070 (R11).
- Peering eBGP avec la confédération (vers R4).

### 2. Confédération (AS 65010)
La confédération divise un grand AS en sous-AS pour des raisons de scalabilité. La configuration spécifique requiert l'identification de la confédération globale :
```cisco
router bgp 65011
 bgp confederation identifier 65010
 bgp confederation peers 65011 65012
```
- **Sub-AS 65011 (R4, R5, R6)** : Connecté au backbone via R4.
- **Sub-AS 65012 (R7, R8, R9)** : Connecté aux clients via R7 et R8.

### 3. AS Clients
Les AS stub (65060, 65070, 65080, 65090) ont une relation eBGP unique avec le fournisseur et annoncent leurs réseaux locaux (`network 192.168.x.0 mask 255.255.255.0`).

---

## 📁 Structure du Projet

```text
TP_CONFED_BGP/
│
├── 1.Documentation/
│   ├── Doc_TP_Confed_BGP.pdf               # Guide réseau exhaustif
│   └── TOPOLOGIE 4.png                     # Diagramme réseau
│
├── 2.Configs/                              # Fichiers de config prêts à l'emploi
│
├── README.md                               # La présentation du projet
└── LICENSE                                 # MIT
```

---

## Procédure de Déploiement

### 1. Prérequis
- **Émulateur réseau** : GNS3 (v2.2+) ou EVE-NG.
- **Images IOS** : Cisco IOSv ou IOL compatibles BGP.

### 2. Importer la topologie
Créez la topologie sur GNS3/EVE-NG en respectant scrupuleusement les interfaces et connexions du schéma de la topologie. Allumez les routeurs de R1 à R13.

### 3. Déployer les configurations
Transférez les configurations standardisées sur vos routeurs (par exemple, en utilisant SCP ou en copiant le contenu directement dans la console) :
```bash
# Exemple de transfert via SCP vers l'AS 65000
scp 2.Configs/configs_R1.cfg admin@10.0.0.2:/nvram:startup-config
scp 2.Configs/Configs_R2.cfg admin@<IP-R2>:/nvram:startup-config
scp 2.Configs/Configs_R3.cfg admin@<IP-R3>:/nvram:startup-config

# Poursuivre de la même manière pour R4 à R13...
```
*Note : Si SSH n'est pas activé initialement sur vos routeurs virtuels, le plus simple est de copier-coller le contenu des fichiers `Configs_R*.cfg` directement dans la console série.*

### 4. Sauvegarde
Sur chaque routeur, assurez-vous de sauvegarder :
```cisco
copy running-config startup-config
```

---

## Vérification et Tests

Une fois le réseau en place, vous devez vérifier son fonctionnement.

### 1. Connectivité de base
```cisco
show ip interface brief
show ip route
ping <neighbor-ip>
```

### 2. Vérification BGP
Les sessions BGP doivent toutes être dans l'état `Established` :
```cisco
show ip bgp summary
show ip bgp neighbors | include BGP state
```
Vérification des routes annoncées et reçues :
```cisco
show ip bgp
show ip bgp neighbors 10.0.0.2 received-routes
show ip bgp neighbors 10.0.0.2 advertised-routes
```

### 3. Vérification de la Confédération
Sur R4 à R9, l'attribut `AS_CONFED_SEQUENCE` doit apparaître dans les chemins d'AS pour le routage interne à la confédération.
```cisco
show ip bgp
show run | section bgp
```

### 4. Tests de bout en bout (Ping & Traceroute)

**WEB-SRV vers PC1 (Traversée du Backbone)** :
```cisco
! Sur R10 (ou le serveur Web)
ping 192.168.70.1 source 192.168.60.1
traceroute 192.168.70.1 source 192.168.60.1
! Chemin attendu : R10 → R1 → R3 → R11
```

**WEB-SRV vers FTP-SRV (Traversée de la Confédération)** :
```cisco
! Sur R10
ping 192.168.80.1 source 192.168.60.1
! Chemin attendu : R10 → R1 → R4 → R5/R6 → R7 → R12
```

---

## Dépannage BGP

### Session en état "Idle" ou "Active"
* **Diagnostic** : Problème de peering ou connectivité TCP (port 179).
* **Commandes utiles** : `show ip bgp summary`, `ping <voisin>`.
* **Solutions courantes** : Vérifier le routage IP, l'exactitude des `remote-as`, ou relancer la session avec `clear ip bgp <neighbor-ip>`.

### Routes BGP absentes ou non propagées
* **Diagnostic** : Commande `network` manquante, ou problème de Next-Hop.
* **Solutions courantes** : Vérifier que la route exacte est présente dans la table de routage globale, et que l'iBGP n'est pas bloqué par la règle Split-Horizon. Ajouter `next-hop-self` si besoin sur le routeur de bordure.

### Boucle de Routage ou Reachability
* **Diagnostic** : Le traceroute s'arrête en plein milieu. 
* **Vérification** : `show ip bgp <destination>`, observez le `Next Hop`. L'IP du saut suivant doit être joignable via l'IGP ou statiquement.

### Commandes de Debugging
*(À utiliser avec modération, risque de surcharge CPU)*
```cisco
debug ip bgp events
debug ip bgp updates
undebug all
```

---

## 🤝 Guide de Contribution et Utilisation Pédagogique

Ce projet constitue une **base idéale pour les laboratoires de certification (CCNA/CCNP/CCIE)**. Que ce soit pour ajouter des tunnels IPsec, des VPN MPLS, ou améliorer la documentation, vos contributions sont les bienvenues !

### Comment contribuer ?

1. **Forkez** le projet en haut à droite du dépôt.
2. **Clonez** votre fork en local et créez une branche :
   ```bash
   git checkout -b feature/ma-nouvelle-fonctionnalite
   ```
3. **Committez** vos modifications en respectant notre standard de messages :
   * `feat:` Nouvelle fonctionnalité (ex: `feat: ajout route-reflector pour R1`)
   * `fix:` Correction de bug
   * `docs:` Modification de documentation
   * `style:` Formatage des configurations Cisco
   
4. **Poussez** vers votre fork :
   ```bash
   git push origin feature/ma-nouvelle-fonctionnalite
   ```
5. Ouvrez une **Pull Request** sur GitHub !

### Standards de Code (Cisco IOS)
Veuillez suivre le format standardisé utilisé dans le dossier `2.Configs/` (bannières de début/fin, `enable`, `configure terminal`, indentation des sous-interfaces, et `write memory` à la fin).

### Signaler un Bug
Si vous rencontrez un problème (ex: boucle de routage, session BGP Idle), ouvrez une **Issue** en indiquant :
- L'OS et l'émulateur utilisé (GNS3 / EVE-NG).
- L'image IOS testée.
- Les étapes pour reproduire, et idéalement un résultat de commande `show ip bgp summary` ou `traceroute`.

## Licence et Auteur

**Licence** : Projet distribué sous licence MIT. Voir le fichier [LICENSE](LICENSE) pour plus de détails.

**Nelson Bandos**
- [GitHub](https://github.com/Nelson2410)
- [LinkedIn](https://linkedin.com/in/nelson-bandos)

Pour tout problème, n'hésitez pas à ouvrir une [issue](https://github.com/Nelson2410/_BGP/issues) ou à consulter la [documentation PDF complète](./1.Documentation/Doc_TP_Confed_BGP.pdf) présente dans ce dépôt.