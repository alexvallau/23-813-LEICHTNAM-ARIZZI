### 23-813-LEICHTNAM-ARIZZI
Projet supervision

## Etude théorique préparatoire

#### Question 1 
_Combien de lignes doivent être, théoriquement, présentes dans votre table de routage
lorsque la topologie sera complète et lorsque les protocoles auront convergé ? Donner pour chacun
de vos routeurs, une table de routage partielle qui contiendra 7 routes choisies de manière pertinente._

#### Réponse 1
Il existe 16 réseaux internes (vue dans le tableau) + celui du prof + le réseau 0.0.0.0 +le réseau externe qui interconnecte tous les routeurs + le réseau vlan 140 + le réseau vlan 176. Donc 16+1+1+1+1+1. Donc 21 sous réseaux.

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

#### Question 2
_Expliquer en 4 lignes le rôle du protocole VRRP qui sera mis en place dans les routeurs._

#### Réponse 2
VRRP (Virtual Router Redundancy Protocol) est un protocole dont le but est d'augmenter la disponibilité de la passerelle par défaut des hôtes d'un même réseau. Dans notre cas, il permet de définir une seule adresse IP virtuelle comme passerelle par défaut référençant nos deux routeurs pour les hôtes de notre réseau.

#### Question 3 
_Expliquer le fonctionnement de VRRP qui permet aux machines A et B d’utiliser le
« bon » routeur. Comment fonctionne ce mécanisme lorsque le routeur utilisé devient défaillant ?_

### Réponse 3 
Commençons de manière simple et descriptive. Nos 2 routeurs ont 2 adresses IP différentes. Lorsqu'ils sont configurés avec le protocole VRRP, une nouvelle interface dite "virtuelle" est créée. Cette interface, comme toutes les interfaces, contient 2 informations :

* Une adresse MAC virtuelle unique et différente des vraies adresses MAC de nos routeurs.
* Une adresse IP virtuelle, qui peut être similaire à l'une des IPs de nos routeurs.

Nos routeurs vont donc faire partie de cette nouvelle interface virtuelle.\\

Le protocole VRRP fonctionne sous le modèle maître-esclave, que l'on peut paramétrer selon nos désirs en donnant une priorité à nos routeurs. Le routeur avec la priorité la plus haute sera le maître. De manière régulière, il annoncera aux autres routeurs de son cluster qu'il est bien fonctionnel et qu'il gère l'IP virtuelle.

À ce propos, c'est bien évidemment lui qui répondra aux requêtes ARP des différents clients(voir RFC3768 section "host ARP requests"), en envoyant la MAC virtuelle au client qui l'a demandée.
1. Si mon client A demande via ARP : "À quelle MAC correspond l'adresse IP virtuelle ?"
2. Le routeur maître répondra : "L'adresse IP virtuelle correspond à la MAC virtuelle de mon cluster."

Si un routeur n'envoie plus d'informations sur l'état de ses liens, ou même plus d'informations du tout, alors un processus de sélection d'un nouveau maître est mis en place. Comme déjà énoncé, le routeur avec la plus haute priorité (voire même avec la plus haute IP selon les modèles) est choisi comme nouveau routeur maître. Ce nouveau maître annoncera à son tour, à ses paires, son état de fonctionnement.

Ainsi, A et B ne "choisissent" pas le bon routeur. Du point de vue de ces clients, il n'existe qu'une seule entité de routage, la virtuelle. C'est par les différentes mécaniques expliquées ci-dessus que les routeurs échangent intelligemment leurs rôles, passant d'esclave à maître et assurant une redondance des liens.


#### Question 4
_Expliquer en quelques lignes le rôle du protocole OSPF dans le réseau ci-dessus.
Justifier entre autres que l’utilisation du routage statique n’aurait pas été pertinente._

