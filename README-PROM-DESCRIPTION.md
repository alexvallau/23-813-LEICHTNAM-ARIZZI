# Introduction

Prometheus est un système de surveillance de parc informatique open source. Son rôle principal est de collecter et stocker les informations provenant de différents systèmes hétérogènes. Ces données peuvent inclure le flux de bits à travers l'interface d'un routeur ou les ressources utilisées par un système d'exploitation Linux. Elles sont ensuite stockées dans une base de données et peuvent être facilement exploitées ou représentées par des applications tierces.

## Fonctionnement/interaction

Pour décrire Prometheus, il est plus approprié de parler d'écosystème, car il ne fonctionne pas seul. Pour comprendre au mieux Prometheus, nous détaillerons les éléments essentiels à son fonctionnement.

### Exportateur

Un exportateur agit comme un interprète entre un système source et Prometheus. Il récupère les informations du système source et les traduit dans un format compréhensible par Prometheus. De plus, Prometheus régule la fréquence à laquelle l'exportateur doit récupérer des informations du système source. En résumé, les exportateurs sont des composants logiciels agissant comme des passerelles entre les applications ou les systèmes à surveiller et Prometheus, facilitant ainsi la collecte de métriques au format approprié.

### Base de données

Les données collectées sont stockées dans une base de données locale spécialement conçue pour les séries temporelles. Prometheus utilise sa propre base de données de séries temporelles pour stocker les métriques collectées avec leurs étiquettes correspondantes, facilitant ainsi la recherche et l'agrégation rapides des données. Cette base de données est optimisée pour des opérations de requête rapides et efficaces, avec son propre langage de requête appelé PromQL, distinct de tout autre langage de requête utilisé dans les bases de données de séries temporelles traditionnelles telles que SQL.

### Visualisation des données

Bien que Prometheus propose une visualisation native des données récupérées, elle peut être considérée comme archaïque en termes de design et de fonctionnalités. Ainsi, l'utilisation d'outils tiers comme Grafana est souvent privilégiée. Grafana est relativement simple d'utilisation et nécessite simplement la création d'une connexion entre le serveur Prometheus et Grafana pour exploiter visuellement toutes les données récupérées.

### Comparaison avec les concurrents

Prometheus se distingue de ses concurrents par sa modernité, sa simplicité d'intégration et son langage de requête puissant, PromQL, offrant une analyse avancée des données de surveillance. En comparaison, Nagios bénéficie d'une large adoption professionnelle et d'une grande communauté, mais souffre d'une configuration complexe. Zabbix offre des fonctionnalités avancées avec une architecture extensible, mais sa configuration initiale peut être ardue. Par rapport à ces concurrents, Prometheus se démarque par son écosystème complet, comprenant des exportateurs, une base de données optimisée et des outils de visualisation, offrant ainsi une solution complète pour la surveillance et l'analyse des systèmes informatiques.

