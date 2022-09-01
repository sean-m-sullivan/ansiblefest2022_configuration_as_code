# Intro

In this section you will configure your private automation hub using the code provided that is missing some critical values/information that you will have to fill in yourself, based on the requirements and looking at readme's for the roles.

## Step 1

Ensure that you have `ansible-navigator` installed on your machine.

```console
dnf install ansible-navigator
```

Further documentation for those who are interested to learn more see:

- [link1]()
- [link2]()

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

Further documentation for those who are interested to learn more see:

- [link1]()
- [link2]()

## Step 3

Create a file `group_vars/all/ah_users.yml` make sure this user has `is_superuser` set to `true` and their `password` is set to `Password1234!`.

```yaml
---
ah_users:
  - username: api
    groups:
      - "admin"
    append: true
    state: "present"
...
```

Further documentation for those who are interested to learn more see:

- [link1]()
- [link2]()

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

- [link1]()
- [link2]()

## Step 5

Create a playbook `hub_config.yml` and `include` the `repository` and `user` roles.

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

Further documentation for those who are interested to learn more see:

- [link1]()
- [link2]()
