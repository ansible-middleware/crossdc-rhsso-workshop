# 6 - Creating a SSO authentication realm

Now that Single Sign-On is deployed and configured, we can create a realm for client application to authenticate user accounts.  To do this we will use the `keycloak_realm` role provided by the `keycloak` collection. 

First thing we'll do is adding the details of the realm we are going to create.  We'll need to provide the following variables to `inventory/group_vars/rhsso.yml`:

```
keycloak_realm: TestRealm
keycloak_clients:
    - name: TestClient1
      roles:
        - TestClient1Admin
        - TestClient1User
      realm: "{{ keycloak_realm }}"
      public_client: True
      web_origins:
        - http://testclient1origin/application
        - http://testclient1origin/other
      users:
      - username: TestUser
        password: password
        client_roles:
          - client: TestClient1
            role: TestClient1User
            realm: "{{ keycloak_realm }}"

```

Next we'll add the following post tasks to the rhsso.yml playbook:

```
post-tasks:
  - name: Keycloak Realm Role
    ansible.builtin.include_role:
      name: keycloak_realm
```

Save changes and re-run the playbook. Once the playbook is complete, you should be able to access the application and navigate to the `TestRealm` realm.


# Testing the realm

To test the install, use the Browser Preview and open the url http://app1.guid01.internal:8080/auth/ : you should be able to authenticate as administrator, and then select the `TestRealm` in the top-left menu.

![TestRealm page](../images/realm-selection.png)

Now that the application is deployed, let's move on to the next section, deploying the load balancer.

Next [Step 7](./07-deploying-jbcs.md)