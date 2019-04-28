Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/bionic64"

  config.vm.define "bastion" do |bastion|
    bastion.vm.hostname = "bastion"
    bastion.vm.network "private_network", ip: "10.30.0.20"
    
    bastion.vm.provision "shell", inline: <<-SHELL
      apt update
      apt-get install -y avahi-daemon libnss-mdns python3-pip
      pip3 install ansible
      echo "dns bootstrap Completed!!"
    SHELL
  end

  config.vm.define "jenkins" do |jenkins|
    jenkins.vm.hostname = "jenkins"
    jenkins.vm.network "private_network", ip: "10.30.0.21"
    jenkins.vm.network "forwarded_port", guest: 8080, host: 8080
    
    jenkins.vm.provision "shell", inline: <<-SHELL
      apt-get install -y avahi-daemon libnss-mdns
      echo "dns bootstrap Completed!!"
    SHELL
  end
  
  config.vm.define "sonar" do |sonar|
    sonar.vm.hostname = "sonar"
    sonar.vm.network "private_network", ip: "10.30.0.22"
    sonar.vm.network "forwarded_port", guest: 9000, host: 9000
    
    sonar.vm.provider "virtualbox" do |vb|
      vb.memory = "2560"
    end
    
    sonar.vm.provision "shell", inline: <<-SHELL
      apt update
      apt-get install -y avahi-daemon libnss-mdns python3-pip
      pip3 install docker
      curl -fsSL https://get.docker.com | sudo sh
      echo "sonar bootstrap Completed!!"
    SHELL
  end  
end