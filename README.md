# ansible-role-rustdesk-server-docker

Ansible role rustdesk server in docker compose.

<img src="https://github.com/foi/ansible-role-rustdesk-server-docker/workflows/ci.yml/badge.svg?branch=main"> Github's CI is cursed. [act](https://github.com/nektos/act) works very well.

Compatibility
--------------

python: 3.12, 3.13, 3.14
ansible: 12, 13

prerequisites
--------------

docker and docker-compose should be installed in advance.

Install
--------------

`ansible-galaxy role install foi.rustdesk_server_docker`

or add it in `requirements.yml`

```
roles:
  - name: foi.rustdesk_server_docker
```

and run `ansible-galaxy install -r requirements.yml`

Role Variables
--------------
```yml
---
rustdesk_server_docker_root_path: /opt/rustdesk-server
rustdesk_server_docker_root_path_mode: '0775'
rustdesk_server_docker_root_path_owner: root
rustdesk_server_docker_root_path_group: root
rustdesk_server_docker_image: rustdesk/rustdesk-server
rustdesk_server_docker_version: 1.1.15
rustdesk_server_docker_domain: "change-it"
rustdesk_server_docker_hbbr_relay_port: 21117
rustdesk_server_docker_hbbr_install: true
rustdesk_server_docker_hbbs_install: true
rustdesk_server_docker_hbbr_container_name: hbbr
rustdesk_server_docker_hbbs_container_name: hbbs
rustdesk_server_docker_hbbs_restart: unless-stopped
rustdesk_server_docker_hbbr_restart: unless-stopped
rustdesk_server_docker_hbbs_bind_address: "0.0.0.0"
rustdesk_server_docker_hbbr_bind_address: "0.0.0.0"
rustdesk_server_docker_hbbs_ports:
  - "{{ rustdesk_server_docker_hbbs_bind_address }}:21114:21114"
  - "{{ rustdesk_server_docker_hbbs_bind_address }}:21115:21115"
  - "{{ rustdesk_server_docker_hbbs_bind_address }}:21116:21116/tcp"
  - "{{ rustdesk_server_docker_hbbs_bind_address }}:21116:21116/udp"
  - "{{ rustdesk_server_docker_hbbs_bind_address }}:21118:21118"
rustdesk_server_docker_hbbr_ports:
  - "{{ rustdesk_server_docker_hbbr_bind_address }}:{{ rustdesk_server_docker_hbbr_relay_port }}:{{ rustdesk_server_docker_hbbr_relay_port }}"
  - "{{ rustdesk_server_docker_hbbr_bind_address }}:21119:21119"
rustdesk_server_docker_volumes:
  - rustdesk-server-data:/root
rustdesk_server_docker_hbbs_command:
  - hbbs
rustdesk_server_docker_hbbr_command:
  - hbbr
rustdesk_server_docker_compose_out_of_service_template:
  volumes:
    rustdesk-server-data: {}
rustdesk_server_docker_hbbs_service_params: {}
rustdesk_server_docker_hbbr_service_params: {}

rustdesk_server_docker_hbbs_environment:
  - "RELAY={{ rustdesk_server_docker_domain }}:{{ rustdesk_server_docker_hbbr_relay_port }}"
  - "ENCRYPTED_ONLY=1"
rustdesk_server_docker_hbbr_environment:
  - "RELAY={{ rustdesk_server_docker_domain }}:{{ rustdesk_server_docker_hbbr_relay_port }}"
  - "ENCRYPTED_ONLY=1"

rustdesk_server_docker_compose_template: |
  services:
  {% if rustdesk_server_docker_hbbs_install %}
    hbbs:
      image: {{ rustdesk_server_docker_image }}:{{ rustdesk_server_docker_version }}
      container_name: "{{ rustdesk_server_docker_hbbs_container_name }}"
      restart: "{{ rustdesk_server_docker_hbbs_restart }}"
  {% if rustdesk_server_docker_hbbs_ports | length > 0 %}
      ports:
  {% for port in rustdesk_server_docker_hbbs_ports %}
        - "{{ port }}"
  {% endfor %}
  {% endif %}
      command:
  {% for command in rustdesk_server_docker_hbbs_command %}
        - {{ command }}
  {% endfor %}
  {% if rustdesk_server_docker_hbbr_install %}
      depends_on:
        - hbbr
  {% endif %}
      environment:
  {% for env in rustdesk_server_docker_hbbs_environment %}
        - "{{ env }}"
  {% endfor %}
      volumes:
  {% for v in rustdesk_server_docker_volumes %}
        - {{ v }}
  {% endfor %}
  {% for k, v in rustdesk_server_docker_hbbs_service_params.items() %}
        {{ k }}: {{ v if (v is number or v is string or v is boolean) else v | to_json }}
  {% endfor %}
  {% endif %}

  {% if rustdesk_server_docker_hbbr_install %}
    hbbr:
      image: {{ rustdesk_server_docker_image }}:{{ rustdesk_server_docker_version }}
      container_name: "{{ rustdesk_server_docker_hbbr_container_name }}"
      restart: "{{ rustdesk_server_docker_hbbr_restart }}"
  {% if rustdesk_server_docker_hbbr_ports | length > 0 %}
      ports:
  {% for port in rustdesk_server_docker_hbbr_ports %}
        - "{{ port }}"
  {% endfor %}
  {% endif %}
      command:
  {% for command in rustdesk_server_docker_hbbr_command %}
        - {{ command }}
  {% endfor %}
  {% if rustdesk_server_docker_hbbr_environment | length > 0 %}
      environment:
  {% for env in rustdesk_server_docker_hbbr_environment %}
        - "{{ env }}"
  {% endfor %}
  {% endif %}
      volumes:
  {% for v in rustdesk_server_docker_volumes %}
        - {{ v }}
  {% endfor %}
  {% for k, v in rustdesk_server_docker_hbbr_service_params.items() %}
        {{ k }}: {{ v if (v is number or v is string or v is boolean) else v | to_json }}
  {% endfor %}
  {% endif %}

  {% if rustdesk_server_docker_compose_out_of_service_template is not none %}
  {{ rustdesk_server_docker_compose_out_of_service_template | to_nice_yaml(indent=2) }}
  {% endif %}
```

Example Playbook
----------------
```yml
# inventory
[rustdesk_server]
rustdesk-server
# playbook.yml
- hosts: rustdesk_server
  roles:
    - role: foi.rustdesk_server_docker
  become: true
# host_vars/rustdesk-server.yml
rustdesk_server_docker_domain: "yourdomainname.com"
# below if you don't want to use automatically created docker volume
rustdesk_server_docker_volumes:
  - /mnt:/root
rustdesk_server_docker_compose_out_of_service_template: null
```
License
-------

MIT
