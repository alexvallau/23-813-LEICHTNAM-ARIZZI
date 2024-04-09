## PARTIE 3

### SOMMAIRE
- [Script bash de mesure de débit en SNMP](#Script-bash-de-mesure-de-débit-en-SNMP)
  - [Question 20](#Question-9)
  - [Récupération du compteur d’octets](#Récupération-du-compteur-d-octets)
  - [Gestion de la date et stockage dans un fichier](#Gestion-de-la-date-et-stockage-dans-un-fichier)
  - [Lecture de la dernière ligne du fichier, calcul et enregistrement du débit](#Lecture-de-la-dernière-ligne-du-fichier,-calcul-et-enregistrement-du-débit)
  - [Gestion du fichier vide et gestion du rebouclage du compteur d'octets](#Gestion-du-fichier-vide-et-gestion-du-rebouclage-du-compteur-d'octets)
  - [Question 21](#Question-21)
  - [Question 22](#Question-22)
  - [Question 23](#Question-23)
  - [Script générique](#Script-générique)
  
    
## Script bash de mesure de débit en SNMP

Dans cette partie, nous allons développer un script bash récupérant, via le protocole SNMP, des informations sur les interfaces de nos routeurs. Nous nous limiterons  à la récupération du nombre d'octet en sortie de notre routeur R2. Avec ces données et un peu de bon sens, nous serons capable de déterminer le débit sortant de l'interface de notre routeur. 


### Question 20
_Il aurait été possible de ne pas utiliser le cron ou les timers systemd pour ce script,
par exemple en utilisant la fonction sleep. Pourquoi est-il plus pertinent d’utiliser le cron ou les timers
systemd plutôt que la fonction sleep._

### Réponse 20
L'utilisation de Cron ou des timers systemd pour automatiser les mesures de débit réseau est préférable à la fonction sleep, car cela garantit une exécution périodique et fiable du script sans qu'il doive rester actif en permanence. Ces outils gèrent l'exécution des tâches en arrière-plan de manière efficace, permettant ainsi une meilleure gestion des ressources et assurant la continuité des mesures, même en cas de redémarrage du système ou d'interruptions du script.


### Récupération du compteur d’octets

``` bash
#!/bin/sh

oid="1.3.6.1.2.1.31.1.1.1.6.1"
agent_ip="10.100.4.2"
community="123test123"
value=$(snmpwalk -v2c -c "$community" "$agent_ip" "$oid" | awk '{print $NF}' )
echo "${value}"
```


### Gestion de la date et stockage dans un fichier
``` bash
oid="1.3.6.1.2.1.31.1.1.1.6.1"
agent_ip="10.100.4.2"
community="123test123"
value=$(snmpwalk -v2c -c "$community" "$agent_ip" "$oid" | awk '{print $NF}' )
date=$(date +%s) #Nous renvoie
echo "$date;$value">>filename
```
### Lecture de la dernière ligne du fichier, calcul et enregistrement du débit

Afin de récupérer la dernière du fichier, nous avons utilisé:
``` bash
#récupère la dernière ligne
last_line=$(tail -n 1 "${filename}")
#Split la dernière ligne et récupère le premier élément, soit la date
last_line_date=$(echo "$last_line" | cut -d ";" -f 1)
#Split la dernière ligne et récupère le deuxième élément, soit le deuxième octet
last_line_octet_value=$(echo "$last_line" | cut -d ";" -f 2)
```

### Gestion du fichier vide et gestion du rebouclage du compteur d'octets

Voici ce qui a été mis en place afin de vérifier si le fichier est vide:
```bash
#Si le fichier est vide alors
if [ ! -s ${filename} ]; then
        #On fait une requête SNMP à notre routeur que l'on sauvegarde dans une variable
        first_octet_value=$(snmpwalk -v2c -c "$community" "$agent_ip" "$oid" | awk '{print $NF}' )

        first_date=$(date +%s)
        echo "$first_date;$first_octet_value" >> "${filename}"
        sleep 10
fi
```
### Question 21
_Expliquer le problème posé par le rebouclage du compteur. Expliquer la solution à mettre en place._

### Réponse 21
Le problème avec le rebouclage du compteur est le suivant : nos équipements réseaux ont une limite de stockage pour leurs informations. Par exemple, le compteur de notre routeur ne peut pas compter indéfiniment le nombre d'octets en sortie. Il peut compter jusqu'à 2^64 bits au maximum. Une fois ce nombre maximal atteint, le compteur se réinitialise, repartant à zéro.

Cela pose un problème lors du calcul du débit. Pour calculer le débit, nous devons faire la différence entre le nombre d'octets à deux instants différents, puis diviser par la durée entre ces instants ($\frac{NombreOctet(t+1)-NombreOctet(t)}{\Delta t}$). Si le compteur est réinitialisé entre ces deux instants, la soustraction donnera un nombre négatif, car $NombreOctet(t)$ sera plus grand que $NombreOctet(t+1)$ si $NombreOctet(t+1) = 0$. Cela faussera le calcul du débit.

Pour remédier à ce problème, nous avons mis en place une vérification dans le script. Nous vérifions si la dernière ligne du fichier contient un nombre négatif :
- Si c'est le cas, cela signifie que le compteur a rebouclé.
  - Nous interrogeons à nouveau le routeur via SNMP.
  - Nous attendons 10 secondes.
  - Nous effectuons le calcul du débit par rapport à cette dernière requête SNMP.
- Si la dernière ligne ne contient pas de nombre négatif, nous exécutons le script normalement.


### Question 22

_Exposez votre script_
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
L'adresse IP de la machine est 10.100.4.2.

### Script générique

Afin de rendre notre script complètement générique, nous avons ajouté les lignes suivantes:

```bash
#!/bin/bash

# Vérifier si le nombre d'arguments est correct
if [ "$#" -ne 4 ]; then
    echo "Usage: $0 <OID> <Agent_IP> <Community> <Filename>"
    exit 1
fi

# Récupérer les valeurs des arguments
oid="$1"
agent_ip="$2"
community="$3"
filename="$4"
```

Ainsi, notre crontab ressemblera à:
``` bash
* * * * * /home/etudiant/snmp-2.sh 1.3.6.1.2.1.31.1.1.1.6.3 10.100.4.2 123test123 /home/etudiant/data
``` 


