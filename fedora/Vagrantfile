# -*- mode: ruby -*-

# If the following variable doesn't match a hash key then all vms will be provisioned
# PROVISION= 'debian'
PROVISION= ''

Vagrant.configure("2") do |config|
	boxes = {
		'fedora' => {
			'image' => 'fedora/33-cloud-base',
			'shell' => 'sudo dnf -y install ansible',
			'ansible' => 'sudo ansible-playbook /vagrant/fedora/fedora.yml',
			'hostname' => 'fedora',
		},
		'centos' => {
			'image' => 'centos/8',
			'shell' => 'sudo dnf -y install epel-release && sudo dnf -y install ansible',
			'ansible' => 'sudo ansible-playbook /vagrant/centos/centos.yml',
			'hostname' => 'centos',
		},
		'ubuntu' => {
			'image' => 'generic/ubuntu2010',
			'shell' => 'sudo apt-get -y install ansible',
			'ansible' => 'sudo ansible-playbook /vagrant/ubuntu/ubuntu.yml',
			'hostname' => 'ubuntu',
		},
		'debian' => {
			'image' => 'debian/buster64',
			'shell' => 'sudo apt-get -y install ansible',
			'ansible' => 'sudo ansible-playbook /vagrant/debian/debian.yml',
			'hostname' => 'debian',
		},
		'arch' => {
			'image' => 'generic/arch',
			'shell' => 'sudo pacman -S ansible --noconfirm && ansible-galaxy collection install community.general',
			'ansible' => 'sudo ansible-playbook /vagrant/arch/arch.yml',
			'hostname' => 'arch',
		}
	}
	def provision_box(box, params)
		box.vm.box = params['image']
		box.ssh.insert_key = 'true'
		box.vm.hostname = params['hostname']
		box.vm.synced_folder ".", "/vagrant/" + params['hostname'], owner: 'vagrant', group: 'vagrant', create: true, type: 'rsync'
		box.vm.provider "virtualbox" do |vb|
			vb.customize ["modifyvm", :id, "--cpus", "2", "--paravirtprovider", "kvm", "--ioapic", "on"]
			vb.memory = 2048
		end
		box.vm.provider :libvirt do |v|
			v.memory = 2048
			v.cpus = 2
			v.username = 'root'
		end
		box.vm.provision "shell", preserve_order:true, inline: params['shell']
		box.vm.provision "shell", preserve_order:true, inline: params['ansible'] + '|| true'
		if Vagrant.has_plugin?("vagrant-proxyconf")
			box.proxy.http     = ENV["http_proxy"]
			box.proxy.https    = ENV["https_proxy"]
			box.proxy.no_proxy = ENV["no_proxy"]
		end
	end

	params = boxes[PROVISION]
	if boxes.key?(PROVISION)
		puts "Provisioning " + PROVISION
		puts "===================="
		config.vm.define PROVISION do |box|
			provision_box(box, params)
		end
	else
		puts "Provisioning all VMs"
		puts "===================="
		boxes.each do | name, params |
			config.vm.define name do |box|
				provision_box(box, params)
			end
		end
	end
end
