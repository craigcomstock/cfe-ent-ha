# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  config.vm.box = "centos/6"

  config.vm.provider :libvirt do |lv|
    lv.cpus = 2
    lv.memory = 4096
    lv.volume_cache = :unsafe
  end

  config.vm.define "node1".to_sym do |node|
    node.vm.network :private_network, ip: "192.168.0.10"
    node.vm.network :private_network, ip: "192.168.10.10"
    node.vm.network :private_network, ip: "192.168.100.10"

    # node.vm.provision "shell", inline: <<-SHELL
    #     yum -y update
    # SHELL

    node.vm.provision "shell", inline: <<-SHELL
        set -xe
        yum -y install pcs pacemaker cman fence-agents
        hostname node1
        echo "192.168.0.10 node1" >> /etc/hosts
        echo "192.168.0.11 node2" >> /etc/hosts
        echo "192.168.10.10 node1-pg" >> /etc/hosts
        echo "192.168.10.11 node2-pg" >> /etc/hosts
        service pcsd start
        chkconfig pcsd on
        cp /vagrant/pgsql /usr/lib/ocf/resource.d/heartbeat/pgsql
        chown --reference /usr/lib/ocf/resource.d/heartbeat/{IPaddr,pgsql}
        chmod --reference /usr/lib/ocf/resource.d/heartbeat/{IPaddr,pgsql}
        printf "hacluster\\nhacluster\\n" | passwd hacluster
        pcs cluster auth node{1,2} -u hacluster -p hacluster
        pcs cluster setup --name cfcluster node{1,2}
        pcs cluster start --all
        sleep 1m
        pcs property set stonith-enabled=false
        pcs property set no-quorum-policy=ignore
        pcs resource defaults resource-stickiness="INFINITY"
        pcs resource defaults migration-threshold="1"
        pcs cluster status
        pcs status
        pcs resource create cfvirtip IPaddr2 ip=192.168.10.100 cidr_netmask=24 --group cfengine
        pcs cluster enable --all node{1,2}
        pcs status
    SHELL

    node.vm.provision "shell", inline: <<-SHELL
        set -xe
        rpm -i /vagrant/cfengine-nova-hub-3.12.0-1.x86_64.rpm
        service cfengine3 stop
        chkconfig cfengine3 off
    SHELL

    node.vm.provision "shell", inline: <<-SHELL
        set -xe
        mkdir -p /var/cfengine/state/pg/{data/pg_arch,tmp}
        chown -R cfpostgres:cfpostgres /var/cfengine/state/pg/{data/pg_arch,tmp}
        chmod --reference /var/cfengine/state/pg/data/postgresql.conf /vagrant/postgresql.conf.ha
        chown --reference /var/cfengine/state/pg/data/postgresql.conf /vagrant/postgresql.conf.ha
        cp /vagrant/postgresql.conf.ha /var/cfengine/state/pg/data/postgresql.conf
        echo "host replication all node2-pg trust" >> /var/cfengine/state/pg/data/pg_hba.conf
        echo "host replication all node1-pg trust" >> /var/cfengine/state/pg/data/pg_hba.conf
    SHELL
  end

  config.vm.define "node2".to_sym do |node|
    node.vm.network :private_network, ip: "192.168.0.11"
    node.vm.network :private_network, ip: "192.168.10.11"
    node.vm.network :private_network, ip: "192.168.100.11"

    # node.vm.provision "shell", inline: <<-SHELL
    #     yum -y update
    # SHELL

    node.vm.provision "shell", inline: <<-SHELL
        set -xe
        yum -y install pcs pacemaker cman fence-agents
        hostname node2
        echo "192.168.0.10 node1" >> /etc/hosts
        echo "192.168.0.11 node2" >> /etc/hosts
        echo "192.168.10.10 node1-pg" >> /etc/hosts
        echo "192.168.10.11 node2-pg" >> /etc/hosts
        service pcsd start
        chkconfig pcsd on
        printf "hacluster\\nhacluster\\n" | passwd hacluster
        rm -f /usr/lib/ocf/resource.d/heartbeat/pgsql
        cp -f /vagrant/pgsql /usr/lib/ocf/resource.d/heartbeat/pgsql
        chown --reference /usr/lib/ocf/resource.d/heartbeat/{IPaddr,pgsql}
        chmod --reference /usr/lib/ocf/resource.d/heartbeat/{IPaddr,pgsql}
    SHELL

    node.vm.provision "shell", inline: <<-SHELL
        rpm -i /vagrant/cfengine-nova-hub-3.12.0-1.x86_64.rpm
        service cfengine3 stop
        chkconfig cfengine3 off
    SHELL
  end
end
