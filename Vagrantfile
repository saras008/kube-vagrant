  Vagrant.configure("2") do |config|
    config.vm.box = "bento/debian-12-arm64"

    config.vm.provider "vmware_desktop" do |v|
      v.memory = "2048"
      v.cpus = 1
    end

    config.vm.define "k8s-master" do |master|
      master.vm.hostname = "k8s-master"
      master.vm.network "private_network", type: "dhcp", dns: ["8.8.8.8", "8.8.4.4"]
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
    config.vm.define "jenkins-server" do |jenkins|
      jenkins.vm.hostname = "jenkins-server"
      jenkins.vm.network "private_network", type: "dhcp", dns: ["8.8.8.8", "8.8.4.4"]
      jenkins.vm.provider "vmware_desktop" do |v|
        v.memory = "4096"
        v.cpus = 2
      end
      jenkins.vm.provision "shell", inline: <<-SHELL
        sudo apt update -y
        sudo apt install -y gnupg curl net-tools
        curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
        echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
        sudo apt update -y
        sudo apt install -y openjdk-17-jdk jenkins
        sudo systemctl enable --now jenkins
      SHELL
    end
  end
