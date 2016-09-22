# -*- mode: ruby -*-
# vi: set ft=ruby :
# We set the last octet in IPV4 address here
 nodes = {
 'aio' => [1, 100],
 'nagios' => [1, 101],
 'stackstorm' => [1, 102],
}

Vagrant.configure("2") do |config| 
  config.vm.box = "trusty64"
  config.vm.box_url = "http://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box"
  config.vm.usable_port_range= 2800..2900

  nodes.each do |prefix, (count, ip_start)|
    count.times do |i|
      hostname = "%s" % [prefix, (i+1)]
      config.vm.define "#{hostname}" do |box|
        box.vm.hostname = "#{hostname}.local"
        if prefix == "aio"
          box.vm.network :private_network, ip: "172.29.236.#{ip_start+i}", :netmask => "255.255.252.0", auto_config: false
          box.vm.network :private_network, ip: "172.17.0.#{ip_start+i}", :netmask => "255.255.255.0"
        else
          box.vm.network :private_network, ip: "172.29.236.#{ip_start+i}", :netmask => "255.255.252.0"
        end # prefix
        box.vm.provider :virtualbox do |vbox|
          # Defaults
          vbox.linked_clone = true
          vbox.name = "#{hostname}"
          vbox.customize ["modifyvm", :id, "--groups", "/test"]
          vbox.customize ["modifyvm", :id, "--memory", 1024]
          vbox.customize ["modifyvm", :id, "--cpus", 1]
          vbox.customize ["modifyvm", :id, "--pae", "on"]
          vbox.customize ["modifyvm", :id, "--paravirtprovider", "kvm"]
          vbox.customize ["modifyvm", :id, "--nictype1", "Am79C973"]
          vbox.customize ["modifyvm", :id, "--nictype2", "Am79C973"]
          if prefix == "aio"
            vbox.customize ["modifyvm", :id, "--memory", 8192]
            vbox.customize ["modifyvm", :id, "--cpus", 4]
            vbox.customize ["modifyvm", :id, "--nictype3", "Am79C973"]
            vbox.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
            vbox.customize ["modifyvm", :id, "--nicpromisc3", "allow-all"]
            dir = "#{ENV['HOME']}/vagrant-additional-disk"
            unless File.directory?( dir )
                Dir.mkdir dir
            end # unless
            file_to_disk = "#{dir}/#{hostname}-sdb.vmdk"
            unless File.exists?( file_to_disk )
              vbox.customize ['createhd', '--filename', file_to_disk, '--size', 120 * 1024]
            end # unless
            vbox.customize ['storageattach', :id, '--storagectl', 'SATAController', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', file_to_disk]
          end # aio
          if prefix == "stackstorm"
            vbox.customize ["modifyvm", :id, "--memory", 2048]
            vbox.customize ["modifyvm", :id, "--cpus", 2]
          end # stackstorm
        end # box.vm virtualbox
        if prefix == "stackstorm" # only run once the stackstorm VM has been brought on line
          config.vm.provision "ansible" do |ansible|
            # Disable default limit to connect to all the machines
            ansible.limit = "all"
            ansible.playbook = "site.yml"
            ansible.groups = {
              "vagrant" => ["aio", "nagios", "stackstorm"],
              "ntp-server" => ["aio"],
              "ntp-client" => ["nagios", "stackstorm"],
              "nagios-mon" => ["aio"],
              "stackstorm" => ["stackstorm"]
            }
          end # do ansible
        end # if prefix == stackstorm
      end # config.vm.define
    end # count.times
  end # nodes.each
end # Vagrant.configure("2")