# **YAML for Ansible**

YAML is the backbone of Ansible's automation language. This guide covers Ansible-specific YAML usage with practical examples.
## **Benefits of YAML for Ansible**

### **Human-Friendly Format**
✨ **Readable Syntax** - Clear structure with indentation  
💬 **Comments Support** - `# Explanations can be added`  
📝 **Self-Documenting** - Keys describe their purpose  

### **Technical Advantages**
🔗 **Native Data Types** - Supports strings, numbers, lists, dictionaries  
⚡ **Fast Processing** - Lightweight compared to JSON/XML  
🔄 **Reusability** - Anchors (`&`) and aliases (`*`) reduce duplication  

### **Ansible-Specific Benefits**
🧩 **Perfect for Playbooks** - Natural representation of automation workflows  
🔌 **Module Integration** - Clean parameter passing to Ansible modules  
🌐 **Cross-Platform** - Works the same on all operating systems  

## **1. Ansible YAML Basics**

### **File Structure**
```yaml
---
# All Ansible YAML files start with ---
- name: Descriptive play name  # Playbook level
  hosts: target_group         # Mandatory field
  become: yes                 # Enable privilege escalation
  vars:                       # Variables section
    http_port: 80
  tasks:                      # Action section
    - name: Ensure Nginx is installed
      ansible.builtin.apt:
        name: nginx
        state: present
...
```

## **2. Key Ansible YAML Constructs**

### **Plays (Playbook Level)**
```yaml
- name: Configure web servers
  hosts: webservers
  vars:
    package_name: nginx
  tasks:
    - name: Install web server
      ansible.builtin.apt:
        name: "{{ package_name }}"
        state: latest
```

### **Tasks (Action Level)**
```yaml
tasks:
  - name: Copy config file
    ansible.builtin.copy:
      src: files/nginx.conf
      dest: /etc/nginx/nginx.conf
      owner: root
      group: root
      mode: '0644'
    notify: restart nginx  # Handler trigger
```

### **Handlers (Triggered Tasks)**
```yaml
handlers:
  - name: restart nginx
    ansible.builtin.service:
      name: nginx
      state: restarted
```

## **3. Ansible-Specific YAML Features**

### **Variables**
```yaml
vars:
  - web_user: www-data
  - db_config:
      host: db01.example.com
      port: 5432

tasks:
  - name: Create web directory
    ansible.builtin.file:
      path: "/var/www/{{ web_user }}"
      state: directory
```

### **Conditionals**
```yaml
tasks:
  - name: Install EPEL (RHEL only)
    ansible.builtin.yum:
      name: epel-release
      state: present
    when: ansible_os_family == "RedHat"
```

### **Loops**
```yaml
vars:
  packages:
    - nginx
    - postgresql-client
    - python3-pip

tasks:
  - name: Install multiple packages
    ansible.builtin.apt:
      name: "{{ item }}"
      state: present
    loop: "{{ packages }}"
```

## **4. Advanced Ansible YAML Patterns**

### **Roles Structure**
```yaml
# roles/webserver/tasks/main.yml
- name: Install Nginx
  ansible.builtin.apt:
    name: nginx
    state: present

- name: Ensure Nginx is running
  ansible.builtin.service:
    name: nginx
    state: started
    enabled: yes
```

### **Dynamic Includes**
```yaml
- name: Include OS-specific tasks
  ansible.builtin.include_tasks: "setup_{{ ansible_os_family }}.yml"
```

### **Tags for Selective Execution**
```yaml
tasks:
  - name: Install packages
    ansible.builtin.apt:
      name: nginx
      state: present
    tags: install

  - name: Apply configuration
    ansible.builtin.template:
      src: templates/nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    tags: config
```

## **5. Real-World Ansible YAML Examples**

### **Full LAMP Stack Deployment**
```yaml
---
- name: Deploy LAMP stack
  hosts: webservers
  become: yes
  vars:
    mysql_root_password: "SecurePass123!"
    php_modules: [ 'php', 'php-mysql', 'php-curl' ]
  
  tasks:
    - name: Install Apache
      ansible.builtin.apt:
        name: apache2
        state: present
    
    - name: Install MySQL
      ansible.builtin.apt:
        name: mysql-server
        state: present
    
    - name: Secure MySQL
      ansible.builtin.mysql_user:
        name: root
        password: "{{ mysql_root_password }}"
        priv: "*.*:ALL"
    
    - name: Install PHP
      ansible.builtin.apt:
        name: "{{ item }}"
        state: present
      loop: "{{ php_modules }}"
    
    - name: Copy website files
      ansible.builtin.copy:
        src: ../website/
        dest: /var/www/html/
        owner: www-data
        group: www-data
        mode: '0755'
    
  handlers:
    - name: Restart Apache
      ansible.builtin.service:
        name: apache2
        state: restarted
...
```

### **Multi-Environment Playbook**
```yaml
---
- name: Configure production servers
  hosts: prod_servers
  vars_files:
    - vars/prod.yml
  tasks:
    - name: Deploy production config
      ansible.builtin.template:
        src: templates/prod_config.j2
        dest: /etc/app/config.conf

- name: Configure staging servers
  hosts: staging_servers
  vars_files:
    - vars/staging.yml
  tasks:
    - name: Deploy staging config
      ansible.builtin.template:
        src: templates/staging_config.j2
        dest: /etc/app/config.conf
...
```

## **6. Ansible YAML Best Practices**

1. **Use consistent indentation** (2 spaces)
2. **Quote strings** when they contain `:` or `{{ }}`
   ```yaml
   bad: {{ variable }}_suffix  # May cause parsing issues
   good: "{{ variable }}_suffix"
   ```
3. **Prefer `ansible.builtin` FQCN** (Fully Qualified Collection Names)
   ```yaml
   # Instead of just 'apt'
   ansible.builtin.apt:
   ```
4. **Limit line length** (80-120 chars)
5. **Use `---` and `...`** for clear document boundaries

## **7. Troubleshooting YAML in Ansible**

### **Common Errors**
| Error | Solution |
|-------|----------|
| `mapping values are not allowed` | Check for missing `:` after keys |
| `did not find expected key` | Verify indentation |
| `unhashable type: 'dict'` | Fix malformed dictionaries |

### **Debugging Tools**
```bash
# Validate syntax
ansible-playbook --syntax-check playbook.yml

# Check variable values
ansible-playbook --extra-vars "@vars.yml" playbook.yml --tags debug

# Verbose output
ansible-playbook -vvv playbook.yml
```

## **8. Ansible YAML vs Regular YAML**

| Feature | Ansible YAML | Regular YAML |
|---------|-------------|-------------|
| **Required start/end** | `---` recommended | Optional |
| **Special strings** | `{{ variables }}` | Plain text |
| **Key names** | Ansible modules (`apt`, `copy`) | Arbitrary |
| **Comments** | `#` for documentation | Same |
