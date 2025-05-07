# Install apache in all servers by create and run a playbook
- create .yml file
```
vim werbserver_play.yml
```
- Put the code into it  
```
---
- name: Install and configure httpd on Red Hat
  hosts: all
  become: yes
  tasks:
    - name: Install httpd package
      ansible.builtin.package:
        name: httpd
        state: present
      when: ansible_os_family == "RedHat"

    - name: Ensure httpd service is enabled and running
      ansible.builtin.service:
        name: httpd
        state: started
        enabled: yes

    - name: Create a simple index.html page
      ansible.builtin.copy:
        content: "<h1>Welcome to Apache on Red Hat!</h1>"
        dest: /var/www/html/index.html
        owner: apache
        group: apache
        mode: '0644'
  ```
- Run this play book
```
ansible-playbook -v webserver_play.yml
```

# now your all server have the apache package
