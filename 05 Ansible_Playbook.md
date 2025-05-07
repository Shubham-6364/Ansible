# **Ansible Playbook**

An Ansible playbook is a YAML file containing a set of plays and tasks to automate infrastructure configuration and deployment. Here's a comprehensive guide to creating effective playbooks.

## **Benefits of Ansible Playbooks**

### **Automation Power**
🎯 **Orchestration** - Defines complete multi-tier deployments  
⏱️ **Workflow Control** - Ordered execution with error handling  
🔄 **Reusable Components** - Roles and includes promote DRY principles  

### **Operational Advantages**
📊 **State Declaration** - Describes "what" not "how" (desired state)  
🔀 **Conditional Logic** - `when:` clauses for environment adaptation  
🛠️ **Modular Design** - Break complex tasks into manageable units  

### **Collaboration Benefits**
📑 **Documentation** - Playbooks serve as living documentation  
👥 **Team Sharing** - Version control friendly (Git)  
🔄 **Change Tracking** - Diff shows infrastructure modifications  

## **1. Basic Playbook Structure**

```yaml
---
- name: Playbook name
  hosts: target_group  # Inventory group or host pattern
  become: yes          # Enable privilege escalation (sudo)
  vars:                # Variables section
    http_port: 80
  tasks:               # List of tasks to execute
    - name: First task description
      ansible.builtin.module_name:
        parameter: value
    - name: Second task
      ansible.builtin.another_module:
        key: value
...
```

## **2. Essential Playbook Components**

### **Targeting Hosts**
```yaml
hosts: webservers           # Group from inventory
hosts: server1.example.com  # Specific host
hosts: all                  # All hosts in inventory
hosts: db*                  # Wildcard pattern
```

### **Privilege Escalation**
```yaml
become: yes                  # Default sudo
become: true                 # Alternative syntax
become_user: postgres        # Specific user
become_method: su            # Alternative to sudo
```

## **3. Common Modules in Playbooks**

### **Package Management**
```yaml
- name: Install Nginx
  ansible.builtin.apt:
    name: nginx
    state: present
    update_cache: yes

- name: Remove old package
  ansible.builtin.yum:
    name: httpd
    state: absent
```

### **File Operations**
```yaml
- name: Copy config file
  ansible.builtin.copy:
    src: files/nginx.conf
    dest: /etc/nginx/nginx.conf
    owner: root
    group: root
    mode: '0644'

- name: Create directory
  ansible.builtin.file:
    path: /var/www/html
    state: directory
    mode: '0755'
```

### **Service Management**
```yaml
- name: Start and enable Nginx
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: yes
```

## **4. Advanced Playbook Features**

### **Conditionals**
```yaml
- name: Install EPEL (RHEL only)
  ansible.builtin.yum:
    name: epel-release
    state: present
  when: ansible_os_family == "RedHat"
```

### **Loops**
```yaml
- name: Install multiple packages
  ansible.builtin.apt:
    name: "{{ item }}"
    state: present
  loop:
    - nginx
    - postgresql
    - python3-pip
```

### **Handlers (Triggered Tasks)**
```yaml
tasks:
  - name: Update Nginx config
    ansible.builtin.template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: restart nginx

handlers:
  - name: restart nginx
    ansible.builtin.service:
      name: nginx
      state: restarted
```

## **5. Complete Example Playbooks**

### **Example 1: Web Server Setup**
```yaml
---
- name: Configure web servers
  hosts: webservers
  become: yes
  vars:
    web_root: /var/www/mysite
    packages:
      - nginx
      - python3
      - python3-pip

  tasks:
    - name: Install required packages
      ansible.builtin.apt:
        name: "{{ item }}"
        state: present
        update_cache: yes
      loop: "{{ packages }}"

    - name: Create web directory
      ansible.builtin.file:
        path: "{{ web_root }}"
        state: directory
        mode: '0755'

    - name: Deploy website content
      ansible.builtin.copy:
        src: ../website_files/
        dest: "{{ web_root }}"
        owner: www-data
        group: www-data

    - name: Enable Nginx configuration
      ansible.builtin.template:
        src: templates/mysite.conf.j2
        dest: /etc/nginx/sites-available/mysite.conf
      notify:
        - restart nginx
        - reload nginx

  handlers:
    - name: restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted

    - name: reload nginx
      ansible.builtin.service:
        name: nginx
        state: reloaded
...
```

### **Example 2: Multi-Role Deployment**
```yaml
---
- name: Base system configuration
  hosts: all
  roles:
    - common
    - { role: users, tags: ['users'] }

- name: Database setup
  hosts: dbservers
  roles:
    - postgresql
    - backup

- name: Web application deployment
  hosts: webservers
  vars:
    app_version: 2.4.1
  roles:
    - nginx
    - { role: app_deploy, app_branch: main }
...
```

## **6. Playbook Execution & Debugging**

### **Running Playbooks**
```bash
# Basic execution
ansible-playbook -i inventory.ini playbook.yml

# With extra variables
ansible-playbook playbook.yml --extra-vars "http_port=8080"

# Limit to specific hosts
ansible-playbook playbook.yml --limit webserver1

# Run with tags
ansible-playbook playbook.yml --tags "config,deploy"
```

### **Debugging Commands**
```bash
# Syntax check
ansible-playbook --syntax-check playbook.yml

# Dry run (check mode)
ansible-playbook --check playbook.yml

# Step-by-step execution
ansible-playbook --step playbook.yml

# Verbose output
ansible-playbook -vvv playbook.yml
```

## **7. Best Practices**

1. **Use roles** for reusable components
2. **Tag tasks** for selective execution
3. **Include variables** in separate files
4. **Document plays** with `name` fields
5. **Validate YAML** before running
6. **Use handlers** for service restarts
7. **Version control** your playbooks

## **8. Real-World Use Cases**

### **Cloud Provisioning**
```yaml
- name: Provision AWS EC2 instances
  hosts: localhost
  connection: local
  tasks:
    - name: Launch instances
      amazon.aws.ec2_instance:
        key_name: mykey
        instance_type: t3.micro
        image_id: ami-123456
        count: 3
        tags:
          Environment: Production
```

### **Configuration Drift Remediation**
```yaml
- name: Enforce SSH configuration
  hosts: all
  tasks:
    - name: Ensure SSH config exists
      ansible.builtin.template:
        src: templates/sshd_config.j2
        dest: /etc/ssh/sshd_config
        validate: /usr/sbin/sshd -t -f %s
      notify: restart sshd
```

### **Zero-Downtime Deployments**
```yaml
- name: Rolling update of web servers
  hosts: webservers
  serial: 2  # Update 2 servers at a time
  tasks:
    - name: Pull latest code
      ansible.builtin.git:
        repo: https://github.com/company/app.git
        dest: /var/www/app
        version: main
      notify: restart app
```
