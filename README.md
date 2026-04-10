# nutanix_ansible

Deploying Multi-VMs on NUTANIX with Ansible

Prerequisites:

Install Nutanix Ansible Collection and Requirements:

$ ansible-galaxy collection install nutanix.ncp
$ pip install -r ~/.ansible/collections/ansible_collections/nutanix/ncp/requirements.txt


# Configuring ansible

ansible.cfg

[defaults]
inventory = ./hosts
vault_password_file = ./.vault_pass

Ansible vault

Create Ansible Vault password file
echo 'password_here' > .vault_pass

hosts file

[dev]
localhost ansible_connection=local
