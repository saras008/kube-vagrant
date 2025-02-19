  Vagrant.configure("2") do |config|
    # Use Debian 12 from the "generic" namespace
    config.vm.box = "generic/debian12"

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

    # Worker nodes ...
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
    config.vm.define "jenkins-server" do |jenkins|
      jenkins.vm.hostname = "jenkins-server"
      jenkins.vm.network "private_network", type: "dhcp"
      jenkins.vm.provider "vmware_desktop" do |v|
        v.memory = "4096"
        v.cpus = 2
      end
      jenkins.vm.provision "shell", inline: <<-SHELL
        sudo add-apt-repository universe -y
        sudo apt update -y
        sudo apt install -y openjdk-11-jdk wget git curl gnupg
        curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
        echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] http://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
        sudo apt update -y
        sudo apt install -y jenkins
        sudo systemctl enable --now jenkins
      SHELL
    end
  end
