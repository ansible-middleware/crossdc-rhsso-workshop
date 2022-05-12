# 9 - Testing

To test our deployment we'll add a validate section to our playbook all.yml.  This step will attempt to connect to the Single Sign-On service and check integrity. 

First thing we'll do is add a post_tasks section at the end of all.yml.

```
  post_tasks:
    - include_tasks: validate.yml
      
```

This task requires a file validate.yml, so let's create this file in the same level as jboss.yml. Paste the following into validate.yml:

```
---
TODO
```

To verify reliability, we will first terminate one DataGrid and one Single Sign-On instance, then check the authorization frontend URL is still available.

```
---
TODO
```

Save changes and re-run the playbook.

`ansible-navigator run -i ./inventory/hosts all.yml --extra-vars "rhn_username=<your rhn login> rhn_password=<your rhn password>"`

The Ansible playbook should output the http response from the addressbook application, the script will then complete successfully.

Now that we have our test in place, all that's left is to combine these playbooks into a single playbook.

Next [Step 10](./10-conclusion.md)

