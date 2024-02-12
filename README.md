### 23-813-LEICHTNAM-ARIZZI
Projet supervision

## Etude théorique préparatoire

#### Question 1 
_Combien de lignes doivent être, théoriquement, présentes dans votre table de routage
lorsque la topologie sera complète et lorsque les protocoles auront convergé ? Donner pour chacun
de vos routeurs, une table de routage partielle qui contiendra 7 routes choisies de manière pertinente._

#### Réponse 1
Il existe 16 réseaux internes (vue dans le tableau) + celui du prof + le réseau 0.0.0.0 +le réseau externe qui interconnecte tous les routeurs + le réseau vlan 140 + le réseau vlan 176. Donc 16+1+1+1+1+1. Donc 21 sous réseaux.

#### Tables de routage partielles
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

Ces routes nous semblaient importantes car elles ne font pas partie des réseaux internes étudiants. Nous avons représenté ces réseaux 

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

Nos routeurs vont donc faire partie de cette nouvelle interface virtuelle. Ce qui veut dire que les interfaces de nos routeurs, vont être connectées à cette nouvelle interface virtuelle.\\

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

Dans notre cas, l'intérêt d'OSPF est que si le routeur 1 ne fonctionne plus et qu'on envoie un paquet depuis internet vers une de nos deux machines A ou B en passant par le routeur1 grace à OSPF le paquet envoyé pourra passer par le routeur 2 ce qui n'est pas le si on utilise un routage statique.

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
    * Ping de  10.100.4.1(R1) vers 10.100.4.2(R2)
    * Ping de 10.100.4.1(R1) vers 10.100.4.3(Client A)
  * Interface GigabitEthernet3(Interface réseau côté prof): 10.250.0.7
    * Ping de 10.250.0.7(R1) vers 10.250.0.253(Réussi)
    * Ping de 10.250.0.8(R1) vers 10.250.0.254(Réussi)
   
  #### Question 6
  _Rédiger les tests que vous mettrez en œuvre à la fin de cette étape pour valider la fonctionnement du réseau._

  #### Réponse 6
  Après avoir configuré OSPF sur notre routeur R1 avec la configuration suivante:
  ``` python
  router ospf 1
   router-id 10.10.4.1
   network 10.100.4.0 0.0.0.255 area 0
   network 10.250.0.0 0.0.0.255 area 0 ```

Nous avons regardé la table de routage reçue par OSPF
_Route OSPF reçue par R1:_
``` python
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



#### Question 7
_Relever les commandes saisies pour la configuration de VRRP._

#### Réponse 7
``` python
* Routeur R1
R1(config-if)#vrrp 1 ip 10.100.4.5
R1(config-if)#vrrp 1 prio
R1(config-if)#vrrp 1 priority 150

* Routeur R2
R2(config-if)#vrrp 1 ip 10.100.4.5
R2(config-if)#vrrp 1 prio
R2(config-if)#vrrp 1 priority 100
```

##### Question 8
Test de basculement :

Pour tester le basculement des routeurs, nous avons désactivé sur R1 (routeur maître) l'interface connectée au réseau interne.

Routeur1 (maître) :
``` python
R1# Show running-conf
interface GigabitEthernet2
 ip address 10.100.4.1 255.255.255.0
 shutdown
``` 

Une fois l'interface déconnecteée, le routeur2 qui est esclave va prendre le relais en tant que routeur maitre.
``` python
R2#show vrrp
GigabitEthernet2 - Group 1
  State is Backup
  Virtual IP address is 10.100.4.5
  Virtual MAC address is 0000.5e00.0101
  Advertisement interval is 1.000 sec
  Preemption enabled
  Priority is 100
  Master Router is 10.100.4.1, priority is 150
  Master Advertisement interval is 1.000 sec
  Master Down interval is 3.609 sec (expires in 3.605 sec)
