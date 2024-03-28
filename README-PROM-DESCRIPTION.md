# Introduction

Prometheus est un système de surveillance de parc informatique open source. Son rôle principal est de collecter et stocker les informations provenant de différents systèmes hétérogènes. Ces données peuvent inclure le flux de bits à travers l'interface d'un routeur ou les ressources utilisées par un système d'exploitation Linux. Elles sont ensuite stockées dans une base de données et peuvent être facilement exploitées ou représentées par des applications tierces.

## Fonctionnement/interaction

Pour décrire Prometheus, il est plus approprié de parler d'écosystème, car il ne fonctionne pas seul. Pour comprendre au mieux Prometheus, nous détaillerons les éléments essentiels à son fonctionnement.

### Exportateur

Un exportateur agit comme un interprète entre un système source et Prometheus. Il récupère les informations du système source et les traduit dans un format compréhensible par Prometheus. De plus, Prometheus régule la fréquence à laquelle l'exportateur doit récupérer des informations du système source. En résumé, les exportateurs sont des composants logiciels agissant comme des passerelles entre les applications ou les systèmes à surveiller et Prometheus, facilitant ainsi la collecte de métriques au format approprié.

### Base de données

Les données collectées sont stockées dans une base de données locale spécialement conçue pour les séries temporelles. Prometheus utilise sa propre base de données de séries temporelles pour stocker les métriques collectées avec leurs étiquettes correspondantes, facilitant ainsi la recherche et l'agrégation rapides des données. Cette base de données est optimisée pour des opérations de requête rapides et efficaces, avec son propre langage de requête appelé PromQL, distinct de tout autre langage de requête utilisé dans les bases de données de séries temporelles traditionnelles telles que SQL.

### Visualisation des données

Bien que Prometheus propose nativement une visualisation  des données récupérées, elle peut être considérée comme archaïque en termes de design et de fonctionnalités. Ainsi, l'utilisation d'outils tiers comme Grafana est souvent privilégiée. Grafana est relativement simple d'utilisation et nécessite simplement la création d'une connexion entre le serveur Prometheus et Grafana pour exploiter visuellement toutes les données récupérées.

### Comparaison avec les concurrents

Prometheus se démarque de ses concurrents par plusieurs aspects clés. 

#### Nagios

Nagios est souvent préféré pour sa longue histoire et sa large adoption dans les environnements professionnels. Il offre une surveillance des systèmes et des réseaux avec une interface web pour la gestion et la visualisation. Cependant, sa configuration peut être complexe et nécessiter un effort important. Malgré sa grande communauté et son support actif, Nagios peut être perçu comme moins moderne que Prometheus en raison de ses fonctionnalités parfois limitées et de son interface utilisateur moins intuitive.

#### Zabbix

Zabbix est reconnu pour ses fonctionnalités avancées de surveillance de la performance et de la disponibilité. Il propose une architecture extensible et modulaire, avec une gestion avancée des alertes et des notifications. Cependant, la configuration initiale de Zabbix peut être plus complexe que celle de Prometheus, et il peut nécessiter des ressources système plus importantes pour les déploiements à grande échelle. Malgré ses visualisations graphiques puissantes, l'interface utilisateur de Zabbix peut être considérée comme moins conviviale que celle de Prometheus.

#### Conclusion

Comparé à Nagios et Zabbix, Prometheus offre une solution plus moderne et flexible pour la surveillance des systèmes informatiques. Avec son écosystème complet, comprenant des exportateurs, une base de données optimisée et des outils de visualisation, Prometheus simplifie la collecte, le stockage et l'analyse des données de surveillance. Son langage de requête PromQL permet une analyse avancée des données, tandis que sa simplicité d'intégration en fait un choix attractif pour les environnements cloud-native et distribués.


