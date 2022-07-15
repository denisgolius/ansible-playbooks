Vagrant.configure("2") do |config|
  config.vm.base_mac = nil
  config.vm.synced_folder ".", "/vagrant", disabled: true
#   config.vbguest.auto_update = false

  config.vm.provider "virtualbox" do |vb|
    vb.gui = false
    vb.memory = "1024"
    vb.cpus = 1
    vb.linked_clone = true
  end

  config.vm.provider "libvirt" do |vb|
    vb.memory = "1024"
    vb.cpus = 1
  end

  N = 3
  (1..N).each do |machine_id|
    config.vm.define "load-balancer-1" do |n|
      n.vm.hostname = "load-balancer-1"
      n.vm.network "private_network", ip: "192.168.77.2", libvirt__dhcp_enabled: false
      n.vm.box = "generic/freebsd13"
      n.vm.box_version = "202206.03.0"
    end

    config.vm.define "victoria-#{machine_id}" do |n|
      n.vm.hostname = "victoria-#{machine_id}"
      n.vm.network "private_network", ip: "192.168.77.#{20+machine_id}", libvirt__dhcp_enabled: false
      n.vm.box = "generic/freebsd13"
      n.vm.box_version = "202206.03.0"

      if machine_id == N
        n.vm.provision :ansible do |ansible|
          ansible.limit = "all"
          ansible.playbook = "playbooks/monitoring.yml"
          ansible.groups = {
            "victoria_cluster" => [ "victoria-[1:#{N}]" ],
            "victoria_cluster:vars" => { "if_name" => "eth1", "vminsert_replication_factor" => 2 },
            "load_balancer" => [ "load-balancer-1" ],
            "load_balancer:vars" => { "if_name" => "eth1" }
          }
          ansible.raw_arguments = [ "-D"]
        end
      end
    end
  end
end
