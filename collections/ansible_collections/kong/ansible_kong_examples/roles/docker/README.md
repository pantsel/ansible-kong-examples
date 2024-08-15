Ansible Docker Installation Playbook
## Ansible Playbook: Docker Setup

This README.md file provides an overview of the Ansible playbook designed to automate the installation and setup of Docker on an Ubuntu system. The playbook performs several key tasks including updating the system, installing dependencies, adding Docker's repository, and configuring Docker services.

### Table of Contents
- [Ansible Playbook: Docker Setup](#ansible-playbook-docker-setup)
  - [Table of Contents](#table-of-contents)
  - [Playbook Overview](#playbook-overview)
  - [Tasks Breakdown](#tasks-breakdown)
  - [Requirements](#requirements)
  - [Usage](#usage)

### Playbook Overview
This Ansible playbook performs the following operations:

1. Updates and upgrades all system packages.
2. Installs required packages.
3. Configures the Docker repository.
4. Installs Docker and related packages.
5. Manages Docker group and user permissions.
6. Enables and starts Docker services.

### Tasks Breakdown
1. **Update and Upgrade All Packages**
```yaml
name: Update and upgrade all packages to the latest version
ansible.builtin.apt:
    update_cache: true
    upgrade: dist
    cache_valid_time: 3600
```

2. **Install Required Packages**
```yaml
name: Install required packages
ansible.builtin.apt:
    pkg:
        - apt-transport-https
        - ca-certificates
        - curl
        - gnupg
        - software-properties-common
        - python3-pip
        - python3-requests
        - virtualenv
        - python3-setuptools
```

3. **Create Directory for Docker's GPG Key**
```yaml
name: Create directory for Docker's GPG key
ansible.builtin.file:
    path: /etc/apt/keyrings
    state: directory
    mode: '0755'
```

4. **Add Docker's Official GPG Key**
```yaml
name: Add Docker's official GPG key
ansible.builtin.apt_key:
    url: https://download.docker.com/linux/ubuntu/gpg
    keyring: /etc/apt/keyrings/docker.gpg
    state: present
```

5. **Determine Architecture**
```yaml
name: Determine architecture
ansible.builtin.command: dpkg --print-architecture
register: dpkg_architecture
changed_when: false
```

6. **Print Architecture Variables**
```yaml
name: Print architecture variables
ansible.builtin.debug:
    msg: "Architecture: {{ dpkg_architecture.stdout }}, Codename: {{ ansible_lsb.codename }}"
```

7. **Add Docker Repository**
```yaml
name: Add Docker repository
ansible.builtin.apt_repository:
    repo: >-
        deb [arch={{ dpkg_architecture.stdout }}
        signed-by=/etc/apt/keyrings/docker.gpg]
        https://download.docker.com/linux/ubuntu {{ ansible_lsb.codename }} stable
    filename: docker
    state: present
```

8. **Install Docker and Related Packages**
```yaml
name: Install Docker and related packages
ansible.builtin.apt:
    name: "{{ item }}"
    state: present
    update_cache: true
    loop:
        - docker-ce
        - docker-ce-cli
        - containerd.io
        - docker-buildx-plugin
        - docker-compose-plugin
```

9. **Add Docker Group**
```yaml
name: Add Docker group
ansible.builtin.group:
    name: docker
    state: present
```

10. **Add User to Docker Group**
```yaml
name: Add user to Docker group
ansible.builtin.user:
    name: "{{ ansible_user }}"
    groups: docker
    append: true
```

11. **Enable and Start Docker Services**
```yaml
name: Enable and start Docker services
ansible.builtin.systemd:
    name: "{{ item }}"
    enabled: true
    state: started
    loop:
        - docker.service
        - containerd.service
```

### Requirements
- Ansible: Make sure you have Ansible installed on the control machine.
- Ubuntu System: This playbook is designed for Ubuntu-based systems.
- Privileges: Ensure you have the necessary privileges to run system-level commands and manage services.

### Usage
1. Save the playbook to a file, e.g., `docker-setup.yml`.
2. Run the playbook using the following command:
     ```bash
     ansible-playbook docker-setup.yml -i your_inventory_file
     ```
     Replace `your_inventory_file` with the path to your Ansible inventory file.

This playbook automates the installation and configuration of Docker, making it easier to set up Docker on multiple Ubuntu systems. If you have any questions or need further customization, feel free to adjust the tasks as needed.