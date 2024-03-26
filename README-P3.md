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

### Question 23
_Décrire la configuration mise en place. Donner dans votre compte-rendu l’adresse IP
de la machine sur laquelle le script s’exécute._

### Réponse 23
``` bash
* * * * * /home/etudiant/snmp-2.sh
```
