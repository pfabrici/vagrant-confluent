Vagrant - Confluent
===================

This is a Vagrant project to set up a 3-node Confluent Kafka cluster with
* Confluent Kafka
* Confluent Schema Registry
* Apache Zookeeper 3.9
* Datamountaineers Kafka Connect
* Cassandra Sink

This configuration will start and provision three CentOS7/64-Bit VMs. All of the three nodes host a zookeeper a kafka node.

Prerequisites
-------------------------
* Vagrant (tested on 1.8.1)
* VirtualBox (tested on 5.1.10)


Nodes
------------
The nodes are members of a "Private Network". They have IPs 10.30.3.2 ... 10.30.3.4.
Each node can see the others but the host can not access the boxes. As the
netmask is 255.255.0.0 the cluster nodes can access other Vagrant boxes with
private network and IPs like 10.30.x.y

Zookeeper
------------------
Zookeeper is installed in /opt/zookeeper. It is run by its own zookeeper user.
Switch to zookeeper from the vagrant user by typing :
`su - zookeeper`
Provide the password `zookeeper`. Zookeeper will be started during provisioning.

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
