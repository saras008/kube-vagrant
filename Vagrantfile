Vagrant.configure("2") do |config|
  config.vm.box = "bento/ubuntu-22.04-arm64"
  config.vm.provider "vmware_desktop" do |v|
    v.memory = "2048"
    v.cpus = 1
  end
  
  config.vm.define "k8s-master" do |master|
    master.vm.hostname = "k8s-master"
    master.vm.network "private_network", type: "dhcp"
    master.vm.provider "vmware_desktop" do |v|
      v.memory = "4096"
      v.cpus = 2
    end
    master.vm.provision "shell", inline: <<-SHELL
      sudo apt-get update -y
    SHELL
  end
  
  (1..2).each do |i|
    config.vm.define "k8s-worker#{i}" do |worker|
      worker.vm.hostname = "k8s-worker#{i}"
      worker.vm.network "private_network", type: "dhcp"
      worker.vm.provider "vmware_desktop" do |v|
        v.memory = "2048"
        v.cpus = 1
      end
      worker.vm.provision "shell", inline: <<-SHELL
        sudo apt-get update -y
      SHELL
    end
  end
end
