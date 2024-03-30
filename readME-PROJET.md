# Brouillon
### Quelques idées
* Schéma sur le fonctionnement général de notre infra
* Capture d'écran du dashbooard
* Expliquer les intéractions entre les diff éléments:
  * Contenarisation des différentes applications
  * Expliquer le fichier prométheus.yml
  * Expliquer les différentes applications présentes
     * Blackbox
     * cadvisor
     * SNMP-exporter
     * node-exporter 
       * Montrer les différentes config sur la machine hôte




# Intération entre les différents éléments

Ce que nous vous proposons ici est le fruit de plusieurs heures de travail et de recherche. Bien que les différents éléments présentés vont avoir une allure assez simple dans leur fonctionnement, il n'en reste pas moins qu'en partant de 0 il faut: Savoir ce que l'on veut, trouver ce que l'on veut et corriger les bugs.

Parlons de ce que nous voulions. EN  bon français, nous voulions déployer un outil de surveillance réseau. Cet outil devait être capable d'analyser plusieurs types de machines ainsi que plusieurs types de protocoles. Prométhéus, l'outil donc, sera le coeur de notre surveillance réseau en jouant plusieurs rôle. Il sera chargé, selon sa configuration, de faire des requêtes à ses différents exporteurs. A leurs tours, les exporteurs iront faire des requêtes aux différents noeuds(machine, routeur, switch etc..) et renverront les résultats à prométhéus. Prométhéus aura également un rôle de stockeur de données dans sa base PROMQL. Les possibilités visuelles de prométhéus restant assez maigres, il nous a été demandé d'utiliser grafana afin d'observer les données.


## Conteneurisation

Le lecteur est grand expert docker, nous ne détaillerons donc pas notre docker-compose. A la place, je ferai simplement le listing ce que qui a été déployé et où:

Sur l'hôte de proméhétus(192.168.170.81):

| Nom de conteneurs | Mappage des ports | Utilité                             |
|-------------------|-------------------|-------------------------------------|
| prometheus        | 9090:9090         | Système de surveillance             |
| grafana           | 3000:3000         | Tableau de bord pour les données   |
| snmp-exporter     | 9116:9116, 161:161/udp | Exportateur SNMP                |
| blackbox_exporter| 9115:9115         | Surveille des pages WEB                |



Sur une machine linux monitorée(10.100.4.4):
| Nom de conteneurs | Mappage des ports | Utilité                             |
|-------------------|-------------------|-------------------------------------|
| node-exporter     | 9100:9100         | Exportateur de métriques système   |