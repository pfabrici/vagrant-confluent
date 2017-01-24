Vagrant - Confluent
===================

This is a Vagrant project to set up a 3-node Confluent Kafka cluster with
* Confluent Kafka
* Confluent Schema Registry
* Apache Zookeeper 3.9
* Datamountaineers Kafka Connect
* Cassandra Sink

This configuration will start and provision three CentOS7/64-Bit VMs each of them
containing a zookeeper, kafka-server, schema-registry and kafka connect server.

Prerequisites
-------------------------
* Vagrant (tested on 1.8.1)
* VirtualBox (tested on 5.1.10)

Node Configuration
------------
All three nodes base on a core Centos 7. They are configured to get 2GB RAM so
you will need >= 6GB RAM for the cluster. The nodes are members of a "Private Network".
These IPs are assigned to node names :

|Nodename | IP-Adress|
|-------| ---------|
|consumer1|10.30.3.2|
|consumer2|10.30.3.3|
|consumer3|10.30.3.4|

Each node can see the others but the host can not access the boxes. As the
netmask is 255.255.0.0 the cluster nodes can access other Vagrant boxes with
private network and IPs like 10.30.x.y

IPV6 is turned off to avoid any impacts.

Only a few CentOS packages will be installed during provisioning :
* wget to simplify download of additional software
* net-tools ( for /sbin/ifconfig )
* nc
* ntp ( to have same date on each node )

Two operating system users have been prepared :
* zookeeper with homedir /opt/zookeeper
* kafka with homedir /opt/kafka

Both users have an password equal to their username. You can su to the users by typing
`su - zookeeper ` from the vagrant users shell.

Zookeeper
------------------
The Zookeeper installation is done independentlty from the Confluent stuff.
As it is pretty standard Zookeeper is running on port 2181 here as well. Zookeepers
data directory is in /tmp/zookeeper.
Plain Apache Zookeeper tar.gz have been used for the installation. You can work
with the zookeeper cli by :

```
su - zookeeper # use zookeeper password
cd bin
./zkCli.sh
```


Schema-Registry
----------------
Confluents schema-registry is part of the kafka installation. In this setup you can find it
under /opt/kafka. As kafka is installed under its own user kafka you have to Switchto it from the vagrant
user before starting or stopping the service :
`su - kafka`
Same here - kafka's password is kafka. Change directory to bin before starting services :
`
cd /opt/kafka/bin
schema-registry-start -daemon ../etc/schema-registry/schema-registry.properties
connect-distributed -daemon ../etc/schema-registry/connect-avro-distributed.properties`


Kafka Connect starten
---------------------
connect-distributed -daemon ../etc/schema-registry/connect-avro-distributed.properties


`kafka-avro-console-producer --broker-list $BROKER_LIST --topic orders-topic --property value.schema='{"type":"record","name":"myrecord","fields":[{"name":"id","type":"int"},{"name":"created","type":"string"},{"name":"product","type":"string"},{"name":"price","type":"double"}]}'`
