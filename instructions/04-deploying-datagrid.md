# 4 - Deploying DataGrid

Next thing we're going to do is deploy DataGrid.  To do this we'll create a playbook by creating the file `playbooks/datagrid.yml`.

Copy the contents of the following file into datagrid.yml:

```
---
- name: Playbook for JDG Hosts
  hosts: datagrid
  become: true
  collections:
    - middleware_automation.redhat_csp_download
    - middleware_automation.infinispan
  roles:
    - redhat_csp_download
    - role: infinispan
      infinispan_users: "{{ user_accounts }}"
      supervisor_password: "{{ admin_pass }}"
```

Have a look at the yaml we just copied, it's relatively straight forward, as it is only invoking the redhat_csp_download and infinispan roles from the Execution Environment collections. 

You can find the full documentation with all parameters [here](https://ansible-middleware.github.io/infinispan/1.0.3/roles/infinispan.html). In our deployment, we will rely on most defaults values,
except those needed to activate the high-availability, database persistence, and caches to serve for the Single Sign-On service. Open the `inventory/group_vars/datagrid.yml` file, and paste the
following:

```
---
admin_pass: admin_password
user_accounts:
  - { name: 'adminuser', password: 'adminpassword', roles: 'admin' }
  - { name: 'rhsso', password: 'rhssopassword', roles: 'application' }
infinispan_keycloak_caches: True
jdg_enable: True
jdg_configure_firewalld: True
jdg_keycloak_persistence: True
jdg_jgroups_relay: True
jdg_jgroups_relay_sites:
  - site1
  - site2

jdb_db_user: keycloak-user
jdg_db_pass: keycloak-pass

jdg_default_realm_tls: True
jdg_keystore_path: /etc/pki/java/keystore.jks
jdg_keystore_alias: "*.rhssocrossdc.com"

install_keystore: True

jdg_bind_address: "{{inventory_hostname}}{{domain_name}}"
jdg_url: "https://{{inventory_hostname}}{{domain_name}}:11222"
```

On the first lines here, we set the administrator password, and define two additional user accounts, one admin and one application account for Single Sign-On; then:

* we enable initialization for Single Sign-On caches
* we enable the config for the firewall
* JDBC persistence
* cross datacenter relaying, defining two sites
* parameters for secure TLS communication

But we are not finished, we now need to create configuration which is not only specific to DataGrid, but to DataGrid depending on which site will be deployed among site1 and site2.
To do this open the `inventory/group_vars/site1.yml` and `inventory/group_vars/site2.yml` and add for site1:

```
jdg_jgroups_relay_site: site1
jdg_jdbc_url: jdbc:mariadb:sequential://site1-database1,site1-database2,site2-database1:3306/keycloak
```

and similarily for site2:

```
jdg_jgroups_relay_site: site2
jdg_jdbc_url: jdbc:mariadb:sequential://site2-database1,site1-database1,site1-database2:3306/keycloak
```

The content of those group_vars will assign the DataGrid instance to their respcetive datacenter, and set the JDBC connection url to the clustered MariaDB instances.


## Running the playbook

We are now ready for deploying our DataGrid cluster; for starters, locate the archive zipfile downloaded from the Red Hat Customer portal, and copy it in the root directory of the workshop:

`cp <path_to_downlaoded_archive> ./`

finally, run the playbook use following command (you will need your Red Hat network credentials as discussed in section 01): 

`ansible-navigator -i ./inventory/hosts playbooks/datagrid.yml --extra-vars "rhn_username=<your rhn login> rhn_password=<your rhn password>"`

Once this command completes, you should see something like:

```

PLAY RECAP ***************************************************************************************************************
192.168.122.224            : ok=51   changed=6    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0   
192.168.122.64             : ok=51   changed=6    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0   
192.168.123.103            : ok=51   changed=6    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0   
192.168.123.22             : ok=51   changed=6    unreachable=0    failed=0    skipped=11   rescued=0    ignored=0 

```

## Testing the installation

To test the install, click the "Browser Preview" tab and enter the url http://app1.guid01.internal:8080/ in the address bar.  You should see the following page:

![DataGrid console](../images/datagrid-console.png)

You should see a html response showing the default DataGrid console landing page.

Now that DataGrid is installed, let's move on to the next section, installing Single Sign-On.

Next [Step 5](./05-deploying-sso.md)