# Ansible role Zabbix-GUI

[![CI Molecule](https://github.com/darexsu/ansible-role-zabbix-gui/actions/workflows/ci.yml/badge.svg)](https://github.com/darexsu/ansible-role-zabbix-gui/actions/workflows/ci.yml)&emsp;![](https://img.shields.io/static/v1?label=idempotence&message=ok&color=success)&emsp;![Ansible Role](https://img.shields.io/ansible/role/d/58156?color=blue&label=downloads)

  - Role:
      - [platforms](#platforms)
      - [install](#install)
      - [requirements](#requirements)
      - [relative](#relative)
      - [behaviour](#behaviour)
  - Playbooks (short version):
      - [install and configure: Zabbix-gui, PHP, Nginx, FirewallD](#install-and-configure-zabbix-gui-php-nginx-short-version)
      - [install and configure: Zabbix-gui, PHP, Apache, FirewallD](#install-and-configure-zabbix-gui-php-apache-short-version)
        - [install: Zabbix-GUI](#install-zabbix-gui-short-version)
        - [configure: zabbix.conf.php](#configure-zabbixconfphp-short-version)  
  - Playbooks (full version):
      - [install and configure: Zabbix-gui, PHP, Nginx, FirewallD](#install-and-configure-zabbix-gui-php-nginx-full-version)
      - [install and configure: Zabbix-gui, PHP, Apache, FirewallD](#install-and-configure-zabbix-gui-php-apache-full-version)
        - [install: Zabbix-GUI](#install-zabbix-gui-full-version)
        - [configure: zabbix.conf.php](#configure-zabbixconfphp-full-version)  

### Platforms

|  Testing         |  Zabbix repo       |  Ready for use     |
| :--------------: | :----------------: | :----------------: |
| Debian 11        |  Zabbix  6.0 lts   |:heavy_check_mark:  |
| Debian 10        |  Zabbix  6.0 lts   |:heavy_check_mark:  |
| Ubuntu 20.04     |  Zabbix  6.0 lts   |:heavy_check_mark:  |
| Ubuntu 18.04     |  Zabbix  6.0 lts   |:heavy_check_mark:  |
| Oracle Linux 8   |  Zabbix  6.0 lts   |:heavy_check_mark:  |
| Rocky Linux 8    |  Zabbix  6.0 lts   |:heavy_check_mark:  |  

### Install
```
ansible-galaxy install darexsu.zabbix_gui --force
```

### Requirements

dependencies will automatically be installed: [Nginx](https://github.com/darexsu/ansible-role-nginx), [Apache](https://github.com/darexsu/ansible-role-apache), [PHP](https://github.com/darexsu/ansible-role-php), [FirewallD](https://github.com/darexsu/ansible-role-firewalld)

### Relative

Another Zabbix roles: [Zabbix-server](https://github.com/darexsu/ansible-role-zabbix-server/), [Zabbix-agent](https://github.com/darexsu/ansible-role-zabbix-agent/), [Zabbix-gui](https://github.com/darexsu/ansible-role-zabbix-gui/)

### Behaviour

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
##### Install and configure: Zabbix-GUI, PHP, Nginx (short version)
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
      # Zabbix_GUI -> config
      zabbix_gui_config:
        enabled: true

      # PHP
      php:
        enabled: true
        version: "7.4"
        src: "third_party"
      # PHP -> install
      php_install:
        enabled: true
      # PHP -> config
      php_fpm:
        zabbix_conf:
          enabled: true

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
      # Nginx -> config -> nginx.conf
      nginx_conf:
        enabled: true
      # Nginx -> config -> virtualhost.conf
      nginx_virtualhost:
        default_conf:
          enabled: true
          file: "default.conf"
          state: "absent"
      # Nginx -> config -> virtualhost.conf
        zabbix_conf:
          enabled: true

      # FirewallD
      firewalld:
        enabled: true
      # FirewallD -> rules
      firewalld_rules:
        service_http:
          enabled: true
        service_https:
          enabled: true

  tasks:
    - name: include role darexsu.zabbix_gui
      include_role:
        name: darexsu.zabbix_gui

```
##### Install and configure: Zabbix-GUI, PHP, Apache (short version)
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
      # Zabbix_GUI -> config
      zabbix_gui_config:
        enabled: true
      
      # PHP
      php:
        enabled: true
        version: "7.4"
        src: "third_party"
      # PHP -> install
      php_install:
        enabled: true
      # PHP -> config
      php_fpm:
        zabbix_conf:
          enabled: true

      # Apache
      apache:
        enabled: true
      # Apache -> install
      apache_install:
        enabled: true
      # Apache -> config -> modules
      apache_modules:
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

      # FirewallD
      firewalld:
        enabled: true
      # FirewallD -> rules
      firewalld_rules:
        service_http:
          enabled: true
        service_https:
          enabled: true

  tasks:
    - name: include role darexsu.zabbix_gui
      include_role: 
        name: darexsu.zabbix_gui

```

##### Install: Zabbix-GUI (short version)

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

  tasks:
    - name: include role darexsu.zabbix_gui
      include_role: 
        name: darexsu.zabbix_gui

```
##### Configure: zabbix.conf.php (short version)
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
##### Install and configure: Zabbix-GUI, PHP, Nginx (full version)
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
      
      # PHP
      php:
        enabled: true
        version: "7.4"
        src: "third_party"
        service:
          enabled: true
          state: "started"
      # PHP -> install
      php_install:
        enabled: true
        packages: [bcmath, ctype, common, fpm, cli, gd, ldap, curl, xml, xmlrpc, mbstring, mysql, zip]
        dependencies:
          Debian: [apt-transport-https, ca-certificates, curl, gnupg2, lsb-release]
          RedHat: []
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
          enabled: false
          state: "stopped"

      # Nginx
      nginx:
        enabled: true
        src: "distribution"
        service:
          enabled: true
          state: "started"
      # Nginx -> install
      nginx_install:
        enabled: true
        packages: [nginx]
        dependencies:
          Debian: [apt-transport-https, ca-certificates, curl, gnupg2, lsb-release, debian-archive-keyring]
          RedHat: []
      # Nginx -> config -> nginx.conf
      nginx_conf:
        enabled: true
        file: "nginx.conf"
        src: "nginx__conf.j2"
        backup: false
        vars:
          user: "www-data"
          worker_processes: "auto"
          error_log: "/var/log/nginx/error.log notice"
          pidfile: "/var/run/nginx.pid"
          worker_connections: "1024"
          multi_accept: "off"
          mime_file_path: "/etc/nginx/mime.types"
          access_log: "/var/log/nginx/access.log"
          sendfile: "off"
          tcp_nopush: "off"
          tcp_nodelay: "on"
          keepalive_timeout: "75s"
          keepalive_requests: "1000"
      # Nginx -> config -> virtualhost.conf
      nginx_virtualhost:
        default_conf:
          enabled: true
          file: "default.conf"
          state: "absent"
      # Nginx -> config -> zabbix.conf
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

      # FirewallD
      firewalld:
        enabled: true
      # FirewallD -> rules
      firewalld_rules:
        service_http:
          enabled: true
          zone: "public"
          state: "enabled"
          service: "http"
          permanent: true
        service_https:
          enabled: true
          zone: "public"
          state: "enabled"
          service: "https"
          permanent: true

  tasks:
    - name: include role darexsu.zabbix_gui
      include_role:
        name: darexsu.zabbix_gui

```
##### Install and configure: Zabbix-GUI, PHP, Apache (full version)
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
      
      # PHP
      php:
        enabled: true
        version: "7.4"
        src: "third_party"
        service:
          enabled: true
          state: "started"
      # PHP -> install
      php_install:
        enabled: true
        packages: [bcmath, ctype, common, fpm, cli, gd, ldap, curl, xml, xmlrpc, mbstring, mysql, zip]
        dependencies:
          Debian: [apt-transport-https, ca-certificates, curl, gnupg2, lsb-release]
          RedHat: []
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
        packages:
          Debian: [apache2]
          RedHat: [httpd]
        dependencies:
          Debian: []
          RedHat: []
      # Apache -> config -> modules
      apache_modules:
        enabled: false
        mods_enabled: [proxy, proxy_fcgi]
        mods_disabled: []
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

      # FirewallD
      firewalld:
        enabled: true
      # FirewallD -> rules
      firewalld_rules:
        service_http:
          enabled: true
          zone: "public"
          state: "enabled"
          service: "http"
          permanent: true
        service_https:
          enabled: true
          zone: "public"
          state: "enabled"
          service: "https"
          permanent: true

  tasks:
    - name: include role darexsu.zabbix_gui
      include_role: 
        name: darexsu.zabbix_gui

```

##### Install: Zabbix-GUI (full version)

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
##### Configure: zabbix.conf.php (full version)
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
