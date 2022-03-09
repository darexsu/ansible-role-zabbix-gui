# Ansible role Zabbix-GUI

[![CI Molecule](https://github.com/darexsu/ansible-role-zabbix-gui/actions/workflows/ci.yml/badge.svg)](https://github.com/darexsu/ansible-role-zabbix-gui/actions/workflows/ci.yml)&emsp;![](https://img.shields.io/static/v1?label=idempotence&message=ok&color=success)&emsp;![Ansible Role](https://img.shields.io/ansible/role/d/58156?color=blue&label=downloads)

|  Testing         |  Zabbix repo       |
| :--------------: | :----------------: |
| Debian 11        |  Zabbix  6.0 lts   |
| Debian 10        |  Zabbix  6.0 lts   |
| Ubuntu 20.04     |  Zabbix  6.0 lts   |
| Ubuntu 18.04     |  Zabbix  6.0 lts   |
| Oracle Linux 8   |  Zabbix  6.0 lts   |
| Rocky Linux 8    |  Zabbix  6.0 lts   |     
                                          
Zabbix Roles: [zabbix-server](https://github.com/darexsu/ansible-role-zabbix-server/), [zabbix-agent](https://github.com/darexsu/ansible-role-zabbix-agent/), [zabbix-gui](https://github.com/darexsu/ansible-role-zabbix-gui/)
Compatible with Roles: [Nginx](https://github.com/darexsu/ansible-role-nginx), [Apache](https://github.com/darexsu/ansible-role-apache), [PHP](https://github.com/darexsu/ansible-role-php)

### 1) Install role from Galaxy
```
ansible-galaxy install darexsu.nginx --force
```
### 2) Example playbooks: 
  
  - [full playbook (Zabbix-gui, PHP, Nginx)](#full-playbook-zabbix-gui-php-nginx)
  - [full playbook (Zabbix-gui, PHP, Apache)](#full-playbook-zabbix-gui-php-apache)  
    - install
      - [Zabbix-GUI](#install-zabbix-gui)     
    - config
      - [zabbix.conf.php](#configure-zabbixconfphp)

Replace or Merge dictionaries (with "hash_behaviour=replace" in ansible.cfg):
```
# Replace             # Merge
---                   ---
  vars:                 vars:
    dict:                 merge:
      a: "value"            dict: 
      b: "value"              a: "value" 
                              b: "value"

# How does merge work?
Your vars [host_vars]  -->  default vars [current role] --> default vars [include role]
  
  dict:          dict:              dict:
    a: "1" -->     a: "1"    -->      a: "1"
                   b: "2"    -->      b: "2"
                                      c: "3"
    
```
##### Full playbook (Zabbix-gui, PHP, Nginx)
```yaml
- hosts: all
  become: true

  vars:
    merge:
      # Zabbix_GUI
      zabbix_gui:
        enabled: true
        src: "third_party"
        version: "6.0"
        server_name: "Zabbix server"
        db_type: "MYSQL"
        db_port: "3306"
        db_server: "localhost"
        db_name: "zabbix"
        db_user: "zabbix"
        db_password: "change_me"
      # Zabbix_GUI -> install
      zabbix_gui_install:
        enabled: true
        packages:
          Debian: [zabbix-frontend-php]
          RedHat: [zabbix-web-mysql]
        dependencies:
          Debian: [apt-transport-https, ca-certificates, gnupg2, lsb-release]
          RedHat: []
      # Zabbix_GUI -> config
      zabbix_gui_config:
        enabled: true
        file: "zabbix.conf.php"
        src: "zabbix_gui__zabbix.conf.php.j2"
        backup: false
        vars:
          db_type: "{{ zabbix_gui.db_type }}"
          db_server: "{{ zabbix_gui.db_server }}"
          db_port: "{{ zabbix_gui.db_port }}"
          db_database: "{{ zabbix_gui.db_name }}"
          db_user: "{{ zabbix_gui.db_user }}"
          db_password: "{{ zabbix_gui.db_password }}"
      
      # PHP
      php:
        enabled: true
        version: "7.4"
        src: "third_party"
      # PHP -> install
      php_install:
        enabled: true
        packages: [bcmath, ctype, common, fpm, cli, gd, ldap, curl, xml, xmlrpc, mbstring, mysql, zip]
      # PHP -> config -> php-fpm
      php_fpm:
        zabbix_conf:
          enabled: true
          file: "zabbix.conf"
          state: "present"
          src: "../../darexsu.zabbix_gui/templates/zabbix_gui__php.j2"
          backup: false
          vars:
            webserver_user: "www-data"
            webserver_group: "www-data"
            pm: "dynamic"
            pm_max_children: "10"
            pm_start_servers: "5"
            pm_min_spare_servers: "5"
            pm_max_spare_servers: "5"
            pm_max_requests: "500"
            date_timezone: "Europe/Moscow"
            tcp_ip_socket:
              enabled: false
              listen: "127.0.0.1:9000"
            unix_socket:
              enabled: true
              file: "php{{ php.version }}-fpm-zabbix.sock"
              user: "www-data"
              group: "www-data"

      # Apache
      apache:
        enabled: true
        service:
          enabled: false
          state: "stopped"

      # Nginx
      nginx:
        enabled: true
        src: "distribution"
      # Nginx -> install
      nginx_install:
        enabled: true
        packages: [nginx]
        service:
          enabled: true
          state: "started"
      # Nginx -> config
      nginx_conf:
        enabled: true
        vars:
          user: "www-data"
      # Nginx -> config -> virtualhost.conf
      nginx_virtualhost:
        default_conf:
          enabled: true
          file: "default.conf"
          state: "absent"
        zabbix_conf:
          enabled: true
          file: "zabbix.conf"
          state: "present"
          src: "../../darexsu.zabbix_gui/templates/zabbix_gui__nginx.j2"
          backup: false
          vars:
            listen_port: "80"
            listen_ipv6: false
            server_name: "localhost"
            root: "/usr/share/zabbix/"
            access_log: false
            error_log: false
            tcp_ip_socket:
              enabled: false
              listen: "127.0.0.1:9000"
            unix_socket:
              enabled: true
              file: "php{{ php.version }}-fpm-zabbix.sock"

  tasks:
    - name: include role darexsu.zabbix_gui
      include_role:
        name: darexsu.zabbix_gui

```
##### Full playbook (Zabbix-gui, PHP, Apache)
```yaml
- hosts: all
  become: true

  vars:
    merge:
      # Zabbix_GUI
      zabbix_gui:
        enabled: true
        src: "third_party"
        version: "6.0"
        server_name: "Zabbix server"
        db_type: "MYSQL"
        db_port: "3306"
        db_server: "localhost"
        db_name: "zabbix"
        db_user: "zabbix"
        db_password: "change_me"
      # Zabbix_GUI -> install
      zabbix_gui_install:
        enabled: true
        packages:
          Debian: [zabbix-frontend-php, zabbix-apache-conf]
          RedHat: [zabbix-web-mysql, zabbix-apache-conf]
        dependencies:
          Debian: [apt-transport-https, ca-certificates, gnupg2, lsb-release]
          RedHat: []
      # Zabbix_GUI -> config
      zabbix_gui_config:
        enabled: true
        file: "zabbix.conf.php"
        src: "zabbix_gui__zabbix.conf.php.j2"
        backup: false
        vars:
          db_type: "{{ zabbix_gui.db_type }}"
          db_server: "{{ zabbix_gui.db_server }}"
          db_port: "{{ zabbix_gui.db_port }}"
          db_database: "{{ zabbix_gui.db_name }}"
          db_user: "{{ zabbix_gui.db_user }}"
          db_password: "{{ zabbix_gui.db_password }}"

      # PHP
      php:
        enabled: true
        version: "7.4"
        src: "third_party"
      # PHP -> install
      php_install:
        enabled: true
        packages: [bcmath, ctype, common, fpm, cli, gd, ldap, curl, xml, xmlrpc, mbstring, mysql, zip]
      # PHP -> config
      php_fpm:
        zabbix_conf:
          enabled: true
          file: "zabbix.conf"
          state: "present"
          src: "../../darexsu.zabbix_gui/templates/zabbix_gui__php.j2"
          backup: false
          vars:
            webserver_user: "www-data"
            webserver_group: "www-data"
            pm: "dynamic"
            pm_max_children: "10"
            pm_start_servers: "5"
            pm_min_spare_servers: "5"
            pm_max_spare_servers: "5"
            pm_max_requests: "500"
            date_timezone: "Europe/Moscow"
            tcp_ip_socket:
              enabled: false
              listen: "127.0.0.1:9000"
            unix_socket:
              enabled: true
              file: "php{{ php.version }}-fpm-zabbix.sock"
              user: "www-data"
              group: "www-data"

      # Apache
      apache:
        enabled: true
        service:
          enabled: true
          state: "started"
      # Apache -> install
      apache_install:
        enabled: true
      # Apache -> config -> virtualhost.conf
      apache_virtualhost:
      # Apache -> config -> delete default virtual_host
        default_conf:
          enabled: true
          state: "absent"
      # Apache -> config -> add zabbix virtual_host
        zabbix_conf:
          enabled: true
          file: "zabbix.conf"
          state: "present"
          src: "../../darexsu.zabbix_gui/templates/zabbix_gui__apache.j2"
          backup: false
          vars:
            ip: "*"
            port: "80"
            ServerName: "www.example.com"
            ServerAdmin: "webmaster@localhost"
            DocumentRoot: "/usr/share/zabbix/"


  tasks:
  - name: include role darexsu.zabbix_gui
    include_role: 
      name: darexsu.zabbix_gui

```

##### Install Zabbix-gui

```yaml
- hosts: all
  become: true

  vars:
    merge:
      # Zabbix_GUI
      zabbix_gui:
        enabled: true
        src: "third_party"
        version: "6.0"
        server_name: "Zabbix server"
        db_type: "MYSQL"
        db_port: "3306"
        db_server: "localhost"
        db_name: "zabbix"
        db_user: "zabbix"
        db_password: "change_me"
      # Zabbix_GUI -> install
      zabbix_gui_install:
        enabled: true
        packages:
          Debian: [zabbix-frontend-php]
          RedHat: [zabbix-web-mysql]
        dependencies:
          Debian: [apt-transport-https, ca-certificates, gnupg2, lsb-release]
          RedHat: []
      # Zabbix_GUI -> config
      zabbix_gui_config:
        enabled: true
        file: "zabbix.conf.php"
        src: "zabbix_gui__zabbix.conf.php.j2"
        backup: false
        vars:
          db_type: "{{ zabbix_gui.db_type }}"
          db_server: "{{ zabbix_gui.db_server }}"
          db_port: "{{ zabbix_gui.db_port }}"
          db_database: "{{ zabbix_gui.db_name }}"
          db_user: "{{ zabbix_gui.db_user }}"
          db_password: "{{ zabbix_gui.db_password }}"

  tasks:
  - name: include role darexsu.zabbix_gui
    include_role: 
      name: darexsu.zabbix_gui

```
##### configure zabbix.conf.php
```yaml
- name: Converge
  hosts: all
  become: true

  vars:
    merge:
      # Zabbix_GUI
      zabbix_gui:
        enabled: true
        src: "third_party"
        version: "6.0"
        server_name: "Zabbix server"
        db_type: "MYSQL"
        db_port: "3306"
        db_server: "localhost"
        db_name: "zabbix"
        db_user: "zabbix"
        db_password: "change_me"
      # Zabbix_GUI -> config
      zabbix_gui_config:
        enabled: true
        file: "zabbix.conf.php"
        src: "zabbix_gui__zabbix.conf.php.j2"
        backup: false
        vars:
          db_type: "{{ zabbix_gui.db_type }}"
          db_server: "{{ zabbix_gui.db_server }}"
          db_port: "{{ zabbix_gui.db_port }}"
          db_database: "{{ zabbix_gui.db_name }}"
          db_user: "{{ zabbix_gui.db_user }}"
          db_password: "{{ zabbix_gui.db_password }}"

  tasks:
  - name: include role darexsu.zabbix_gui
    include_role: 
      name: darexsu.zabbix_gui

```