```
En faisant un show vrrp, on voit que le routeur est devenu le routeur maitre.
En faisant un ping depuis Client B vers l'adresse virtuelle 10.100.4.5, et qu'on coupe l'interface du routeur maitre(R1) on voit que le routeur esclave prend le relais car le ping ne se coupe pas.
```python
PING 10.100.4.5 (10.100.4.5) 56(84) octets de données.
64 octets de 10.100.4.5 : icmp_seq=1 ttl=255 temps=0.703 ms
64 octets de 10.100.4.5 : icmp_seq=2 ttl=255 temps=0.616 ms
64 octets de 10.100.4.5 : icmp_seq=3 ttl=255 temps=0.726 ms
64 octets de 10.100.4.5 : icmp_seq=4 ttl=255 temps=0.600 ms
```


##  Supervision et métrologie avec SNMP

#### Question 9
_Relever les commandes saisies dans un des routeurs pour la mise en place de
SNMPv3. Relever la commande snmpget saisie pour récupérer l’objet syslocation._

#### Réponse 9

``` python
snmp-server group snmpv3group v3 priv
snmp-server user snmpuser snmpv3group v3 auth sha auth_pass priv aes 128 crypt_pass
snmp-server location "pm19"
snmp-server contact "leichtnam"
```
En testant du client B vers le routeur 2:

``` python
[etudiant@localhost ~]$ snmpget -v 3 -l authPriv -u snmpuser -a SHA -A auth_pass -x AES -X crypt_pass 10.100.4.2 sysLocation.0
SNMPv2-MIB::sysLocation.0 = STRING: "pm19"
[etudiant@localhost ~]$ snmpget -v 3 -l authPriv -u snmpuser -a SHA -A auth_pass -x AES -X crypt_pass 10.100.4.2 sysContact.0
SNMPv2-MIB::sysContact.0 = STRING: "leichtnam"
```

### Question 10
_Rappeler l’encodage utilisé lors de l’émission de données en SNMP._

### Réponse 10
 L'encodage utilisé est BER. Cet encodage intervient lors de la mise en trame des données SNMP.


### Question 11
_Capturer (inclure la transcription au format octet) la trame et sa réponse lorsque vous
faites un SNMP-GET sur le MTU de la 2ème interface d’un de vos routeurs. Vérifier de manière
exhaustive que cette trame correspond bien à la théorie vue en cours._

### Réponse 11
Sur le client A(10.100.4.3) nous avons éxécuté la commande (en sachant que l'ip de notre interface virtuelle VRRP est 10.100.4.5)
``` python
snmpget -v2c -c 123test123 10.100.4.5 ifMtu.2
```
Cette commande aurait également pu être:
``` python
snmpget -v2c -c 123test123 10.100.4.5 1.3.6.1.2.1.2.2.1.4.2
``` 

Nous avons ensuite initié une capture de trame du côté de notre client avec les résultats suivants:

``` python
[root@localhost ~]# tshark -i enp0s8  -f "udp"
    1 0.000000000   10.100.4.3 → 10.100.4.5   SNMP 91 get-request 1.3.6.1.2.1.2.2.1.4.2
    2 0.000979607   10.100.4.5 → 10.100.4.3   SNMP 93 get-response 1.3.6.1.2.1.2.2.1.4.2
```
Comme vu dans le cours, la commande get permet de récupérer la valeur de l'oid indiqué dans la commande. 
Il est donc donc normal de recevoir la même valeur de l'oid, avec la valeur en plus.


snmpget -v2c -c 123test123 10.100.4.5 ifMtu.2
1.3.6.1.2.1.2.2.1.4.2



### Question 12 
_Copier-coller la ligne du fichier de la MIB VRRP qui indique l’OID relatif de la branche
VRRP par rapport à mib-2._

### Réponse 12

``` python
 vrrpMIB  MODULE-IDENTITY
     LAST-UPDATED "200003030000Z"
     ORGANIZATION "IETF VRRP Working Group"
     CONTACT-INFO
            "Brian R. Jewell
     Postal: Copper Mountain Networks, Inc.
             2470 Embarcadero Way
             Palo Alto, California 94303
     Tel:    +1 650 687 3367
     E-Mail: bjewell@coppermountain.com"

     DESCRIPTION
         This MIB describes objects used for managing Virtual Router
          Redundancy Protocol (VRRP) routers."
     REVISION "200003030000Z"    -- 03 Mar 2000
     DESCRIPTION "Initial version as published in RFC 2787.
     ::= { mib-2 68 }```



 





