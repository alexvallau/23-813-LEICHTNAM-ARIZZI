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
Pour l'instant, nous avons configuré les machines R1 Et client A. Afin d'être au plus clair sur les tests, nous presenterons les tests à effectuer par machine et par interface.
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

``` python3
snmpget -v2c -c 123test123 10.100.4.5 ifMtu.2
1.3.6.1.2.1.2.2.1.4.2
```


### Question 12 
_Copier-coller la ligne du fichier de la MIB VRRP qui indique l’OID relatif de la branche
VRRP par rapport à mib-2._

### Réponse 12

``` python
 vrrpMIB  MODULE-IDENTITY
     LAST-UPDATED 200003030000Z
     ORGANIZATION IETF VRRP Working Group
     CONTACT-INFO
            Brian R. Jewell
     Postal: Copper Mountain Networks, Inc.
             2470 Embarcadero Way
             Palo Alto, California 94303
     Tel:    +1 650 687 3367
     E-Mail: bjewell@coppermountain.com
     DESCRIPTION
         This MIB describes objects used for managing Virtual Router
          Redundancy Protocol (VRRP) routers.
     REVISION 200003030000Z    -- 03 Mar 2000
     DESCRIPTION "Initial version as published in RFC 2787.
     ::= { mib-2 68 }
```

### Question 13
_Pourquoi la première commande échoue alors que la deuxième réussie ?_

### Réponse 13
La première commande échoue car l'OID spécifié, "vrrpMIB", n'est pas reconnu par le périphérique SNMP ciblé. En revanche, la deuxième commande réussit car elle utilise un OID standard, "mib-2.68", qui fait partie du MIB-II et est pris en charge par la plupart des périphériques SNMP. Cela suggère que le périphérique ne supporte pas ou n'a pas activé la partie du MIB associée à "vrrpMIB".


### Question 14
_On s’intéresse à la table vrrpOperTable. Donner l’OID par rapport à mib-2 de cette
table. Relever dans la vrrpOperTable de R1 et expliquer les 8 premières colonnes et comment est constitué l’index._

### Réponse 14

L'OID par rapport à la mib-2 de la table vrrpOperTable est .1.3.6.1.2.1.68.1.3

 En tapant la commande

 Nous obtenons les 8 colonnes suivantes:
 ```python3
 SNMPv2-SMI::mib-2.68.1.3.1.2.2.1 = Hex-STRING: 00 00 5E 00 01 01 #Adresse MAC de l'interface virtuelle
SNMPv2-SMI::mib-2.68.1.3.1.3.2.1 = INTEGER: 3 #vrrpOperState    
SNMPv2-SMI::mib-2.68.1.3.1.4.2.1 = INTEGER: 1 # vrrpOperAdminState        
SNMPv2-SMI::mib-2.68.1.3.1.5.2.1 = INTEGER: 150 # vrrpOperPriority    
SNMPv2-SMI::mib-2.68.1.3.1.6.2.1 = INTEGER: 1 # vrrpOperIpAddrCount: Nombre adresse ip associé à notre interface virtuelle, 0 et 1, soit 2 Ip   
SNMPv2-SMI::mib-2.68.1.3.1.7.2.1 = IpAddress: 10.100.4.1 #IP Master
SNMPv2-SMI::mib-2.68.1.3.1.8.2.1 = IpAddress: 10.100.4.5 # Ip de l'interface virtuelle
SNMPv2-SMI::mib-2.68.1.3.1.9.2.1 = INTEGER: 1
```
# Faire l'index

### Execution de iperf

Nous obtenons les résultats suivants
``` python3
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec  1.09 GBytes   941 Mbits/sec    0             sender
[  5]   0.00-10.03  sec  1.09 GBytes   937 Mbits/sec                  receiver
```

Ce qui représente comme débit de montée et de déscente à peu près 117 o/s

### Question 15
_Sur quel pare-feu avez-vous ajouté une exception ?_

### Réponse 15
Le parefeu sur lequel nous ajouté une exception est le parefeu de la machine B. Nous avons dû ouvrir le port 5201, qui est le port natif d'utilisation de iperf.

### Question 16
_Quel est le protocole de transport utilisé pour le test de débit entre les deux machines ?_

### Réponse 16
Le protocole utilisé est UDP

### Question 17
_Pourquoi selon vous le débit calculé par capinfos est légèrement plus élevé que le débit généré par iperf ?_

### Réponse 17
Dans un premier temps, nous avons limité notre débit avec i perf à  500 Kbit/s
```python3
[ ID] Interval           Transfer     Bitrate         Jitter    Lost/Total Datagrams
[  5]   0.00-10.00  sec   611 KBytes   500 Kbits/sec  0.015 ms  0/432 (0%)  receiver ```
 et dans la capture effectuée par wireshark, nous voyons entre autres:

