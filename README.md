
![Openstack](https://github.com/tulamelkii/openstack/blob/main/Openstack.png)

1 - An interface or command line for obtaining authentication information from keystone via the RESTful API. 

2 - Keystone requests authentication information from users and generates a token to return to the corresponding authentication request.

3 - The interface or command line sends a download instance request (with an authentication token) to the nova api via the RESTful API.

4 - After receiving the request, the nova api sends an authentication request to keystone to verify whether the token is a valid user and token.

5 - Keystone checks whether the token is valid. If it is valid, it returns valid authentication and the corresponding role (Note. Some operations require role permissions).

6 - After passing the certification, the nova api is linked to the database.

7 - Initialize the database entry of the newly created VM. 

8 - The nova api requests nova-scheduler via rpc.call if there is a resource (host ID) to create a VM.

9 - The nova-scheduler process listens to the message queue to receive nova api requests.

10 - nova-scheduler requests computing resources in the nova database and calculates hosts that meet the needs of creating a virtual machine using a scheduling algorithm.

11 - For hosts that correspond to the creation of virtual machines, nova-scheduler updates the information about the physical host corresponding to the virtual machines in the database.

12 - The nova scheduler sends nova-compute a corresponding request to create a virtual machine via rpc.cast.

13 - Nova-compute will receive a message about the request to create a VM from the corresponding message queue.

14 - Nova-compute requests nova-wire to get information about the VM via rpc.call. (Flavor)



15 - The Nova receiver receives the nova-compute request message from the message queue.

16 - The new explorer requests information corresponding to the virtual machine based on the message.

17 - Nova Explorer receives the relevant information about the virtual machine from the database.

18 - The new explorer sends information about the virtual machine to the message queue.

19 - Nova-compute receives information messages from the virtual machine from the corresponding message queue.

20 - Nova-compute receives an authenticated token via the RESTfull keystone API and receives the image needed to create a virtual machine via an HTTP request from the glance api.

21 - The glance api authenticates to keystone whether the token is valid and returns the verification result.

22 - The token verification is passed, and nova-compute receives information about the virtual machine image (URL).

23 - Nova-compute receives the k authentication token via the keystone RESTfull API and requests the neutron server via HTTP to obtain the network information necessary to create a virtual machine.

24 - The neutron server authenticates with keystone whether the token is valid and returns the verification result.

25 - The token verification is passed, and nova-compute receives information about the virtual machine network.

26 - Nova-compute receives an authenticated token via the RESTfull keystone API and receives the persistent storage information needed to create a virtual machine via an HTTP request from the cinder api.
 
27 - The Cinder api authenticates Keystone whether the token is valid and returns the verification result.

28 - The token verification is passed, and nova-compute receives information about the permanent storage of the virtual machine.

29 - Nova-compute calls the configured virtualization driver to create a virtual machine based on instance information.







- dnf config-manager --enable crb # enable full packages for centos(this if extra package)
- yum update
- change   vim /etc/chrony.conf and add server for ntp and location
```
pool ntp0.ntp-servers.net iburst
```
change host vim /etc/hosts.. <name vm>
- enter ip and <name vm>   in /etc/hosts

                                                                   --1--

## Keystone Service 

Keystone - install on the controller, this is service who create main catalog for  autinfication and authorization

Keystone needs for interactions services and users

This is fist identifity service, who use users with autinfication and authorization
change host vim /etc/hosts.. <name vm>

- add repo list   /etc/yum.repos.d/centos.repo for maria db
  
```
[mariadb]
name = MariaDB
baseurl = https://rpm.mariadb.org/10.6/rhel/$releasever/$basearch
gpgkey= https://rpm.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```
second istall server and client Maria db
```
- sudo yum install MariaDB-server MariaDB-client
```  
(and how watch package yum whatprovides "Maria*"

Start db
 ```
  systemctl start mariadb
  systemctl enable mariadb
```
   check status active and go to maria
```
-  mysql -u root -p                                 # login for db  and press enter  )
-  MariaDB [(none)]> CREATE DATABASE keystone;      # Create db keystone
```
- Grant privilaeges for db keystone
```
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
IDENTIFIED BY '<password for keystone>';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
IDENTIFIED BY '<password for keystone>';
```
install httpd, mod_wsgi                 # install apache and plus module wsgi  

- install packages dnf centos-release-openstack-antelope for antilopa and install openstack-keystone 
```
 sudo dnf install -y centos-release-openstack-antelope
 sudo yum install openstack-keystone
 ```
install client !
```
- sudo yum install python3-openstackclient
```
  
- edit config /etc/keystone/keystone.conf add password
- create coonection who connected 
```
[database]
# ...
connection = mysql+pymysql://keystone:<password_for_db_ceystone>@<name_host:port>/keystone   #change password and ip port controller
``` 
- edit token provider
```
[token]
# ...
provider = fernet
```
- run sync mariadb and module keystone 
- check connection mariadb  
```
su -s /bin/sh -c "keystone-manage db_sync" keystone

...log its ok connect....
2024-03-06 06:17:19.570 9546 INFO alembic.runtime.migration [-] Context impl MySQLImpl.
2024-03-06 06:17:19.571 9546 INFO alembic.runtime.migration [-] Will assume non-transactional DDL.
2024-03-06 06:17:19.583 9546 INFO alembic.runtime.migration [-] Context impl MySQLImpl.
2024-03-06 06:17:19.583 9546 INFO alembic.runtime.migration [-] Will assume non-transactional DDL.
2024-03-06 06:17:19.596 9546 INFO alembic.runtime.migration [-] Running upgrade  -> 27e647c0fad4, Initial version.
2024-03-06 06:17:20.074 9546 INFO alembic.runtime.migration [-] Running upgrade 27e647c0fad4 -> e25ffa003242, Initial no-op Yoga contract migration.
2024-03-06 06:17:20.075 9546 INFO alembic.runtime.migration [-] Running upgrade 27e647c0fad4 -> 29e87d24a316, Initial no-op Yoga expand migration.
..........................
```
create user keystone and group keysone  that will be used to run keystone
```
 keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
 keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone

...log..........
2024-03-06 07:57:49.585 9642 INFO keystone.common.utils [-] /etc/keystone/credential-keys/ does not appear to exist; attempting to create it
2024-03-06 07:57:49.586 9642 INFO keystone.common.fernet_utils [-] Created a new temporary key: /etc/keystone/credential-keys/0.tmp
2024-03-06 07:57:49.586 9642 INFO keystone.common.fernet_utils [-] Become a valid new key: /etc/keystone/credential-keys/0
2024-03-06 07:57:49.586 9642 INFO keystone.common.fernet_utils [-] Starting key rotation with 1 key files: ['/etc/keystone/credential-keys/0']
2024-03-06 07:57:49.586 9642 INFO keystone.common.fernet_utils [-] Created a new temporary key: /etc/keystone/credential-keys/0.tmp
2024-03-06 07:57:49.586 9642 INFO keystone.common.fernet_utils [-] Current primary key is: 0
2024-03-06 07:57:49.586 9642 INFO keystone.common.fernet_utils [-] Next primary key will be: 1
2024-03-06 07:57:49.586 9642 INFO keystone.common.fernet_utils [-] Promoted key 0 to be the primary: 1
2024-03-06 07:57:49.587 9642 INFO keystone.common.fernet_utils [-] Become a valid new key: /etc/keystone/credential-keys/0
.................
```
create api for user admin, add password and 
```
 keystone-manage bootstrap --bootstrap-password <password_admin> \
  --bootstrap-admin-url http://<host_name>:5000/v3/ \
  --bootstrap-internal-url http://<host_name>:5000/v3/ \
  --bootstrap-public-url http://<host_name> 5000/v3/ \
  --bootstrap-region-id RegionOn
```
add server name for apcahe

change preference   /etc/httpd/conf/httpd.conf
```
servername <name_host>
```
create link /usr/share/keystone/wsgi-keystone.conf
```
ln -s /usr/share/keystone/wsgi-keystone.conf /etc/httpd/conf.d/   #he KeystoneWSGI application is a WSGI (Web Server Gateway Interface) application that provides a web service interface to Keystone.
 ```
start httpd service
```
systemctl enable httpd.service
systemctl start httpd.service
```
check status service and listen 5000 port
```
httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; preset: disabled)
     Active: active (running) since Wed 2024-03-06 09:08:16 EST; 6s ago
       Docs: man:httpd.service(8)
   Main PID: 9778 (httpd)
     Status: "Started, listening on: port 5000, port 80"
      Tasks: 197 (limit: 100452)
     Memory: 80.7M
        CPU: 298ms
     CGroup: /system.slice/httpd.service
             ├─9778 /usr/sbin/httpd -DFOREGROUND
             ├─9779 /usr/sbin/httpd -DFOREGROUND
             ├─9780 "(wsgi:keystone-" -DFOREGROUND
             ├─9781 "(wsgi:keystone-" -DFOREGROUND
             ├─9782 "(wsgi:keystone-" -DFOREGROUND
             ├─9783 "(wsgi:keystone-" -DFOREGROUND
             ├─9784 "(wsgi:keystone-" -DFOREGROUND
             ├─9785 /usr/sbin/httpd -DFOREGROUND
             ├─9786 /usr/sbin/httpd -DFOREGROUND
             └─9787 /usr/sbin/httpd -DFOREGROUND

Mar 06 09:08:16 localhost.localdomain systemd[1]: Starting The Apache HTTP Server...
Mar 06 09:08:16 localhost.localdomain systemd[1]: Started The Apache HTTP Server.
Mar 06 09:08:16 localhost.localdomain httpd[9778]: Server configured, listening on: port 5000, port 80

...
ss -tuln
tcp          LISTEN        0             511                               *:5000                            *:*
```
 ```
 export OS_USERNAME=admin
 export OS_PASSWORD=ADMIN_PASS
 export OS_PROJECT_NAME=admin
 export OS_USER_DOMAIN_NAME=Default
 export OS_PROJECT_DOMAIN_NAME=Default
 export OS_AUTH_URL=http://<host_vm>:5000/v3
 export OS_IDENTITY_API_VERSION=3
```
***
create scripts for envroment admin-openrc
- sudo -i
```
 vim admin-openrc
 export OS_USERNAME=admin
 export OS_PASSWORD=ADMIN_PASS
 export OS_PROJECT_NAME=admin
 export OS_USER_DOMAIN_NAME=Default
 export OS_PROJECT_DOMAIN_NAME=Default
 export OS_AUTH_URL=http://<host_vm>:5000/v3
 export OS_IDENTITY_API_VERSION=3
```
debug keystone create log
....../etc/keystone/keystone.conf.........................................
```
debug = true
The name of a logging configuration file. This file is appended to any
existing logging configuration files. For details about logging configuration
files, see the Python logging module documentation. Note that when logging
configuration files are used then all logging configuration is set in the
configuration file and other logging configuration options are ignored (for
example, log-date-format). (string value)
Note: This option can be changed without restarting.
Deprecated group/name - [DEFAULT]/log_config
log_config_append = <None>

Defines the format string for %%(asctime)s in log records. Default:
%(default)s . This option is ignored if log_config_append is set. (string
value)
log_date_format = %Y-%m-%d %H:%M:%S

(Optional) Name of log file to send logging output to. If no default is set,
logging will go to stderr as defined by use_stderr. This option is ignored if
log_config_append is set. (string value)
Deprecated group/name - [DEFAULT]/logfile
log_file = <None>

(Optional) The base directory used for relative log_file  paths. This option
is ignored if log_config_append is set. (string value)
Deprecated group/name - [DEFAULT]/logdir
log_dir = /var/log/keystone/
............................................................................



                                                                          --2--

----------------------        ----------------------         ----------------------
|                     |       |                     |        |                    |
    glance- api server  -------  glance registry       ------        db
|                     |       |                     |        |                    |
-----------------------       -----------------------        ----------------------
           |
           |
-----------------------
|                     |
      Image Store
|                     |
-----------------------
## Glance Service (images) 
The OpenStack Image service includes the following components:
1)glance-api
  Accepts Image API calls for image discovery, retrieval, and storage.
2)Database
Stores image metadata and you can choose your database depending on your preference
3)Storage repository
 filesystem mounted on the glance-api controller node
4) Metadata definition service
A common API for vendors, admins, services, and users to meaningfully define their own custom metadata.
```
- create database
```
mysql -u root -p
MariaDB [(none)]> CREATE DATABASE glance;
```
- add roles for user glance

```
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' \
  IDENTIFIED BY '<password_user>';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
  IDENTIFIED BY '<password_user>';
exit 
```
go to home directory and run env
```
sudo -i
. admin-openrc
```
-create user glance to defaul domen 
```
openstack user create --domain default --password-prompt glance
```
- create Project service
```
openstack project create service --domain default
```
- create Role to user glance in project service
```
openstack role add --project service --user glance admin
```
- create Service glance

```
openstack service create --name glance  --description "OpenStack Image" image
```
- create Api endpoint for images
- 1. punglic
```
openstack endpoint create --region RegionOne image public http://<host_name>:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | bb455b258a3d4c7cb8b0f5b77abab92c |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2e3d7b05e80048ae9e4e5f705a9fcf36 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://Only:9292                 |
+--------------+----------------------------------+
```
2.internal
```
 openstack endpoint create --region RegionOne image internal http://<host_name>:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 992ab299d4344d5aa3cb5561442513dc |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2e3d7b05e80048ae9e4e5f705a9fcf36 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://Only:9292                 |
+--------------+----------------------------------+

```
3.admin
```
openstack endpoint create --region RegionOne image admin http://<hostname>:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 641709b7345142128a7ddc13e872c46d |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2e3d7b05e80048ae9e4e5f705a9fcf36 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://Only:9292                 |
+--------------+----------------------------------+
```
install glance
 
if not install openstack-glance and we see error

```
Error:
 Problem: package openstack-glance-1:26.0.0-1.el9s.noarch from centos-openstack-antelope requires python3-glance = 1:26.0.0-1.el9s, but none of the providers can be installed
  - conflicting requests
  - nothing provides python3-pyxattr needed by python3-glance-1:26.0.0-1.el9s.noarch from centos-openstack-antelope
(try to add '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages)
...
You don't enable full CRB repository and we can or download package python3-pyxattr
https://mirror.stream.centos.org/9-stream/CRB/x86_64/os/Packages/    #and find python3-pyxattr (package)
```

- First install openstack-glance
```
yum install openstack-glance
```
- Second change preference glance /etc/glance/glance-api.conf
- change connectin to sync glance
```
[database]
# ...
connection = mysql+pymysql://glance:<glance_pass>@<hostname:port>/glance

```
- edit keystoun autotoken  /etc/glance/glance-api.conf
```
 [keystone_authtoken]

www_authenticate_uri  = http://<host_name>:5000         # change hostname for autenficate 
auth_url = http://<hostname>:5000                       # change hostname for auth_url 
memcached_servers = <hostname> :11211                   #  chanhe host
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = glance
password = <password_glance>                            #  change pass

[paste_deploy]
...
flavor = keystone                                       # enable flavor 
```
- create store for images, change  /etc/glance/glance-api.conf
```
[glance_store]
...
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/
```
- add role for user glance only reader
```
openstack role add --user glance --user-domain Default --system all reader
```
sync db glance 
```
su -s /bin/sh -c "glance-manage db_sync" glance
```
- enable service glance
```
 systemctl enable openstack-glance-api.service
 systemctl start openstack-glance-api.service
```
                                                            ---3--- 
                                                       

#Placement service shedule

1.Resource tracking: Placement maintains a real-time inventory of compute resources available in the cloud. It tracks information such as CPU, memory, storage, network

2.Resource allocation: When a user requests the creation of a VM or other resource, Placement is responsible for selecting the appropriate compute host to accommodate the workload

3.Scheduler filters and weights: Placement uses a set of filters and weights to evaluate compute hosts and determine the best placement option

4.Placement API: OpenStack Placement exposes a RESTful API that allows clients (such as Nova, the OpenStack compute service) to interact with the Placement service. 

5.Class and train: Resource classes represent different types of resources available in the cloud (e.g., CPU, RAM), while traits describe additional capabilities or features of a resource (e.g., hardware acceleration, SSD storage)

To create the database, complete these steps:
```
mysql -u root -p
MariaDB [(none)]> CREATE DATABASE placement;
```
add privleges for user placement

```
MariaDB [(none)]> GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'localhost' \
  IDENTIFIED BY '<password>';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON placement.* TO 'placement'@'%' \
  IDENTIFIED BY 'password';
```
configure user and endpoint 
```
.admin-openrc
openstack user create --domain default --password-prompt placement
User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | fa742015a6494a949f67629884fc7ec8 |
| name                | placement                        |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```
add placement user
```
openstack role add --project service --user placement admin
```
create placement Api
```
openstack service create --name placement \
  --description "Placement API" placement
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Placement API                    |
| enabled     | True                             |
| id          | 2d1a27022e6e4185b86adac4444c495f |
| name        | placement                        |
| type        | placement                        |
+-------------+----------------------------------
```
create endpoint api public
```
openstack endpoint create --region RegionOne \
  placement public http://<hostname>:8778     # change hostname
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 2b1b2637908b4137a9c2e0470487cbc0 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2d1a27022e6e4185b86adac4444c495f |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+

```
create endpoint api internal 
```
openstack endpoint create --region RegionOne \
  placement internal http://<hostname>:8778
--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 02bcda9a150a4bd7993ff4879df971ab |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2d1a27022e6e4185b86adac4444c495f |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+
```
openstack endpoint create --region RegionOne \
  placement admin http://<hostname>:8778
```
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 3d71177b9e0f406f98cbff198d74b182 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 2d1a27022e6e4185b86adac4444c495f |
| service_name | placement                        |
| service_type | placement                        |
| url          | http://controller:8778           |
+--------------+----------------------------------+
```
Install placement

```
yum install openstack-placement-api
```
edit placement.conf
vim  /etc/placement/placement.conf
```
[placement_database]
# ...
connection = mysql+pymysql://placement:<password>@<hostname>/placement

[api]
# ...
auth_strategy = keystone

[keystone_authtoken]
# ...
auth_url = http://<hostname>:5000/v3
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = placement
password = 1qaz2wsx
```
- synhronise db
```
su -s /bin/sh -c "placement-manage db sync" placement
```
restart httpd
```
systemctl restart httpd
```
- Change file vim /etc/keystone/policy.json to => vim /etc/keystone/policy.yaml !!!
  
- Change file vim /etc/keystone/policy.json to => vim /etc/placement/policy.yaml !!!
```
mv policy.json policy.yaml
```
if not change format you will error  for format depricated!!
- yum install osc-placement

# Verify

- yum install osc-placemen
```
yum install  python3-osc-placement
```

```
. admin-openrc
 placement-status upgrade check
+-------------------------------------------+
| Upgrade Check Results                     |
+-------------------------------------------+
| Check: Missing Root Provider IDs          |
| Result: Success                           |
| Details: None                             |
+-------------------------------------------+
| Check: Incomplete Consumers               |
| Result: Success                           |
| Details: None                             |
+-------------------------------------------+
| Check: Policy File JSON to YAML Migration |
| Result: Success                           |        it's ok 
| Details: None                             |
+-------------------------------------------+
```
error

[root@Only ~]# openstack allocation candidate list --resource VCPU=1

***Expecting value: line 1 column 1 (char 0)

- Problem access for web to direcrory 
- edit 00-placement-api.conf and add access for directory /usr/bin
```
[root@controller ~]# vim /etc/httpd/conf.d/00-placement-api.conf
...

[root@controller ~]# vim /etc/httpd/conf.d/00-placement-api.conf

Listen 8778

<VirtualHost *:8778>
  WSGIProcessGroup placement-api
  WSGIApplicationGroup%{GLOBAL}
  WSGIPassAuthorization On
  WSGIDaemonProcess placement-api processes=3 threads=1 user=placement group=placement
  WSGIScriptAlias/ /usr/bin/placement-api
  <IfVersion >= 2.4>
    ErrorLogFormat "%M"
  </IfVersion>
  ErrorLog /var/log/placement/placement-api.log
  #SSLEngine On
  #SSLCertificateFile...
  #SSLCertificateKeyFile...
  <Directory /usr/bin>                              # add
     <IfVersion >= 2.4>                             # add 
       Require all granted                          # add
     </IfVersion>                                   # add
  </Directory>
</VirtualHost>
Alias /placement-api /usr/bin/placement-api
<Location /placement-api>
  SetHandler wsgi-script
  Options + ExecCGI
  WSGIProcessGroup placement-api
  WSGIApplicationGroup%{GLOBAL}
  WSGIPassAuthorization On
</Location>
```
- check
```
openstack --os-placement-api-version 1.2 resource class list --sort-column name
...
[root@Only ~]# openstack --os-placement-api-version 1.2 resource class list --sort-column name
+----------------------------------------+
| name                                   |
+----------------------------------------+
| DISK_GB                                |
| FPGA                                   |
| IPV4_ADDRESS                           |
| MEMORY_MB                              |
| MEM_ENCRYPTION_CONTEXT                 |
| NET_BW_EGR_KILOBIT_PER_SEC             |
| NET_BW_IGR_KILOBIT_PER_SEC             |

openstack --os-placement-api-version 1.6 trait list --sort-column name
+---------------------------------------+
| name                                  |
+---------------------------------------+
| COMPUTE_ACCELERATORS                  |
| COMPUTE_ADDRESS_SPACE_EMULATED        |
| COMPUTE_ADDRESS_SPACE_PASSTHROUGH     |
| COMPUTE_ARCH_AARCH64                  |
```

A request to create a virtual machine (VM) is sent to the cloud infrastructure management system (for example, OpenStack Nova),

which then accesses the placement service to clarify the available resources. Placement provides information about available 

computing power, memory, storage and other resources, based on which Nova creates a virtual machine based on the required 

characteristics and the level of available resources.

                                                 ---4---

- nova-api - Accepts and responds to end user compute API calls. The service supports the OpenStack Compute API. Running an 

instance.

- nova-api-metadata - provides metadata about virtual machines that are managed by OpenStack Compute (Nova). This service provides 

information about virtual machines such as their ID, name, IP address, image, resources used, status, etc. 

- nova-compute service - The nova-compute service monitors the allocation and management of computing resources (both physical and 

virtual) for instances (virtual machines) and ensures the execution of requests for the creation, management and deletion of 

virtual machines.

It interacts with hypervisors such as KVM, Xen, VMware and others to manage virtual machines and dedicated resources such as 

processor, memory and storage.

The nova-compute service is also responsible for monitoring and managing the state of virtual machines, including monitoring 

their lifecycle (creation, startup, suspension, resumption, shutdown, deletion) and resource allocation.


- nova-scheduler  -  Takes a virtual machine instance request from the queue and determines on which compute server host it runs.
 
- nova-conductor  -  nova-conductor module scales horizontally. However, do not deploy it on nodes where the nova-compute service runs

- nova-novncproxy daemon
Provides a proxy for accessing running instances through a VNC connection. Supports browser-based novnc clients.

- nova-spicehtml5proxy daemon
Provides a proxy for accessing running instances through a SPICE connection. Supports browser-based HTML5 client.

                                                                                                                        --- install for controller ---
  

- create bd nova_api; nova; nova_cel0
```
   mysql -u root -p

   MariaDB [(none)]> CREATE DATABASE nova_api;
   MariaDB [(none)]> CREATE DATABASE nova;
   MariaDB [(none)]> CREATE DATABASE nova_cell0;
  ```
- grant access to db
```
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' \
  IDENTIFIED BY '<password>'';                                                             #change pass

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' \
  IDENTIFIED BY '<password>';                                                               #change pass

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' \
  IDENTIFIED BY 'NOVA_DBPASS';                                                              #change pass
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' \
  IDENTIFIED BY '<password>';
                                                           
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'localhost' \               #change pass
  IDENTIFIED BY '<password>';
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_cell0.* TO 'nova'@'%' \                      change password 
  IDENTIFIED BY '<password>';
```
 
. admin-openrc

- create computer service credentional(create nova instance)
```
 openstack user create --domain default --password-prompt nova
User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 8a7dbf5279404537b1c7b86c033620fe |
| name                | nova                             |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```
- add role to user nova admin
```
 openstack role add --project service --user nova admin
```
- create nova service
```
 openstack service create --name nova  --description "OpenStack Compute" compute
```
 - create compute api endpoint admin, internal and public
```
  openstack endpoint create --region RegionOne compute public http://<host>:8774/v2.1  #change host
+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | 3c1caa473bfe4390a11e7177894bcc7b          |
| interface    | public                                    |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | 060d59eac51b4594815603d75a00aba2          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://controller:8774/v2.1               |
+--------------+-------------------------------------------+

openstack endpoint create --region RegionOne compute internal http://<host>:8774/v2.1    chaange host 
+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | e3c918de680746a586eac1f2d9bc10ab          |
| interface    | internal                                  |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | 060d59eac51b4594815603d75a00aba2          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://controller:8774/v2.1               |
+--------------+-------------------------------------------+
openstack endpoint create --region RegionOne compute admin http://<hostname>:8774/v2.1    #change host
+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | 38f7af91666a47cfb97b4dc790b94424          |
| interface    | admin                                     |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | 060d59eac51b4594815603d75a00aba2          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://controller:8774/v2.1               |
+--------------+-------------------------------------------+
```
install rabbit, nova-api,nova-conductor,nova-novncproxy,nova-sceduler
```
 sudo yum install openstack-nova-api openstack-nova-conductor openstack-nova-novncproxy openstack-nova-scheduler  
 sudo  dnf -y install rabbitmq-server memcached
```
- change preference nova.conf
```
vim /etc/nova/nova.conf
```
- enable config api for os and api metadata 
```
[DEFAULT]
# ...
enabled_apis = osapi_compute,metadata
```
create access for api database 
api base -*** authorization user for api, reqwest for create , dell vm and managment vm 
```
[api_database]
# ...
connection = mysql+pymysql://nova:<password>@<hostname>/nova_api
```
The nova database is used to store data about virtual machines, images, networks, projects, as well as information about how they are interconnected.
```
[database]
# ...
connection = mysql+pymysql://nova:<password>@<hostname>/nova
```
- change transprt
```  
[DEFAULT]
# ...
transport_url = rabbit://openstack:<password>@<hostname>:5672/    #Change password and name host
```
Add strategy and authorization nova with token (/etc/nova/nova.conf)

```
[api]
# ...
auth_strategy = keystone

[keystone_authtoken]
# ...
www_authenticate_uri = http://<host>:5000/   #change host
auth_url = http://<host>:5000/               #change host
memcached_servers = controller:11211
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = nova
password = <pass>                           #change pass

```
- create service user for nova need for lond seesion(live migration or snapshot take long enough to exceed the expiry of the user token vim /etc/nova/nova.conf)
```
[service_user]
send_service_user_token = true
auth_url = https://controller/identity
auth_strategy = keystone
auth_type = password
project_domain_name = Default
project_name = service
user_domain_name = Default
username = nova
password = NOVA_PASS
```
- add ip managment interface /etc/nova/nova.conf

```
[DEFAULT]
# ...
my_ip = <ip for managment interface>  ## external
```
- enable vnc
```
[vnc]
enabled = true
# ...
server_listen = $my_ip
server_proxyclient_address = $my_ip
```
- add glance server 

```
[glance]
# ...
api_servers = http://<host>:9292
```
- lock path nova tmp ( This path is used to temporarily store lock files, which are used to synchronize access to shared resources and prevent conflicts between parallel threads or processes)
```
[oslo_concurrency]
# ...
lock_path = /var/lib/nova/tmp
```
add accecss for placement
```
[placement]
# ...
region_name = RegionOne
project_domain_name = Default
project_name = service
auth_type = password
user_domain_name = Default
auth_url = http://<host>:5000/v3          #change host        
username = placement
password = <password>                     # change password
```
- Register cell0 ( responsible for coordinating actions between different cells ) main cell 
 ```
su -s /bin/sh -c "nova-manage cell_v2 map_cell0" nova
```
- create cell1  for scaling nova space
  ```
   su -s /bin/sh -c "nova-manage cell_v2 create_cell --name=cell1 --verbose" nova
  ```
synhronise bd  

```
su -s /bin/sh -c "nova-manage db sync" nova
```

- Verify nova cell0 and cell1 are registered correctly (0 and 1 cell)
```
su -s /bin/sh -c "nova-manage cell_v2 list_cells" nova
+-------+--------------------------------------+----------------------------------------------------+--------------------------------------------------------------+----------+
|  Name |                 UUID                 |                   Transport URL                    |                     Database Connection                      | Disabled |
+-------+--------------------------------------+----------------------------------------------------+--------------------------------------------------------------+----------+
| cell0 | 00000000-0000-0000-0000-000000000000 |                       none:/                       | mysql+pymysql://nova:****@controller/nova_cell0?charset=utf8 |  False   |
| cell1 | f690f4fd-2bc5-4f15-8145-db561a7b9d3d | rabbit://openstack:****@controller:5672/nova_cell1 | mysql+pymysql://nova:****@controller/nova_cell1?charset=utf8 |  False   |
+-------+--------------------------------------+----------------------------------------------------+--------------------------------------------------------------+----------+
```
- enable service and start
```
# systemctl enable \
    openstack-nova-api.service \
    openstack-nova-scheduler.service \
    openstack-nova-conductor.service \
    openstack-nova-novncproxy.service
# systemctl start \
    openstack-nova-api.service \
    openstack-nova-scheduler.service \
    openstack-nova-conductor.service \
    openstack-nova-novncproxy.service
```

                                                                                                         ---install compute node ---
                                                                                                         
                                                                                                         
- install nova compute
```                                                                                                       
yum install openstack-nova-compute                                                                                                       
```
vim  /etc/nova/nova.conf and enable api 

```
[DEFAULT]
# ...
enabled_apis = osapi_compute,metadata
```
---
- install libvirt, qemu-kvm, libvirt-client
```
yum install qemu-kvm qemu-img libvirt virt-install libvirt-client 
```

- change path for nova.conf (error access for nova/tmp) !!! 
```
lock_path = /var/lib/nova (  lock_path = /var/lib/nova/tmp wrong)
log_dir = /var/log/nova
state_path=var/lib/nova

```
check hardware acceleration for virtual machine 
```
egrep -c '(vmx|svm)' /proc/cpuinfo
```
- enable qemu
``` 
[libvirt]
# ...
virt_type = qemu
```
enable driver 
```
 compute_driver=libvirt.LibvirtDriver
```
start and enabled service nova
```
systemctl enable libvirtd.service openstack-nova-compute.service
systemctl start libvirtd.service openstack-nova-compute.service
```
. admin-openrc
```
$ openstack compute service list --service nova-compute
+----+-------+--------------+------+-------+---------+----------------------------+
| ID | Host  | Binary       | Zone | State | Status  | Updated At                 |
+----+-------+--------------+------+-------+---------+----------------------------+
| 1  | node1 | nova-compute | nova | up    | enabled | 2017-04-14T15:30:44.000000 |
+----+-------+--------------+------+-------+---------+----------------------------+
```
Discover compute hosts:(find host)
```
su -s /bin/sh -c "nova-manage cell_v2 discover_hosts --verbose" nova

... log ....
Found 2 cell mappings.
Skipping cell0 since it does not contain hosts.
Getting compute nodes from cell 'cell1': ad5a5985-a719-4567-98d8-8d148aaae4bc
Found 1 computes in cell: ad5a5985-a719-4567-98d8-8d148aaae4bc
Checking host mapping for compute host 'compute': fe58ddc1-1d65-4f87-9456-bc040dc106b3
Creating host mapping for compute host 'compute': fe58ddc1-1d65-4f87-9456-bc040dc106b3
...log...
```
add sceduller for discover 
```
[scheduler]
discover_hosts_in_cells_interval = 300
```

## Check nova status 
----
```
openstack compute service list

+----+--------------------+------------+----------+---------+-------+----------------------------+
| Id | Binary             | Host       | Zone     | Status  | State | Updated At                 |
+----+--------------------+------------+----------+---------+-------+----------------------------+
|  1 | nova-scheduler     | controller | internal | enabled | up    | 2016-02-09T23:11:15.000000 |
|  2 | nova-conductor     | controller | internal | enabled | up    | 2016-02-09T23:11:16.000000 |
|  3 | nova-compute       | compute1   | nova     | enabled | up    | 2016-02-09T23:11:20.000000 |
+----+--------------------+------------+----------+---------+-------+----------------------------+
```
list api and chec access

```
 openstack catalog list

+-----------+-----------+-----------------------------------------+
| Name      | Type      | Endpoints                               |
+-----------+-----------+-----------------------------------------+
| keystone  | identity  | RegionOne                               |
|           |           |   public: http://controller:5000/v3/    |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:5000/v3/  |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:5000/v3/     |
|           |           |                                         |
| glance    | image     | RegionOne                               |
|           |           |   admin: http://controller:9292         |
|           |           | RegionOne                               |
|           |           |   public: http://controller:9292        |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:9292      |
|           |           |                                         |
| nova      | compute   | RegionOne                               |
|           |           |   admin: http://controller:8774/v2.1    |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:8774/v2.1 |
|           |           | RegionOne                               |
|           |           |   public: http://controller:8774/v2.1   |
|           |           |                                         |
| placement | placement | RegionOne                               |
|           |           |   public: http://controller:8778        |
|           |           | RegionOne                               |
|           |           |   admin: http://controller:8778         |
|           |           | RegionOne                               |
|           |           |   internal: http://controller:8778      |
|           |           |                                         |
+-----------+-----------+-----------------------------------------+
```
- list images 
```
openstack image list

+--------------------------------------+-------------+-------------+
| ID                                   | Name        | Status      |
+--------------------------------------+-------------+-------------+
| 9a76d9f9-9620-4f2e-8c69-6c5691fae163 | cirros      | active      |
+--------------------------------------+-------------+-------------+
```
- Check the cells and placement API are working successfull
```
 nova-status upgrade check
+--------------------------------------------------------------------+
| Upgrade Check Results                                              |
+--------------------------------------------------------------------+
| Check: Cells v2                                                    |
| Result: Success                                                    |
| Details: None                                                      |
+--------------------------------------------------------------------+
| Check: Placement API                                               |
| Result: Success                                                    |
| Details: None                                                      |
+--------------------------------------------------------------------+
| Check: Cinder API                                                  |
| Result: Success                                                    |
| Details: None                                                      |
+--------------------------------------------------------------------+
| Check: Policy File JSON to YAML Migration                          |
| Result: Success                                                    |
| Details: None                                                      |
+--------------------------------------------------------------------+
| Check: Older than N-1 computes                                     |
| Result: Success                                                    |
| Details: None                                                      |

```
                                                              --- Neutron ---


OpenStack Networking plug-ins and agents- Plug and unplug ports, create networks or subnets, and provide IP addressing. this 

use NEC OpenFlow products, Open vSwitch, Linux bridging, Open Virtual Network (OVN) The common agents are L3 (layer 3 ip ), 

DHCP (dynamic host IP addressing), and a plug-in agent.

- nova-api service-Accepts and responds to end user compute API calls. The service supports the OpenStack Compute API

- nova-api-metadata service - The metadata service supports two sets of APIs - an OpenStack metadata API and an EC2-compatible API and also exposes vendordata and user data

- nova-compute service  - daemon that creates and terminates virtual machine instances through hypervisor(libvirt for KVM or QEMU)

- nova-scheduler service - Takes a virtual machine instance request from the queue and Определяет on which compute server host it runs.

- nova-conductor module - The nova-conductor module scales horizontally. However, do not deploy it on nodes where the nova-compute service runs.

nova-novncproxy daemon -vnc Provides a proxy for accessing running instances through a VNC connection

nova-spicehtml5proxy daemon Provides a proxy for accessing running instances through a SPICE connection

                                                            For Controller
                                                            

 - create db
```
mysql -u root -p
MariaDB [(none)]> CREATE DATABASE neutron;
```
- grant access
```
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' \
  IDENTIFIED BY '<password>';                                                               # password
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' \  
  IDENTIFIED BY '<password>';                                                               # password
```
go admin openrc

```
 . admin-openrc
```
- create user netron
```
openstack user create --domain default --password-prompt neutron
User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | fdb0f541e28141719b6a43c8944bf1fb |
| name                | neutron                          |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+----------------------------------+
```
- add role for user neutron
```
openstack role add --project service --user neutron admin
```
- create neutron service 
```
openstack service create --name neutron \
  --description "OpenStack Networking" network

+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Networking             |
| enabled     | True                             |
| id          | f71529314dab4a4d8eca427e701d209e |
| name        | neutron                          |
| type        | network                          |
+-------------+----------------------------------+

```
- create endpoins for neutron public
```
openstack endpoint create --region RegionOne \
  network public http://controller:9696

+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 85d80a6d02fc4b7683f611d7fc1493a3 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | f71529314dab4a4d8eca427e701d209e |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+

```
create ep internal
```

openstack endpoint create --region RegionOne \
  network internal http://controller:9696

+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 09753b537ac74422a68d2d791cf3714f |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | f71529314dab4a4d8eca427e701d209e |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+
```
- create ep admin
```
 openstack endpoint create --region RegionOne \
  network admin http://controller:9696

+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 1ee14289c9374dffb5db92a5c112fc4e |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | f71529314dab4a4d8eca427e701d209e |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+
```
- Networking provders and we use 2 options
  
  Networking Option 1: Provider networks
  
  Networking Option 2: Self-service networks

- install the components
```
  yum install openstack-neutron openstack-neutron-ml2 openstack-neutron-openvswitch ebtables
```
- Edit /etc/neutron/neutron.conf
```
[database]
connection = mysql+pymysql://neutron:<password>@<host:port>/neutron  # add password and host port
```
- add preference  Modular Layer 2 (ML2) plug-in, router service
```
[DEFAULT]
core_plugin = ml2
service_plugins = router
```
 - add configure RabbitMQ
```
[DEFAULT]
transport_url = rabbit://openstack:<password>@<host>   # pass + host
```
 - edit keystone preference autotocken
```
[DEFAULT]
auth_strategy = keystone

[keystone_authtoken]

www_authenticate_uri = http://<host>:5000      # host auth
auth_url = http://<host>:5000                  # host auth           
memcached_servers = <host>:11211               # memcached  host 
auth_type = password
project_domain_name = Default
user_domain_name = Default
project_name = service
username = neutron
password = < password>                         # password
```
- nova section 

```
[DEFAULT]
notify_nova_on_port_status_changes = true          # "enable change status for nova"
notify_nova_on_port_data_changes = true


[nova]
auth_url = http://<hostname>:5000          #host for auth nova
auth_type = password
project_domain_name = Default
user_domain_name = Default
region_name = RegionOne
project_name = service
username = nova
password = <password>                      # password

```
edit 

lock_path = /var/lib/neutron/tmp











acess to dash
----------------------------
1. in </etc/httpd/conf.d/openstack-dashboard.conf>
   change "WSGIScriptAlias /dashboard /usr/share/openstack-dashboard/openstack_dashboard/wsgi/django.wsgi"
   to "WSGIScriptAlias /dashboard /usr/share/openstack-dashboard/openstack_dashboard/wsgi.py"
   because there is no django.wsgi, but wsgi.py on your server.

2. in </etc/httpd/conf.d/openstack-dashboard.conf>
   change "<Directory /usr/share/openstack-dashboard/openstack_dashboard/wsgi>"
   to "<Directory /usr/share/openstack-dashboard/openstack_dashboard>"
   because of the same reason to 1

3. in </etc/openstack-dashboard/local_settings>
   add "WEBROOT = '/dashboard/'"

4. in </etc/openstack-dashboard/local_settings>
   change "http://%s/identity/v3"
   to "http://%s:5000/identity/v3"

good luck
--------------------------------------------



