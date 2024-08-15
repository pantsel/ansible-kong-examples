Ansible Kong Installation Playbook
## README.md

This README.md file explains the Ansible playbook designed to automate the installation and setup of Kong on an Ubuntu system. The playbook performs several key tasks including updating the system, downloading and installing the Kong package, configuring SSL certificates, and managing the Kong service.

### Table of Contents
- [README.md](#readmemd)
  - [Table of Contents](#table-of-contents)
  - [Playbook Overview](#playbook-overview)
  - [Tasks Breakdown](#tasks-breakdown)
  - [Requirements](#requirements)
  - [Usage](#usage)

### Playbook Overview
This Ansible playbook performs the following operations:

1. Updates and upgrades all system packages.
2. Determines the Ubuntu release codename and system architecture.
3. Sets the Kong package URL, downloads the Kong package, and installs it.
4. Creates directories and copies SSL certificates.
5. Configures Kong by copying a configuration template.
6. Reloads and manages the Kong service.

### Tasks Breakdown
1. **Update and Upgrade All Packages**
```yaml
name: Update and upgrade all packages to the latest version
ansible.builtin.apt:
    update_cache: true
    upgrade: dist
    cache_valid_time: 3600
```

2. **Determine Ubuntu Release Codename**
```yaml
name: Determine Ubuntu release codename
ansible.builtin.command: lsb_release -sc
register: ubuntu_release
changed_when: false
```

3. **Determine Architecture**
```yaml
name: Determine architecture
ansible.builtin.command: dpkg --print-architecture
register: dpkg_architecture
changed_when: false
```

4. **Print Architecture Variables**
```yaml
name: Print architecture variables
ansible.builtin.debug:
    msg: "Architecture: {{ dpkg_architecture.stdout }}, Codename: {{ ansible_lsb.codename }}"
```

5. **Print Kong Version**
```yaml
name: Print Kong version
ansible.builtin.debug:
    msg: "Kong version: {{ kong_version }}"
```

6. **Set the Kong Package URL**
```yaml
name: Set the Kong package URL
ansible.builtin.set_fact:
    kong_package_url: "{{ kong_gateway_package_url | replace('ANSIBLE_LSB_CODENAME', ubuntu_release.stdout) | replace('ARCH', dpkg_architecture.stdout) }}"
```

7. **Download Kong Package**
```yaml
name: Download Kong package
ansible.builtin.get_url:
    url: "{{ kong_package_url }}"
    dest: "/tmp/kong-enterprise-edition-{{ kong_version }}.deb"
```

8. **Install Kong Package**
```yaml
name: Install Kong package
ansible.builtin.apt:
    deb: "/tmp/kong-enterprise-edition-{{ kong_version }}.deb"
    state: present
```

9. **Create Directory for the Cluster Certificate and Key**
```yaml
name: Create directory for the cluster certificate and key
ansible.builtin.file:
    path: /etc/ssl/private/kong
    state: directory
    group: kong
    owner: kong
    mode: '0700'
```

10. **Copy cluster.crt**
```yaml
name: Copy cluster.crt
ansible.builtin.copy:
    src: "{{ kong_cluster_crt_path_local }}"
    dest: /etc/ssl/private/kong/cluster.crt
    group: kong
    owner: kong
    mode: 0600
```

11. **Copy cluster.key**
```yaml
name: Copy cluster.key
ansible.builtin.copy:
    src: "{{ kong_cluster_key_path_local }}"
    dest: /etc/ssl/private/kong/cluster.key
    group: kong
    owner: kong
    mode: 0600
```

12. **Copy kong.conf**
```yaml
name: Copy kong.conf
ansible.builtin.template:
    src: "{{ role_path }}/templates/kong.conf.j2"
    dest: /etc/kong/kong.conf
    owner: kong
    group: kong
    mode: 0600
    register: template_result
```

13. **Reload Kong Service if kong.conf is Changed or if Force Reload is Requested**
```yaml
name: Reload Kong service if kong.conf is changed or if force reload is requested
ansible.builtin.systemd:
    name: kong-enterprise-edition
    state: reloaded
    when: template_result.changed or force_reload
```

14. **Ensure Kong Service is Enabled and Started**
```yaml
name: Ensure Kong service is enabled and started
ansible.builtin.systemd:
    name: kong-enterprise-edition
    state: started
    enabled: yes
```

15. **Wait for Kong Service to Become Available**
```yaml
name: Wait for Kong service to become available
ansible.builtin.wait_for:
    host: localhost
    port: "{{ kong_proxy_port_http_ssl }}"
    delay: 5
    timeout: 60
    state: started
    msg: "Kong is not running"
```

### Requirements
- Ansible: Make sure you have Ansible installed on the control machine.
- Ubuntu System: This playbook is designed for Ubuntu-based systems.
- Kong: Ensure that the appropriate Kong version and package URL are defined.
- Privileges: Ensure you have the necessary privileges to run system-level commands and manage services.

### Usage
1. Save the playbook to a file, e.g., `kong-setup.yml`.
2. Run the playbook using the following command:
     ```
     ansible-playbook kong-setup.yml -i your_inventory_file
     ```
     Replace `your_inventory_file` with the path to your Ansible inventory file.

This playbook automates the installation and configuration of Kong, making it easier to set up Kong on multiple Ubuntu systems. If you have any questions or need further customization, feel free to adjust the tasks as needed.