#### Réponse 4
Dans ce réseau, OSPF a pour mission de définir les itinéraires les plus efficaces pour le transit des paquets au sein du réseau. Dans notre contexte, OSPF actualise les tables de routage des routeurs dans l'aire OSPF en évaluant les coûts associés à chaque chemin. Le chemin avec le coût le plus bas est considéré comme le plus rapide pour atteindre le réseau de destination.

Le recours au routage statique serait inapproprié en raison du nombre élevé de réseaux et de routeurs qui sont interconnectés. Configurer le routage statique sur l'ensemble des routeurs aurait été contraignant et laborieux. De plus, grâce à l'utilisation de VRRP, l'intégration d'OSPF permet une mise à jour automatique des tables de routage en cas de défaillance d'un lien, assurant ainsi une haute disponibilité du réseau.

## Mise en place et configuration des machines virtuelles

#### Question 5
_Rédiger les tests que vous mettrez en œuvre à la fin de cette étape pour valider le
fonctionnement du réseau. L’objectif est d’écrire le moins de tests possibles pour tester le plus de
fonctionnalités possibles._

#### Réponse 5
Pour l'instant, nous avons configuré les machines R1 Et client A. Afin d'être au plus clair sur les tests, nous presenterons les tests à effectuer par machine.
\\
* Machine A :
  IP: 10.100.4.3/24
  De machine A vers Routeur R1: On ping l'interface du routeur.

* Routeur R1
  * Interface loopback0 : 10.10.4.1
  * Interface GigabitEthernet1: DHCP(Interface de gestion de réseau)
  * Interface GigabitEthernet2(Interface réseau interne): 10.100.4.1
    * Ping de R1 vers R2
    * Ping de R1 vers Client A
  * Interface GigabitEthernet3(Interface réseau côté prof): 10.250.0.7
    * Ping de R1 vers 10.250.0.253(Réussi)
    * Ping de R1 vers 10.250.0.254(Réussi)
   
  #### Question 6
  _Rédiger les tests que vous mettrez en œuvre à la fin de cette étape pour valider la fonctionnement du réseau._

  #### Réponse 6
  Après avoir configuré OSPF sur notre routeur avec la configuration suivante
  
  ``` router ospf 1
   router-id 10.10.4.1
   network 10.100.4.0 0.0.0.255 area 0
   network 10.250.0.0 0.0.0.255 area 0 ```

Nous avons regardé la table de routage reçue par OSPF
  
```
R1#show ip route ospf
O*E2  0.0.0.0/0 [110/10] via 10.250.0.254, 00:54:58, GigabitEthernet3
                [110/10] via 10.250.0.253, 5d20h, GigabitEthernet3
      10.0.0.0/8 is variably subnetted, 13 subnets, 2 masks
O        10.10.2.1/32 [110/2] via 10.250.0.3, 00:38:20, GigabitEthernet3
O        10.10.2.2/32 [110/2] via 10.250.0.4, 00:00:31, GigabitEthernet3
O        10.100.1.0/24 [110/2] via 10.250.0.2, 00:22:23, GigabitEthernet3
O        10.100.2.0/24 [110/2] via 10.250.0.4, 00:00:31, GigabitEthernet3
                       [110/2] via 10.250.0.3, 00:38:20, GigabitEthernet3
O        10.100.5.0/24 [110/2] via 10.250.0.9, 00:17:30, GigabitEthernet3
O        10.200.3.0/24 [110/2] via 10.250.0.105, 00:42:14, GigabitEthernet3
O        10.200.4.0/24 [110/2] via 10.250.0.108, 6d19h, GigabitEthernet3
                       [110/2] via 10.250.0.107, 6d19h, GigabitEthernet3
O        10.200.6.0/24 [110/2] via 10.250.0.112, 5d20h, GigabitEthernet3
O     192.168.140.0/23 [110/101] via 10.250.0.253, 5d20h, GigabitEthernet3
O     192.168.176.0/24 [110/101] via 10.250.0.254, 00:54:58, GigabitEthernet3

```

