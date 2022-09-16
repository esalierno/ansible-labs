# -*- mode: ruby -*-
# vi: set ft=ruby :
# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    #ENV['VAGRANT_EXPERIMENTAL'] = 'disks'
    ENV["LC_ALL"] = "en_US.UTF-8"

    config.vm.define "ansible" do |hostconfig|
        hostconfig.vm.box = "centos/stream8"
        hostconfig.vm.box_download_insecure=true
        hostconfig.vm.hostname = "ansible.example.com"
        hostconfig.vm.network "private_network", ip: "192.168.60.10", hostname: true

        $update_hosts = <<-SCRIPT
        echo '192.168.60.100  lb lb.example.com' >> /etc/hosts
        echo '192.168.60.20  pweb01 pweb01.example.com' >> /etc/hosts
        echo '192.168.60.21  pweb02 pweb02.example.com' >> /etc/hosts
        echo '192.168.60.30  pdb01 pdb01.example.com' >> /etc/hosts
        echo '192.168.60.31  pdb02 pdb02.example.com' >> /etc/hosts
SCRIPT

        hostconfig.vm.provider "virtualbox" do |v|
            v.linked_clone = true
            v.memory = 512
            v.cpus = 1
        end
        hostconfig.vm.provision "Generate ansible ssh keys", type: "shell", inline: "[ -f /home/vagrant/.ssh/id_rsa ] || ssh-keygen -q -f /home/vagrant/.ssh/id_rsa -N ''"
        hostconfig.vm.provision "Changing ownership of ssh keys", type: "shell", inline: "chown vagrant:vagrant /home/vagrant/.ssh/*"
        hostconfig.vm.provision "Disable StrictHostKeyChecking", type: "shell", inline: "echo 'StrictHostKeyChecking no'>/home/vagrant/.ssh/config && chown vagrant:vagrant /home/vagrant/.ssh/config && chmod 600 /home/vagrant/.ssh/config"
        hostconfig.vm.provision "Copy labs stuff", type: "shell", inline: "cp -r /vagrant/labs /home/vagrant && chown -R vagrant:vagrant /home/vagrant/labs"
        hostconfig.vm.provision "Update /etc/hosts", type: "shell", inline: $update_hosts
        hostconfig.trigger.after :up do |trigger|
            trigger.name = "Retrieve ansible ssh keys"
            if Vagrant::Util::Platform.windows? then
                trigger.run = {inline: "./OpenSSH/scp.exe -F ssh.config vagrant@ansible:/home/vagrant/.ssh/id_rsa.pub ansible_id_rsa.pub"}
            else
                trigger.run = {inline: "scp -F ssh.config vagrant@ansible:/home/vagrant/.ssh/id_rsa.pub ansible_id_rsa.pub"}
            end
            
        end
    end
    config.vm.define "lb" do |hostconfig|
        hostconfig.vm.box = "centos/stream8"
        hostconfig.vm.hostname = "lb.example.com"
        hostconfig.vm.network "private_network", ip: "192.168.60.100", hostname: true
        hostconfig.vm.provider "virtualbox" do |v|
            v.linked_clone = true
            v.memory = 256
            v.cpus = 1
        end
        hostconfig.vm.provision "Add ansible public key to authorized_keys", type: "shell", inline: "cat /vagrant/ansible_id_rsa.pub >> /home/vagrant/.ssh/authorized_keys"
    end
    config.vm.define "pweb01" do |hostconfig|
        hostconfig.vm.box = "centos/stream8"
        hostconfig.vm.hostname = "pweb01.example.com"
        hostconfig.vm.network "private_network", ip: "192.168.60.20", hostname: true
        hostconfig.vm.provider "virtualbox" do |v|
            v.linked_clone = true
            v.memory = 256
            v.cpus = 1
        end
        hostconfig.vm.provision "Add ansible public key to authorized_keys", type: "shell", inline: "cat /vagrant/ansible_id_rsa.pub >> /home/vagrant/.ssh/authorized_keys"
    end
    config.vm.define "pweb02" do |hostconfig|
        hostconfig.vm.box = "centos/stream8"
        hostconfig.vm.hostname = "pweb02.example.com"
        hostconfig.vm.network "private_network", ip: "192.168.60.21", hostname: true
        hostconfig.vm.provider "virtualbox" do |v|
            v.linked_clone = true
            v.memory = 256
            v.cpus = 1
        end
        hostconfig.vm.provision "Add ansible public key to authorized_keys", type: "shell", inline: "cat /vagrant/ansible_id_rsa.pub >> /home/vagrant/.ssh/authorized_keys"
    end
    config.vm.define "pdb01" do |hostconfig|
        hostconfig.vm.box = "centos/stream8"
        hostconfig.vm.hostname = "pdb01.example.com"
        hostconfig.vm.network "private_network", ip: "192.168.60.30", hostname: true
        hostconfig.vm.disk :disk, name: "data1", size: "5GB", disk_ext: "vmdk"
        hostconfig.vm.disk :disk, name: "data2", size: "5GB", disk_ext: "vmdk"
        hostconfig.vm.provider "virtualbox" do |v|
            v.linked_clone = true
            v.memory = 256
            v.cpus = 1
        end
        hostconfig.vm.provision "Add ansible public key to authorized_keys", type: "shell", inline: "cat /vagrant/ansible_id_rsa.pub >> /home/vagrant/.ssh/authorized_keys"
    end
    config.vm.define "pdb02" do |hostconfig|
        hostconfig.vm.box = "centos/stream8"
        hostconfig.vm.hostname = "pdb01.example.com"
        hostconfig.vm.network "private_network", ip: "192.168.60.31", hostname: true
        hostconfig.vm.disk :disk, name: "data1", size: "5GB", disk_ext: "vmdk"
        hostconfig.vm.disk :disk, name: "data2", size: "5GB", disk_ext: "vmdk"
        hostconfig.vm.provider "virtualbox" do |v|
            v.linked_clone = true
            v.memory = 256
            v.cpus = 1
        end
        hostconfig.vm.provision "Add ansible public key to authorized_keys", type: "shell", inline: "cat /vagrant/ansible_id_rsa.pub >> /home/vagrant/.ssh/authorized_keys"
    end
    
end