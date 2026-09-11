# Projet Réseau d'Entreprise — Redstone Engineering
### Documentation technique
> Réalisé sur Cisco Packet Tracer

## Présentation du projet

J'ai conçu et simulé l'infrastructure réseau complète d'une entreprise fictive appelée **Redstone Engineering**, une startup spécialisée dans l'hydrogène grand public. Ce projet m'a permis de mettre en pratique les compétences réseau attendues en entreprise : segmentation par VLANs, routage dynamique, sécurisation des accès et automatisation de l'adressage IP.

L'infrastructure couvre deux sites interconnectés :
- **Site A — Siège social (Paris)** : pôles Direction, RH, IT et Serveurs
- **Site B — Agence (Lyon)** : pôles RH et Recherche

Les objectifs techniques que je me suis fixés sont :
- Segmenter le réseau par VLANs pour isoler les pôles
- Automatiser l'adressage IP via DHCP
- Assurer le routage dynamique entre les sites avec OSPF
- Sécuriser les accès aux équipements via SSH
- Protéger les ports contre les connexions non autorisées avec Port Security
- Filtrer le trafic réseau avec des ACLs

## Étape 1 — Topologie et plan d'adressage

### Choix des équipements

Pour ce projet, j'ai sélectionné des équipements Cisco réalistes, représentatifs de ce qu'on trouve en entreprise.

**Routeur ISR4331** — J'ai choisi ce routeur de gamme entreprise pour gérer l'interconnexion WAN entre les deux sites et le routage dynamique OSPF. J'y ai ajouté le module **NIM-2T** pour disposer d'interfaces Serial, nécessaires à la simulation de la liaison WAN.

**Switch cœur 3560-24PS (Multilayer/L3)** — J'ai opté pour ce switch Layer 3 car il est capable de router les paquets entre les VLANs directement au cœur du réseau, sans surcharger le routeur qui se concentre uniquement sur le WAN. C'est une architecture plus performante et réaliste.

**Switches d'accès 2960-24TT (L2)** — Pour les switches d'accès, j'ai utilisé des 2960, modèle standard en entreprise. Le routage n'étant pas nécessaire à ce niveau, un switch Layer 2 suffit et est plus économique.

### Équipements déployés

| Équipement | Site A (Paris) | Site B (Lyon) |
|---|---|---|
| Routeur | 1x ISR4331 | 1x ISR4331 |
| Switch cœur L3 | 1x 3560-24PS | 1x 3560-24PS |
| Switches d'accès L2 | 4x 2960-24TT | 2x 2960-24TT |
| Serveurs | 3 (DHCP, DNS, WEB) | 0 |
| Postes utilisateurs | 2-3 par VLAN | 2-3 par VLAN |

### Plan d'adressage VLANs

J'ai choisi une convention de nommage en dizaines pour les VLANs (10, 20, 30...), ce qui est une pratique courante en entreprise pour faciliter la lecture et la gestion.

| VLAN | Nom | Site | Réseau | Passerelle |
|---|---|---|---|---|
| VLAN 10 | Direction | A | 192.168.10.0/24 | 192.168.10.1 |
| VLAN 20 | RH | A | 192.168.20.0/24 | 192.168.20.1 |
| VLAN 30 | IT | A | 192.168.30.0/24 | 192.168.30.1 |
| VLAN 40 | Servers | A | 192.168.40.0/24 | 192.168.40.1 |
| VLAN 50 | RH-2 | B | 192.168.50.0/24 | 192.168.50.1 |
| VLAN 60 | Research | B | 192.168.60.0/24 | 192.168.60.1 |

### Plan d'adressage liaisons réseau

Une **liaison point-à-point** est une liaison directe entre deux équipements uniquement, sans aucun autre équipement sur ce lien. Pour ce type de liaison, j'ai utilisé des réseaux **/30** qui offrent exactement 2 adresses hôtes utilisables — parfait pour économiser les adresses IP.

| Liaison | Réseau | IP SW-Core / Routeur |
|---|---|---|
| SW-Core-SiteA ↔ Router-SiteA | 192.168.0.0/30 | 192.168.0.1 / 192.168.0.2 |
| SW-Core-SiteB ↔ Router-SiteB | 192.168.0.4/30 | 192.168.0.5 / 192.168.0.6 |
| WAN Router-SiteA ↔ Router-SiteB | 10.0.0.0/30 | 10.0.0.1 / 10.0.0.2 |

