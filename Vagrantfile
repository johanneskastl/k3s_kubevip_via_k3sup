Vagrant.configure("2") do |config|

  ###################################################################################
  # define number of agents
  W = 1

  # provision W VMs as agents
  (1..W).each do |i|

    # name the VMs
    config.vm.define "k3sagent#{i}" do |node|

      # which image to use
      node.vm.box = "opensuse/Leap-15.2.x86_64"

      # sizing of the VMs
      node.vm.provider "libvirt" do |lv|
        lv.random_hostname = true
        lv.memory = 1024
        lv.cpus = 2
      end

      # set the hostname
      node.vm.hostname = "k3sagent#{i}"

    end # config.vm.define agents

  end # each-loop agents

  ###############################################

  # define number of server nodes
  M = 3

  # provision N VMs as server nodes
  (1..M).each do |i|

    # name the VMs
    config.vm.define "k3sserver#{i}" do |node|

      # which image to use
      node.vm.box = "opensuse/Leap-15.2.x86_64"

      # sizing of the VMs
      node.vm.provider "libvirt" do |lv|
        lv.random_hostname = true
        lv.memory = 1024
        lv.cpus = 2
      end

      # set the hostname
      node.vm.hostname = "k3sserver#{i}"

      # if this is the last machine
      if i == M

        #################################################
        # only target one node to not run ansible twice
        # but do not limit this to this node (set ansible.limit to all)
        node.vm.provision "ansible" do |ansible|
          ansible.compatibility_mode = "2.0"
          ansible.limit = "all"
          ansible.groups = {
            "k3sservers"  => [ "k3sserver1", "k3sserver2", "k3sserver3" ],
            "k3sagents"  => [ "k3sagent1" ]
          }
          ansible.playbook = "ansible/playbook-vagrant.yml"

        end # node.vm.provision
      end # if-condition last machine

    end # config.vm.define server nodes

  end # each-loop server nodes

end # Vagrant.configure