``` python3

[root@localhost etudiant]# capinfos /tmp/capture-500k.pcap
File name:           /tmp/capture-500k.pcap
File type:           Wireshark/... - pcapng
File encapsulation:  Ethernet
File timestamp precision:  nanoseconds (9)
Packet size limit:   file hdr: (not set)
Number of packets:   413
File size:           629kB
Data size:           615kB
Capture duration:    9,545075832 seconds
First packet time:   2024-03-11 14:55:25,971538993
Last packet time:    2024-03-11 14:55:35,516614825
Data byte rate:      64kBps
Data bit rate:       515kbps
```

Iperf mesure le débit au niveau de la couche de transport (TCP ou UDP), tandis que capinfos mesure le débit au niveau des trames capturées, ce qui inclut les en-têtes de protocole, les encapsulations et éventuellement d'autres surcharges de protocole. Par conséquent, si les trames capturées incluent des en-têtes supplémentaires (par exemple, des en-têtes VLAN), cela peut augmenter légèrement le débit calculé par capinfos par rapport au débit mesuré par iperf.

### Question 18
 _Les compteurs d’octets sont disponibles en version 32 bits (ifInOctets, ifOutOctets) ou en version 64 bits (ifHCInOctets, ifHCOutOctets). Justifier précisément quels OID il faut utiliser._

### Réponse 18
Nous utiliserons l'OID de l'interface out en 64 bits

### Question 19
_Décrire une manipulation « simple » permettant de trouver le débit entrant ou sortant
du réseau en utilisant les compteurs d’octets snmp. Comparer les débits obtenus par SNMP avec les débits générés._

### Réponse 19
Une méthode simple consiste à prendre le nombre d'octet(appelons ce nombre octet0) en entrée ou en sortie(en fonction de nos besoins) à l'instant t_{0}.
Nous attendons l'instant t+1 et nous reprennons le nombre d'octet(appelons ce nombre octet1) ayant normalement évolués.
Nous faisons ensuite le calcul suivant: (octet1-octet0)/(t+1)-t_{0}


### Script bash de mesure de débit en SNMP

## Récupération du compteur d’octets

``` bash
#!/bin/sh

oid="1.3.6.1.2.1.31.1.1.1.6.1"
agent_ip="10.100.4.2"
community="123test123"
value=$(snmpwalk -v2c -c "$community" "$agent_ip" "$oid" | awk '{print $NF}' )
echo "${value}"
```



### Question 20
_Il aurait été possible de ne pas utiliser le cron ou les timers systemd pour ce script,
par exemple en utilisant la fonction sleep. Pourquoi est-il plus pertinent d’utiliser le cron ou les timers
systemd plutôt que la fonction sleep._

### Réponse 20
Le cron appelera ponctuellement le script pour qu'il s'éxécute. SI nous utilisons seulement une fonction sleep, cela signifique que le script est lancé constamment, ce qui utilise des ressources inutilement.


### Question 21


### Réponse 21
Voici un exemple de notre fichier avec les colonnes qui sont respectivement; 
Nombre de seconde depuis 1970; Nombre d'octets en interface de sortie; Débit
1710246662;4522703987;2032
1710246721;4522830471;2143
1710246782;4522956964;2073
1710246841;4523072989;1966

Nous savons que le Nombre d'octets en interface de sortie est codé sur 64 bits. Nous avons donc 1.8 * 10^19 octets au maximum. Une fois que ce nombre est atteint, le compteur repart à 0.
Nous savons également que pour calculer le début, nous devons faire(comme expliqué ci-dessus) NombreOctet(t+1)-NombreOctet(t). Si le compteur est réinitialisé, cette soustraction sera donc un nombre négatif et le calcul du débit sera faussé. 
Afin de pallier à ce problème, nous avons décidé de vérifier si la dernière ligne du script contient un nombre négatif
* Si la dernère ligne est un nombre négatif
  * On refait une demande snmp au routeur
  * On attend 10 secondes
  * On fait notre calcul de débit par rapport à cette dernière requête SNMP
* Si la dernière ligne ne contient pas de nombre négatif, on éxecute le script normalement.

### Question 22
Exposez votre script

### Réponse 22

