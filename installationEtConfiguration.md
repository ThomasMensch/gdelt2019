# Installation et configuration

Dans ce document, nous décrivons les différentes étapes d'installation et de configuration que nous avons réalisé pour ce projet. 

## Etapes d'installation et de configuration en local

Nous avons opté pour une installation de Zeppelin et de Cassandra en local pour commencer

### Zeppelin en local:

- download package : http://zeppelin.apache.org/download.html
- `tar zxvf …`
- `cd zeppelin …`
- `bin/zeppelin-daemon.sh start`
- `sudo service zeppelin start`
- `sudo service zeppelin stop`
- `sudo service zeppelin restart`

### Installation de CCM

CCM: https://academy.datastax.com/planet-cassandra/getting-started-with-ccm-cassandra-cluster-manager

CCM est plus approprié pour une installation en local, sans cluster déployé sur plusieurs machines.

### Cassandra/Zeppelin en local

Pour tester nos requêtes, nous avons installé Cassandra et Zeppelin en local, ce qui nous a permis de concevoir les requêtes demandées par l'énoncé du projet. (voir dossier AWSnotebooks)

- `sudo apt update`
- `sudo apt install cassandra`

## Déploiement du cluster sur AWS - Etapes d'installation et de configuration sur EC2

*Sources utiles:*

 - https://www.linode.com/docs/databases/cassandra/set-up-a-cassandra-node-cluster-on-ubuntu-and-centos/
 - https://www.vultr.com/docs/how-to-install-apache-cassandra-3-11-x-on-centos-7

Nous avons installé Cassandra manuellement sur chaque noeud EC2 généré par EMR. Nous rappelons à ce stade, que EMR génère 1 noeud Master et plusieurs noeuds Slave. Les services lancés par EMR (Spark, Zeppelin, ...) sont tous lancés au niveau du noeud Master, ce qui impose de ne rien installer d'autre sur ce noeud, sous peine d'avoir des soucis de mémoire (la mémoire octroyée par le compte educate étant très limitée). C'est ce qui nous a valu l'erreur "Broken pipe" au niveau de Zeppelin, car mémoire des machine EC2 pas assez puissante pour prendre en charge, et Cassandra et Zeppelin ainsi que Spark.
Nous avons continué de travailler sur le sujet, après la soutenance, afin de résoudre le problème. Nous avons donc cerné le problème, qui était que le noeud Master ne devait pas contenir d'instance Cassandra. Ceci nous a permis de charger un mois de données pour les requêtes suivantes:
- Requête 1
- Requête 2
- Requête 3_1
- Requête 3_2
- Requête 4

### Chargement des fichiers dans AWS S3

Création d'un bucket S3 pour chargement des fichiers (suivant le TP AWS).
Chargement d'un an de données dans le bucket S3.
Pour ce faire, nous avons suivi le TP AWS, avec le code Scala permettant de charger les données dans le bucket S3.

### Création d'un Cluster

Le lancement du Cluster est fait à l'aide du module EMR d'AWS, avec 3 instances. Dans ce cas, EMR lance automatiquement 1 Master et 2 Slaves.

### Installation Cassandra sur Centos-7 (Amazon EMR)

https://www.vultr.com/docs/how-to-install-apache-cassandra-3-11-x-on-centos-7