### Adressage statique des serveurs

J'ai attribué des adresses IP statiques aux serveurs car ils doivent être joignables en permanence à la même adresse. J'ai choisi de les placer en fin de plage (*.250, *.251, *.252) pour les distinguer facilement des adresses distribuées par DHCP.

| Serveur | IP | Masque | Passerelle |
|---|---|---|---|
| SRV-DHCP | 192.168.40.250 | 255.255.255.0 | 192.168.40.1 |
| SRV-DNS | 192.168.40.251 | 255.255.255.0 | 192.168.40.1 |
| SRV-WEB | 192.168.40.252 | 255.255.255.0 | 192.168.40.1 |

---

## Étape 2 — Câblage

Pour le câblage, j'ai appliqué les règles standard Cisco selon les types d'équipements connectés :

| Liaison | Câble |
|---|---|
| PC / Laptop / Serveur → Switch accès | Copper Straight-Through |
| Switch accès → Switch cœur | Copper Cross-Over |
| Switch cœur → Routeur | Copper Straight-Through |
| Router SiteA → Router SiteB | Serial DCE/DTE |

Pour la liaison WAN entre les deux routeurs, j'ai utilisé un câble **Serial DCE/DTE** qui simule une vraie liaison WAN. Le côté DCE (Router-SiteA) fournit l'horloge à la liaison via la commande `clock rate 64000`.

---

## Étape 3 — Configuration des VLANs et SVIs

### Bonne pratique appliquée
Sur chaque switch, j'ai respecté l'ordre suivant pour éviter les erreurs de configuration :
1. Créer les VLANs et leur donner un nom
2. Vérifier les ports connectés avec `show interfaces status`
3. Configurer les ports Access
4. Configurer les ports Trunk
5. Vérifier avec `show vlan brief`

### Création des VLANs

J'ai créé les VLANs sur chaque switch **avant** toute assignation de ports, ce qui est la bonne pratique pour garantir une configuration propre.

```
vlan 10
name Direction
exit
```

### Configuration des ports Access et Trunk

J'ai configuré chaque port selon l'équipement auquel il est connecté :
- **Access** pour les équipements finaux (PCs, serveurs) — un seul VLAN par port
- **Trunk** pour les liaisons entre équipements réseau — transporte plusieurs VLANs tagués (802.1Q)

```
! Ports Access
interface range FastEthernet0/1 - 2
switchport mode access
switchport access vlan 10
exit

! Port Trunk
interface FastEthernet0/3
switchport mode trunk
exit
```

> ⚠️ Sur un switch L3 (3560/3750), j'ai dû spécifier l'encapsulation dot1q avant de configurer le mode trunk, contrairement au switch L2 (2960) qui l'utilise par défaut :
> ```
> switchport trunk encapsulation dot1q
> switchport mode trunk
> ```

### Activation du routage IP sur le Switch L3

Sur un switch Cisco L3, le routage est désactivé par défaut. J'ai activé cette fonctionnalité avec la commande suivante, indispensable pour le routage inter-VLAN :

```
ip routing
```

### Interfaces virtuelles (SVI)

J'ai créé une SVI (Switch Virtual Interface) par VLAN sur chaque switch cœur. Ces interfaces virtuelles servent de **passerelle par défaut** pour les équipements de chaque VLAN et assurent le routage inter-VLAN directement au niveau du switch L3, sans solliciter le routeur.

```
interface vlan 10
ip address 192.168.10.1 255.255.255.0
no shutdown
exit
```

#### SVIs Site A (SW-Core-SiteA)

| SVI | IP Passerelle |
|---|---|
| VLAN 10 Direction | 192.168.10.1 |
| VLAN 20 RH | 192.168.20.1 |
| VLAN 30 IT | 192.168.30.1 |
| VLAN 40 Servers | 192.168.40.1 |

#### SVIs Site B (SW-Core-SiteB)

| SVI | IP Passerelle |
|---|---|
| VLAN 50 RH-2 | 192.168.50.1 |
| VLAN 60 Research | 192.168.60.1 |

### Port routed vers le routeur