``` bash

[root@localhost etudiant]# cat snmp-2.sh
#!/bin/sh

# C'est l oid correspondant au trafique entrant sur l interface 3(10.250.0.7)
oid="1.3.6.1.2.1.31.1.1.1.6.3"
agent_ip="10.100.4.2"
community="123test123"
filename="/home/etudiant/data"

#Si le fichier est vide
if [ ! -s ${filename} ]; then
        first_octet_value=$(snmpwalk -v2c -c "$community" "$agent_ip" "$oid" | awk '{print $NF}' )
        first_date=$(date +%s)
        echo "$first_date;$first_octet_value" >> "${filename}"
        sleep 10
fi
#Récupération de la dernière ligne et des valeurs
last_line=$(tail -n 1 "${filename}")
last_line_date=$(echo "$last_line" | cut -d ";" -f 1)
last_line_octet_value=$(echo "$last_line" | cut -d ";" -f 2)

#Récupération des toutes dernières données
#octet_value=$(snmpget -v2c -Oq -c ${community} ${agent_ip} ${oid} | cut -d" -f 2)
octet_value=$(snmpwalk -v2c -c "$community" "$agent_ip" "$oid" | awk '{print $NF}' )
date=$(date +%s)


#Calcul de l'octet
delta_t=$(($date - $last_line_date))
delta_octet=$(($octet_value - $last_line_octet_value))
#Ici, on test si deltat octet est négative,ce qui signfierai une boucle
if [ $delta_octet -lt 0 ]; then
    sleep 10
    first_octet_value=$(snmpwalk -v2c -c "$community" "$agent_ip" "$oid" | awk '{print $NF}' )
    first_date=$(date +%s)
    echo "$first_date;$first_octet_value" >> "${filename}"
    last_line=$(tail -n 1 "${filename}")
    last_line_date=$(echo "$last_line" | cut -d ";" -f 1)
    last_line_octet_value=$(echo "$last_line" | cut -d ";" -f 2)
    delta_octet=$(($octet_value - $last_line_octet_value))
fi
octet_seconde=$(($delta_octet / $delta_t))
echo "$date;$octet_value;$octet_seconde" >> "${filename}"
``` 

### Projet Prometheus / Grafana / Netflow

## Synthèse Prometheus

Fonctionnement de Prometheus :

Prometheus est un système de surveillance moderne, spécialisé dans la collecte et le stockage de métriques provenant d'environnements informatiques. 
Son architecture repose sur plusieurs éléments essentiels travaillant de concert pour offrir une surveillance performante et fiable.
Prometheus utilise une collecte basée sur le scraping, permettant de récupérer directement les métriques à partir des applications et des services surveillés. 
Il interroge périodiquement les endpoints HTTP exposés par ces entités pour extraire les métriques au format Prometheus.

Une fois collectées, les métriques sont stockées localement dans une base de données en séries temporelles. 
Cette base de données est conçue pour permettre un accès rapide aux données historiques, assurant ainsi une conservation efficace des métriques sur une période définie, tout en optimisant l'utilisation de l'espace de stockage disponible.
Prometheus propose également des fonctionnalités avancées de traitement et d'analyse des métriques grâce à son langage de requête PromQL. 
Cela donne aux utilisateurs la possibilité d'exécuter des requêtes sophistiquées pour analyser les données, mettre en place des alertes et créer des tableaux de bord personnalisés pour superviser les performances de leur infrastructure.
Enfin, l'intégration de Prometheus avec des outils de visualisation tels que Grafana permet de créer des tableaux de bord interactifs et des graphiques basés sur les métriques collectées. 
Cette fonctionnalité offre aux utilisateurs une visualisation claire et détaillée de leurs données, facilitant ainsi l'obtention d'informations précieuses sur les performances du système.

Comparaison avec d'autres Solutions de Supervision :

Comparé à des solutions de supervision telles que Nagios ou Zabbix Prometheus présente plusieurs avantages significatifs qui en font un choix privilégié pour de nombreuses entreprises.
D'abord, sa méthode de collecte moderne basée sur le scraping le rend particulièrement adapté aux environnements cloud et conteneurisés. 
Contrairement aux solutions traditionnelles, Prometheus est conçu pour fonctionner efficacement dans des environnements dynamiques où les charges de travail sont souvent transitoires.
De plus, le modèle de stockage à double échantillonnage de Prometheus offre un équilibre optimal entre précision et efficacité. 
Cela permet de conserver les données historiques de manière économique, ce qui est crucial dans les environnements de grande envergure où la gestion du stockage des métriques peut rapidement devenir complexe.
Prometheus se distingue également par sa grande extensibilité, avec une bibliothèque étendue de plug-ins et d'intégrations disponibles. 
Son langage de requête PromQL permet une analyse avancée et personnalisée des données, offrant ainsi une flexibilité significative pour surveiller et analyser les systèmes selon les besoins spécifiques de chaque organisation.
Enfin, l'intégration étroite de Prometheus avec Kubernetes en fait une solution populaire pour la surveillance des applications et des services déployés dans des clusters Kubernetes. 
Cette compatibilité native en fait un choix stratégique pour les organisations qui adoptent Kubernetes et recherchent une solution de supervision moderne et efficace.







