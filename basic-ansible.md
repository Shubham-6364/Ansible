### Ansible
ansible is configuration management tool
terraform is infrastructure management too (l
chef is working on pull mechnism
ansible is working on push mechnism

let's crearte 4 instance (1 main instance, 2 developer instance and 1 production instance)
install ansible in main instance
yum install ansible -y
ansible --version
vim /etc/ansible/host
[development]
dev_server1 ansible_host=<ip address>
dev_server2 ansible_host=<ip address>

[production]
prod_server1 ansible_host=<ip address>


#[development:vars]
[all:vars]
ansible_python_interpreter=/usr/bin/python3
ansible_user=ec2-user
ansible_ssh_private_key_file=<path of key>

# now import a .pem file in main server 
mkdir /ansible_key
cd ansible_key
paste the key content in this file
vim new1.pem
change the permission of file
chmod 700 new1.pem
now check the ansible host file is working or not
ansible development -m ping
-m for module
ansible development -a "free -h"
-a for adhoc command
update the packages of development servers
ansible development -a "sudo apt update"
