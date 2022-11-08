# Ansible Template

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
│   ├── vars
│   └── <playbookname>.yml
└── roles
    └── requirements.yml
```

## Designing Playbooks

### Ansible version usage

Used Ansible version is 2.10+
Due to this also the FQCNs are used, to not run in any problems with same naming.

### Pre flight and post flight

At a minimum, there should be `pre_tasks` and `post_tasks` that can judge whether ansible can or has been run on a system. Some playbooks will not necessarily need this (eg if you're running an adhoc playbook to create a user). But operations done on a host should at least have these in the playbook, with an optional `handlers:` include.

```yaml
  handlers:
    - include: handlers/main.yml

  pre_tasks:
    - name: Check if ansible cannot be run here
      stat:
        path: /etc/no-ansible
      register: no_ansible

    - name: Verify if we can run ansible
      assert:
        that:
          - "not no_ansible.stat.exists"
        success_msg: "We are able to run on this node"
        fail_msg: "/etc/no-ansible exists - skipping run on this node"

  # Import roles/tasks here

  post_tasks:
    - name: Touching run file that ansible has ran here
      file:
        path: /var/log/ansible.run
        state: touch
        mode: '0644'
        owner: root
        group: root
```

### Comments

Each playbook should have comments or a name descriptor that explains what the playbook does or how it is used. If not available, README-... files can be used in place, especially in the case of adhoc playbooks that take input. Documentation for each playbook/role does not have to be on this wiki. Comments or README's should be sufficient.

### Tags

Ensure that you use relevant tags where necessary for your tasks.

### Roles

If you are using roles or collections, you will need to list them in `./roles/requirements.yml` or `.collections/requirements.yml`. For example, we use the `freeipa` collection and a `mysql` role from `geerlingguy`.

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

### Pre-commits / linting

When pushing to your own forked version of this repository, pre-commit must run to verify your changes. They must be passing to be pushed up. This is an absolute requirement, even for roles.

When the linter passes, the push will complete and you will be able to open a PR.
