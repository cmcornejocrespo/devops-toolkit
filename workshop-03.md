# HP DevOps Toolkit

## Workshop 3 - Ansible

### SSH keys are your friends

```sh
ssh-agent bash
ssh-add ~/.ssh/id_rsa
```

### Manage your inventory in simple text files (yml)

```sh
[webservers]
www1.example.com
www2.example.com

[dbservers]
db0.example.com
db1.example.com
```

### Ansible For Ad Hoc Parallel Task Execution

- [modules](https://docs.ansible.com/ansible/latest/modules/modules_by_category.html) index

```sh
ansible all -m ping
ansible db0.example.com -m yum -a "name=httpd state=installed"
ansible webservers -a "/usr/sbin/reboot"
```

### Playbooks are Ansible’s configuration, deployment, and orchestration language

```yml
production                # inventory file for production servers
staging                   # inventory file for staging environment

group_vars/
   group1.yml             # here we assign variables to particular groups
   group2.yml
host_vars/
   hostname1.yml          # here we assign variables to particular systems
   hostname2.yml

site.yml                  # master playbook
webservers.yml            # playbook for webserver tier
dbservers.yml             # playbook for dbserver tier

roles/
    common/               # this hierarchy represents a "role"
        tasks/            #
            main.yml      #  <-- tasks file can include smaller files if warranted
        handlers/         #
            main.yml      #  <-- handlers file
        templates/        #  <-- files for use with the template resource
            ntp.conf.j2   #  <------- templates end in .j2
        files/            #
            bar.txt       #  <-- files for use with the copy resource
            foo.sh        #  <-- script files for use with the script resource
        vars/             #
            main.yml      #  <-- variables associated with this role
        defaults/         #
            main.yml      #  <-- default lower priority variables for this role

```

### Workshop 03 - Ansible

- Create Vagrantfile

```sh
vagrant init ubuntu/bionic64
```

- Create Multi-Machine setup

```sh
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
```

- Start vagrant

```sh
vagrant up
```

- Run ping modules

```sh
vagrant ssh bastion

# check hostfile
cd /vagrant/workshop-03
ansible -i hosts all -m ping
```

- Run Jenkins playbook

Check tutor instructions

- Run Sonar playbook

Check tutor instructions

- Run jenkins pipeline with sonar integration

```sh
sh "'${mvnHome}/bin/mvn' clean install sonar:sonar -Dsonar.host.url=http://sonar.local:9000 -Psonar-coverage"
```