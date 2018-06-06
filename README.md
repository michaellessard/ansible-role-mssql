Role Name
=========

This role will perform the following:
- Install and uninstall of the Microsoft SQL Server
- Create, delete, or import (from .sql file) a database
- MSSQL command line tools for RHEL 7

Requirements
------------

In order for this role to work, you need some core repositories configured for your RHEL instance. If running this in a public cloud provider, this has likely already been done for you. If necessary, register the system to Red Hat's content repositories or Red Hat Satellite using `subscription-manager`.


Role Variables
--------------

The only variables in Defaults are around the mssql packages and should not need to be changed. Pip is also included in order to handle the expect module for accepting the EULA.

Within Vars, you must explicitly agree to the End User's License Agreement for both the server setup script and the command line tools. To do this, add Y or YES where applicable to the variables for each EULA.

The default user is 'SA' when logging in via command line tools. The SA user is mandatory for initial creation, this role does not currently offer the ability to create additional users.

You will have to specify (or get) the user directory (see the full example). 

Additionally, there are some predefined default values including:
```yaml
# These are required for database installation
end_user_license_aggreement_consent_server: # Must be Y or N
end_user_license_aggreement_consent_cli: "" # Must be YES or NO in all caps within quotes
edition: Developer
db_user: SA

# For use when creating, importing, or deleting databases
db_name:
db_host: 127.0.0.1
db_port: 1433

import_file:
import_file_dest:

#System Config options
enable_iptables: false
install_cli: true

```
I would strongly recommend modifying these for anything beyond a basic proof of concept.

Probably the best way to approach this is to copy these values to an extra vars file and including it in your playbook or by running them from the command line like so:
`ansible-playbook site.yml -e @extra_vars.yml`

Dependencies
------------

No additional galaxy roles are required.

Example Playbook
----------------

To use the default installation tasks:

    - hosts: db
      roles:
         - { role: michaellessard.mssql }

To use the installation and create a new db, I usually give the service a few seconds to come up before attempting to login:
```yaml
---
- hosts: db
  become: yes
  roles:
    - { role: michaellessard.mssql }
  tasks:
    - name: Wait up to 60 seconds for server to become available after creation
      wait_for:
        port: 1433
        timeout: 60
    - name: Create new db
      include_role:
        name: michaellessard.mssql
        tasks_from: new_db
```

To use the uninstall tasks:
```yaml
---
- hosts: db
  name: Removes mssql-server
  become: yes

  tasks:
  - name: Run remove tasks from mssql-server role
    include_role:
      name: michaellessard.mssql
      tasks_from: uninstall
```

A complete example that will install mssql, a db and the command line tool 
```yaml
---
- name: Take user directory information
  hosts: db
  tasks:
    - name: set userdirectory fact
      set_fact:
        userdirectory: "{{ ansible_user_dir}}"
    - debug: var=userdirectory

- name: install MsSQL on RHEL
  hosts: db
  become: yes
  vars:
    db_name: ansibleDemo
    db_host: localhost
    db_user: SA
    db_password: 1qaz2wsX
  roles:
    -  { role: michaellessard.mssql }
  tasks:
    - name: Pause for 15 seconds for DB connection to come up
      pause:
        seconds: 15
    - name: Create new database
      mssql_db:
        name: "{{ db_name }}"
        state: present
        login_host: "{{ db_host }}"
        login_user: "{{ db_user }}"
        login_password: "{{ db_password }}"

```

To test mssql : 

```
$ sqlcmd -S localhost -U SA
1> SELECT Name from sys.Databases
2> GO 
```



License
-------

BSD

Author Information
------------------

Contributions and issues with this role are welcome at the associated git repo.
