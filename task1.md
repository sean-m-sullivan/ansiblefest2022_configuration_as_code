# Intro

In this section we will show you step by step how to build an Execution Environment (EE) from code. You will also be publishing this EE to your own private automation hub which you will use in the rest of the tasks.

## Step 1

Ensure that you have `ansible-core` and `ansible-builder` installed on your machine.

```console
sudo dnf install ansible-core ansible-builder
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

### **In the next few steps pay attention to the folder paths and make sure to put the files in the correct folders**

Set the variables to be used in the collections for use. These include hosts, usernames, and other variables that are reused in each role.

Create a file in this folder path `group_vars/all/auth.yml`

```yaml
---
controller_hostname: "{{ groups['automationcontroller'][0] }}"
controller_username: "{{ controller_user | default('admin') }}"
controller_password: "{{ controller_pass }}"
controller_validate_certs: false

ah_host: "{{ groups['automationhub'][0] }}"
ah_username: "{{ ah_user | default('admin') }}"
ah_password: "{{ ah_pass }}"
ah_path_prefix: 'galaxy' # this is for private automation hub
ah_verify_ssl: false
ah_validate_certs: false

ee_registry_username: "{{ ah_username }}"
ee_registry_password: "{{ ah_password }}"
ee_registry_dest: "{{ ah_host }}"
...
```

Further documentation for those who are interested to learn more see:

- [more about group_vars](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#organizing-host-and-group-variables)
- link2

## Step 4

Create your inventory file `inventory.yml`, copy in the username and password into the correct fields a long with the servers. For the builder group put the automation hub server.

```yaml
---
all:
  children:
    automationcontroller:
      hosts:
        HOST_HERE:

    automationhub:
      hosts:
        HOST_HERE:

    builder:
      hosts:
        VSCODE_HOST:
...
```

Further documentation for those who are interested to learn more see:

- [more about inventories](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#inventory-basics-formats-hosts-and-groups)
- [how to use this source in AAP](https://docs.ansible.com/ansible-tower/latest/html/userguide/inventories.html#add-source)

## Step 5

Create a vault file `vault.yml` and fill in the correct passwords for each variable

```yaml
---
cloud_token: 'this is the one from console.redhat.com'
root_machine_pass: 'password for root user on hub'
ah_api_user_pass: 'this will create and use this password can be generated'
controller_api_user_pass: 'this will create and use this password can be generated'
controller_pass: 'account pass for controller'
ah_pass: 'hub admin account pass'
ah_token_password: "{{ ah_api_user_pass }}"
vault_pass: 'the password to decrypt this vault'
...
```

Create a `.password` file put your generated password in this file. (remember we are not committing this file into git because we have it in our ignore list)

```text
Generated_Password
```

Further documentation for those who are interested to learn more see:

- [ansible vaults](https://docs.ansible.com/ansible/latest/user_guide/vault.html)
- [vault with navigator](https://ansible-navigator.readthedocs.io/en/latest/faq/#how-can-i-use-a-vault-password-with-ansible-navigator)

## Step 6

Create a new playbook called `buildEE.yml` and make the hosts use the group builder (which for this lab we are using automation hub, see note) and turn gather_facts on. Then add include role redhat_cop.ee_utilities.ee_builder

Note: this we would normally suggest being a small cli only server for deploying config as code and running installer/upgrades for AAP

```yaml
---
- name: playbook to configure execution environments
  hosts: builder
  gather_facts: true
  vars_files: "vault.yml"
  tasks:
    - name: include ee_builder role
      ansible.builtin.include_role:
        name: redhat_cop.ee_utilities.ee_builder
...
```

Further documentation for those who are interested to learn more see:

- [include vs import](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_role_module.html)
- [ee_builder role](https://github.com/redhat-cop/ee_utilities/tree/main/roles/ee_builder)

## Step 7

Create a file `group_vars/all/ah_ee_list.yml` where we will create a list called `ee_list` that has 4 variables per item which are:

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
      - name: redhat_cop.aap_utilities
      - name: awx.awx

ee_image_push: true
ee_create_ansible_config: false
...
```

Further documentation for those who are interested to learn more see:

- [YAML lists and more](https://docs.ansible.com/ansible/latest/reference_appendices/YAMLSyntax.html)
- [builder role documentation](https://github.com/redhat-cop/ee_utilities/blob/main/roles/ee_builder/README.md#build-argument-defaults)

## Step 8

Run the playbook pointing to the recently created inventory file and limit the run to just builder to build your new custom EE and publish it to private automation hub.

```console
ansible-playbook -i inventory.yml -l builder buildEE.yml
```

Further documentation for those who are interested to learn more see:

- [more on ansible-playbook](https://docs.ansible.com/ansible/latest/cli/ansible-playbook.html#ansible-playbook)
- link2

[previous task](task0.md)<div align="right">[next task](task2.md)</div>
