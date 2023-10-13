# Ansible Role: docker_compose

Installs and configures docker and docker_compose on Debian/Ubuntu servers.

## Requirements

No special requirements; note that this role requires root access, so either run it in a playbook with a global `become: yes`, or invoke the role in your playbook like:

```yaml
- hosts: web_servers
  roles:
    - role: hikimory.docker_compose
      become: yes
```

Specify root or any other user with sudo privileges as the remote user.
There are two ways you can do this:

### 1. ansible.cfg file

```ini
[defaults]
inventory = hosts.yml
private_key_file = ~/.ssh/cluster
stdout_callback = yaml
stderr_callback = yaml
remote_user = root
```

### 2. hosts file

```yaml
all:
  children:
    web_servers:
      hosts:
        web1:
          ansible_host: 192.168.0.104
          ansible_user: root
```

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

By default, this role will install docker and necessary packages. If you do not plan to install packages, update the install_packages variable to `no`.

```yaml
# Installation
install_packages: true
```

Packages to be installed. In some situations, you may need to add additional packages.

```yaml
# Define a custom list of packages to install
prep_packages:
  - apt-transport-https
  - ca-certificates
  - curl
  - gnupg-agent
  - software-properties-common
  - python3-pip

pip_packages:
  - docker
  - docker-compose

docker_packages:
  - docker-ce
  - docker-ce-cli
  - containerd.io
```

You can specify the user who will be connected to the docker group

```yaml
# Docker group
docker_group: docker
docker_user: root  
```

Default project-path settings. 
Copy the entire project into the files directory of the ansible role and specify the path to the project directory.
Or specify the relative path to your docker project

```yaml
# Project files destination
docker_remote_app_dir: /tmp/myapp
docker_local_app_dir: docker_app/ # role-directory/files/docker_app
```

By default, this role will use template files. If you do not plan to use it, update the install_packages variable to `false`.
It means that it will copy the project files along the path `docker_remote_app_dir` from `docker_local_app_dir`.

```yaml
# Templates
use_templates: false
# true
# false
```

If you plan to use it, update the `use_templates` variable and sprcify container names.

```yaml
# Templates
use_templates: true
# true
# false

#Templates
container_names: []
# - container_name_1
# - container_name_2
```

The value of the `compose_state` variable controls the state of `docker-compose`.

```yaml
# Containers state.
compose_state: restart
# run - run and build containers
# stop - stop all containers
# restart - restart all containers
# rebuild - rebuild all containers
```

The values of the `environment_variables` variable will be passed to the `env.j2` template. Subsequently, a `.env` file will be generated along the path `docker_remote_app_dir`.

```yaml
# Environment variables - .env file.
environment_variables: []
# - MYSQL_DATABASE=docker_demo
# - MYSQL_USER=user
# - MYSQL_PASSWORD=user_pass
# - MYSQL_ROOT_PASSWORD=root_pass
# - PMA_HOST=mysql
# - PMA_PORT=3306
```
The values of the `environment_variables` variable will be passed to the `env.j2` template. Subsequently, a `.env` file will be generated along the path `docker_remote_app_dir`.

The rest of the settings in defaults/main.yml control docker containers and some other common settings.

```yaml
# Containers.
containers: []
# - service_name: test
#   image: test_image:5.7.37
#   container_name: test
#   restart: unless-stopped
#   ports:
#     - "3306:3306"
#   volumes:
#     - .local/path:/remote/path/
#     - volume_name:/remote/path/
#   labels:
#     - "com.example.description=Database volume"
#     - "com.example.department=IT/Ops"
#     - "com.example.label-with-empty-value"
#   environment:
#     - "key=value"
#     - "key=value"
#   networks:
#     - network_name
#   depends_on:
#     - service_name

# Volumes.
volumes: []
  # - name: dbdata
  #   driver: dbdata

# Networks.
networks: []
  # - name: app-network
  #   driver: bridge
```

## Example Playbook

```yaml
---
- hosts: web_servers
  roles:
    - role: hikimory.docker_compose
      become: yes
```

*Inside `vars/main.yml`*:

```yaml
compose_state: run

use_templates: true

environment_variables:
  - MYSQL_DATABASE=docker_demo
  - MYSQL_USER=bob
  - MYSQL_PASSWORD=bob_pass
  - MYSQL_ROOT_PASSWORD=root_pass
  - PMA_HOST=mysql
  - PMA_PORT=3306

containers:
  - service_name: mysql
    image: mysql:5.7.37
    container_name: db
    restart: unless-stopped
    ports:
      - "3306:3306"
    volumes:
      - ./mysql/sql/:/docker-entrypoint-initdb.d/
      - dbdata:/var/lib/mysql
    environment:
      - MYSQL_DATABASE=docker_demo
      - MYSQL_USER=bob
      - MYSQL_PASSWORD=bobPassword
      - MYSQL_ROOT_PASSWORD=rootPassword
    networks:
      - app-network

  - service_name: phpmyadmin
    image: phpmyadmin/phpmyadmin
    container_name: myadmin
    restart: unless-stopped
    ports:
      - "8080:80"
    environment:
      - PMA_HOST=mysql
      - PMA_PORT=3306
    networks:
      - app-network
    depends_on:
      - mysql

  - service_name: php
    build:
      context: .
      dockerfile: php/Dockerfile
    container_name: app
    restart: unless-stopped
    volumes:
      - ./webapp:/var/www/html
    networks:
      - app-network
    depends_on:
      - mysql

  - service_name: nginx
    image: nginx:alpine
    container_name: webserver
    restart: unless-stopped
    volumes:
      - ./webapp:/var/www/html
      - ./nginx/site.conf:/etc/nginx/conf.d/app.conf
    ports:
      - "8000:80"
    networks:
      - app-network
    depends_on:
      - mysql
      - php

volumes:
  - name: dbdata

networks:
  - name: app-network
    driver: bridge
```

## License

MIT / BSD

## Author Information

This role was created in 2023 by [Nick Bykon](https://github.com/hikimory).