Pour la liaison entre le switch cœur et le routeur, j'ai transformé le port en **routed port** (interface L3 pure) avec la commande `no switchport`. Cela crée une liaison point-à-point propre entre les deux équipements.

```
interface FastEthernet0/X
no switchport
ip address 192.168.0.1 255.255.255.252
no shutdown
exit
```

### Vérification

```
show ip interface brief           ! Vérifier les SVIs et leur statut
show interfaces Fa0/1 switchport  ! Vérifier la config d'un port
show vlan brief                   ! Vérifier les VLANs et leurs ports
```

---

## Étape 4 — Configuration du DHCP

### Pools DHCP

J'ai configuré le service DHCP sur SRV-DHCP. Un pool DHCP contient toutes les informations nécessaires pour qu'un équipement obtienne automatiquement sa configuration réseau : nom du pool, passerelle, serveur DNS et plage d'IPs distribuables.

Je n'ai pas créé de pool pour le VLAN 40 car il ne contient que des serveurs avec des adresses IP statiques.

#### Pools Site A

| Pool | Gateway | DNS | Start IP |
|---|---|---|---|
| Direction | 192.168.10.1 | 192.168.40.251 | 192.168.10.100 |
| RH | 192.168.20.1 | 192.168.40.251 | 192.168.20.100 |
| IT | 192.168.30.1 | 192.168.40.251 | 192.168.30.100 |

#### Pools Site B

| Pool | Gateway | DNS | Start IP |
|---|---|---|---|
| RH-2 | 192.168.50.1 | 192.168.40.251 | 192.168.50.100 |
| Research | 192.168.60.1 | 192.168.40.251 | 192.168.60.100 |

### DHCP Relay

Sans configuration supplémentaire, les requêtes DHCP des PCs restent locales à leur VLAN — elles sont envoyées en broadcast et ne franchissent pas le switch d'accès. Le serveur DHCP étant en VLAN 40, il ne reçoit jamais ces requêtes.

J'ai résolu ce problème en configurant `ip helper-address` sur chaque SVI des switches cœur. Cette commande indique au switch de **relayer** les broadcasts DHCP vers l'adresse IP du serveur DHCP.

```
interface vlan 10
ip helper-address 192.168.40.250
exit
```

Cette configuration a été appliquée sur les SVIs des VLANs 10, 20, 30 du Site A et des VLANs 50, 60 du Site B.

### Test de connectivité
Après activation du DHCP sur tous les postes, j'ai effectué un ping entre deux machines de VLANs différents avec succès, confirmant que le routage inter-VLAN fonctionne correctement via les SVIs du SW-Core.

---

## Étape 5 — Interconnexion des sites via OSPF

### Pourquoi OSPF ?

J'ai choisi OSPF comme protocole de routage dynamique plutôt que du routage statique. Avec le routage statique, chaque route doit être saisie manuellement sur chaque équipement — c'est fastidieux et peu flexible. OSPF permet aux équipements réseau de **découvrir et partager automatiquement** leurs routes. Si un nouveau VLAN est ajouté, il est propagé sans intervention manuelle.

### Configuration des interfaces routeurs

Avant de configurer OSPF, j'ai attribué des adresses IP aux interfaces des deux routeurs :

```
! Router-SiteA
interface GigabitEthernet0/0/0
ip address 192.168.0.2 255.255.255.252
no shutdown
exit

interface Serial0/1/0
ip address 10.0.0.1 255.255.255.252
clock rate 64000
no shutdown
exit

! Router-SiteB
interface GigabitEthernet0/0/0
ip address 192.168.0.6 255.255.255.252
no shutdown
exit

interface Serial0/1/0
ip address 10.0.0.2 255.255.255.252
no shutdown
exit
```

### Configuration OSPF

J'ai configuré OSPF sur les deux routeurs et les deux switches cœur avec la commande `router ospf 1`. Le **"1"** est un numéro de processus local à chaque équipement — il identifie l'instance OSPF et n'a pas besoin d'être identique sur tous les équipements. L'**area 0** est la zone backbone obligatoire dans tout réseau OSPF — dans notre cas on n'a qu'une seule area car le réseau est simple.

Chaque équipement n'annonce que ses réseaux directement connectés — OSPF se charge ensuite de propager ces routes à tous les autres équipements.