- `sudo yum install -y java-1.8.0-openjdk` ou `sudo alternatives --config java` (Normalement, les instances EC2 centOS contiennent déjà une version java, compatible avec l'objectif du projet)
- `java -version`
- `echo $JAVA_HOME`

1. `ssh -i "test.pem" ec2-user@ec2-35-168-111-46.compute-1.amazonaws.com` (pour connexion en ssh au noeud EC2 CentOS)
2. `java -version`
3. `sudo nano /etc/yum.repos.d/cassandra311x.repo`
=> copier le texte suivant dans le fichier .repo:
[cassandra] 
name=Apache Cassandra baseurl=https://www.apache.org/dist/cassandra/redhat/311x/ gpgcheck=1 repo_gpgcheck=1 gpgkey=https://www.apache.org/dist/cassandra/KEYS

4. sudo rm -rf /var/lib/cassandra/data/system/* 
5. sudo yum install cassandra -y
6. Configuration du cassandra.yaml, avec la commande suivante: 
=> sudo nano /etc/cassandra/conf/cassandra.yaml

Avec les paramètres suivants:

- seeds: "172.31.81.86, 172.31.83.94, 172.31.91.18" (étant les adresses IP privées des instances EC2)
- listen_address: "172.31.81.86"
- rpc_address: “172.31.81.86”
(l'adresse 712.31.81.86 étant l'adresse privée du noeud EC2, et non l'adresse publique)
- endpoint_snitch GossipingPropertyFileSnitch (ce type de Gossip est le mieux adapté aux environnements de production)
- auto_bootstrap: false (option adaptée à un cluster ne contenant pas de données au départ)

7. sudo nano /etc/cassandra/conf/cassandra-rackdc.properties

- lsb_release -a
- sudo yum install epel-release -y
- sudo yum install --enablerepo='epel' ufw -y
- sudo ufw enable
- sudo ufw allow proto tcp from 172.31.53.37 to any port 9042
- sudo ufw allow proto tcp from 172.31.53.37 to any port 7000
- sudo ufw allow proto tcp from 172.31.58.36 to any port 9042
- sudo ufw allow proto tcp from 172.31.58.36 to any port 7000
- sudo ufw allow proto tcp from 172.31.53.163 to any port 9042
- sudo service cassandra start
- sudo nodetool status
- cqlsh

8. Afin de permettre aux différents noeuds Cassandra de communiquer (sachant que chaque noeud Cassandra est sur une machine EC2 différente), il faut configurer des règles de sécurité, au niveau des règles entrantes de chaque instance EC2:

- Aller sur l'écran du cluster en cours, et chercher un lien "subnet", cliquer dessus.
- Rechercher le bloc CIDR, dans les colonnes affichées. Copier cette valeur.
- Saisir le bloc CIDR du subnet du cluster EMR, au niveau des adresses entrantes de chaque noeud EC2 du cluster AWS, de type "Tous les TCP"

### Création du keysapce Cassandra et des tables:
Création du keyspace: `CREATE KEYSPACE gdelt_project WITH REPLICATION = {'class':'SimpleStrategy', 'replication_factor' : 3};`
Création des tables:
- Table pour requête 1: `CREATE TABLE request1 (year int, month int, day int, actionCountry text, language text, eventid int, numarticles int, PRIMARY KEY ( (year, month, day, actionCountry,language) , eventid ));`
- Table pour requête 2:`CREATE TABLE request2 (year int, month int, day int, actioncountry  text, eventid int, nummentions int, PRIMARY KEY ((actioncountry, year, month, day), nummentions)) WITH CLUSTERING ORDER BY (nummentions DESC);`
- Tables pour requête 3:
=> Table 1:`CREATE TABLE request31 (sourcecommonname text,year int,month int,day int,theme text,numarticles int,avgtone float, PRIMARY KEY (sourcecommonname,theme,year,month,day));`
=> Table 2:`CREATE TABLE request32 (sourcecommonname text,year int,month int,day int,person text,numarticles int,avgtone float, PRIMARY KEY  (sourcecommonname,year,month,day,person));`
=> Table 3:`CREATE TABLE request33 (sourcecommonname text,year int,month int,day int,location text,numarticles int,avgtone float, PRIMARY KEY  (sourcecommonname,year,month,day,location));`
- Table pour requête 4:`CREATE TABLE request4 (actor1countrycode text,actor2countrycode text,year int,month int,day int,avgtone int,numarticles int, PRIMARY KEY ((actor1countrycode,actor2countrycode),year,month,day);`

### Installation d'Ansible (amélioration de notre rendu)

Notre rendu, peut être amélioré, en utilisant ansible, afin d'automatiser le lancement des cluster et ainsi d'économiser du crédit sur AWS.

Setting Up Cassandra Cluster Through Ansible - Knoldus Blogs

 - https://blog.knoldus.com/setting-up-cassandra-cluster-through-ansible/

### Démarrage de Zeppelin

Ouvrir un tunnel SSH en lançant la commande suivante dans un terminal `ssh -i gdeltKeyPair.pem -ND 8157 hadoop@ec2-34-205-166-40.compute-1.amazonaws.com`. 
Il est nécessaire d'installer FoxyProxy, et suivre la procédure décrite par AWS, qui consiste à charger un fichier de configuration XML dans FoxyProxy. Après l'ajout de l'extension et import du fichier proxyproxy-settings.xml, vérifier que l'option "use proxies based on their predefined patterns and priorities" est cochée.

### Ajout du connecteur / interpréteur Cassandra dans Zeppelin

Cette étape n'est pas nécessaire si l'on travaille sur Zeppelin en local.

Aller dans l'option interpreter à droite (petite roue dentée), ensuite sur spark en mode edit, et rajouter les connecteurs ou interpréteurs nécessaires:
=> `"spark.jars.packages"` avec la valeur suivante	`"datastax:spark-cassandra-connector:2.4.0-s_2.11"`
=> `"spark.cassandra.connection.host"` avec la valeur suivante `"172.31.81.86, 172.31.83.94, 172.31.91.18"`, ie les adresses privées des noeuds EC2.

Il faut ajouter les imports suivants dans chaque notebook, afin de pouvoir lancer des requêtes vers le cluster Cassandra:

```
import org.apache.spark.sql.cassandra._
import com.datastax.spark.connector._
```

https://github.com/datastax/spark-cassandra-connector/blob/master/doc/14_data_frames.md
