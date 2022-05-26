# 10 - Combining the playbooks

The last step of the workshop is to tie everything together.  To do this, fist we'll create a playbook called crossdc.yml, and paste the following into it:

```
---
- import_playbook: datagrid.yml

- import_playbook: rhsso.yml

```

This playbook contains the execution of the main service playbooks, ie. Red Hat Single Sign-On and Data Grid.

Next, we will create another top-level playbook, named all.yml, which in turn will incorporate the dependencies playbooks for certificates, mariadb and jbcs, and the crossdc.yml playbook we created in the former step.

```
---
- import_playbook: prerequisites.yml

- import_playbook: certificates.yml

- import_playbook: loadbalancer.yml

- import_playbook: database.yml

- import_playbook: crossdc.yml
```

Save those files and run the playbook with the following command:

`ansible-playbook -i ./inventory/hosts all.yml --extra-vars "rhn_username=<your rhn login> rhn_password=<your rhn password>" `

Because we made no changes to the configuration since we scaled up the infrastructure on Step 08, the execution will run making no changes to the deployments, validating the playbook idempotency.
Idempotency is a very important concept of automated deployments, as it allows to report instantly any unexpected divergence of the target deployment while setting it back to that declared in the configuration, and at the same time only apply the necessary changes to the deployment after changes are made into the configuration.

# Conclusion

You can download the completed workshop from here:  https://github.com/ansible-middleware/crossdc-rhsso-workshop/

The workshop is now complete, we've demonstrated the use of the ansible-middleware collection to deploy a multi-regional highly available Red Had Single Sign-On service, using JBoss Core Services as a load balancer and MariaDB/galera for the persistence layer.
