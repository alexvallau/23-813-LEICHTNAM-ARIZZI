# Introduction

Prometheus est un système de surveillance de parc informatique open source. Son rôle est de collecter et stocker les informations provenant de différents systèmes hétérogènes. Ces données peuvent par exemple être le flux de bits à travers l'interface d'un routeur ou les ressources utilisées par un système d'exploitation Linux. Elles sont ensuite stockées dans une base de données et peuvent être facilement exploitées ou représentées par des applications tierces.

## Fonctionnement/interaction

Pour décrire Prometheus, il est plus approprié de parler d'écosystème, car il ne fonctionne pas seul. Pour le comprendre au mieux, nous détaillerons les éléments essentiels à son fonctionnement.

### Exportateur

Si nous devions comparer un exportateur à un métier, nous prendrions l'exemple de l'interprète. 
Un interprète est capable de comprendre des informations dans une langue A, afin de  les retranscrire dans une langue B. 
C'est exactement le rôle que jouera un exportateur entre un système donné, donc un système à qui l'on veut recueillir des informations, et prométhéus. L'exportateur demandera des informations au système(routeur, ordinateure) et les traduira dans un format que prométhéus est capable de comprendre et de stocker. De plus, prométhéus régira la candence à laquelle notre exportateur doit demander des informations au système. Concrètement: Les exportateurs sont des composants logiciels qui agissent comme des passerelles entre les applications ou les systèmes à surveiller et Prometheus. Ils permettent aux cibles de fournir des métriques au format compréhensible par Prometheus.

### Base de données

Les données collectées sont stockées dans une base de données locale spécialement conçue pour les séries temporelles. Prometheus utilise sa propre base de données de séries temporelles pour stocker les métriques collectées avec leurs étiquettes correspondantes, facilitant ainsi la recherche et l'agrégation rapides des données. Cette base de données est optimisée pour des opérations de requête rapides et efficaces, avec son propre langage de requête appelé PromQL, distinct de tout autre langage de requête utilisé dans les bases de données de séries temporelles traditionnelles telles que SQL.

### Visualisation des données

Bien que Prometheus propose nativement une visualisation  des données récupérées, elle peut être considérée comme archaïque en termes de design et de fonctionnalités. Ainsi, l'utilisation d'outils tiers comme Grafana est souvent privilégiée. Grafana est relativement simple d'utilisation et nécessite simplement la création d'une connexion entre le serveur Prometheus et Grafana pour exploiter visuellement toutes les données récupérées.

### Schéma issu de la documentatioon
![Texte alternatif](./images/architecture.png "Titre de l'image")


### Comparaison avec les concurrents

Les principaux concurents de Prométhéus sont Zabbix et Nagios. Nous commencerons par comparer Nagios et Prométhéus, ce qui nous aidera à consolider nos bases sur prométhéus. Nous parlerons ensuite de Zabbix seul.

#### Comparaison entre Prometheus et Nagios

#### Capacités
Prometheus et Nagios offrent différentes fonctionnalités. Nagios se concentre principalement sur le trafic réseau et la sécurité des applications, tandis que Prometheus s'intéresse davantage aux applications de son infrastructure

#### Collecte de données
- Prometheus collecte des données à partir des exportateurs expliqués en début de ce document.
- Nagios utilise des agents installés sur les différents noeuds du réseau surveillés.

#### Visualisations
- Les visualisations fournies par Prometheus ne répondent pas totalement aux besoins actuels, nécessitant souvent l'utilisation d'outils supplémentaires comme Grafana.
- Nagios propose des tableaux de bord adaptés à la surveillance des réseaux et des infrastructures, mais manque de graphiques pour les problèmes liés aux applications.

#### Alertes
Prométhéus est doté d'un système d'alertes. Ce système, comme sur grafana, fonctionnera avec des conditions à établir. Si ces conditions sont atteintes, l'alerte se déclenchera. Il est également possible de notifier l'alerte par mail.

#### Configuration et Maintenance
- La configuration initiale de Prometheus est plus simple grâce à des images Docker, tandis que Nagios nécessite une configuration plus complexe.
- Les intégrations de Prometheus sont vastes grâce à sa capacité à écrire de nouveaux exportateurs, tandis que Nagios a une liste d'intégrations officielles limitée.

#### Alertes
- Prometheus propose Alertmanager pour définir des seuils et déclencher des alertes en cas de dépassement.
- Nagios utilise divers canaux tels que l'e-mail et les SMS pour les alertes.

#### Communauté
- La communauté de Prometheus est plus active et en constante évolution, tandis que Nagios connaît un développement plus lent.

#### Zabbix

#### Capacités

Zabbix est un outil de surveillance complet pour les serveurs, les applications ains que les bases de données. Il offre des fonctionnalités de surveillance des performances, du dépannage, de la sécurité et de la détection des menaces. Zabbix est adapté à la surveillance d'une infrastructure disposant jusqu'à 1000 end-point

#### Collecte de données

Zabbix peut collecter des données à partir de diverses sources, y compris SNMP, JMX, IPMI, et d'autres protocoles. Comme nagios, il dispose d'un agent installable sur différents systèmes.

#### Visualisations

En termes de visualisation, Zabbix offre des tableaux de bord et des graphiques mais nécéssite également grafana

#### Configuration et maintenance

Zabbix nécessite une configuration initiale plus complexe par rapport à certains autres outils de surveillance. La maintenance de Zabbix peut être relativement simple une fois qu'il est correctement configuré, avec des mises à jour régulières et la gestion des alertes.

#### Communauté

Zabbix bénéficie d'une communauté active d'utilisateurs et de contributeurs, bien que peut-être moins importante que celle de certaines autres solutions de surveillance. Il existe des forums, des groupes de discussion et d'autres ressources en ligne pour obtenir de l'aide et des conseils sur Zabbix.

