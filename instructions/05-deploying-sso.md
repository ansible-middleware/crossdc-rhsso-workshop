# 5 - Deploying Single Sign-On

Next thing we're going to do is deploy Single Sign-On.  To do this we'll create a playbook by creating the file `playbooks/rhsso.yml`.

Copy the contents of the following file into datagrid.yml:

```
---
- name: Playbook for rhsso Hosts
  hosts: rhsso
  become: true
  collections:
    - middleware_automation.redhat_csp_download
    - middleware_automation.keycloak
  roles:
    - redhat_csp_download
    - role: keycloak
      keycloak_admin_password: "{{ admin_pass }}"
```

Very similarily to what we did for deploying DataGrid, in here we are only setting the console administrator user password, and invoking the execution of the `redhat_csp_download` and `keycloak` roles.
Indeed, most of the parameters we need to override are included in the group_vars for rhsso, as follows:

```
---
keycloak_ha_enabled: True
keycloak_modcluster_url: "loadbalancer0{{ domain_name }}"
keycloak_frontend_url: "https://{{keycloak_modcluster_url}}/auth"
keycloak_jdbc_engine: mariadb
keycloak_db_user: keycloak-user
keycloak_db_pass: keycloak-pass
keycloak_rhsso_enable: True
keycloak_configure_firewalld: True

admin_pass: "redhat1!but12long"

infinispan_user: rhsso
infinispan_pass: rhssopassword
infinispan_use_ssl: True
```

In this file, we start by enabling the clustering with `keycloak_ha_enabled`; then we setup credentials for MariaDB (with `keycloak_db_*` and `keycloak_jdbc_*` variables), and DataGrid connectivy (`infinispan_*` parameters). 

`keycloak_configure_firewalld` enables the configuration for opening the cluster ports, to allow cluster nodes to connect to each other. _Note_: when deploying to a cloud environment, configuring the
firewall on the system is not usually enough, ports must also be made available via infrastructure configuration (ie. "security groups").

`keycloak_frontend_url` and `keycloak_modcluster_url` are important settings which tell Single Sign-On respectively what is the exposed public URL for the service, and the internal URL for connecting to load-balancer, which also reverse proxy the service: in our case we use Red Hat JBoss Core Service as a reverse proxy, and we will deploy in the following Step 07.

Finally, we set the JDBC url for Single Sign-On per each site, following what we did for DataGrid, in `group_vars/site1.yml`:

```
keycloak_jdbc_url: jdbc:mariadb:sequential://site1-database1,site2-database1:3306/keycloak
```

and `group_vars/site2.yml`

```
keycloak_jdbc_url: jdbc:mariadb:sequential://site2-database1,site1-database1:3306/keycloak
```

Note how we use the local site mariadb instance in the first position of the list in the JDBC url: that assign a preference to connect to the local lower latency host.

You can find the full documentation with all parameters and more details [here](https://ansible-middleware.github.io/keycloak/1.0.5/roles/keycloak.html). 


## Running the playbook

We are now ready for deploying our Single Sign-On cluster; for starters, locate the archive zipfile downloaded from the Red Hat Customer portal, and copy it in the root directory of the workshop:

`cp <path_to_downlaoded_archive>/ ./`

finally, run the playbook use following command (you will need your Red Hat network credentials as discussed in section 01): 

`ansible-navigator -i ./inventory/hosts playbooks/rhsso.yml --extra-vars "rhn_username=<your rhn login> rhn_password=<your rhn password>"`

Once this command completes, you should see something like:

```

PLAY RECAP ***************************************************************************************************************
192.168.122.224            : ok=51   changed=6    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0   
192.168.122.64             : ok=51   changed=6    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0   
192.168.123.103            : ok=51   changed=6    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0   
192.168.123.22             : ok=51   changed=6    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0 

```


Next [Step 6](./06-create-sso-realm.md) will configure an Authentication Realm.


