# Intro

In this section you will configure your private automation hub using the code provided that is missing some critical values/information that you will have to fill in yourself, based on the requirements and looking at readme's for the roles.

## Step 1

Ensure that you have `ansible-navigator` installed on your machine.

```console
sudo dnf install ansible-navigator
```

Further documentation for those who are interested to learn more see:

- [ansible navigator docs](https://ansible-navigator.readthedocs.io/en/latest/installation/#install-ansible-navigator)

## Step 2

Create a file `group_vars/all/ah_repositories.yml` you will need to add `redhat_cop.ah_configuration` and `redhat_cop.controller_configuration` to the current list of community repositories.

```yaml
---
# ah_repository_certified:
#   token: "{{ cloud_token }}"
#   url: 'https://console.redhat.com/api/automation-hub/'
#   auth_url: 'https://sso.redhat.com/auth/realms/redhat-external/protocol/openid-connect/token'
#   wait: true

ah_repository_community:
  requirements:
    - redhat_cop.aap_utilities
    - redhat_cop.ee_utilities
    - containers.podman
    - awx.awx
  wait: true
...
```

Note: We have ah_repository_certified commented out at this time due to token issues.

Further documentation for those who are interested to learn more see:

- [ah config repository](https://github.com/redhat-cop/ah_configuration/blob/devel/roles/repository/README.md)

## Step 3

Create a file `group_vars/all/ah_users.yml` make sure this user has `is_superuser` set to `true` and their `password` is set to `"{{ ah_token_password }}"`.

```yaml
---
ah_token_username: token_user
ah_users:
  - username: token_user
    groups:
      - "admin"
    append: true
    state: "present"
...
```

Further documentation for those who are interested to learn more see:

- [ah config users](https://github.com/redhat-cop/ah_configuration/blob/devel/roles/user/README.md)

## Step 4

Create a file `group_vars/all/ah_groups.yml` and add `ah_groups` list with one (1) item in it with the `name` of `admin` who's `perms` are `all` and `state` is `present`.
If you need more information expand out this section that was copied from the group role's readme in ah_configuration.

<details>
  <summary>More info</summary>

### Variables

|Variable Name|Default Value|Required|Type|Description|
|:---:|:---:|:---:|:---:|:---:|
|`name`|""|yes|str|Group Name. Must be lower case containing only alphanumeric characters and underscores.|
|`perms`|""|yes|str|The list of permissions to add to or remove from the given group. See below for options.|
|`state`|`present`|no|str|Desired state of the group.|
<!-- |`new_name`|""|yes|str|Setting this option will change the existing name (looked up via the name field.| -->

#### perms

The module accepts the following roles:

- For user management, `add_user`, `change_user`, `delete_user`, and `view_user`.
- For group management, `add_group`, `change_group`, `delete_group`, and `view_group`.
- For collection namespace management, `add_namespace`, `change_namespace`, `upload_to_namespace`, and `delete_namespace`.
- For collection content management, `modify_ansible_repo_content`, and `delete_collection`.
- For remote repository configuration, `change_collectionremote` and `view_collectionremote`.
- For container image management, only with private automation hub v4.3.2
  or later, `change_containernamespace_perms`, `change_container`,
  `change_image_tag`, `create_container`, `push_container`, and `delete_containerrepository`.
- For task management, `change_task`, `view_task`, and `delete_task`.
- You can also grant or revoke all permissions with `*` or `all`.

### Standard Project Data Structure

#### Yaml Example

```yaml
---
ah_groups:
  - name: group1
    state: present
```

</details>

Further documentation for those who are interested to learn more see:

- [ah config groups](https://github.com/redhat-cop/ah_configuration/blob/devel/roles/group/README.md)

## Step 5

Create a playbook `hub_config.yml` and `include` the `repository` role as the first task and `include` the `user` role as the last task.

```yaml
---
- name: configure private automation hub after installation
  hosts: all
  gather_facts: false
  connection: local
  vars_files:
    - "vault.yml"
  tasks:
    - name: include repository sync role
      ansible.builtin.include_role:
        name: redhat_cop.ah_configuration.repository_sync

    - name: include group role
      ansible.builtin.include_role:
        name: redhat_cop.ah_configuration.group
      when: ah_groups | length is not match('0')
...
```

## Step 6

The next step is to run the playbook, for demonstration purposes we are going to show how to get the Execution Environment(EE) that was built in the previous step and run the playbook.

If you wish to skip this step run the playbook this way[^1].

[^1]: `ansible-galaxy install redhat_cop.ah_configuration` then `ansible-playbook -i inventory.yml -l automationhub hub_config.yml`

Login to the automation hub using the podman login command. This will ask for a user:pass. After authenticating pull the config_as_code image.

```console
podman login --tls-verify=false hub.node
podman pull --tls-verify=false hub.node/config_as_code:latest
```

Ansible navigator takes the following commands.
The options used are

|CLI Option|Use|
|:---:|:---:|
|`eei`|execution environment to use.|
|`i`|inventory to use.|
|`pa`|pull arguments to use, in this case ignore tls.|
|`m`|which mode to use, defaults to interactive.|

Use these options to run the playbook in the execution environment.

```console
ansible-navigator run hub_config.yml --eei hub.node/config_as_code -i inventory.yml -l automationhub --pa='--tls-verify=false' -m stdout
```

[previous task](task1.md) [next task](task3.md)
