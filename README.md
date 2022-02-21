# Ansible role Zabbix-GUI

[![CI Molecule](https://github.com/darexsu/ansible-role-zabbix-gui/actions/workflows/ci.yml/badge.svg)](https://github.com/darexsu/ansible-role-zabbix-gui/actions/workflows/ci.yml)&emsp;![](https://img.shields.io/static/v1?label=idempotence&message=ok&color=success)&emsp;![Ansible Role](https://img.shields.io/ansible/role/d/57603?color=blue&label=downloads)

|  Testing         |  Debian            |  Ubuntu         |  Rocky Linux  | Oracle Linux |
| :--------------: | :----------------: | :-------------: | :-----------: | :----------: |
| Distro release   |  10, 11            | 18.04, 20.04    |       8       |      8       |
| Zabbix version   |  6.0 LTS           |   6.0 LTS       |    6.0 LTS    |  6.0 LTS     |             
                                          
Zabbix Roles: [zabbix-server](https://github.com/darexsu/ansible-role-zabbix-server/), [zabbix-agent](https://github.com/darexsu/ansible-role-zabbix-agent/), [zabbix-gui](https://github.com/darexsu/ansible-role-zabbix-gui/)    

### 1) Install role from Galaxy
```
ansible-galaxy install darexsu.nginx --force
```
### 2) Example playbooks: 
  
  - [full playbook (Zabbix-gui, PHP, Nginx)](#full-playbook-zabbix-gui-php-nginx)  
    - install
      - [Zabbix-GUI](#install-zabbix-gui)
      - [Zabbix-GUI, PHP, Nginx from distro repo](#install-zabbix-gui-from-distro) 
    - config
      - [zabbix.conf.php](#configure-zabbix-conf-php)

Replace dictionary and Merge dictionary (with "hash_behaviour=replace" in ansible.cfg)
```
[host_vars]           [host_vars]
---                   ---
  vars:                 vars:
    dict:                 merge:  <-- # Enable Merge
      a: "value"            dict: 
      b: "value"              a: "value" 
                              b: "value"
```
Role recursive merge:
```
[host_vars]     [current role]    [include_role]
  
  dict:          dict:              dict:
    a: "1" -->     a: "1"    -->      a: "1"
                   b: "2"    -->      b: "2"
                                      c: "3"
    
```
##### Full playbook (Zabbix-gui, PHP, Nginx)
```yaml
- name: Converge
  hosts: all
  become: true

  vars:
    merge:
# Zabbix-GUI --------     
      zabbix_gui:
        enabled: true
        version: "6.0"
        server_name: "Zabbix server"  
        db_type: "MYSQL"
        db_port: "3306"
        db_name: "zabbix"
        db_user: "zabbix"
        db_password: "change_me"
      zabbix_gui_install:
        enabled: true
      zabbix_gui_repo:
        enabled: true
      zabbix_gui_config:
        enabled: true
# PHP --------------
      php:
        enabled: true
        version: "7.4"
      php_repo:
        enabled: true
      php_install:
        enabled: true
      php_fpm: 
        enabled: true
        vars:
          date_timezone: "Europe/Moscow" 
          unix_socket:
            enabled: true
# Nginx ------------
      nginx:
        enabled: true
      nginx_repo:
        enabled: true
      nginx_install:
        enabled: true
      nginx_conf:
        enabled: true
      nginx_virtualhost: 
        enabled: true


  tasks:
  - name: include role darexsu.zabbix_gui
    include_role: 
      name: darexsu.zabbix_gui

```

##### Install Zabbix-gui

```yaml
- name: Converge
  hosts: all
  become: true

  vars:
    merge:
# Zabbix-GUI --------     
      zabbix_gui:
        enabled: true
        version: "6.0"
        server_name: "Zabbix server"  
        db_type: "MYSQL"
        db_port: "3306"
        db_name: "zabbix"
        db_user: "zabbix"
        db_password: "change_me"
      zabbix_gui_install:
        enabled: true
      zabbix_gui_repo:
        enabled: true
      zabbix_gui_config:
        enabled: true


  tasks:
  - name: include role darexsu.zabbix_gui
    include_role: 
      name: darexsu.zabbix_gui

```
##### install zabbix-gui from distro

```yaml
- name: Converge
  hosts: all
  become: true

  vars:
    merge:
# Zabbix-GUI --------        
      zabbix_gui:
        enabled: true
        version: "6.0"
        server_name: "Zabbix server"  
        db_type: "MYSQL"
        db_port: "3306"
        db_name: "zabbix"
        db_user: "zabbix"
        db_password: "change_me"
      zabbix_gui_install:
        enabled: true
      zabbix_gui_repo:
        enabled: true
      zabbix_gui_config:
        enabled: true
# PHP --------------
      php:
        enabled: true
        version: "7.4"
      php_repo:
        enabled: false                    # <-- disable third-party repo
      php_install:
        enabled: true
      php_fpm: 
        enabled: true
        vars:
          date_timezone: "Europe/Moscow" 
          unix_socket:
            enabled: true
# Nginx ------------      
      nginx:
        enabled: true
      nginx_repo:
        enabled: false                    # <-- disable third-party repo
      nginx_install:
        enabled: true
      nginx_conf:
        enabled: true
      nginx_virtualhost: 
        enabled: true

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
# Zabbix-GUI --------     
      zabbix_gui:
        enabled: true
        version: "6.0"
        server_name: "Zabbix server"  
        db_type: "MYSQL"
        db_port: "3306"
        db_name: "zabbix"
        db_user: "zabbix"
        db_password: "change_me"
      zabbix_gui_config:
        enabled: false
        file: "zabbix.conf.php"
        src: "zabbix_gui_gui.j2"
        path:
          debian: "/etc/zabbix/web/"
          redhat: "/etc/zabbix/web/"
        backup: false
        vars:
          db_type: "{{ zabbix_gui.db_type }}"
          db_server: "{{ zabbix_gui.db_server }}"
          db_port: "{{ zabbix_gui.db_port }}"
          db_database: "{{ zabbix_gui.db_name }}"
          db_user: "{{ zabbix_gui.db_user }}"
          db_password: "{{ zabbix_gui.db_password }}"
          db_schema: ""
          db_encryption: "false"
          db_key_file: ""
          db_cert_file: ""
          db_ca_file: ""
          db_verify_host: "false"
          db_sypher_list: ""
          db_vault_url: ""
          db_vault_db_path: ""
          db_vault_token: ""
          db_double_ieee754: "true"
          zbx_server_name: "{{ zabbix_gui.server_name }}"
          image_format_default: "IMAGE_FORMAT_PNG"


  tasks:
  - name: include role darexsu.zabbix_gui
    include_role: 
      name: darexsu.zabbix_gui

```