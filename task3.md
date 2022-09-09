# Intro

In this section, you will only be given a summary of the objects you need to create along with some screenshots of a controller that is configured with the completed code. You will also be provided the variables sections from the readme's for each of the required roles to help you complete this task.

## Step 1

Create a file `group_vars/all/projects.yml` and add the required information to the list `controller_projects` to configure the UI to look like the screenshot

```yaml
---
controller_projects:

...
```

![project](assets/images/projects.png)

Further documentation for those who are interested to learn more see:

- [projects role](https://github.com/redhat-cop/controller_configuration/blob/devel/roles/projects/README.md)

## Step 2

Create a file `group_vars/all/credential_types.yml` and add the required information to the list `controller_credential_types` to configure the UI to look like the screenshot

```yaml
---
controller_credential_types:
  - name: ssh_priv_file
    kind: cloud
    description: creates temp ssh priv key to use
    inputs:
      fields:
        - id: priv_key
          type: string
          label: Certificate
          format: ssh_private_key
          secret: true
    injectors:
      env:
        MY_CERT_FILE_PATH: !unsafe '{{ tower.filename.cert_file }}'
      file:
        template.cert_file: !unsafe '{{ priv_key }}'
...
```

![credential_type_input](assets/images/cred_type_input.png)
![credential_type_injector](assets/images/cred_type_injector.png)

Further documentation for those who are interested to learn more see:

- [credential types role](https://github.com/redhat-cop/controller_configuration/blob/devel/roles/credential_types/README.md)

## Step 3

Create a file `group_vars/all/credentials.yml` and add the required information to the list `controller_credentials` to configure the UI to look like the screenshot

```yaml
---
controller_credentials:
  - name: aap_admin
    credential_type: Red Hat Ansible Automation Platform
    organization: config_as_code
    description: aap admin account
    inputs:
      host: "{{ controller_hostname }}"
      username: "{{ controller_username }}"
      password: "{{ controller_password }}"
      verify_ssl: false

  - name: ah_certified
    credential_type: Ansible Galaxy/Automation Hub API Token
    organization: config_as_code
    inputs:
      url: "https://{{ ah_host }}/api/galaxy/content/rh-certified/"
      token: "{{ ah_token }}"

  - name: ah_published
    credential_type: Ansible Galaxy/Automation Hub API Token
    organization: config_as_code
    inputs:
      url: "https://{{ ah_host }}/api/galaxy/content/published/"
      token: "{{ ah_token }}"

  - name: ah_community
    credential_type: Ansible Galaxy/Automation Hub API Token
    organization: config_as_code
    inputs:
      url: "https://{{ ah_host }}/api/galaxy/content/community/"
      token: "{{ ah_token }}"

  - name: cr_ah
    credential_type: Container Registry
    organization: config_as_code
    inputs:
      host: "https://{{ ah_host }}/"
      username: "{{ ah_username }}"
      password: "{{ ah_password }}"
      verify_ssl: false

  - name: root
    credential_type: Machine
    organization: config_as_code
    description: root local password
    inputs:
      username: root
      password: "Password1234!"

  - name: vault
    credential_type: Vault
    organization: config_as_code
    description: vault password
    inputs:
      vault_password: "{{ vault_pass }}"
...
```

![credential](assets/images/cred_ah_admin.png)

Further documentation for those who are interested to learn more see:

- [credentials role](https://github.com/redhat-cop/controller_configuration/blob/devel/roles/credentials/README.md)

## Step 4

Create a file `group_vars/all/inventories.yml` and add the required information to the list `controller_inventories` to configure the UI to look like the screenshot

```yaml
---
controller_inventories:
  - name: controller_config
    description: inventory for configuring controller
    organization: config_as_code

  - name: custom_collections
    description: inventory for building custom collections
    organization: config_as_code

  - name: execution_environments
    description: inventory for building execution environments
    organization: config_as_code
...
```

![inventory](assets/images/inv_ah_config.png)

Further documentation for those who are interested to learn more see:

- [inventories role](https://github.com/redhat-cop/controller_configuration/blob/devel/roles/inventories/README.md)

## Step 5

Create a file `group_vars/all/inventory_sources.yml` and add the required information to the list `controller_inventory_sources` to configure the UI to look like the screenshot

```yaml
---
controller_inventory_sources:
  - name: controller_config_source
    organization: config_as_code
    source: scm
    source_project: config_as_code
    source_path: "inventory_{{ env }}.yml"
    inventory: controller_config
    credential: ""
    overwrite: true
    update_on_launch: true
    update_cache_timeout: 0
    wait: true

  - name: custom_collections_inv
    organization: config_as_code
    source: scm
    source_project: config_as_code
    source_path: "inventory_{{ env }}.yml"
    inventory: custom_collections
    credential: ""
    overwrite: true
    update_on_launch: true
    update_cache_timeout: 0
    wait: true

  - name: execution_environments_inv
    organization: config_as_code
    source: scm
    source_project: config_as_code
    source_path: "inventory_{{ env }}.yml"
    inventory: execution_environments
    credential: ""
    overwrite: true
    update_on_launch: true
    update_cache_timeout: 0
    wait: true
...
```

![inventory_source](assets/images/inv_source_ah_config.png)

Further documentation for those who are interested to learn more see:

- [inventory sources role](https://github.com/redhat-cop/controller_configuration/blob/devel/roles/inventory_sources/README.md)

## Step 6

Create a file `group_vars/all/job_templates.yml` and add the required information to the list `controller_templates` to configure the UI to look like the screenshot

```yaml
---
controller_templates:

...
```

![job_template_ee](assets/images/jt_build_ee.png)
![job_template_custom_collections](assets/images/jt_custom_collections.png)
![job_template_ah_config](assets/images/jt_ah_config.png)
![job_template_controller_config](assets/images/jt_controller_config.png)

Further documentation for those who are interested to learn more see:

- [job templates role](https://github.com/redhat-cop/controller_configuration/blob/devel/roles/job_templates/README.md)
