# 7 - Deploying JBoss Core Services

Now that the application is deployed, we can access the application by connecting to each individual instance.  But what we really need is a load balancer to sit in front of these two nodes, providing a single IP address to access all instances of the application.  JBoss Core Services will provide this functionality using the [mod_cluster](https://www.modcluster.io/) module.

To deploy JBoss Core Services, we've provided a zip file containing the role required to deploy jbcs.  To add this role the the project, run the following command:

`unzip workshop/jbcs.zip -d roles`

We'll create a playbook to use this role.  Create a file called jbcs.yml in the top level folder.  Copy the following snippet to the top of the file:

```
---
- name: Playbook for JBCS Hosts
  hosts: jbcs
  become: true
  collections:
    - middleware_automation.redhat_csp_download
  roles:
    - redhat_csp_download
    - jbcs
```

Save this file, and test the playbook by running the following command:

`ansible-navigator run -i ./inventory/hosts jbcs.yml  --extra-vars "rhn_username=<your rhn login> rhn_password=<your rhn password>"`


# Testing the JBCS installation

To test the JBoss Core Services are installed correctly, use your browser and navigate to the external hostname of your JBoss Core Server.

e.g. `https://frontend1.guid01.domainname.com`

You should see the default apache landing page.

![default apache landing page](../images/apache.png)

# Using mod_cluster to configure JBCS

TODO

# Testing the JBCS installation

Once the playbook has completed, you should be able to access the application by connecting to the load balancer with:

 `https://<load balancer ip address>/auth/`

 ![sso admin console](../images/)

 Next [Step 8](./08-scaling-up.md)