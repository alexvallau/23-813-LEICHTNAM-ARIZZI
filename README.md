### 23-813-LEICHTNAM-ARIZZI
Projet supervision

## Etude théorique préparatoire

### Question 1 
Combien de lignes doivent être, théoriquement, présentes dans votre table de routage
lorsque la topologie sera complète et lorsque les protocoles auront convergé ? Donner pour chacun
de vos routeurs, une table de routage partielle qui contiendra 7 routes choisies de manière pertinente.

### Réponse 1
Il existe 16 réseaux internes + celui du prof + le réseau 0.0.0.0 +le réseau externe qui interconnecte tous les routeurs + le réseau vlan 140 + le réseau vlan 176. Donc 16+1+1+1+1+1. Donc 21 sous réseaux.

#### Tables de routage 
#### R1
| Destination   |      Passerelles      |  Coût |
|----------|:-------------:|------:|
| 10.X.Y.0/24 |  DC | 0 |
| 10.250.0.0/24 |    DC   |   0 |
| 192.168.140.0/23 | 10.250.0.253 |    1 |
| 192.168.176.0/24 | 10.250.0.254 |    1 |
| 0.0.0.0/0 | 10.250.0.253 |    1 |

#### R2

| Destination   |      Passerelles      |  Coût |
|----------|:-------------:|------:|
| 10.X.Y.0/24 |  DC | 0 |
| 10.250.0.0/24 |    DC   |   0 |
| 192.168.140.0/23 | 10.250.0.253 |    1 |
| 192.168.176.0/24 | 10.250.0.254 |    1 |
| 0.0.0.0/0 | 10.250.0.254 |    1 |

### Question 2
Expliquer en 4 lignes le rôle du protocole VRRP qui sera mis en place dans les routeurs.

VRRP (Virtual Router Redundancy Protocol) est un protocole dont le but est d'augmenter la disponibilité de la passerelle par défaut des hôtes d'un même réseau. Dans notre cas, il permet de définir une seule adresse IP virtuelle comme passerelle par défaut référençant nos deux routeurs pour les hôtes de notre réseau.
