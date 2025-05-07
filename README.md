# **Ansible**

## **1. What is Ansible?**
Ansible is an **open-source automation tool** for:
- Configuration management
- Application deployment
- Infrastructure provisioning
- Continuous delivery
- **Agentless** (uses SSH/WinRM, no client software needed)

### **Key Features**
✅ **YAML-based** playbooks (human-readable)  
✅ **Idempotent** operations (safe to rerun)  
✅ **Huge module library** (4500+ built-in modules)  
✅ **Multi-platform** (Linux, Windows, network devices)  

## **2. Ansible Architecture**

### **Core Components**
| Component | Purpose |
|-----------|---------|
| **Inventory** | Defines managed nodes (`/etc/ansible/hosts`) |
| **Playbooks** | Automation scripts (`.yml` files) |
| **Modules** | Reusable units of work (`ansible.builtin.copy`) |
| **Plugins** | Extend functionality (callback, connection) |
| **Roles** | Pre-packaged automation bundles |

### **How It Works**
1. Ansible reads **inventory** to find hosts
2. Executes **playbooks** via SSH
3. Runs **modules** on target nodes
4. Returns results to **control node**

## **3. Installation & Setup**

### **Install Ansible**
```bash
# Ubuntu/Debian
sudo apt update
sudo apt install ansible

# RHEL/CentOS
sudo dnf install ansible-core

# macOS
brew install ansible

# Python (pip)
pip install ansible
```

### **Verify Installation**
```bash
ansible --version
```

## **4. Basic Commands**

### **Ad-Hoc Commands**
```bash
ansible all -i inventory -m ping  # Test connectivity
ansible webservers -m apt -a "name=nginx state=present"  # Install package
ansible dbservers -m command -a "free -h"  # Run shell command
```

### **Playbook Execution**
```bash
ansible-playbook -i inventory playbook.yml
```

## **5. Inventory Files**

### **Basic Inventory (`hosts`)**
```ini
[webservers]
web1.example.com
web2.example.com ansible_port=2222

[dbservers]
db1.example.com
db2.example.com

[all:vars]
ansible_user=admin
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### **Dynamic Inventory**
```bash
# AWS EC2 example
ansible-inventory -i aws_ec2.yml --graph
```

## **6. Writing Playbooks**

### **Basic Playbook Structure**
```yaml
---
- name: Configure web servers
  hosts: webservers
  become: yes  # sudo
  tasks:
    - name: Install Nginx
      ansible.builtin.apt:
        name: nginx
        state: present

    - name: Start Nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: yes
```

### **Common Modules**
| Module | Purpose | Example |
|--------|---------|---------|
| `apt`/`yum` | Package management | `name=nginx state=latest` |
| `copy` | File transfer | `src=file.conf dest=/etc/` |
| `template` | Config files with variables | `src=template.j2 dest=/etc/` |
| `user` | User management | `name=test groups=wheel` |
| `file` | File operations | `path=/tmp/test state=directory` |

## **7. Variables & Facts**

### **Using Variables**
```yaml
vars:
  http_port: 80
  web_pkg: nginx

tasks:
  - name: Install {{ web_pkg }}
    apt:
      name: "{{ web_pkg }}"
```

### **Gathering Facts**
```bash
ansible all -m setup  # View all system facts
```
Use facts in playbooks:
```yaml
- name: Print OS
  debug:
    msg: "This is {{ ansible_distribution }} {{ ansible_distribution_version }}"
```

## **8. Roles & Reusability**

### **Directory Structure**
```
roles/
└── webserver/
    ├── tasks/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── templates/
    │   └── nginx.conf.j2
    └── vars/
        └── main.yml
```

### **Using Roles**
```yaml
---
- name: Apply web server role
  hosts: webservers
  roles:
    - webserver
```

## **9. Ansible Vault (Secrets)**
```bash
ansible-vault create secret.yml  # Create encrypted file
ansible-vault edit secret.yml   # Edit encrypted file
ansible-playbook --ask-vault-pass playbook.yml  # Run with vault
```

## **10. Real-World Use Cases**

### **Web Server Deployment**
```yaml
- name: Deploy LAMP stack
  hosts: webservers
  tasks:
    - name: Install Apache
      apt: name=apache2 state=latest
    - name: Install PHP
      apt: name=php state=latest
    - name: Copy website
      copy: src=site/ dest=/var/www/html/
```

### **User Management**
```yaml
- name: Add admin users
  hosts: all
  vars:
    admins:
      - name: alice
        key: "ssh-rsa AAA..."
      - name: bob
        key: "ssh-rsa BBB..."
  tasks:
    - name: Create users
      user:
        name: "{{ item.name }}"
        groups: sudo
        append: yes
      loop: "{{ admins }}"
```

## **11. Best Practices**

1. **Use Roles** for reusable components
2. **Tag tasks** for selective execution:
   ```yaml
   tasks:
     - name: Install packages
       apt: name=nginx
       tags: install
   ```
   Run with: `ansible-playbook --tags install playbook.yml`

3. **Check syntax** before running:
   ```bash
   ansible-playbook --syntax-check playbook.yml
   ```

4. **Use `--check` mode** for dry runs:
   ```bash
   ansible-playbook --check playbook.yml
   ```

## **12. Ansible vs Alternatives**

| Feature | Ansible | Puppet | Chef |
|---------|---------|--------|------|
| **Architecture** | Agentless | Agent-based | Agent-based |
| **Language** | YAML | DSL (Ruby) | Ruby |
| **Learning Curve** | Easy | Moderate | Steep |
| **Speed** | Moderate | Slow | Slow |
| **Community** | Large | Large | Medium |

## **13. Advanced Topics**

### **Dynamic Inventories**
```bash
# AWS EC2 example
ansible-playbook -i aws_ec2.yml playbook.yml
```

### **Custom Modules**
```python
#!/usr/bin/python
from ansible.module_utils.basic import *

def main():
    module = AnsibleModule(argument_spec={})
    module.exit_json(changed=False, message="Hello World!")

if __name__ == '__main__':
    main()
```

### **Performance Tuning**
```ini
# ansible.cfg
[defaults]
forks = 50  # Parallel execution
host_key_checking = False
```

## **14. Getting Help**
```bash
ansible-doc -l  # List all modules
ansible-doc apt  # Show module documentation
```