```
! Exemple Router-SiteA
router ospf 1
network 10.0.0.0 0.0.0.3 area 0
network 192.168.0.0 0.0.0.3 area 0

! Exemple SW-Core-SiteA
router ospf 1
network 192.168.10.0 0.0.0.255 area 0
network 192.168.20.0 0.0.0.255 area 0
network 192.168.30.0 0.0.0.255 area 0
network 192.168.40.0 0.0.0.255 area 0
network 192.168.0.0 0.0.0.3 area 0
```

> 💡 Le **wildcard mask** utilisé dans OSPF est l'inverse du masque de sous-réseau :
> - /24 (255.255.255.0) → wildcard **0.0.0.255**
> - /30 (255.255.255.252) → wildcard **0.0.0.3**

### Vérification OSPF

```
show ip ospf neighbor    ! Vérifier les voisins OSPF (état FULL = OK)
show ip route            ! Vérifier la table de routage (routes "O" = apprises via OSPF)
```

### Test de connectivité inter-sites
J'ai effectué un ping depuis PC-Research-1 (Site B — 192.168.60.x) vers PC-Direction-1 (Site A — 192.168.10.x) avec succès. Ce test confirme que le routage OSPF fonctionne correctement entre les deux sites et que tous les VLANs peuvent communiquer à travers la liaison WAN.

---

## Étape 6 — Sécurisation SSH

### Pourquoi SSH plutôt que Telnet ?

J'ai configuré SSH sur tous les équipements réseau pour sécuriser leur administration à distance. Telnet, l'alternative non sécurisée, transmet toutes les données **en clair** — un simple outil de capture comme Wireshark suffit à intercepter les mots de passe. SSH chiffre l'intégralité de la communication, rendant toute interception illisible. C'est le standard en production.

### Ce que j'ai configuré

Sur chaque équipement réseau, j'ai mis en place les éléments suivants :

- **Nom de domaine** `redstone-eng.local` — requis pour la génération des clés RSA
- **Clé RSA 1024 bits** — assure le chiffrement de la session SSH
- **SSH version 2** — plus sécurisé que la v1
- **Compte local** `admin` — authentification par couple username/password
- **5 lignes VTY (0-4)** — permet jusqu'à 5 sessions d'administration simultanées
- **`login local`** — force l'utilisation des comptes locaux plutôt qu'un mot de passe partagé, ce qui permet de tracer les connexions et de gérer les accès individuellement

```
ip domain-name redstone-eng.local
username admin secret RedstoneEng2024
crypto key generate rsa
ip ssh version 2
line vty 0 4
transport input ssh
login local
exit
```

### Équipements concernés

J'ai appliqué cette configuration sur **tous les équipements réseau** : 2 routeurs, 2 switches cœur et 6 switches d'accès — soit 10 équipements au total. Les postes utilisateurs ne sont pas concernés par SSH ; dans un environnement réel, leur administration passerait par des outils de gestion centralisée dédiés.

### Vérification
```
show ip ssh    ! Doit afficher "SSH Enabled - version 2.0"
```

---

## Étape 7 — Port Security

### Pourquoi Port Security ?

Port Security permet de contrôler quelles machines peuvent se connecter physiquement sur les ports des switches d'accès. Sans cette protection, n'importe qui pourrait brancher un équipement non autorisé sur une prise réseau et accéder au réseau de l'entreprise.

### Choix de la méthode

J'ai réfléchi aux deux approches disponibles pour l'apprentissage des adresses MAC autorisées :

- **Statique** → on saisit manuellement chaque adresse MAC. Fastidieux à maintenir et peu pratique à chaque nouvel équipement.
- **Sticky** → le switch apprend automatiquement les premières MACs connectées et les mémorise. Plus pratique, mais le premier équipement connecté est automatiquement autorisé sans vérification préalable.

Dans un environnement de production, on utiliserait une solution d'authentification réseau plus robuste pour un contrôle granulaire des accès. Dans le cadre de ce projet, j'ai opté pour la méthode **Sticky avec violation Shutdown** — le meilleur compromis disponible : apprentissage automatique des machines en place et blocage complet de tout nouvel équipement non reconnu.

### Configuration

J'ai appliqué Port Security uniquement sur les **switches d'accès**, sur les ports en mode Access uniquement (jamais sur un port Trunk).

