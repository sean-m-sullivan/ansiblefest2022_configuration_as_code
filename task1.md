# Intro

In this section we will show you step by step how to build an Execution Environment (EE) from code. You will also be publishing this EE to your own private automation hub which you will use in the rest of the tasks.

## Step 1

Ensure that you have `ansible-core` and `ansible-builder` installed on your machine.

```console
dnf install ansible-core ansible-builder
```

Further documentation for those who are interested to learn more see:

- [upgrading from ansible to ansible-core](https://access.redhat.com/discussions/6962395)
- [using pip (non supported upstream version)](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

## Step 2

Install our ee_utilities collection and containers.podman using `ansible-galaxy` command.

```console
ansible-galaxy collection install redhat_cop.ee_utilities containers.podman
```

Further documentation for those who are interested to learn more see:

- [installing collections using cli](https://docs.ansible.com/ansible/devel/user_guide/collections_using.html#collections)
- [using collections in AAP](https://docs.ansible.com/ansible-tower/latest/html/userguide/projects.html#collections-support)

## Step 3

Create a file in this folder path group_vars/all/auth.yml

```yaml
# User may update controller/hub auth creds to this file and encrypt it using `ansible-vault`
---
controller_hostname: "{{ groups['automationcontroller'][0] }}"
controller_username: "{{ controller_user | default('admin') }}"
controller_password: "{{ controller_pass | default('Password1234!') }}"
controller_validate_certs: false

ah_host: "{{ groups['automationhub'][0] }}"
ah_username: "{{ ah_user | default('admin') }}"
ah_password: "{{ ah_pass | default('Password1234!') }}"
ah_path_prefix: 'galaxy' # this is for private automation hub
ah_verify_ssl: false
ah_validate_certs: false

ee_registry_username: "{{ ah_username }}"
ee_registry_password: "{{ ah_password }}"
ee_registry_dest: "{{ ah_host }}"
ee_validate_certs: false
...
```

Further documentation for those who are interested to learn more see:

- [more about group_vars](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#organizing-host-and-group-variables)
- link2

## Step 4

Create your inventory file inventory.yml, copy in the username and password into the correct fields a long with the servers. For the builder group put the automation hub server.

```yaml
---
all:
  children:
    automationcontroller:
      hosts:
        HOST_HERE

    automationhub:
      hosts:
        HOST_HERE

    builder:
      hosts:
        SAME_AS_HUB
  vars:
    ansible_user: HERE
    ansible_password: HERE
    controller_pass: HERE
    ah_pass: HERE
    ansible_ssh_common_args: '-o StrictHostKeyChecking=no' # might need this, test without on lab
...
```

Further documentation for those who are interested to learn more see:

- [more about inventories](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#inventory-basics-formats-hosts-and-groups)
- [how to use this source in AAP](https://docs.ansible.com/ansible-tower/latest/html/userguide/inventories.html#add-source)

## Step 5

Create a new playbook called buildEE.yml and make the hosts use the group builder (which for this lab we are using automation hub, see note) and turn gather_facts on. Then add include role redhat_cop.ee_utilities.ee_builder

Note: this we would normally suggest being a small cli only server for deploying config as code and running installer/upgrades for AAP

```yaml
---
- name: playbook to configure execution environments
  hosts: builder
  gather_facts: true
  tasks:
    - name: include ee_builder role
      ansible.builtin.include_role:
        name: redhat_cop.ee_utilities.ee_builder
...
```

Further documentation for those who are interested to learn more see:

- [include vs import](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_role_module.html)
- [ee_builder role](https://github.com/redhat-cop/ee_utilities/tree/main/roles/ee_builder)

## Step 6

Create a file called ah_ee_list.yml in group_vars/all where we will create a list called `ee_list` that has 4 variables per item which are:

- `ee_name` this is required and will be what the EE image will be called
- `bindep` this is any system packages that would be needed
- `python` these are any python modules that need to be added through pip (excluding ansible)
- `collections` any collections that you would like to be built into your EE image

which the role will loop over and for each item in this list it will create and publish an EE using the provided variables. For this lab we will just pass it 3 of the 4 redhat_cop configuration as code collections which are:

- redhat_cop.controller_configuration
- redhat_cop.ah_configuration
- redhat_cop.ee_utilities

```yaml
---
ee_list:
  - ee_name: config_as_code
    collections:
      - name: redhat_cop.controller_configuration
      - name: redhat_cop.ah_configuration
      - name: redhat_cop.ee_utilities

ee_image_push: true
ee_create_ansible_config: false
...
```

Further documentation for those who are interested to learn more see:

- [YAML lists and more](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)
- [builder role documentation](https://github.com/redhat-cop/ee_utilities/blob/main/roles/ee_builder/README.md#build-argument-defaults)

## Step 7

Run the playbook pointing to the recently created inventory file and limit the run to just builder to build your new custom EE and publish it to private automation hub.

```console
ansible-playbook -i inventory.yml -l builder buildEE.yml
```

Further documentation for those who are interested to learn more see:

- [more on ansible-playbook](https://docs.ansible.com/ansible/latest/cli/ansible-playbook.html#ansible-playbook)
- link2
