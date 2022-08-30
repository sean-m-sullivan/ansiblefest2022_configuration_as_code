# Intro

In this section we will show you step by step how to build an Execution Environment (EE) from code. You will also be publishing this EE to your own private automation hub which you will use in the rest of the tasks.

## Step 1

Ensure that you have `ansible-core` installed on your machine.

```console
dnf install ansible-core
```

Further documentation for those who are interested to learn more see:

- link1
- link2

## Step 2

Install our ee_utilities collection using `ansible-galaxy` command.

```console
ansible-galaxy collection install redhat_cop.ee_utilities
```

Further documentation for those who are interested to learn more see:

- link1
- link2

### I think we need to add group_vars here actually... and make sure to add the auth.yml file

## Step 3

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
    admin_password: HERE
...
```

Further documentation for those who are interested to learn more see:

- link1
- link2

## Step 4

Create a new playbook called buildEE.yml and make the hosts use the group builder (which for this lab we are using automation hub, see note) and turn gather_facts on.

Note: this we would normally suggest being a small cli only server for deploying config as code and running installer/upgrades for AAP

```yaml
---
- name: playbook to configure execution environments
  hosts: builder
  gather_facts: true
  tasks:
```

Further documentation for those who are interested to learn more see:

- link1
- link2

## Step 5

Include the role redhat_cop.ee_utilities.ee_builder and have a vars section for that include role

```yaml
    - name: include ee_builder role
      ansible.builtin.include_role:
        name: redhat_cop.ee_utilities.ee_builder
      vars:
```

Further documentation for those who are interested to learn more see:

- link1
- link2

## Step 6

In the vars we will pass it a list called `ee_list` that has 4 variables per item which are:

- `ee_name` this is required and will be what the EE image will be called
- `ee_bindep` this is any system packages that would be needed
- `ee_python` these are any python modules that need to be added through pip (excluding ansible)
- `ee_collections` any collections that you would like to be built into your EE image

which the role will loop over and for each item in this list it will create and publish an EE using the provided variables. For this lab we will just pass it 3 of the 4 redhat_cop configuration as code collections which are:

- redhat_cop.controller_configuration
- redhat_cop.ah_configuration
- redhat_cop.ee_utilities

```yaml
ee_list:
  - ee_name: config_as_code
    ee_collections:
      - name: redhat_cop.controller_configuration
      - name: redhat_cop.ah_configuration
      - name: redhat_cop.ee_utilities
```

Further documentation for those who are interested to learn more see:

- link1
- link2

## Step 7

Run the playbook pointing to the recently created inventory file and limit the run to just builder to build your new custom EE and publish it to private automation hub.

```console
ansible-playbook -i inventory.yml -l builder buildEE.yml
```

Further documentation for those who are interested to learn more see:

- link1
- link2
