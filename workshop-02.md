# HP DevOps Toolkit

## Workshop 2 - Vagrant

### Setup

- Install [vagrant](https://www.vagrantup.com/downloads.html) (requires [Virtualbox](https://www.virtualbox.org/wiki/Downloads))
- Download ubuntu 18.04 image

```sh
vagrant box add ubuntu/bionic64
```

### Create Jenkins vm

- Create a Vagrantfile init template

```sh
vagrant init ubuntu/bionic64
```

- Check vm access

```sh
vagrant ssh
```

- Open Vagrantfile, add vm name, update network settings to have dhcp enabled

```sh
config.vm.hostname = 'jenkins'
config.vm.network "private_network", type: "dhcp"
```

- Save and reload config. Ssh into vm, install Jenkins and retrieve IP

```sh
# validate Vagrantfile
vagrant validate

vagrant ssh
$ sudo apt update && sudo apt install openjdk-8-jre-headless -y
$ wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
# start jenkins nohup
$ nohup java -jar jenkins.war </dev/null &>/dev/null &
# get vm ip
$ ip a
```

- Open browser and access Jenkins IP:8080

- Open Vagrantfile and add port-forwarding

```sh
config.vm.network "forwarded_port", guest: 8080, host: 8080
```

- Reload config. Restart Jenkins

```sh
# validate Vagrantfile
vagrant validate

vagrant reload
vagrant ssh
$ nohup java -jar jenkins.war </dev/null &>/dev/null &
```

- Open browser and access Jenkins localhost:8080

Provisioners in Vagrant allow you to automatically install software, alter configurations, and more on the machine as part of the vagrant up process.

- Now add the provisioner section inline [shell](https://www.vagrantup.com/docs/provisioning/shell.html#inline-scripts)

```sh
config.vm.provision "shell", privileged: false, inline: <<-SHELL
  sudo apt update &>/dev/null && sudo apt install openjdk-8-jdk-headless -y &>/dev/null
  wget --quiet http://mirrors.jenkins.io/war-stable/latest/jenkins.war
  nohup java -jar jenkins.war </dev/null &>/dev/null &
  echo "Completed!!"
SHELL
```

- Recreate the vm

```sh
# validate Vagrantfile
vagrant validate

vagrant destroy --force
vagrant up
```

### Create Sonar vm

- Create a Vagrantfile init template

```sh
vagrant init ubuntu/bionic64
```

- Check vm access

```sh
vagrant ssh
```

- Open Vagrantfile, add vm name, update network settings to have dhcp enabled and port-forwarding

```sh
config.vm.hostname = 'sonar'
config.vm.network "private_network", type: "dhcp"
config.vm.network "forwarded_port", guest: 9000, host: 9000
```

- Increase the memory of the vm to 2GB

```sh
sonar.vm.provider "virtualbox" do |vb|
  vb.memory = "2048"
end
```

- Now add the provisioner section via inline shell

```sh
sonar.vm.provision "shell", inline: <<-SHELL
  apt-get install -y avahi-daemon libnss-mdns
  echo "dns bootstrap Completed!!"
SHELL
```

- Now add the provisioner section using [docker](https://www.vagrantup.com/docs/provisioning/docker.html)

```sh
sonar.vm.provision "docker" do |d|
  d.run "sonarqube",
    args: "-p 9000:9000"
end
```

- Test the vm's

```sh
# validate Vagrantfile
vagrant validate

vagrant up
```

- Open browser and access Sonarqube localhost:9000

### Create Multi-Machine setup

- Create a Vagrantfile init template

´´´sh
vagrant init ubuntu/bionic64
´´´

- Add the following snippets inside the VagrantFile

```sh
config.vm.define "jenkins" do |jenkins|
  jenkins.vm.hostname = "jenkins"
  jenkins.vm.network "private_network", type: "dhcp"
  jenkins.vm.network "forwarded_port", guest: 8080, host: 8080

  jenkins.vm.provision "shell", privileged: false, inline: <<-SHELL
    sudo apt update &>/dev/null && sudo apt install openjdk-8-jdk-headless avahi-daemon libnss-mdns -y &>/dev/null
    wget --quiet http://mirrors.jenkins.io/war-stable/latest/jenkins.war
    nohup java -jar jenkins.war </dev/null &>/dev/null &
    echo "Jenkins bootstrap Completed!!"
  SHELL

end
```

```sh
config.vm.define "sonar" do |sonar|
  sonar.vm.hostname = "sonar"
  sonar.vm.network "private_network", type: "dhcp"
  sonar.vm.network "forwarded_port", guest: 9000, host: 9000

  sonar.vm.provider "virtualbox" do |vb|
    vb.memory = "2560"
  end

  sonar.vm.provision "shell", inline: <<-SHELL
    apt-get install -y avahi-daemon libnss-mdns
    echo "dns bootstrap Completed!!"
  SHELL

  sonar.vm.provision "docker" do |d|
    d.run "sonarqube",
      args: "-p 9000:9000"
  end
end
```

- We need to create a Jenkins job as we did before and update the sonar step. Scroll down to Pipeline section and paste [this](https://raw.githubusercontent.com/cmcornejocrespo/devops-training-material/develop/jenkins/Jenkinsfile) or this:

```yml

node {
    def mvnHome = tool 'M3'

    stage('Preparation') { // for display purposes
        // Get some code from a GitHub repository
        git url: 'https://github.com/cmcornejocrespo/webinar-bat-desk.git', branch: 'feature/jbcnconf-2017'
        // Get the Maven tool.
        // ** NOTE: This 'M3' Maven tool must be configured
        // **       in the global configuration.

    }
    stage('Build') {
        // Run the maven build
        sh "'${mvnHome}/bin/mvn' clean package"
    }

    stage('Run unit tests') {
        // Run the maven test
        sh "'${mvnHome}/bin/mvn' test"
    }

    stage('Run Sonar reports') {
        // Run the maven test
        echo "something"
    }

    stage('Run Integration Tests') {
        // Run integration tests
        sh "'${mvnHome}/bin/mvn' clean verify -Pintegration-tests"
    }

    stage('Results') {
        junit '**/target/surefire-reports/TEST-*.xml'
        archiveArtifacts 'bat-desk-common/target/*.jar'
    }
}
```

- Change the sonar step

```sh
sh "'${mvnHome}/bin/mvn' clean install sonar:sonar -Dsonar.host.url=http://sonar.local:9000 -Psonar-coverage"
```