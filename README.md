Vagrant - Confluent
===================

This is a Vagrant project to set up a 3-node Confluent.io Kafka cluster with
* Confluent Kafka
* Confluent Schema Registry
* Apache Zookeeper 3.9
* Datamountaineers Kafka Connect
* Cassandra Sink

This configuration will start and provision three CentOS7/64-Bit VMs each of them
containing a zookeeper, kafka-server, schema-registry and kafka connect server.

Whereever possible the vendors ( Confluent.io/Datamountaineer ) tar.gz / deb
packages have been used to have greatest flexibility in installation.


Purpose
------------
A kafka installation without source and sink does not make so much sense. This
vagrant setup is meant to be part of a larger environment. It is prepared to
connect to a Apache Cassandra cluster on the sink-side.

Prerequisites
-------------------------
* Vagrant (tested on 1.8.1)
* VirtualBox (tested on 5.1.10)
* Linux host. The Vagrantfile has been tested on a Linux Mint 18 machine. There are still known issues when running on Windows platform ( e.g. rsync problems, Guest Addition issues )

You can improve provision times and downloads by providing all necessary
downloads one in the pkg folder.


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

To ease the network handling all name/ip combinations are put into /etc/hosts so
we have a simple name resolution system.

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
Zookeeper is more-or-less the configuration memory for the kafka,schema-registry and kafka-connect
cluster described in this README.

The Zookeeper installation is done independently from the Confluent stuff.
As it is pretty standard, Zookeeper is running on port 2181 here as well. Zookeepers
data directory is in /tmp/zookeeper. Zookeeper is configured to run in cluster mode,
all three nodes of this vagrant cluster are part of it.
Plain Apache Zookeeper tar.gz have been used for the installation. You can work
with the zookeeper cli by :

```
su - zookeeper # use zookeeper password
cd bin
./zkCli.sh
```

Zookeeper is started during the provision phase of this vagrant cluster by simply
calling the command line. To stop the zookeeper process you have to find out the
process id first, e.g. by typing

```
su - zookeeper
cd bin
./zkServer stop
```

To start zookeeper again use 'start' as a parameter to zkServer.sh .


kafka-server
-------------
Kafka Server is setup as a 3-node cluster like zookeeper, too. Each kafka server is listening on
port 9092. As kafka server is installed in user kafkas home directory /opt/kafka you
have to switch to that user before starting and stopping the server.

Kafka server will be started during cluster provisioning.

During the provision phase a /vagrant/config/server[1..3].properties will be copied
to /opt/kafka/etc/kafka/server.properties. The /vagrant/config folder within the
vagrant box is a copy of the config folder in this git project. The properties files will
be copied regarding to the following map :

| Node Name | Property file |
| ------- | -------|
| consumer1 | server1.properties|
| consumer2 | server2.properties|
| consumer3 | server3.properties|

We need a special server.properties file for each node, because the IP adress is
hardcoded in there.


Schema-Registry
----------------

The schema-registry is not started by default. You have to do that on each
node of the cluster by

```
cd /opt/kafka/bin
schema-registry-start -daemon ../etc/schema-registry/schema-registry.properties
```
Currently the logfiles of the schema-registry go to /tmp/schema-registry. To verify
if the service runs please use `ps -ef | grep schema` or equivalent. Check the logfiles
for warnings or errors.

The schema-registry depends on the zookeeper service. If zookeeper is not available
schema-registry does not recover and has to be restarted manually.

To stop schema-registry simply run

```
cd /opt/kafka/bin
schema-registry-stop
```
on the node you want schema-registry to stop.


kafka-connect
-------------
Kafka Connect is included in the Confluent.io Open Source package. As we want to
check out the Cassandra sink capabilities later on some parts of Datamountaineers
Stream Reactor software is installed during provisioning.
Part of the Stream Reactor software is a CLI for kafka connect, that enables the
operator to simply maintain sinks and sources.

To start the kafka-connect server run

```
cd /opt/kafka/bin
connect-distributed -daemon ../etc/schema-registry/connect-avro-distributed.properties
tail -f ../log/connectDistributed.out
```

as user kafka for each node.


A Cassandra sink Use-Case
--------------------------

[See here] (http://docs.datamountaineer.com/en/latest/cassandra-sink.html)


`kafka-avro-console-producer --broker-list $BROKER_LIST --topic orders-topic --property value.schema='{"type":"record","name":"myrecord","fields":[{"name":"id","type":"int"},{"name":"created","type":"string"},{"name":"product","type":"string"},{"name":"price","type":"double"}]}'`
