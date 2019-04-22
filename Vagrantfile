# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|


	config.vm.define "ldap_server", primary: true do |ldap_server|
	   ldap_server.vm.hostname = "ldap-server"
	   ldap_server.vm.box      = "debian/jessie64"
           ldap_server.vm.box_check_update = false
           ldap_server.vm.network "forwarded_port", guest: 389, host: 3890
	   ldap_server.vm.network :private_network, ip: "192.168.100.101"

	ldap_server.vm.provider "virtualbox" do |v|
           v.customize [ "modifyvm", :id, "--cpus", "1" ]
	   v.customize [ "modifyvm", :id, "--memory", "256" ]	
	end
           ldap_server.vm.provision "shell", inline: <<-SHELL

        # Criação de grupos
        sudo adduser --group cordenacao
        sudo adduser --group secretaria
        sudo adduser --group estagiario

        #Adição de usuários ao grupo cordenação:
        sudo useradd -m -g cordenacao -p FDBu190.2vfwo joao
        sudo useradd -m -g cordenacao -p FDBu190.2vfwo maria
        sudo useradd -m -g cordenacao -p FDBu190.2vfwo -G secretaria jose

        #Adição de usuários ao grupo secretaria:
        sudo useradd -m -g secretaria -p FDBu190.2vfwo -G cordenacao alice
        sudo useradd -m -g secretaria -p FDBu190.2vfwo bob


        #Adição de usuários ao grupo estagiários:
        sudo useradd -m -g estagiario -p FDBu190.2vfwo joaquim
        sudo useradd -m -g estagiario -p FDBu190.2vfwo manoel
        sudo useradd -m -g estagiario -p FDBu190.2vfwo isabel

           SHELL
	end
	

	#......................................................................

        config.vm.define "nfs_server" do |nfs_server|
           nfs_server.vm.hostname = "nfs-server"
           nfs_server.vm.box      = "debian/jessie64"
           nfs_server.vm.box_check_update = false
           nfs_server.vm.network "private_network", ip: "192.168.100.104"

        nfs_server.vm.provider "virtualbox" do |z|
           z.customize [ "modifyvm", :id, "--cpus", "1" ]
           z.customize [ "modifyvm", :id, "--memory", "256" ]
        end
        end

	#......................................................................

	config.vm.define "client_01" do |client_01|
	   client_01.vm.hostname = "client-01"
	   client_01.vm.box = "debian/jessie64"
           client_01.vm.box_check_update = false
	   client_01.vm.network "private_network", ip: "192.168.100.102"
   
	client_01.vm.provider "virtualbox" do |y|
           y.customize [ "modifyvm", :id, "--cpus", "1" ]
           y.customize [ "modifyvm", :id, "--memory", "256" ]
        end
	end
	
	#......................................................................

        config.vm.define "client_02" do |client_02|
           client_02.vm.hostname = "client-02"
           client_02.vm.box      = "debian/jessie64"
           client_02.vm.box_check_update = false
           client_02.vm.network "private_network", ip: "192.168.100.103"
   
        client_02.vm.provider "virtualbox" do |z|
           z.customize [ "modifyvm", :id, "--cpus", "1" ]
           z.customize [ "modifyvm", :id, "--memory", "256" ]
        end
	end

end