```
interface range FastEthernet0/1 - 2
switchport port-security
switchport port-security maximum 1
switchport port-security mac-address sticky
switchport port-security violation shutdown
exit
```

- **maximum 1** → une seule adresse MAC autorisée par port
- **sticky** → apprentissage automatique et mémorisation de la MAC
- **violation shutdown** → le port se désactive complètement si une MAC non autorisée tente de se connecter

### Équipements concernés
Port Security a été configuré sur les 6 switches d'accès des deux sites, sur tous les ports connectés à des équipements utilisateurs.

### Vérification
```
show port-security interface FastEthernet0/1
```

---

## Étape 8 — Configuration DNS

J'ai configuré le service DNS sur **SRV-DNS** (192.168.40.251) afin de permettre aux utilisateurs de joindre les serveurs par leur nom plutôt que par leur adresse IP. Sans DNS, il faudrait retenir l'IP de chaque serveur.

J'ai créé des enregistrements de type **A** (Address) — qui associent un nom de domaine à une adresse IPv4. Le type **AAAA** est son équivalent pour les adresses IPv6. L'extension **.local** a été choisie car il s'agit d'un domaine interne, non accessible depuis l'extérieur du réseau.

| Enregistrement | Type | IP |
|---|---|---|
| srv-web.redstone-eng.local | A | 192.168.40.252 |
| srv-dhcp.redstone-eng.local | A | 192.168.40.250 |
| srv-dns.redstone-eng.local | A | 192.168.40.251 |

Test effectué avec succès : `ping srv-web.redstone-eng.local` depuis PC-Direction-1 résout correctement l'IP et confirme la connectivité.

---

## Étape 9 — ACLs — Politique de sécurité

### Politique choisie
J'ai défini une politique de **moindre privilège** sur l'accès au serveur DHCP : seul le pôle **IT (VLAN 30)** est autorisé à y accéder, car c'est l'équipe qui gère l'infrastructure réseau. Les VLANs Direction (10), RH Site A (20), RH Site B (50) et Research (60) sont bloqués.

Le serveur DNS reste accessible à tous les VLANs — le bloquer empêcherait la résolution de noms sur l'ensemble du réseau.

### Règles configurées
J'ai utilisé une **ACL étendue** car il fallait filtrer à la fois sur l'adresse IP source (le VLAN) et la destination (le serveur DHCP). Une ACL standard n'aurait pas permis de cibler une destination précise.

```
ip access-list extended PROTECT-SERVERS
deny ip 192.168.10.0 0.0.0.255 host 192.168.40.250
deny ip 192.168.20.0 0.0.0.255 host 192.168.40.250
deny ip 192.168.50.0 0.0.0.255 host 192.168.40.250
deny ip 192.168.60.0 0.0.0.255 host 192.168.40.250
permit ip any any
```

### Limitation rencontrée
Durant la configuration, j'ai rencontré une limitation connue de Packet Tracer : le switch 3560-24PS ne supporte pas correctement l'application des ACLs sur les interfaces SVI dans le simulateur. Cette limitation est documentée sur les forums Cisco Community et ne reflète pas le comportement du vrai matériel.

En environnement réel (GNS3 ou matériel physique), cette ACL appliquée en **in** sur les SVIs sources bloquerait effectivement l'accès au serveur DHCP pour les VLANs concernés.

---

## Conclusion

Ce projet m'a permis de concevoir et simuler une infrastructure réseau d'entreprise complète, de la topologie physique jusqu'à la sécurisation des accès. J'ai mis en pratique les compétences fondamentales attendues en BTS SIO SISR : segmentation par VLANs, routage inter-VLAN avec un switch L3, routage dynamique OSPF entre deux sites, automatisation de l'adressage IP via DHCP, et sécurisation des équipements via SSH et Port Security.

Ce projet m'a également appris à faire face aux limitations d'un simulateur et à documenter honnêtement les écarts entre la théorie et la pratique — une compétence essentielle dans le monde professionnel.

Les prochaines étapes d'amélioration possibles seraient :
- Tester le projet sur **GNS3** pour valider les ACLs en environnement réel
- Mettre en place le protocole **VTP** pour la propagation automatique des VLANs
- Simuler un accès **Internet** avec NAT/PAT sur les routeurs
- Mettre en place une **redondance** avec STP ou HSRP

