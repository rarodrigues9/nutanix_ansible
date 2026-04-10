
# Deploying Multi-VMs on NUTANIX with Ansible



Prerequisites:

Installing Nutanix Ansible Collection and Requirements:
```
ansible-galaxy collection install nutanix.ncp
pip install -r ~/.ansible/collections/ansible_collections/nutanix/ncp/requirements.txt
```

##  Configuring ansible controller 

#### ansible.cfg
```
[defaults]
inventory = ./hosts
vault_password_file = ./.vault_pass
```
#### Ansible vault

Create Ansible Vault password file:

```
echo 'password_here' > .vault_pass
```
#### hosts file
```
[dev]
localhost ansible_connection=local
```
#### Create a directory structure for Ansible variables
```
mkdir -p group_vars/dev
```
Ansible Vault for creating the encrypted file containing credentials.
```
ansible-vault create group_vars/dev/vault
```
Insert the credentials:
```
pc_host: prism_central_hostname/ip
pc_user: prism_central_username
pc_pass: prism_central_password
```

## Playbook for Multi-VMs

```
#deploy.yml

---
- name: Provisionamento de VMs no Nutanix Prism Central
  hosts: localhost
  gather_facts: false
  collections:
    - nutanix.ncp
  vars_files:
    - vms.yml
  module_defaults:
    group/nutanix.ncp.ntnx:
      nutanix_host: "{{ pc_host }}"
      nutanix_username: "{{ pc_user }}"
      nutanix_password: "{{ pc_pass }}"
      validate_certs: false

  tasks:
    - name: Criar instâncias
      nutanix.ncp.ntnx_vms:
        state: present
        cluster:
          name: "{{ item.0.cluster }}"     
        name: "{{ item.0.prefix }}-{{ item.1 }}"
        cores_per_vcpu: "{{ item.0.cores }}"
        vcpus: "{{ item.0.vcpu }}"
        memory_gb: "{{ item.0.mem }}"
        networks:
          - is_connected: true
            subnet:
              name: "{{ item.0.vlan }}"
        disks:
          - type: DISK
            size_gb: "{{ item.0.disk }}"
            bus: SCSI
            clone_image:
              name: "{{ item.0.image }}"
        boot_config:
          boot_type: UEFI
          boot_order:
            - DISK
        guest_customization:
          type: "cloud_init"
          script_path: "./cloud_init.yml"
          is_overridable: true
      loop: "{{ vms_def | subelements('indices') }}"
```
```
#vms.yml

---
# Definição e quantidade dos recursos das VMs
vms_def:
  - prefix: "vm-web-"
    cluster: "CLUSTER NAME HERE"
    cores: 1
    vcpu: 2
    mem: 4
    disk: 10
    image: "ubuntu-2404-cloudimg"
    vlan: "VLAN HERE"
    indices: [1,2] # Criará vm-web-1 e vm-web-2
  # - prefix: "vm-db"
  #   vcpu: 4
  #   ram_gb: 8
  #   indices: [1]    # Criará vm-db-1
```
```
#cloud_init.yml

---
#cloud-config
users:
  - default
  - name: user
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    passwd: $6$rounds=4 ...
    ssh_authorized_keys:
      - ssh-ed25519 AAAAC3NzaC1lZ ...
write_files:
  - path: /etc/netplan/50-cloud-init.yaml
    content: |
      network:
        version: 2
        ethernets:
          ens3:
            dhcp4: true
            dhcp6: false
            nameservers:
              search: [domain.local]
              addresses: [10.0.0.10]
runcmd:
  - chmod 600 /etc/netplan/*
  - netplan apply 
  - timedatectl set-timezone America/Cuiaba
```
