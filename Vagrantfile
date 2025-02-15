# Vagrantfile for Kubernetes Lab on Apple M1 with Ubuntu 22.04 (ARM64)
Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-22.04-arm64"  # Ensure the box supports ARM64 for VMware Fusion
  config.vm.provider "vmware_desktop" do |v|
    v.memory = "2048"  # Reduced memory usage
    v.cpus = 1  # Reduced CPU allocation
  end
  
  # Define the control plane node
  config.vm.define "k8s-master" do |master|
    master.vm.hostname = "k8s-master"
    master.vm.network "private_network", type: "dhcp"
    master.vm.provider "vmware_desktop" do |v|
      v.memory = "4096"  # Reduced from 8192
      v.cpus = 2  # Reduced from 4
    end
    master.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update -y
    SHELL
  end
  
  # Define worker nodes
  (1..2).each do |i|
    config.vm.define "k8s-worker#{i}" do |worker|
      worker.vm.hostname = "k8s-worker#{i}"
      worker.vm.network "private_network", type: "dhcp"
      worker.vm.provider "vmware_desktop" do |v|
        v.memory = "2048"  # Reduced from 4096
        v.cpus = 1  # Reduced from 2
      end
      worker.vm.provision "shell", inline: <<-SHELL
        sudo apt-get update -y
      SHELL
    end
  end
end
