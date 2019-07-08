##############################################################################
# Devstack
###############################################################################
Vagrant.configure(2) do |config|
  config.vm.define 'devstack' do |nodeconfig|

    nodeconfig.vm.hostname = 'devstack'
    nodeconfig.vm.box = 'accanto/xenial'
    nodeconfig.vm.box_check_update = false
    nodeconfig.vm.network 'private_network', ip: '192.168.56.130'

    nodeconfig.vm.provider "virtualbox" do |virtualbox|
      virtualbox.name = 'devstack'
      virtualbox.gui = false
      virtualbox.cpus = 4
      virtualbox.memory = 16384
      virtualbox.customize [ "modifyvm", :id, "--uartmode1", "disconnected" ]
      virtualbox.customize [ "modifyvm", :id, "--natnet1", "192.168.222.0/24"]
    end

    nodeconfig.vm.provider :libvirt do |libvirt|
      libvirt.cpus = 4
      libvirt.memory = 16384
      libvirt.nested = true
      libvirt.cpu_mode = 'host-passthrough'
    end

    nodeconfig.vm.provision "ansible_local" do |ansible|
      ansible.become = true
      ansible.inventory_path = "/vagrant/inventory.yml"
      ansible.playbook = "/vagrant/setup-devstack.yml"
    end
  end
end
