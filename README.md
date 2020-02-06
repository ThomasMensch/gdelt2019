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
- Requête 1: 1 mois (ie request_1_1mois.csv)
- Requête 2: 1 mois (ie request_2_1mois.csv)
- Requête 31: 1 mois (ie request_31_1mois.csv)
- Requête 32: 1 mois (ie request_32_1mois.csv)
- Requête 33: 1 jour (ie request_33_1jour.csv)
- Requête 4: 1 mois (ie request_4_1mois.csv)

=> Ces CSV ne remontent pas les données d'un mois totalement, car l'exécution de la requête d'extraction mettait en risque la consommation de la mémoire. Donc on s'est limité à l'extraction de 1000 lignes, ie la limite par default de Spark. Chaque CSV contient des données sur tout le mois 12. 

### Suite à la présentation:
Lors de la présentation et la démonstration, nous avons pu montrer la résilience et la persistance de nos données.
Nous avions choisi d'utiliser EC2 couplé à EMR et d'installer Cassandra sur tous les noeuds (3 noeuds) pour pouvoir répliquer l'information autant que nécessaire. Nous avions choisi de faire interagir Zeppelin sur AWS avec Cassandra en installant des interpréteurs / connecteur spark cassandra.
Nous avons rencontré le problème suivant majoritairement qui est celui que nos noeuds masters finissaient par tomber au bout d'un certain temps et que les slaves persistaient mais qu'on perdait donc notre cluster en entier.


### Budget consommé:
Nous avons consommé 100$, puis avons réactivé notre compte educate afin d'investiguer encore l'origine des erreurs que nous avons rencontrées. Il en ressort la résolution de la majorité de nos erreurs, et le chargement d'un mois sur toutes les requêtes, sauf la 3-1 de la requête 3.
Nous avons consommé 120$ en tout.

### Post-projet:
A la suite de la présentation, nous avons poursuivi ce projet, en essayant de résoudre et surtout comprendre l'origine des problèmes auxquels nous avions fait face pendant la démonstration. En voici ce que nous retenons maintenant: 
#### Le lancement d'EMR, avec 3 instances, avec un compte éducate, restreint notablement les ressources matérielles. Lorsque nous avions lancé le cluster, EMR génère un maître et plusieurs Slaves. EMR lance Spark et Zeppelin sur le noeud maître, ce qui consomme notablement la mémoire java. Ce souci impose de ne pas installer Cassandra dans le noeud maître sous peine de:
- Crash Zeppelin en pleine requête Spark ou Cassandra => Broken pipe, puis Connection refused.
- Crash serveur Cassandra au niveau du noeud, avec l'erreur "not enough java memory heap", la mémoire étant consommée principalement par Zeppelin et Spark lancés sur le noeud maître.


