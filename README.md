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
  - [inventory/group\_vars/all.yml](#inventorygroup_varsallyml)
  - [Playbooks](#playbooks)
    - [install\_docker.yml](#install_dockeryml)
    - [install\_kong.yml](#install_kongyml)
- [Getting started](#getting-started)
  - [1. Prepare ENV Variables](#1-prepare-env-variables)
  - [2. Add Kong Cluster Certificates](#2-add-kong-cluster-certificates)
  - [3. Spin up the ubuntu Virtual Machines (Orbstack)](#3-spin-up-the-ubuntu-virtual-machines-orbstack)
  - [4. Install docker on the docker hosts](#4-install-docker-on-the-docker-hosts)
  - [5. Install and Run Kong on the docker hosts](#5-install-and-run-kong-on-the-docker-hosts)
  - [6. Install and Run Kong directly on the ubuntu hosts](#6-install-and-run-kong-directly-on-the-ubuntu-hosts)
  - [7. Verify Kong is up and running on all hosts](#7-verify-kong-is-up-and-running-on-all-hosts)

<!-- /TOC -->

## Included content/ Directory Structure

```
├── README.md
├── ansible.cfg
├── collections
│   ├── ansible_collections
│   │   └── kong
│   │       └── ansible_kong_examples
│   │           └── roles
│   │               ├── docker
│   │               │   ├── README.md
│   │               │   └── tasks
│   │               │       └── main.yml
│   │               ├── kong
│   │               │   ├── README.md
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

### inventory/group_vars/all.yml

The variables defined in this file will be applied globally across all hosts in the inventory.

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

### 1. Prepare ENV Variables

Duplicate the `.env_example` file and rename it to `.env.` Then, fill in the required values for the environment variables.

| Variable Name                            | Description                                                                                                                                                                                                                                                                                                                                  |
| ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ANS_KONG_EE_VERSION`                    | The Kong EE version to install                                                                                                                                                                                                                                                                                                               |
| `ANS_KONG_CLUSTER_CONTROL_PLANE`         | The fully qualified domain name (FQDN) and port of the Kong Control Plane. Format: `<control_fqdn:443>`. This variable specifies where the Kong Control Plane is accessible.                                                                                                                                                                 |
| `ANS_KONG_CLUSTER_SERVER_NAME`           | The fully qualified domain name (FQDN) of the Kong Control Plane server. Format: `<control_fqdn>`. This variable is used to identify the server name for the Kong Control Plane.                                                                                                                                                             |
| `ANS_KONG_CLUSTER_TELEMETRY_ENDPOINT`    | The fully qualified domain name (FQDN) and port of the Kong Telemetry Endpoint. Format: `<control_telemetry_fqdn:443>`. This variable specifies where the Kong Telemetry service can be reached.                                                                                                                                             |
| `ANS_KONG_CLUSTER_TELEMETRY_SERVER_NAME` | The fully qualified domain name (FQDN) of the Kong Telemetry server. Format: `<control_telemetry_fqdn>`. This variable is used to identify the server name for the Kong Telemetry service.                                                                                                                                                   |
| `ANSIBLE_PRIVATE_KEY_FILE`               | The path to the private key file used for SSH authentication in Ansible. This file should be in PEM format and must have the appropriate permissions set to ensure secure access to the remote hosts. Make sure to specify the correct path to the private key file in this variable. `Orbstack` users can use `~/.orbstack/ssh/id_ed25519`. |

***Export env variables***

```bash
$ export $(grep -v '^#' .env | xargs)         
```

### 2. Add Kong Cluster Certificates

1. Create a folder named `.tls`.

2. Place the Kong Cluster certificate and private key into this directory with the following names:
   - **Certificate**: `cluster.crt`
   - **Private Key**: `cluster.key`

   These files are used for mutual TLS (mTLS) connections to the Control Plane.

### 3. Spin up the ubuntu Virtual Machines (Orbstack)

For Orbstack users:

**Open Orbstack** and create VMs:
   - **VM 1**: `ubuntu` (Ubuntu 22.04 LTS - Jammy Jellyfish amd46)
   - **VM 2**: `ubuntu2` (Ubuntu 22.04 LTS - Jammy Jellyfish amd46)
   - **VM 3**: `docker1` (Ubuntu 22.04 LTS - Jammy Jellyfish amd46)
   - **VM 4**: `docker2` (Ubuntu 22.04 LTS - Jammy Jellyfish amd46)

### 4. Install docker on the docker hosts

***Role***: `collections/ansible_collections/kong/ansible_kong_examples/roles/docker`

Install Docker on the designated Docker hosts by running the following Ansible playbook:

```bash
$ ansible-playbook install_docker.yml  
```

### 5. Install and Run Kong on the docker hosts

***Role***: `collections/ansible_collections/kong/ansible_kong_examples/roles/kong_docker`

Deploy and start Kong within Docker containers on the Docker hosts with this command:

```bash
$ ansible-playbook install_kong.yml --tags "kong_docker"    
```

### 6. Install and Run Kong directly on the ubuntu hosts

***Role***: `collections/ansible_collections/kong/ansible_kong_examples/roles/kong`

Install and configure Kong directly on Ubuntu hosts using the following playbook:

```bash
$ ansible-playbook install_kong.yml --tags "kong_host"    
```

### 7. Verify Kong is up and running on all hosts

To ensure Kong is running correctly on all hosts:

```bash
$ curl http://ubuntu.orb.local:8000

$ curl http://ubuntu2.orb.local:8000

$ curl http://docker1.orb.local:8000

$ curl http://docker2.orb.local:8000
```






