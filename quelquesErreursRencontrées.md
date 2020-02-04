# Quelques erreurs rencontrées et leur résolution

Il est à noter que pour ce projet, nous utilisons un compte Amazon Educate.Il se peut donc qu'il y ait des erreurs très spécifiques à cette version.

## Erreur rencontrée: Cluster résilié avec erreurs EC2 liés aux groupes de sécurité

ClusterRésilié avec des erreursThe EC2 Security Groups [sg-0d3e49b7d23d67d03] contain one or more ingress rules to ports other than [22] which allow public access.

→ Cette erreur a été résolu, en enlevant les règles de sécurité entrantes, publiques, qui ouvraient l'accès à tout le monde.
→ Les règles de sécurité ont été configurées, moyennant les bloc CIDR, du subnet du cluster EMR. 

## Erreur rencontrée: Permissions de la clé

```
Permissions 0664 for 'key.pem' are too open. It is required that your private key files are NOT accessible by others. This private key will be ignored. Load key "key.pem": bad permissions hadoop@ec2-xxx-xxx-xxx-xxx.compute-1.amazonaws.com: Permission denied (publickey)
```

→ `chmod 400 key.pem` puis relancement de la tentative de connexion ssh `sudo ssh -i key.pem hadoop@ec2-xxx-xxx-xxx-xxx.compute-1.amazonaws.com`

## Erreur rencontrée: Installation Cassandra

Downloading packages: cassandra-3.11.5-1.noarch.rpm FAILED
https://www.apache.org/dist/cassandra/redhat/311x/cassandra-3.11.5-1.noarch.rpm: [Errno 14] curl#35 - "OpenSSL SSL_connect: SSL_ERROR_SYSCALL in connection to [www.apache.org:443](http://www.apache.org:443/) " Essai d'un autre miroir.

Error downloading packages: cassandra-3.11.5-1.noarch: [Errno 256] No more mirrors to try.

→ Il faut faire attention à pointer vers le bon dossier `https://www.apache.org/dist/cassandra/redhat/311x/"`


Les instances EC2, doivent permettre des connexions ssh, et permettre aux noeuds Cassandra de communiquer, en ajoutant les bloc CIDR issus du subnet du cluster généré: "Tous les TCP" + range de ports "0 - 65000" + X.X.X.X/XYZ (=bloc CIDR du subnet)

##  Erreur rencontrée: Interpréteur Cassandra sur Zeppelin

Aller dans zeppelin --> interpreter --> Spark --> Edit et rajouter : spark.jars.packages => Value: datastax:spark-cassandra-connector:2.4.0-s_2.11 spark.cassandra.connection.host => Value: les adresses IP des noeuds

## Erreur rencontrée : Erreur liée aux noeuds sur différents datacenters ou racks

```
Cannot start node if snitch's data center (dc1) differs from previous data center (datacenter1)
```
http://prajeeth.com/tech_blog/apache/cassandra/cassandra_snitch_error/

## Erreur rencontrée: Port d'installation 127.0.0.2:9042 already in use

```sh
Traceback (most recent call last):File "/usr/local/bin/ccm", line 112, in cmd.run() File "/usr/local/lib/python2.7/dist-packages/ccmlib/cmds/cluster_cmds.py", line 510, in run allow_root=self.options.allow_root) is None: File "/usr/local/lib/python2.7/dist-packages/ccmlib/cluster.py", line 390, in start common.assert_socket_available(itf) File "/usr/local/lib/python2.7/dist-packages/ccmlib/common.py", line 521, in assert_socket_available raise UnavailableSocketError("Inet address %s:%s is not available: %s; a cluster may already be running or you may need to add the loopback alias" % (addr, port, msg)) ccmlib.common.UnavailableSocketError: Inet address 127.0.0.1:9042 is not available: [Errno 98] Address already in use; a cluster may already be running or you may need to add the loopback alias
```

Bien vérifier les propriétés IP (seeds/listen_address/rpc_address) du fichier cassandra.yaml, `sudo sh cassandra -R`. Saisir ensuite `cqlsh adresse_privée_de_l'instance_EC2 9042`, afin de se connecter en CQL au noeud Cassandra.

## Erreur rencontrée lors de l'exécution des requêtes dans notebook Zeppelin: `Broken pipe`, puis `Java Connection refused`:

Cette erreur intervient de façon aléatoire, mais très récurrente, lors de l'exécution de code dans le notebook Zeppelin. Celle-ci est due au manque de mémoire dans l'instance EC2 d'AWS, et au fait que nous avions installé Cassandra dans le noeud Master du cluster EMR d'AWS. De ce fait, Zeppelin tombait en erreur, car la mémoire était utilisée par Cassandra aussi, qui était installé sur le même noeud que Zeppelin et Spark.

Résolution:
Nous avons désinstallé Cassandra du noeud Master du cluster, et avons pu charger 1 mois de données pour toutes nos requêtes. Le noeud Master est le noeud utilisé par le cluster EMR, pour lancer Zeppelin et Spark, donc ne doit pas contenir de serveur Cassandra.
