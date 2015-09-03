# -*- mode: ruby -*-
# vi: set ft=ruby :


Vagrant.configure(2) do |config|
 
  config.vm.box = "ubuntu/trusty64"

  config.vm.provider "virtualbox" do |vb|
  
	vb.memory = "256"

  end



  config.vm.define :happyclient do |client|
	client.vm.hostname = "happyclient"
	client.vm.network "private_network", ip: "192.168.0.3", auto_config: false
	
  end

  config.vm.define :happyrouter do |router|
	router.vm.hostname = "happyrouter"

	router.vm.network "private_network", ip: "192.168.0.2", auto_config: false
	#router.vm.network "public_network", bridge: "eth0", ip:"192.168.0.2" 
	
  end	



end
