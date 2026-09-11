# Redstone Engineering - Projet Réseau Cisco
> Réalisé en mai 2026

## Description du projet
### Simulation d'une infrastructure réseau d'entreprise multi-sites
Contexte: **Redstone Engineering**, une startup fictive spécialisée dans l'hydrogène grand public.

L'infrastructure couvre **deux sites interconnectés** :
- **Site A, Siège social PARIS** : pôles Direction, RH, IT et Serveurs,
- **Site B, Agence LYON** : pôles RH et Recherche.

## Schéma topologie
![schema-cisco](./schema.png)

## Compétences mises en place

| Domaine | Technologie | Description |
|---|---|---|
| **Segmentation** | VLANs 802.1Q | 6 VLANs répartis sur 2 sites |
| **Routage L3** | Switch 3560 + SVIs | Routage inter-VLAN sans passer par le routeur |
| **Routage dynamique** | OSPFv2 | Propagation automatique des routes entre sites |
| **Adressage automatique** | DHCP + Relay | Serveur centralisé sur Site A, relay sur Site B |
| **Résolution de noms** | DNS | Enregistrements A pour les serveurs internes |
| **Administration sécurisée** | SSH v2 | Configuré sur les 10 équipements réseau |
| **Sécurité des ports** | Port Security Sticky | Détection et blocage des équipements non autorisés |
| **Liaison WAN** | Serial DCE/DTE + OSPF | Interconnexion simulée entre les deux sites |

## Structure du projet

```
redstone-eng/
 ┣ README.md              
 ┣ documentation.md       # Documentation technique complète
 ┣ schema.png             # Schema PNG du réseau
 ┗ main.pkt               # Fichier Packet Tracer
```

## Documentation complète

La documentation technique détaillée est [disponible ici](./documentation.md).

## Auteur

**Lucas Goulain-Roubanoff**
