# gdelt2019
Projet GDELT

### Présentation du projet
Le projet GDELT surveille les informations diffusées, imprimées et données Web du monde entier. Ceci porte sur presque tous les coins de chaque pays, dans plus de 100 langues en identifiant les personnes, les lieux, les organisations, les thèmes, les sources, les émotions, les comptes, les citations, les images et les événements qui animent notre société, créant ainsi une plate-forme ouverte gratuite pour l'informatique dans le monde entier.

A partir d’une date (ou d’un mois), afficher sur une carte des pays dans lesquels des évènements ont eu lieu.
Possibilité de visualiser pour chaque pays le nombre de mentions ainsi que le caractère + ou - de chacune d’elles.
Choix d’un événement particulier:
- Pays impactés
- Acteurs
- Liste des articles

Afficher :
- Le nombre d’articles / événements pour chaque (jour, pays de l’événement, langue de l’article).
- Pour un acteur (pays/organisation...), afficher les évènements qui y font reference.
- Les sujets (acteurs) qui ont eu le plus d’articles positifs/négatifs (mois,pays, langue de l’article).
- Acteurs/pays/organisations qui divisent le plus.

### Objectifs:
- Concevoir un système permettant d’analyser l’évolution des relations entre les différents pays
- Proposer un système de stockage distribué, résilient et performant sur AWS pour les données de GDELT et faire persister les données même si un noeud venait à tomber.

### Contraintes
- Utiliser au moins 1 technologie du  cours (SQL / Cassandra / Spark)
- Concevoir un système distribué et tolérant aux pannes
- Charger une année de données dans votre cluster
- Déployer le cluster sur AWS (compte Amazon Educate) / Budget < 300 euros

### Choix technologique:
- Machine locale -- Comprendre les données / requêtes
- AWS (Amazon Educate) -- Configuration de l’infrastructure / Mise en production
- Cassandra: Stockage de  gros volumes de données possible /passage à l’échelle/ Robustesse / résilience, RF à 3
- Spark: Moteur de traitement de données très performant (nécessite bcp de mémoire) / flexibilité dans la manipulation des données avec des dataframes.

### Volumes chargés:
Comme mentionné, nous avons pu résoudre notre problème de compilation sur Zeppelin, en évitant d'installer Cassandra dans le noeud Master EMR, qui contient
Nous avons pu charger les volumes suivants:
- Requête 1: 1 mois
- Requête 2: 1 mois
- Requête 31: 1 mois
- Requête 32: 1 mois
- Requête 33: 1 jour
- Requête 4: 1 mois

### Suite à la présentation:
Lors de la présentation et la démonstration, nous avons pu montrer la résilience et la persistance de nos données.
Nous avions choisi d'utiliser EC2 couplé à EMR et d'installer Cassandra sur tous les noeuds (3 noeuds) pour pouvoir répliquer l'information autant que nécessaire. Nous avions choisi de faire interagir Zeppelin sur AWS avec Cassandra en installant des interpréteurs / connecteur spark cassandra.
Nous avons rencontré le problème suivant majoritairement qui est celui que nos noeuds master finissaient par tomber au bout d'un certain temps et que les slaves persistaient mais qu'on perdait donc notre cluster en entier.

### Post-projet:
A la suite de la présentation, nous avons poursuivit ce projet, en essayant de résoudre et surtout comprendre l'origine des problèmes auxquels nous avions fait face pendant la démonstration. En voici ce que nous retenons maintenant: 
#### Le lancement d'EMR, avec 3 instances, avec un compte éducate, restreint notablement les ressources matérielles. Lorsque nous avions lancé le cluster, EMR génère un maître et plusieurs Slave. EMR lance Spark et Zeppelin sur le noeud maître, ce qui consomme notablement la mémoire java. Ce souci impose de ne pas installer Cassandra dans le noeud maître sous peine de:
- Crash Zeppelin en plein requête Spark ou Cassandra => Broken pipe, puis Connection refused.
- Crash serveur Cassandra au niveau du noeud, avec l'erreur "not enough java memory heap", la mémoire étant consommée principalement par Zeppelin et Spark lancés sur le noeud maître.


