# Kong Ansible Examples Project <!-- omit in toc -->

This sample repository offers foundational examples demonstrating how to install and run Kong Data Planes on Ubuntu hosts using Ansible.

This repository and its associated examples utilize [Orbstack](https://orbstack.dev/) to efficiently spin up and manage virtual machines (VMs). If you do not have OrbStack, you might need to adjust some files and settings accordingly, such as `ansible.cfg` and `hosts.yml`, to ensure compatibility with your VM management setup.

## Table of Contents <!-- omit in toc -->

<!-- TOC -->
- [Included content/ Directory Structure](#included-content-directory-structure)
- [Prerequisites](#prerequisites)
- [Use Cases Covered](#use-cases-covered)
- [Important files](#important-files)
  - [ansible.cfg](#ansiblecfg)
  - [inventory/hosts.yml](#inventoryhostsyml)
  - [Playbooks](#playbooks)
    - [install\_docker.yml](#install_dockeryml)
    - [install\_kong.yml](#install_kongyml)
- [Getting started](#getting-started)
  - [1. Spin up the ubuntu Virtual Machines](#1-spin-up-the-ubuntu-virtual-machines)

<!-- /TOC -->

## Included content/ Directory Structure

```
├── README.md
├── ansible.cfg
├── collections
│   ├── ansible_collections
│   │   └── kong
│   │       └── ansible_kong_examples
│   │           ├── README.md
│   │           └── roles
│   │               ├── docker
│   │               │   └── tasks
│   │               │       └── main.yml
│   │               ├── kong
│   │               │   ├── tasks
│   │               │   │   └── main.yml
│   │               │   └── templates
│   │               │       └── kong.conf.j2
│   │               └── kong_docker
│   │                   ├── README.md
│   │                   └── tasks
│   │                       └── main.yml
│   └── requirements.yml
├── images
├── install_docker.yml
├── install_kong.yml
└── inventory
    ├── group_vars
    │   └── all.yml
    ├── host_vars
    │   ├── server1.yml
    │   ├── server2.yml
    │   ├── server3.yml
    │   └── server4.yml
    └── hosts.yml
```

## Prerequisites

- [Ansible](https://www.ansible.com/)

## Use Cases Covered

The examples cover two distinct use cases:

1. **Install and run Kong Data Plane directly on Ubuntu/Debian hosts**: This approach involves deploying Kong Data Plane natively on the operating system.

2. **Run Kong Data Plane on Docker-enabled hosts (Ubuntu/Debian)**: This method involves deploying Kong Data Plane within Docker containers on hosts that have Docker installed.


## Important files

### ansible.cfg

The ansible configuration file

`private_key_file` is set to the default `Orbstack` private key used to SSH in the hosts. If nor using `Orbstack`, adjust the value accordingly.

### inventory/hosts.yml

> Modify the file as needed if you are not using OrbStack to ensure compatibility with the provided example.

This Ansible hosts file defines a set of hosts organized into two groups: `ubuntu_hosts` and `docker_hosts`. The `all` group includes four servers with the following configurations:

- **server1**: 
  - `ansible_host`: `ubuntu.orb.local@orb`
  - `ansible_user`: `ubuntu`

- **server2**:
  - `ansible_host`: `ubuntu2.orb.local@orb`
  - `ansible_user`: `ubuntu2`

- **server3**:
  - `ansible_host`: `docker1.orb.local@orb`
  - `ansible_user`: `docker1`

- **server4**:
  - `ansible_host`: `docker2.orb.local@orb`
  - `ansible_user`: `docker2`

The `children` section groups these servers into:
- **ubuntu_hosts**:
  - `server1`
  - `server2`

- **docker_hosts**:
  - `server3`
  - `server4`

This structure allows for organized management and task targeting based on server categories. For more details on configuring Ansible inventory files, refer to the [Ansible documentation](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html).

### Playbooks

#### install_docker.yml

This Ansible playbook is designed to install Docker on hosts categorized under the `docker_hosts` group. The playbook performs the following actions:

- **Play Name**: Install Docker
- **Target Hosts**: `docker_hosts`
- **Privilege Escalation**: Enabled (`become: yes`)
- **Roles**: Uses the `kong.ansible_kong_examples.docker` role to handle the Docker installation process. (Can be found in `collections/ansible_collections/kong/ansible_kong_examples/roles/docker`)

This setup ensures that Docker is installed on all servers within the `docker_hosts` group. For more information on Ansible roles and playbooks, see the [Ansible documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_roles.html).

#### install_kong.yml

This Ansible playbook installs Kong on Docker and Ubuntu hosts, organizing the tasks into separate plays for each host group. It performs the following actions:

1. **Play Name**: Install Kong Docker
   - **Target Hosts**: `docker_hosts`
   - **Privilege Escalation**: Enabled (`become: yes`)
   - **Gather Facts**: Limited to minimal subset (`gather_subset: min`)
   - **Roles**: Uses the `kong.ansible_kong_examples.kong_docker` role for installing Kong on Docker (Can be found in `collections/ansible_collections/kong/ansible_kong_examples/roles/kong_docker`).
   - **Tags**: `kong_docker`

2. **Play Name**: Install Kong
   - **Target Hosts**: `ubuntu_hosts`
   - **Privilege Escalation**: Enabled (`become: yes`)
   - **Gather Facts**: Limited to minimal subset (`gather_subset: min`)
   - **Roles**: Uses the `kong.ansible_kong_examples.kong` role for installing Kong on Ubuntu servers (Can be found in `/Users/panagis.tselentis@konghq.com/ansible-projects/ansible_kong_examples/collections/ansible_collections/kong/ansible_kong_examples/roles/kong`).
   - **Tags**: `kong_host`

This playbook ensures that Kong is installed appropriately based on the host type, with Docker hosts receiving the `kong_docker` role and Ubuntu hosts receiving the `kong` role. For more information on Ansible playbooks and roles, refer to the [Ansible documentation](https://docs.ansible.com/ansible/latest/user_guide/playbooks_roles.html).

## Getting started

### 1. Spin up the ubuntu Virtual Machines

To get started with this repository, you'll need to spin up two Ubuntu virtual machines using OrbStack. Follow these steps to set up your environment:

> You can skip this step if you are not using `OrbStack` and have your VMs set up using a different method.

1. **Open OrbStack** and start the VM creation process.

2. **Create Virtual Machines**:
   - **VM 1**:
     - **Name**: `ubuntu1`
     - **Operating System**: Ubuntu 22.04 LTS (Jammy Jellyfish)
     - **Architecture**: Intel

   - **VM 2**:
     - **Name**: `ubuntu2`
     - **Operating System**: Ubuntu 22.04 LTS (Jammy Jellyfish)
     - **Architecture**: Intel

   - **VM 3**:
     - **Name**: `docker1`
     - **Operating System**: Ubuntu 22.04 LTS (Jammy Jellyfish)
     - **Architecture**: Intel

   - **VM 4**:
     - **Name**: `docker2`
     - **Operating System**: Ubuntu 22.04 LTS (Jammy Jellyfish)
     - **Architecture**: Intel

You now have four Ubuntu virtual machines ready for configuration. Follow the subsequent instructions in the repository to proceed with your setup.





