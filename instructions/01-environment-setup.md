# 1 - Prerequisites

You will need an Red Hat network account to be able to download the DataGrid and Single Sign-On installation zip files.  
The account you use must not be managed by SSO in order for the Ansible scripts to be able to authenticate.  You can create a new account by following the instructions in the [Red Hat Customer Portal](https://sso.redhat.com/auth/realms/redhat-external/login-actions/registration?client_id=customer-portal&tab_id=RiPOv96eZ74).  You will need to use these credentials when you run the ansible playbooks later in the workshop.

## Set up the environment

This workshop requires the provisioning of the following RHPDS environment.  Once this environment is provisioned your will have the following:

* A bastion node, serving a VS Code Server IDE, which can be used to run the workshop
* 4 x App Servers which will host Data Grid
* 4 x App Servers which will host Single Sign-On
* 3 x DB Server, which will host MariaDB
* 1 x Frontend Server, which will host JBCS as load balancer

To start the workshop, open the url of your bastion server in a browser. You will be prompted to login.  The default password is `remembertochangeme`.  Once you login to the IDE you will be able to clone the workshop repository.

Open the git panel of the IDE and click on "Clone Repository" and enter the following url:

https://github.com/ansible-middleware/crossdc-rhsso-workshop.git

![VS Code git panel](../images/git.png)

You will be prompted to select the folder to clone the repo to, choose the default folder, `/home/devops`

You will then be prompted to open the folder in VS Code.


## Prepare the execution environment

For this workshop we will rely on [ansible-navigator](https://github.com/ansible/ansible-navigator), and the middleware-automation execution environment located at this [URL](https://quay.io/repository/ansible-middleware/ansible-middleware-ee). Everything will run in a python (virtualenv)[https://docs.python.org/3/tutorial/venv.html] to make sure tool and library versions are compatible with the time of writing this workshop; while the execution environment image, aka the ansible controller node, will run inside a podman container.


### Create the python virtualenv and install ansible-navigator

From the VS Code IDE, open a new terminal by clicking on the "Terminal" menu and selecting "New Terminal"; then ytpe the following commands:

* `dnf install python3 python3-virtualenv podman` to install the required RPM packages
* `virtualenv --python python3 venv` to create the virtual environment directory using the default python 3.x 
* `source venv/bin/activate` to load the virtual environment
* `pip install ansible-navigator` to install ansible-navigator into the virtual environment

and finally to check the installation was successful:

`ansible-navigator --version` should return something like `ansible-navigator 2.x.y`


### Configure ansible-navigator

Now it is time to configure ansible-navigator defaults, and instruct it to use the middleware-automation execution environment; do to so create the file `ansible-navigator.yml`, and populate as follows:

```
ansible-navigator:
  execution-environment:
    container-engine: podman
    image: quay.io/ansible-middleware/ansible-middleware-ee:latest
  

```

Verify the configuration by issuing the command:

```ansible-navigator collections```

You should see navigator fetching the images from quay.io, and finally display the included collections:


```
 NAME                                      VERSION SHADOWED TYPE      PATH                                                       
0│ansible.posix                             1.3.0      False contained /usr/share/ansible/collections/ansible_collections/ansible 
1│community.general                         4.5.0      False contained /usr/share/ansible/collections/ansible_collections/communi 
2│middleware_automation.infinispan          1.0.1      False contained /usr/share/ansible/collections/ansible_collections/middlew 
3│middleware_automation.jcliff              0.0.21     False contained /usr/share/ansible/collections/ansible_collections/middlew 
4│middleware_automation.jws                 0.0.3      False contained /usr/share/ansible/collections/ansible_collections/middlew 
5│middleware_automation.keycloak            1.0.0      False contained /usr/share/ansible/collections/ansible_collections/middlew 
6│middleware_automation.redhat_csp_download 1.2.1      False contained /usr/share/ansible/collections/ansible_collections/middlew 
7│middleware_automation.wildfly             1.0.1      False contained /usr/share/ansible/collections/ansible_collections/middlew 
```


## Check ansible hosts

From the VS Code IDE, open a new terminal by clicking on the "Terminal" menu and selecting "New Terminal".

From the terminal enter the follwing command to copy over the pre-prepared ansible hosts file:

```cp /etc/ansible/hosts ./inventory```


Open the file `inventory/hosts`, you should see the hostnames of your nodes similar to those shown below; if not amend as necessary:

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

You will notice a few hosts are commented; in a later step we will use those additional hosts to demonstrate scaling-up the deployment by adding more resources.


## Test ansible hosts

Once we have checked the hosts file we can test the ansible hosts file by running the following command.

```ansible-navigator run -i inventory/hosts -m stdout playbooks/ping.yml```

You should see the following output:

```
PLAY [Ping all crossdc hosts] ***********************************************************************************

TASK [Gathering Facts] ******************************************************************************************
ok: [site2-datagrid1]
ok: [site1-datagrid2]
ok: [site1-datagrid1]
ok: [site2-datagrid2]
ok: [loadbalancer0]
[...]

TASK [debug] ****************************************************************************************************
ok: [site1-datagrid1] => {
    "msg": "Ansible Ping on site1-datagrid1"
}
ok: [site1-datagrid2] => {
    "msg": "Ansible Ping on site1-datagrid2"
}
[...]
```

We've checked our environment, now we can continue with the workshop and add the required ansible collections.

Next [Step 2](./02-adding-collections.md)