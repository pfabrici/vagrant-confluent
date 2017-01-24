# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.box = 'centos/7'
  config.vm.synced_folder ".", "/vagrant"

  config.ssh.forward_agent = true # So that boxes don't have to setup key-less ssh
  config.ssh.insert_key = false # To generate a new ssh key and don't use the default Vagrant one

  # common provisioning for all
  config.vm.provision "shell", path: "scripts/init.sh"

  # configure zookeeper cluster
  (1..3).each do |i|
    config.vm.define "consumer#{i}" do |s|
      s.vm.hostname = "consumer#{i}"
      s.vm.network "private_network", ip: "10.30.3.#{i+1}", netmask: "255.255.0.0", virtualbox__intnet: "replic-network"

      s.vm.provision "shell" do |cmd|
        cmd.inline = "echo $1 > /tmp/zookeeper/myid && su -l -c 'cp /vagrant/config/zoo.cfg /opt/zookeeper/conf && zkServer.sh start' zookeeper"
        cmd.args   = ["#{i}"]
      end

      s.vm.provision "shell", inline: "sleep 10"

      s.vm.provision "shell" do |cmd|
        cmd.inline = "cp /vagrant/config/server${1}.properties /opt/kafka/etc/kafka/server.properties && chown -R kafka.kafka /opt/kafka/etc && su -l -c 'kafka-server-start -daemon /opt/kafka/etc/kafka/server.properties' kafka"
        cmd.args   = ["#{i}"]
      end


      s.vm.provision "shell" do |cmd|
        #cmd.inline = "cp /vagrant/config/registry${1}.properties /opt/kafka/etc/schema-registry/schema-registry.properties && chown -R kafka.kafka /opt/kafka/etc && su -l -c 'schema-registry-start -daemon /opt/kafka/etc/schema-registry/schema-registry.properties' kafka"
        cmd.inline = "cp /vagrant/config/registry${1}.properties /opt/kafka/etc/schema-registry/schema-registry.properties && chown -R kafka.kafka /opt/kafka/etc"
        cmd.args   = ["#{i}"]
      end

      s.vm.provision "shell" do |cmd|
        #cmd.inline = "cp /vagrant/config/connect-avro-distributed.properties /opt/kafka/etc/schema-registry/connect-avro-distributed.properties && chown -R kafka.kafka /opt/kafka/etc && su -l -c 'connect-distributed -daemon /opt/kafka/etc/schema-registry/connect-avro-distributed.properties' kafka"
        cmd.inline = "cp /vagrant/config/connect-avro-distributed.properties /opt/kafka/etc/schema-registry/connect-avro-distributed.properties && chown -R kafka.kafka /opt/kafka/etc"
        cmd.args   = ["#{i}"]
      end


    end
  end

  config.vm.provider "virtualbox" do |v|
    #  This setting controls how much cpu time a virtual CPU can use. A value of 50 implies a single virtual CPU can use up to 50% of a single host CPU.
    v.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
    v.memory = 2000
  end
end