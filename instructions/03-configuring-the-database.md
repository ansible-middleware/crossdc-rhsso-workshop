# 3 - Installing mariadb

Our workshop uses a mariadb database configured in an active replication cluster using galera/wsrep.  
We will use the mariadb role to install and configure the mariadb database.  

First thing we need to do is create a "roles" folder.  To do this, run the following command from the root of the workshop folder:

    mkdir roles

We have included the boiler plate mariadb role in an archive in the workshop folder.  We will extract this archive to the `roles` folder.  To do this, run the following command:

`unzip workshop/mariadb.zip -d roles`

You should now see a `roles/mariadb` folder.  We shouldn't need to make any changes to these files.  

Configuring a clustered database service is not a straight-forward tasks, and while it is necessary for the workshop to run a highly available Single Sign-On service on it, it goes
beyond the scope of this tutorial. Yet, before we move on, let's take a look at what these files are doing.

There are two main files we'll look at:

* defaults/main.yml
* tasks/main.yml

## Reviewing the mariadb role

### defaults/main.yml

The `defaults/main.yml` file is the main file that contains the default variables that are used by the mariadb role.  

Included in these variables are the following:

* installation directories
* mariadb configuration options
* memory settings
* db variables containing databases, users to create
* cluster settings

From the full set of defaults, we are setting only a subset for our installation, in the file `inventory/group_vars/database.yml`:

```
---
galera_enable_mariadb_repo: False
galera_cluster_name: 'rhsso-db-cluster'
galera_cluster_nodes_group: 'database'
mariadb_charset_server: utf8
mariadb_collation_server: utf8_general_ci
mariadb_charset_client: utf8
mariadb_mysql_root_password: ''
mariadb_databases:
  - name: keycloak
mariadb_mysql_users:
  - name: keycloak-user
    hosts: "{{ (groups['rhsso'] + groups['datagrid']) | flatten | map('regex_replace', '$', '%') | list }}"
    password: keycloak-pass
    priv: keycloak.*:ALL
```

As you can see, we name the database and the cluster, we are interested into setting the database character set and collation to "utf8", and finally we create a `keycloak` database and an user account
that can access it from any hosts that will run Single Sign-On or DataGrid services.


### tasks/main.yml

The `tasks/main.yml` orchestrates the varius configuration steps needed to deploy the mariadb database. The tasks are as follows:

* Determine network setup and configure firewall ports

* Install the mariadb packages

* Deploy base configuration for the mariadb service

* Create database users, allowing access from hosts int he rhsso and datagrid groups

* Setup the clustering among the mariadb nodes

* Configure access for the adminitrator user

* Create the databases specified in `mariadb_databases`


## Installing the fastpackages role

The mariadb role requires the fastpackes role to install the packages required by mariadb.  This role is included in the workshop folder.  We will extract this archive to the roles folder.  To do this, run the following command: 

`unzip workshop/fastpackages.zip -d roles`

## Create the mariadb playbook

To use the mariadb role, we need to create a playbook.  We will create a playbook called `playbooks/mariadb.yml`.  This playbook will just list the mariadb role, as follows:

```
---
- name: Playbook for database Setup
  hosts: database
  roles:
    - mariadb
```

## Run the mariadb playbook

To execute the mariadb playbook, we will run the following command:

`ansible-navigator -i ./inventory/hosts -m stdout playbooks/mariadb.yml`

Once this command has completed, you should be able to connect to and test the mariadb service.  To do this, connect to the mariadb server using ssh.  Open inventory/hosts and find the ip address listed under the database group.  Connect to this host with the command 

`ssh appdb1.guid01.internal`

Once connected to the mariadb server, you should be able to connect to the mariadb service using the user account we configured above in the group_vars for the database. Type the following command: 

`mysql -u keycloak-user keycloak` 

and at the password prompt type: `keycloak-pass`

Once connected to mariadb, you should see the following output:

```
mariadb > 
```

At this prompt you should be able to list the created databases.  To do this, run the following command:

`SHOW DATABASES;`

Once this command has completed, you should see the following output:

```
+--------------------+
| Database           |
+--------------------+
| mysql              | 
| keycloak           |
| test               |
 +-------------------+
```

This shows that mariadb is installed and configured correctly with the correct `keycloak` databases created.

To quit the mariadb client, enter the following command:

`\q`

Our database is now installed and configured, we can now move on to the next step, installing DataGrid.

Next [Step 4](./04-deploying-datagrid.md)