# 8 - Scaling up the deployment


## Adding additional nodes to the DataGrid and Single Sign-On clusters

Let's go back to our initial inventory:

```
[crossdc]

[crossdc:children]
sites
database

[sites:children]
site1
site2

[scenario:children]
datagrid
rhsso
loadbalancer
database

[site1]
site1-datagrid1
site1-datagrid2
site1-rhsso1
#site1-rhsso2
site1-database1
#site1-database2

[site2]
site2-datagrid1
site2-datagrid2
site2-rhsso1
#site2-rhsso2
site2-database1

[datagrid]
site1-datagrid1
site1-datagrid2
site2-datagrid1
site2-datagrid2

[rhsso]
site1-rhsso1
#site1-rhsso2
site2-rhsso1
#site2-rhsso2

[database]
site1-database1
#site1-database2
site2-database1

[loadbalancer]
loadbalancer0
```

Then, we will edit it to achieve the following:

- add a third mariadb instance to join the galera cluster, resulting in a more robust database service (ie. 3 nodes is the very minimum, read more here https://severalnines.com/database-blog/how-deploy-mariadb-cluster-high-availability )
  - uncomment `site1-database2` from the `site1` and `database` groups
- add intra-region fault-tolerance by adding a second Single Sign-On instance per site
  - uncomment `site1-rhsso2` from the `site1` and `rhsso` groups, and similarly uncomment the `site2-rhsso2` lines.

What about the configuration? Indeed there isn't much additional configuration to setup, as we already have the rhsso instances in a cluster, and we already have a cluster of mariaDB/galera nodes;
eventually, the following JDBC url is the only change we will need to make, in `group_vars/site1.yml`:

```
keycloak_jdbc_url: jdbc:mariadb:sequential://site1-database1,site1-database2,site2-database1:3306/keycloak
```

and `group_vars/site2.yml`

```
keycloak_jdbc_url: jdbc:mariadb:sequential://site2-database1,site1-database1,site1-database2:3306/keycloak
```

That is, adding the new database endpoint to the list of nodes for rhsso and datagrid.

Save those files, and execute the playbook by running the following command:

    ansible-navigator run -i ./inventory/hosts crossdc.yml  --extra-vars "rhn_username=<your rhn login> rhn_password=<your rhn password>"

The execution should return changes for the three new instances, it's brand new deployments after all, and no other change for the already running instances.


Next [Step 9](./09-testing.md)
