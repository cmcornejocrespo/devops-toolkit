# HP DevOps Toolkit

## Workshop 3 - Ansible - Azure

### Example 1

- Create bastion vm

```sh
vagrant init ubuntu/bionic64
```

- Setup Azure creds

```sh
vagrant ssh

#install dependencies
sudo apt-get update && sudo apt-get install -y libssl-dev libffi-dev python-dev python-pip

sudo pip install ansible[azure]

mkdir ~/.azure
vi ~/.azure/credentials

#add creds
[default]
subscription_id=<your-subscription_id>
client_id=<security-principal-appid>
secret=<security-principal-password>
tenant=<security-principal-tenant>

#verify configuration

vi test-azure.yml

#add this
---
- hosts: localhost
  connection: local
  tasks:
    - name: Create resource group
      azure_rm_resourcegroup:
        name: ansible-rg-<your-name>
        location: eastus
      register: rg
    - debug:
        var: rg

#run the test
ansible-playbook test-azure.yml
```

### Example 2

- Create jenkins.yml

```yml
- name: Create Azure Jenkins VM
  hosts: localhost
  connection: local
  tasks:
  - name: Create resource group
    azure_rm_resourcegroup:
      name: azure-rg-<your-name>
      location: eastus
  - name: Create virtual network
    azure_rm_virtualnetwork:
      resource_group: azure-rg-<your-name>
      name: vnet-<your-name>
      address_prefixes: "10.0.0.0/16"
  - name: Add subnet
    azure_rm_subnet:
      resource_group: azure-rg-<your-name>
      name: subnet-<your-name>
      address_prefix: "10.0.1.0/24"
      virtual_network: vnet-<your-name>
  - name: Create public IP address
    azure_rm_publicipaddress:
      resource_group: azure-rg-<your-name>
      allocation_method: Static
      name: public-ip-<your-name>
    register: output_ip_address
  - name: Dump public IP for VM which will be created
    debug:
      msg: "The public IP is {{ output_ip_address.state.ip_address }}."
  - name: Create Network Security Group that allows SSH
    azure_rm_securitygroup:
      resource_group: azure-rg-<your-name>
      name: sg-<your-name>
      rules:
        - name: SSH
          protocol: Tcp
          destination_port_range: 22
          access: Allow
          priority: 1001
          direction: Inbound
        - name: jenkins
          protocol: Tcp
          destination_port_range: 8080
          access: Allow
          priority: 1002
  - name: Create virtual network inteface card
    azure_rm_networkinterface:
      resource_group: azure-rg-<your-name>
      name: nic-<your-name>
      virtual_network: vnet-<your-name>
      subnet: subnet-<your-name>
      public_ip_name: public-ip-<your-name>
      security_group: sg-<your-name>
  - name: Create VM
    azure_rm_virtualmachine:
      resource_group: azure-rg-<your-name>
      name: jenkins-vm-<your-name>
      vm_size: Standard_DS1_v2
      admin_username: azureuser
      ssh_password_enabled: false
      ssh_public_keys:
        - path: /home/azureuser/.ssh/authorized_keys
          key_data: <your-key-data>
      network_interfaces: nic-<your-name>
      image:
        offer: UbuntuServer
        publisher: Canonical
        sku: 18.04-LTS
        version: latest
```

### Example 3

- Create sonar.yml

```yml
- name: Create Azure Sonar VM
  hosts: localhost
  connection: local
  tasks:
  - name: Create resource group
    azure_rm_resourcegroup:
      name: azure-rg-<your-name>
      location: eastus
  - name: Create virtual network
    azure_rm_virtualnetwork:
      resource_group: azure-rg-<your-name>
      name: vnet-<your-name>
      address_prefixes: "10.0.0.0/16"
  - name: Add subnet
    azure_rm_subnet:
      resource_group: azure-rg-<your-name>
      name: subnet-<your-name>
      address_prefix: "10.0.1.0/24"
      virtual_network: vnet-<your-name>
  - name: Create public IP address
    azure_rm_publicipaddress:
      resource_group: azure-rg-<your-name>
      allocation_method: Static
      name: public-ip-<your-name>
    register: output_ip_address
  - name: Dump public IP for VM which will be created
    debug:
      msg: "The public IP is {{ output_ip_address.state.ip_address }}."
  - name: Create Network Security Group that allows SSH
    azure_rm_securitygroup:
      resource_group: azure-rg-<your-name>
      name: sg-<your-name>
      rules:
        - name: SSH
          protocol: Tcp
          destination_port_range: 22
          access: Allow
          priority: 1001
          direction: Inbound
        - name: sonar
          protocol: Tcp
          destination_port_range: 9000
          access: Allow
          priority: 1002
  - name: Create virtual network inteface card
    azure_rm_networkinterface:
      resource_group: azure-rg-<your-name>
      name: nic-<your-name>
      virtual_network: vnet-<your-name>
      subnet: subnet-<your-name>
      public_ip_name: public-ip-<your-name>
      security_group: sg-<your-name>
  - name: Create VM
    azure_rm_virtualmachine:
      resource_group: azure-rg-<your-name>
      name: sonar-vm-<your-name>
      vm_size: Standard_DS1_v2
      admin_username: azureuser
      ssh_password_enabled: false
      ssh_public_keys:
        - path: /home/azureuser/.ssh/authorized_keys
          key_data: <your-key-data>
      network_interfaces: nic-<your-name>
      image:
        offer: UbuntuServer
        publisher: Canonical
        sku: 18.04-LTS
        version: latest
```
