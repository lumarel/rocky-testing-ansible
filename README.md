# Ansible Rocky Testing tools

This ansible-playbook takes care of any tasks that may appear for Rocky testing. These are intended to be run on a AWX instance.

## Structure

```text
├── ansible.cfg
├── collections
│   └── requirements.yml
├── playbooks
│   ├── files # unchanged files for copy
│   ├── handlers
│   ├── roles
│   │   └── <role-name>
│   ├── tasks
│   ├── templates
│   └── vars
└── roles
    └── requirements.yml
```

## Designing Playbooks

### Ansible version usage

Used Ansible version is 5.0 (ansible-core 2.12)

Due to this also the FQCNs are used, to not run in any problems with same naming.

### Comments

Each playbook should have comments or a name descriptor that explains what the playbook does or how it is used. If not available, README-... files can be used in place, especially in the case of adhoc playbooks that take input. Documentation for each playbook/role does not have to be on this wiki. Comments or README's should be sufficient.

### Tags

Ensure that you use relevant tags where necessary for your tasks.

### Roles and Collections

If you are using roles or collections, you will need to list them in `./roles/requirements.yml`. For example, we use the `freeipa` collection and a `mysql` role from `geerlingguy`.

```yaml
---
roles:
  - name: geerlingguy.mysql
```

```yaml
---
collections:
  - name: freeipa.ansible_freeipa
    version: 0.3.1
```

**Note**: There will be cases where you should and must specify the version you're working with, depending on the author and the amount of changes that may occur. There may be a future policy that you have to lock onto a specific version.

### Ad-hoc tasks

If the playbook gets executed in an ad-hoc manner from time to time, please use the add_inventory_from_variable task, as this serves the function of not needing to have a static inventory, which needs to be updated every time. Implement this with the following playbook task in your playbook:

```yaml
- name: Add all hosts from input var to temp inventory
  hosts: localhost
  gather_facts: false
  tasks:
    - name: Add inventory
      import_tasks: tasks/add_inventory_from_variable.yml
```

The variable name is by default `input_hostnames`. Due to this either a empty or a inventory with only the localhost can be used.

### Asserting variables

Always make sure that all variables are given for a role or if tasks are directly in a playbook, that they are available, to not run in any unknown states. For this please use the asserting feature of Ansible:

```yaml
- name: Check if all needed variables are available
  ansible.builtin.assert:
    that:
      - input_hostnames | mandatory
```

## Pipelines / linting

If you are making changes in another branch than master, it is necessary to check the code before a PR gets completed. This is a absolute requirement, so the code quality stays good.
Due to this requirement code linting via the Azure-Pipelines server is configured via the azure-pipelines.yml file, which should have the following content:

```yaml
kind: pipeline
type: docker
name: default

steps:
  - name: ansible-check
    image: quay.io/lumarel/ee-lint:stable-2.12
    commands:
      - "[ -f './collections/requirements.yml' ] && ansible-galaxy install -r ./collections/requirements.yml || echo 'No requirements file active.'"
      - "[ -f './roles/requirements.yml' ] && ansible-galaxy install -r ./roles/requirements.yml || echo 'No requirements file active.'"
      - "find ./playbooks -maxdepth 1 -name '*.yml' | grep .yml | xargs ansible-playbook --syntax-check --list-tasks"
      - "find ./playbooks -maxdepth 1 -name '*.yml' | grep .yml | xargs ansible-lint"
```

This uses a versioned number of the Ansible Toolset, which will have to be updated from time to time.
