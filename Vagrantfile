# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

current_dir    = File.dirname(File.expand_path(__FILE__))

Vagrant.configure(2) do |config|

config.vm.synced_folder '.', '/vagrant', disabled: true
config.vm.box = "generic/centos7"
config.vm.box_version = "2.0.6"


# Create the MDS systems.
config.ssh.insert_key = false
config.vm.define "mds1" do |mds|
  mds.vm.hostname = "mds1"
  mds.vm.network :private_network, ip: "192.168.60.11"
  mds.vm.network "private_network", ip: "192.168.1.251"
  mds.vm.provider "virtualbox" do |vb|
    vb.memory = 1024
    vb.cpus = 2
    vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata", "--controller", "IntelAHCI"]
    datadisk="#{current_dir}/drives/mds1-sdb.vdi"
    unless File.exist?(datadisk)
      vb.customize ['createhd', '--filename', datadisk, '--variant', 'Fixed', '--size', 4 * 1024]
    end
    vb.memory = "1024"
    vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', datadisk]
  end
end

# Create the OSS systems.
  (1..3).each do |i|
    config.ssh.insert_key = false
    config.vm.define "oss#{i}" do |oss|
      oss.vm.hostname = "oss#{i}"
      oss.vm.network :private_network, ip: "192.168.60.2#{i}"
      oss.vm.provider "virtualbox" do |vb|
        vb.memory = 1024
        vb.cpus = 2
        vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata", "--controller", "IntelAHCI"]
        datadisk="#{current_dir}/drives/oss#{i}-sdb.vdi"
        unless File.exist?(datadisk) 
          vb.customize ['createhd', '--filename', datadisk, '--variant', 'Fixed', '--size', 4 * 1024]
        end
        vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', datadisk]
      end
    end
  end

# Create the client.
  config.vm.define "cli1" do |cli1|
    config.ssh.insert_key = false
    cli1.vm.hostname = "cli1"
    cli1.vm.network :private_network, ip: "192.168.60.31"
    cli1.vm.network :private_network, ip: "192.168.60.32"
    cli1.vm.provider "virtualbox" do |vb|
      vb.memory = 1024
    end
  end

# Create the client.
  config.vm.define "cli2" do |cli2|
    config.ssh.insert_key = false
    cli2.vm.box = "generic/centos7"
    cli2.vm.box_version = "2.0.6"
    cli2.vm.hostname = "cli1"
    cli2.vm.network :private_network, ip: "192.168.60.33"
    cli2.vm.network :private_network, ip: "192.168.60.34"
    cli2.vm.provider "virtualbox" do |vb|
      vb.memory = 1024
    end
  end


  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.compatibility_mode="2.0"
    ansible.groups = {
      "mds" => ["mds[1:2]"],
      "oss" => ["oss[1:3]"],
      "cli" => ["cli[1:2]"],
      "lustre:children" => ["mds", "oss"],
    }
    ansible.host_vars = {
      "mds1" => {"mount_point" => "/lustre/testfs-MDT0000",
                 "lustre_index" => "0"},
      "mds2" => {"mount_point" => "/lustre/testfs-MDT0001",
                 "lustre_index" => "1"},
      "oss1" => {"mount_point" => "/lustre/testfs-OST0000",
                 "lustre_index" => "0"},
      "oss2" => {"mount_point" => "/lustre/testfs-OST0001",
                 "lustre_index" => "1"},
      "oss3" => {"mount_point" => "/lustre/testfs-OST0002",
                 "lustre_index" => "2"},
      "cli1" => {"mount_point" => "/dfs/testfs"},
      "cli2" => {"mount_point" => "/dfs/testfs"}
    }
  end
end

