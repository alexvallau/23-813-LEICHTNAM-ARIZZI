
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

Dans SNMP, les données sont transmises en utilisant un format d'encodage spécifique appelé ASN.1, associé aux Basic Encoding Rules (BER). ASN.1, ou Abstract Syntax Notation One, est un ensemble de règles pour structurer les données, tandis que les BER déterminent comment ces structures de données sont converties en séquences d'octets pour la transmission. Ce mécanisme standardise la communication entre les différents dispositifs de réseau, permettant ainsi à un système de gestion de réseau de comprendre et de contrôler efficacement les informations reçues des agents SNMP, malgré la diversité des appareils et des fabricants.


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

Les colonnes de la vrrpOperTable peuvent inclure des informations telles que :

vrrpOperState: L'état opérationnel de l'instance VRRP (par exemple, master, backup).  
vrrpOperAdminState: L'état administratif de l'instance VRRP (par exemple, up, down).  
vrrpOperPriority: La priorité de l'instance, utilisée pour déterminer le routeur maître.  
vrrpOperVirtualMacAddr: L'adresse MAC virtuelle utilisée par cette instance VRRP.  
vrrpOperMasterIpAddr: L'adresse IP du routeur maître actuel.  
vrrpOperMasterRouterId: L'identifiant du routeur maître, souvent l'adresse IP du routeur.  
vrrpOperAuthType: Le type d'authentification utilisé (si applicable).  
vrrpOperAdvertisementInterval: L'intervalle à laquelle les annonces VRRP sont envoyées.  

L'index de cette table est souvent constitué de deux parties : l'identifiant de l'interface sur laquelle l'instance VRRP est configurée et l'ID de l'instance VRRP elle-même. Cela permet d'identifier de manière unique chaque entrée dans la table dans le cas où plusieurs instances VRRP opèrent sur le même appareil.

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
Le protocole utilisé est UDP, on utilise UDP pour les tests de débit car ce protocole offre une transmission rapide des données sans se préoccuper de l'établissement de connexion, de la vérification des erreurs, ou de l'ordre de livraison des paquets. Cela le rend idéal pour mesurer la performance maximale du réseau et pour évaluer le comportement des applications temps réel qui peuvent tolérer une certaine perte de données mais exigent une faible latence.

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
