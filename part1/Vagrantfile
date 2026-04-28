Vagrant.configure("2") do |config|
	$common_script = <<~SCRIPT
		sudo apt update
		sudo apt install -y curl 
	SCRIPT

	$server_script = <<~SCRIPT
		# On force l'utilisation de l'IP du réseau privé pour l'API server
		curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--node-ip=192.168.56.110 --bind-address=192.168.56.110 --advertise-address=192.168.56.110" sh -
		
		# Attendre que le token soit généré
		while [ ! -f /var/lib/rancher/k3s/server/node-token ]; do sleep 2; done
		
		sudo cat /var/lib/rancher/k3s/server/node-token > /vagrant/k3s_token
	SCRIPT

	$install_worker = <<~SCRIPT
		# Attendre que le master ait écrit le token dans le dossier partagé
		while [ ! -f /vagrant/k3s_token ]; do sleep 2; done
		
		K3S_TOKEN=$(cat /vagrant/k3s_token)
		# Correction de l'IP : 192.168.56.110
		curl -sfL https://get.k3s.io | K3S_URL=https://192.168.56.110:6443 K3S_TOKEN=$K3S_TOKEN INSTALL_K3S_EXEC="--node-ip=192.168.56.111" sh -
	SCRIPT

	config.vm.define "server" do |server_config|
		server_config.vm.box = "debian/bookworm64"
		server_config.vm.hostname = "antgabriS"
		server_config.vm.network "private_network", ip: "192.168.56.110", virtualbox__intnet: "k3s-net"
		server_config.vm.provision "shell", inline: $common_script
		server_config.vm.provision "shell", inline: $server_script
		
		server_config.vm.provider "virtualbox" do |vb|
			vb.name = "p1_server"
			vb.memory = "1536"
			vb.cpus = 2
		end
	end

	config.vm.define "server_worker" do |server_worker_config|
		server_worker_config.vm.box = "debian/bookworm64"
		server_worker_config.vm.hostname = "antgabriSW"
		server_worker_config.vm.network "private_network", ip: "192.168.56.111", virtualbox__intnet: "k3s-net"
		server_worker_config.vm.provision "shell", inline: $common_script
		server_worker_config.vm.provision "shell", inline: $install_worker

		server_worker_config.vm.provider "virtualbox" do |vb|
			vb.name = "p1_server_worker"
			vb.memory = "1536"
			vb.cpus = 2
		end
	end
end