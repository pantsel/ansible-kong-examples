Ansible Kong DP Playbook
This README.md file explains the Ansible playbook designed to check for Docker installation, set up directories and certificates, and start Kong Data Plane (DP) in a Docker container.

## Table of Contents
- [Table of Contents](#table-of-contents)
- [Playbook Overview](#playbook-overview)
- [Tasks Breakdown](#tasks-breakdown)
- [Requirements](#requirements)
- [Usage](#usage)

## Playbook Overview
This Ansible playbook performs the following operations:

1. Checks if Docker is installed.
2. Creates directories and copies SSL certificates.
3. Starts Kong Data Plane (DP) in a Docker container with specified configuration.

## Tasks Breakdown
1. Check if Docker is Installed
```yaml
name: Check if docker is installed
ansible.builtin.command: docker --version
register: docker_version
failed_when: docker_version.rc != 0
changed_when: false
ignore_errors: yes
```

2. Print an Error Message if Docker is Not Installed
```yaml
name: Print an error message if docker is not installed
ansible.builtin.fail:
  msg: "Docker is not installed. Please install Docker before running this playbook."
when: docker_version.rc != 0
changed_when: false
```

3. Create a Directory for the Cluster Certificate and Key
```yaml
name: Create a directory for the cluster certificate and key
ansible.builtin.file:
  path: /etc/ssl/private/kong
  state: directory
  group: docker
  owner: "{{ ansible_user }}"
  mode: '0700'
```

4. Copy cluster.crt
```yaml
name: Copy cluster.crt
ansible.builtin.copy:
  src: "{{ kong_cluster_crt_path_local }}"
  dest: /etc/ssl/private/kong/cluster.crt
  group: docker
  owner: "{{ ansible_user }}"
  mode: 0600
```

5. Copy cluster.key
```yaml
name: Copy cluster.key
ansible.builtin.copy:
  src: "{{ kong_cluster_key_path_local }}"
  dest: /etc/ssl/private/kong/cluster.key
  group: docker
  owner: "{{ ansible_user }}"
  mode: 0600
```

6. Start Kong DP
```yaml
name: Start Kong DP
community.docker.docker_container:
  name: kong-dp
  image: kong/kong-gateway:3.7.0.0
  state: started
  published_ports:
    - "8000:8000"
    - "8443:8443"
    - "8100:8100"
  healthcheck:
    test: ["CMD", "kong", "health"]
    interval: 30s
    timeout: 30s
    retries: 3
  volumes:
    - /etc/ssl/private/kong/cluster.crt:/usr/local/kong/cluster.crt
    - /etc/ssl/private/kong/cluster.key:/usr/local/kong/cluster.key
  env:
    KONG_ROLE: "data_plane"
    KONG_DATABASE: "off"
    KONG_VITALS: "off"
    KONG_CLUSTER_MTLS: "pki"
    KONG_CLUSTER_CONTROL_PLANE: "{{ kong_cluster_control_plane }}"
    KONG_CLUSTER_SERVER_NAME: "{{ kong_cluster_server_name }}"
    KONG_CLUSTER_TELEMETRY_ENDPOINT: "{{ kong_cluster_telemetry_endpoint }}"
    KONG_CLUSTER_TELEMETRY_SERVER_NAME: "{{ kong_cluster_telemetry_server_name }}"
    KONG_CLUSTER_CERT: "/usr/local/kong/cluster.crt"
    KONG_CLUSTER_CERT_KEY: "/usr/local/kong/cluster.key"
    KONG_LUA_SSL_TRUSTED_CERTIFICATE: "system"
    KONG_KONNECT_MODE: "on"
```

## Requirements
- Ansible: Make sure you have Ansible installed on the control machine.
- Docker: Ensure Docker is installed and running.
- Kong Image: Ensure the appropriate Kong Docker image is available.
- Privileges: Ensure you have the necessary privileges to run Docker containers and manage files.

## Usage
1. Save the playbook to a file, e.g., `kong-dp-setup.yml`.
2. Run the playbook using the following command:
   ```
   ansible-playbook kong-dp-setup.yml -i your_inventory_file
   ```
   Replace `your_inventory_file` with the path to your Ansible inventory file.

This playbook automates the setup of Kong Data Plane (DP) in Docker, ensuring Docker is installed and configured correctly. If you have any questions or need further customization, feel free to adjust the tasks as needed.