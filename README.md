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

### Réponse 2
VRRP (Virtual Router Redundancy Protocol) est un protocole dont le but est d'augmenter la disponibilité de la passerelle par défaut des hôtes d'un même réseau. Dans notre cas, il permet de définir une seule adresse IP virtuelle comme passerelle par défaut référençant nos deux routeurs pour les hôtes de notre réseau.

### Question 3 
Expliquer le fonctionnement de VRRP qui permet aux machines A et B d’utiliser le
« bon » routeur. Comment fonctionne ce mécanisme lorsque le routeur utilisé devient défaillant ?

### Réponse 3 
Le protocole VRRP permet de répondre au besoin de haute disponibilité. Pour cela, les routeurs s'envoient entre eux des annonces pour signaler leurs disponibilités. Si le routeur maître, à savoir, le routeur principal est défaillant, alors le routeur esclave/secondaire prendra le rôle de maître. Ce qui permettra la haute disponibilité, c'est à dire une communication sans interruption en cas de défaillance.

### Question 4
Expliquer en quelques lignes le rôle du protocole OSPF dans le réseau ci-dessus.
Justifier entre autres que l’utilisation du routage statique n’aurait pas été pertinente.

### Réponse 4
Dans ce réseau, OSPF a pour mission de définir les itinéraires les plus efficaces pour le transit des paquets au sein du réseau. Dans notre contexte, OSPF actualise les tables de routage des routeurs dans l'aire OSPF en évaluant les coûts associés à chaque chemin. Le chemin avec le coût le plus bas est considéré comme le plus rapide pour atteindre le réseau de destination.

Le recours au routage statique serait inapproprié en raison du nombre élevé de réseaux et de routeurs. Configurer le routage statique sur l'ensemble des routeurs aurait été contraignant et laborieux. De plus, grâce à l'utilisation de VRRP, l'intégration d'OSPF permet une mise à jour automatique des tables de routage en cas de défaillance d'un lien, assurant ainsi une haute disponibilité du réseau.
