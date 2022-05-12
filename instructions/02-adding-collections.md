# 2 - Preparing

This workshop uses the following collections:

* [middleware_automation.keycloak](https://ansible-middleware.github.io/keycloak/main/): This collection is used to perform configuration of the Single Sign-On instances.
* [middleware_automation.infinispan](https://ansible-middleware.github.io/infinispan/main/): This collection is used to perform the installation and configuration of DataGrid instances.
* [middleware_automation.redhat_csp_download](https://ansible-middleware.github.io/redhat-csp-download/latest/): This collection is used to perform the installation and configuration of runtimes. In this workshop we will only in part rely on it, because we will provide pre-downloaded archives from the Customer Portal.
* [community.mysql](https://docs.ansible.com/ansible/latest/collections/community/postgresql/index.html): This collection is used by our role to configure MariaDB.
* [community.general](https://docs.ansible.com/ansible/latest/collections/community/general/index.html ): This is the general ansible collection containing most general purpose ansible modules.
* [ansible.netcommon](https://docs.ansible.com/ansible/latest/collections/ansible/netcommon/index.html ): This collection provides network configuration and setup modules.

The first three collection are provided automatically by the Execution Environment we downloaded via ansible-navigator; to add the extra collections to your project, copy and paste the following in the file collections/requirements.yml

```
---
collections:
  - name: community.general
  - name: community.mysql
  - name: ansible.netcommon
```

Save changes to this file, and run the following command to install the collections: 

`ansible-navigator exec -m stdout -- ansible-galaxy collection install -r collections/requirements.yml -p collections`

Once this command has completed, you should see the following output:

```
Process install dependency map
Starting collection install process
Installing 'community.general:4.6.0' to '/home/devops/crossdc-rhsso-workshop/collections/ansible_collections/community/general'
Installing 'community.mysql:3.1.3' to '/home/devops/crossdc-rhsso-workshop/collections/ansible_collections/community/mysql'
[...]
```

To check the status of the collections, run the following command: 

`tree -L 2 ~/.ansible/collections/ansible_collections/`

Once this command has completed, you should see the following output:
```
/home/devops/crossdc-rhsso-workshop/collections/ansible_collections/
├── ansible
│ ├── netcommon
│ ├── posix
│ └── utils
└── community
    ├── general
    └── mysql

```

The collections are installed, we'll now proceed to the next step, installing a postgresql database.

Next [Step 3](./03-configuring-postgresql.md)