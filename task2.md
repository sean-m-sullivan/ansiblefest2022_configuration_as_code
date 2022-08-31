# Intro

In this section you will configure your private automation hub using the code provided that is missing some critical values/information that you will have to fill in yourself, based on the requirements and looking at readme's for the roles.

## Step 1

Ensure that you have `ansible-navigator` installed on your machine.

```console
dnf install ansible-navigator
```

## Step 2

Create a file `group_vars/all/ah_repositories.yml` you will need to add `redhat_cop.ah_configuration` and `redhat_cop.controller_configuration` to the current list of community repositories.

```yaml
---
# ah_repository_certified:
#   token: "{{ cloud_token }}"
#   url: 'https://console.redhat.com/api/automation-hub/content/5910538-synclist/'
#   auth_url: 'https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token'

ah_repository_community:
  requirements:
    - redhat_cop.aap_utilities
    - redhat_cop.ee_utilities
    - containers.podman
...
```

Note: We have ah_repository_certified commented out at this time due to token issues.

## Step 3

Create a playbook `hub_config.yml` and include the repository and user roles.

```yaml
---
- name: configure private automation hub
  hosts: "{{ groups['automationhub'][0] }}"
  gather_facts: false
  connection: local
  tasks:
    - name: include repository_sync role
      ansible.builtin.include_role:
        name: redhat_cop.ah_configuration.repository_sync

    - name: include group role
      ansible.builtin.include_role:
        name: redhat_cop.ah_configuration.group
      when: ah_groups is defined
...
```
