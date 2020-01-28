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

### Cassandra en local

- `sudo apt update`
- `sudo apt install cassandra`

## Etapes d'installation et de configuration sur EC2

*Sources utiles:*

 - https://www.linode.com/docs/databases/cassandra/set-up-a-cassandra-node-cluster-on-ubuntu-and-centos/
 - https://www.vultr.com/docs/how-to-install-apache-cassandra-3-11-x-on-centos-7

### Création d'un Cluster

Configurer SSH everywhere avec les règles entrantes

### Installation Cassandra sur Centos-7 (Amazon EMR)

https://www.vultr.com/docs/how-to-install-apache-cassandra-3-11-x-on-centos-7

- `sudo yum install -y java-1.8.0-openjdk` ou `sudo alternatives --config java`
- `java -version`
- `echo $JAVA_HOME`

1. `ssh -i "test.pem" ec2-user@ec2-35-168-111-46.compute-1.amazonaws.com`
2. `java -version`
3. `sudo nano /etc/yum.repos.d/cassandra311x.repo`

[cassandra] name=Apache Cassandra baseurl=https://www.apache.org/dist/cassandra/redhat/311x/ gpgcheck=1 repo_gpgcheck=1 gpgkey=https://www.apache.org/dist/cassandra/KEYS

4. sudo rm -rf /var/lib/cassandra/data/system/* 
5. sudo yum install cassandra -y
6. sudo nano /etc/cassandra/conf/cassandra.yaml

Avec les paramètres suivants:

- seeds: "172.31.81.86, 172.31.83.94, 172.31.91.18, 172.31.87.57, 172.31.87.67"
- listen_address: "172.31.81.86"
- rpc_address: “172.31.81.86”
- endpoint_snitch GossipingPropertyFileSnitch
- auto_bootstrap: false

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


### Installation d'Ansible

On peut automatiser le processus d'installation de cassandra sur les noeuds à l'aide d'Ansible

Setting Up Cassandra Cluster Through Ansible - Knoldus Blogs

 - https://blog.knoldus.com/setting-up-cassandra-cluster-through-ansible/

### Démarrage de Zeppelin

Ouvrir un tunnel SSH en copiant le lien trouvé sur SSH dans un terminal dans le même répertoire que la clé. Si tous les flux dans les groupes de sécurité ne sont pas ouverts, et que vous ne voulez pas autoriser tous les flux publics, il est nécessaire d'installer FoxyProxy. Après l'ajout de l'extension et import du fichier proxyproxy-settings.xml, vérifier que l'option "use proxies based on their predefined patterns and priorities" est cochée.

### Ajout du connecteur / interpréteur Cassandra dans Zeppelin

Cette étape n'est pas nécessaire si l'on travaille sur Zeppelin en local

Aller dans l'option interpreter à droite (petite roue dentée), ensuite spark, et rajouter les connecteurs ou interpréteurs nécessaires

Ne pas oublier de les rajouter sur le notebook, d'enregistrer, et d'écrire $nom_interpréteur dans la cellule ou l'interpréteur est appelé

```scala
import org.apache.spark.sql.cassandra._
import com.datastax.spark.connector._
df.write.cassandraFormat("TableName", "KeySpace").save()
```

https://github.com/datastax/spark-cassandra-connector/blob/master/doc/14_data_frames.md