
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
enabled = true
server_listen = 127.0.0.1
server_proxyclient_address = 127.0.0.1
novncproxy_base_url = http://<host_ip>:6080/vnc_auto.html # vnc host ip
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
add neutron section 
```
[neutron]
www_authenticate_uri http://<host_ip>:5000/v3
auth_url = http://<host_ip>:5000/v3
auth_type = password
project_domain_name = Default
user_domain_name = Default
region_name = RegionOne
project_name = service
username = neutron
password = 1qaz2wsx
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
edit lock
```
[oslo_concurrency]
lock_path = /var/lib/neutron/tmp
```
## Configure the Modular Layer 2 (ML2) plug-in
The ML2 plug-in uses the Linux bridge mechanism to build layer-2 (bridging and switching) virtual networking infrastructure for instances.

- Edit the /etc/neutron/plugins/ml2/ml2_conf.ini
- add section "enable flat, VLAN, and VXLAN networks"
 ```
 [ml2]
type_drivers = flat,vlan,vxlan
```
- add section  "enable VXLAN"
```
[ml2]
tenant_network_types = vxlan
```
- add section  "enable the Linux bridge and layer-2"
```
[ml2]
mechanism_drivers = openvswitch,l2population
```
-  add section "the port security extension driver"
```
[ml2]
extension_drivers = port_security
```
- section, configure the provider virtual network 
```
[ml2_type_flat]
flat_networks = provider
```
- range for self-service networks:
```
[ml2_type_vxlan]
vni_ranges = 1:1000
```
## Configure the Open vSwitch agent

The Linux bridge agent builds layer-2 (bridging and switching) virtual networking infrastructure for instances and handles security groups
- Edit the /etc/neutron/plugins/ml2/openvswitch_agent.ini
```
[vxlan]
local_ip = <external_ip>          #change external ip
l2_population = true

[securitygroup]
enable_security_group = true
firewall_driver = openvswitch

[ovs]
physical_interface_mappings = provider:enp6s18
```
## Configure the layer-3 agent

- Edit the /etc/neutron/l3_agent.ini
```
[DEFAULT]
interface_driver = openvswitch
```
## Configure the DHCP agent

- Edit the /etc/neutron/dhcp_agent.ini

```
[DEFAULT]
interface_driver = openvswitch
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = true
```
## Configure the Compute service to use the Networking service

Edit the /etc/nova/nova.conf
```
[neutron]
auth_url = http://Only:5000        #change host
auth_type = password
project_domain_name = Default
user_domain_name = Default
region_name = RegionOne
project_name = service
username = neutron
password = <password>             #change password
service_metadata_proxy = true
```
- Sync DB for neutron.conf and ml2_conf 
```
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```
restart service  

```
systemctl restart neutron-server neutron-openvswitch-agent  neutron-dhcp-agent  neutron-metadata-agent neutron-l3-agent
```

# Check service Open vSwitch agent DHCP agent L3 agent  Metadata agent 
openstack network agent list
+--------------------------------------+--------------------+------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+------+-------------------+-------+-------+---------------------------+
| 1a030327-9378-4e3a-8a13-243bcc66f753 | Open vSwitch agent | Only | None              | :-)   | UP    | neutron-openvswitch-agent |
| 2c5a2b61-40c7-44f5-bff0-3df357116a10 | DHCP agent         | Only | nova              | :-)   | UP    | neutron-dhcp-agent        |
| 5ec2e854-e326-4d35-9f44-e82d914645ec | L3 agent           | Only | nova              | :-)   | UP    | neutron-l3-agent          |
| f5cda2b1-c193-410a-b2f3-6eab2fac3060 | Metadata agent     | Only | None              | :-)   | UP    | neutron-metadata-agent    |
+--------------------------------------+--------------------+------+-------------------+-------+-------+---------------------------+


                                                            --dashbord--
                                                            
- install dashbord
```
yum install openstack-dashboard
```
- Edit the /etc/openstack-dashboard/local_settings
- allow host for dashbord who can connect 
```
ALLOWED_HOSTS = ['*', 'localhost']
```
- edit preference . add WEBROOT = '/dashboard/'
```
OPENSTACK_HOST = "Only"
OPENSTACK_KEYSTONE_URL = "http://Only:5000"
WEBROOT = '/dashboard/'                        ##!!!!
LOGIN_URL = '/dashboard/auth/login/'
LOGOUT_URL = '/dashboard/auth/logout/'
LOGIN_REDIRECT_URL = '/dashboard/' ?
```
- change Cash for File!!
```
SESSION_ENGINE = 'django.contrib.sessions.backends.file'     ##!(error cash session )

CACHES = {
    'default': {
         'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
         'LOCATION': '<host>:11211',                           #change host
    }
}
```
- add api version 
```
OPENSTACK_API_VERSIONS = {
    "identity": 3,
    "image": 2,
    "volume": 3,
}
```
Default domain and rules for user
```
OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = "Default"
OPENSTACK_KEYSTONE_DEFAULT_ROLE = "admin"
OPENSTACK_NEUTRON_NETWORK = {
```
if you use l3 enable
```
    'enable_router': True,
    'enable_quotas': True,
    'enable_ipv6': True,
    'enable_distributed_router': True,
    'enable_ha_router': True,
    'enable_fip_topology_check': True,
}

```
- add time zone
```
TIME_ZONE = " Europe/Moscow"
```
- add  /etc/httpd/conf.d/openstack-dashboard.conf
```
WSGIApplicationGroup %{GLOBAL}
```
change path py !!
```
WSGIScriptAlias /dashboard /usr/share/openstack-dashboard/openstack_dashboard/wsgi.py   # change path
```
- and change directory !! /etc/httpd/conf.d/openstack-dashboard.conf
```
<Directory /usr/share/openstack-dashboard/openstack_dashboard/>
```
restart hhtpd
```
systemctl restart httpd.service memcached.service
```
# check dash
-http://<host>/dashboard
- user: admin:
...
                                                       --problem--
# Literature for errors cash
- https://programmerall.com/article/14441163019/
```
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'
SESSION_ENGINE = 'django.contrib.sessions.backends.file'
```
error for dash
```
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
```
                                               
                                               --Cinder--

mysql -u root -p

CREATE DATABASE cinder;
 
GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'localhost' IDENTIFIED BY '<pass>';

GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' IDENTIFIED BY '<pass>';

. admin-openrc
 
 openstack user create --domain default --password-prompt cinder

openstack role add --project service --user cinder admin

openstack service create --name cinderv3 --description "OpenStack Block Storage" volumev3





[DEFAULT]

transport_url = rabbit://openstack:<pass>@Only
auth_strategy = keystone
my_ip = <ip_nternal_host>
enabled_backends = lvm
glance_api_servers = http://<host>:9292


[lvm]
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_group = cinder-volumes
target_protocol = iscsi
target_helper = lioadm

[database]

connection = mysql+pymysql://cinder:<pass>@<host>:3306/cinder

[keystone_authtoken]

www_authenticate_uri = http://<host>:5000
auth_url = http://<host>:5000
memcached_servers = <host>:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = cinder
password = <pass>




                                               --Heat-

   ![Openstack](https://github.com/tulamelkii/openstack/blob/main/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-04-05%20100332.png)


- heat command-line client - CLI 
  
- heat-api component -An OpenStack-native REST API that processes API requests by sending them to the heat-engine over Remote Procedure Call
  
- heat-api-cfn  It processes API requests by sending them to the heat-engine over RPC(Remote Procedure Calling).
- heat-engine

- create db 
```
mysql -u root -p
CREATE DATABASE heat;
```
- add privileges
```
GRANT ALL PRIVILEGES ON heat.* TO 'heat'@'localhost' IDENTIFIED BY '<pass_host>';
GRANT ALL PRIVILEGES ON heat.* TO 'heat'@'%' IDENTIFIED BY '<pass_host>';
                                               
```
- create user heat
```
. admin-openrc
openstack user create --domain default --password-prompt heat
```
- add role project
```
openstack role add --project service --user heat admin
```
- create two service heat and heat-cfn
```
openstack service create --name heat --description "Orchestration" orchestration
openstack service create --name heat-cfn --description "Orchestration"  cloudformation
```
- create api for heat
```
openstack endpoint create --region RegionOne orchestration public http://<host>:8004/v1/%\(tenant_id\)s
openstack endpoint create --region RegionOne orchestration internal http://<host>:8004/v1/%\(tenant_id\)s
penstack endpoint create --region RegionOne orchestration admin http://<hiost>:8004/v1/%\(tenant_id\)s
```
- create api for heat-cfn
```
openstack endpoint create --region RegionOne cloudformation public http://<host>:8000/v1
openstack endpoint create --region RegionOne cloudformation internal http://<host>:8000/v1
openstack endpoint create --region RegionOne cloudformation admin http://<host>:8000/v1
```
- create new domaib HEAT
```
openstack domain create --description "Stack projects and users" heat
..
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | Stack projects and users         |
| enabled     | True                             |
| id          | 0f4d1bd326f2454dacc72157ba328a47 |
| name        | heat                             |
+-------------+----------------------------------+
..
```
- Create user heat_domain_admin
```
openstack user create --domain heat --password-prompt heat_domain_admin
```
- create role for user heat_domain_admin (admin)
```
openstack role add --domain heat --user-domain heat --user heat_domain_admin admin
```
- create owner role "heat_stack_owner"
  
```
openstack role create heat_stack_owner
```
- create role for user demo in project demo ( role heat_stack_owner)
```
openstack role add --project demo --user demo heat_stack_owner
```
- Create the (heat_stack_owner) role
```
openstack role create heat_stack_owner
```
- add role (heat_stack_owner) for project user demo
```
openstack role add --project demo --user demo heat_stack_owner
```
- Create the heat_stack_user role
```
openstack role create heat_stack_user
```

- install openstack-heat-api openstack-heat-api-cfn openstack-heat-engine
```
yum install openstack-heat-api openstack-heat-api-cfn openstack-heat-engine
```
- change preference /etc/heat/heat.conf 
- create db
```
[database]
...
connection = mysql+pymysql://heat:<pass>@<host>/heat
```
- create transport rabbit
``` 
[DEFAULT]
...
transport_url = rabbit://openstack:<pass_rabbit>@<host>
```
- create autotoken
```
[keystone_authtoken]
...
www_authenticate_uri = http://<host>:5000
auth_url = http://<host>:5000
memcached_servers = <host>:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = heat
password = <pass>
```
???
```
[trustee]
...
auth_type = password
auth_url = http://controller:5000
username = heat
password = HEAT_PASS
user_domain_name = default
```
???
```
[clients_keystone]
...
auth_uri = http://controller:5000
```
- configure metadata 

```
[DEFAULT]
...
heat_metadata_server_url = http://<host>:8000
heat_waitcondition_server_url = http://<host>:8000/v1/waitcondition
```
- add to domain user 
```

[DEFAULT]
...
stack_domain_admin = heat_domain_admin
stack_domain_admin_password = <pass>
stack_user_domain_name = heat
```
- sync db
```
su -s /bin/sh -c "heat-manage db_sync" heat
```
- enable service 
```
systemctl enable openstack-heat-api.service openstack-heat-api-cfn.service openstack-heat-engine.service
```
- start service

```
systemctl start openstack-heat-api.service openstack-heat-api-cfn.service openstack-heat-engine.service
```
- check system
``` 

 openstack orchestration service list
+----------+-------------+--------------------------------------+------+--------+----------------------------+--------+
| Hostname | Binary      | Engine ID                            | Host | Topic  | Updated At                 | Status |
+----------+-------------+--------------------------------------+------+--------+----------------------------+--------+
| Only     | heat-engine | 78244f29-e9f6-432d-aa12-8e33033e0516 | Only | engine | 2024-04-10T06:10:48.000000 | up     |
| Only     | heat-engine | dce65004-ae3c-4764-9452-b6751c355b43 | Only | engine | 2024-04-10T06:10:48.000000 | up     |
| Only     | heat-engine | 8df0aa21-f2b6-4d5f-a990-dd5d734d59df | Only | engine | 2024-04-10T06:10:48.000000 | up     |
| Only     | heat-engine | 5673e9e1-1eef-4eeb-b648-7311ca961e73 | Only | engine | 2024-04-10T06:10:48.000000 | up     |
| Only     | heat-engine | 7d3aec66-a9d4-41c1-8f3a-8fa0e6bb10a0 | Only | engine | 2024-04-10T06:09:55.000000 | up     |
| Only     | heat-engine | 9ef6ee63-eba6-4206-979f-972692fb4086 | Only | engine | 2024-04-10T06:10:48.000000 | up     |
| Only     | heat-engine | 8d6ecdd7-0737-4e1d-b761-daf18f6682ed | Only | engine | 2024-04-10T06:10:48.000000 | up     |
| Only     | heat-engine | 810e9d14-e05a-46d4-b919-026f77ecda18 | Only | engine | 2024-04-10T06:09:55.000000 | up     |
| Only     | heat-engine | 810b8a45-c5de-4ca0-b628-cd22171b7f4d | Only | engine | 2024-04-10T06:10:48.000000 | up     |
| Only     | heat-engine | 4cf523b8-6118-4e32-a141-308289e3729d | Only | engine | 2024-04-10T06:10:48.000000 | up     |
| Only     | heat-engine | 83ac838d-67c7-4c5a-96d6-89c3dc46c450 | Only | engine | 2024-04-10T06:10:48.000000 | up     |
| Only     | heat-engine | bd1a2559-45b0-4b4b-a11f-f4379103742d | Only | engine | 2024-04-10T06:09:55.000000 | up     |
+----------+-------------+--------------------------------------+------+--------+----------------------------+--------+
```
- create instance
```
heat_template_version: 2015-10-15
description: Launch a basic instance with CirrOS image using the
             ``m1.tiny`` flavor, ``mykey`` key,  and one network.

parameters:
  NetID:
    type: string
    description: Network ID to use for the instance.

resources:
  server:
    type: OS::Nova::Server
    properties:
      image: cirros
      flavor: m1.tiny
      key_name: mykey
      networks:
      - network: { get_param: NetID }

outputs:
  instance_name:
    description: Name of the instance.
    value: { get_attr: [ server, name ] }
  instance_ip:
    description: IP address of the instance.
    value: { get_attr: [ server, first_address ] }
```
created instanse
```
 openstack network list
+--------------------------------------+-------------+--------------------------------------+
| ID                                   | Name        | Subnets                              |
+--------------------------------------+-------------+--------------------------------------+
| 8f18c04e-3143-484c-b5f2-2762fd4c5e65 | InternalNet | 0f2f8db2-1875-4ae8-a46f-18764a40d095 |
| c205593e-d1c6-446d-9166-03b26a532b25 | ExternalNet | 46b48ee1-74d5-46e0-a4bb-951c5a63a806 |
+--------------------------------------+-------------+--------------------------------------+
```
export NET_ID=$(openstack network list | awk '/ Internal / { print $2 }')

```
openstack stack list

+--------------------------------------+------------+----------------------------------+-----------------+----------------------+--------------+
| ID                                   | Stack Name | Project                          | Stack Status    | Creation Time        | Updated Time |
+--------------------------------------+------------+----------------------------------+-----------------+----------------------+--------------+
| e4a77a42-108e-46ff-9588-d08c275d14fa | stack      | ef0fc1864fe74a57bf618baa1331dddf | CREATE_COMPLETE | 2024-04-08T10:38:46Z | None         |
+--------------------------------------+------------+----------------------------------+-----------------+----------------------+--------------+
```
create instance example
```
heat_template_version: 2021-04-16
description: only stack

parameters:
  key_pair_name:
    type: string
    label: key
    description: for_key
    default: for_wm
  image_id:
    type: string
    label: deb_11
    description: images deb
    default: deb
  image_cent:
    type: string
    label: centos
    description: images cent
    default: Centos7
  instance_type:
    type: string
    label: vm2
    description: flavor for instance 2cpu_2mem
    default: vm2
  network_id:
    type: string
    label: Net
    description: network
    default: InternalNet
resources:
  instance:
    type: OS::Nova::Server
    properties:
      image: { get_param: image_id }
      flavor: { get_param: instance_type }
      key_name: { get_param: key_pair_name }
      networks:
        - network: { get_param: network_id }
  my_inst:
    type: OS::Nova::Server
    properties:
      image: { get_param: image_cent }
      flavor: { get_param: instance_type }
      key_name: { get_param: key_pair_name }
      networks:
        - network: { get_param: network_id }
```

                                                              --barbican--
- create mysql
```
sudo mysql -u root -p
```
- create database
```
 CREATE DATABASE barbican;
```
- add privileges

```
GRANT ALL PRIVILEGES ON barbican.* TO 'barbican'@'localhost' IDENTIFIED BY '1qaz2wsx';
GRANT ALL PRIVILEGES ON barbican.* TO 'barbican'@'%' IDENTIFIED BY '1qaz2wsx';
```                                                                           
- create user
```
openstack user create --domain default --password-prompt barbican
```
- Add the admin role to the barbican user
```
openstack role add --project service --user barbican admin
```
- Create the creator role:
```
openstack role create creator

```
- Add the creator role to the barbican user
```
openstack role add --project service --user barbican creator
```
- Create the barbican service entities:
```
openstack service create --name barbican --description "Key Manager" key-manager
```
- create endpoint for barbican
```
openstack endpoint create --region RegionOne key-manager public http://<host>:9311                   # change host
openstack endpoint create --region RegionOne key-manager internal http://<host>:9311                 # change host
openstack endpoint create --region RegionOne key-manager admin http://<host>:9311
```

- install barbican 
```
yum install openstack-barbican-api
```
change /etc/barbican/barbican.conf
```
[DEFAULT]
...
sql_connection = mysql+pymysql://barbican:<password>@<host>/barbican   # change password and host

[DEFAULT]
...
transport_url = rabbit://openstack:<password>@<host>
```

change /etc/barbican/barbican.conf authtoken
```
[keystone_authtoken]
...
www_authenticate_uri = http://<host>:5000
auth_url = http://<host>:5000
memcached_servers = <host>:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = barbican
password = <password>
```
- sync db barbican

```
su -s /bin/sh -c "barbican-manage db upgrade" barbican
```
- create folder /etc/httpd/conf.d/wsgi-barbican.conf and change preference
```
  <VirtualHost [::1]:9311>
    ServerName controller

    ## Logging
    ErrorLog "/var/log/httpd/barbican_wsgi_main_error_ssl.log"
    LogLevel debug
    ServerSignature Off
    CustomLog "/var/log/httpd/barbican_wsgi_main_access_ssl.log" combined

    WSGIApplicationGroup %{GLOBAL}
    WSGIDaemonProcess barbican-api display-name=barbican-api group=barbican processes=2 threads=8 user=barbican
    WSGIProcessGroup barbican-api
    WSGIScriptAlias / "/usr/lib/python2.7/site-packages/barbican/api/app.wsgi"
    WSGIPassAuthorization On
</VirtualHost>
```
- enabled service
```
systemctl enable httpd.service
systemctl start httpd.service
```
- check secret 
```
openstack secret list
+-----------------------------------------------------------------------+----------+---------------------------+--------+-----------------------------------------+-----------+------------+-------------+------+------------+
| Secret href                                                           | Name     | Created                   | Status | Content types                           | Algorithm | Bit length | Secret type | Mode | Expiration |
+-----------------------------------------------------------------------+----------+---------------------------+--------+-----------------------------------------+-----------+------------+-------------+------+------------+
| http://localhost:9311/v1/secrets/935db1f4-904c-411d-951b-64d2511d2713 | mysecret | 2024-04-09T14:14:23+00:00 | ACTIVE | {'default': 'application/octet-stream'} | aes       |        256 | opaque      | cbc  | None       |
| http://localhost:9311/v1/secrets/7a7ce42f-fa63-4970-8a5a-0704f71b5a22 | None     | 2024-04-09T14:49:42+00:00 | ACTIVE | {'default': 'application/octet-stream'} | aes       |        256 | opaque      | cbc  | None       |
+-----------------------------------------------------------------------+----------+---------------------------+--------+-----------------------------------------+-----------+------------+-------------+------+------------+
```
- create secret file example 
```
 openstack secret store --name mysec --file new_template.yaml
+---------------+-----------------------------------------------------------------------+
| Field         | Value                                                                 |
+---------------+-----------------------------------------------------------------------+
| Secret href   | http://localhost:9311/v1/secrets/3f2c6f4d-50e2-44f5-95f7-abd2b0d310b5 |
| Name          | mysec                                                                 |
| Created       | None                                                                  |
| Status        | None                                                                  |
| Content types | None                                                                  |
| Algorithm     | aes                                                                   |
| Bit length    | 256                                                                   |
| Secret type   | opaque                                                                |
| Mode          | cbc                                                                   |
| Expiration    | None                                                                  |
+---------------+-----------------------------------------------------------------------+
```
- watch secret 

```
openstack secret get --payload  http://localhost:9311/v1/secrets/3f2c6f4d-50e2-44f5-95f7-abd2b0d310b5
+---------+-------------------------------------------------------+
| Field   | Value                                                 |
+---------+-------------------------------------------------------+
| Payload | heat_template_version: 2021-04-16                     |
|         | description: only stack                               |
|         |                                                       |
|         | parameters:                                           |
|         |   key_pair_name:                                      |
|         |     type: string                                      |
|         |     label: key                                        |
|         |     description: for_key                              |
|         |     default: for_wm                                   |
|         |   image_id:                                           |
|         |     type: string                                      |
|         |     label: deb_11                                     |
|         |     description: images deb                           |
|         |     default: deb                                      |
|         |   image_cent:                                         |
|         |     type: string                                      |
|         |     label: centos                                     |
|         |     description: images cent                          |
|         |     default: Centos7                                  |
|         |   instance_type:                                      |
|         |     type: string                                      |
|         |     label: vm2                                        |
|         |     description: flavor for instance 2cpu_2mem        |
|         |     default: vm2                                      |
|         |   network_id:                                         |
|         |     type: string                                      |
|         |     label: Net                                        |
|         |     description: network                              |
|         |     default: InternalNet                              |
|         | resources:                                            |
|         |   Group_of_VMs:                                       |
|         |     type: OS::Heat::ResourceGroup                     |
|         |     properties:                                       |
|         |       count: 4                                        |
|         |       resource_def:                                   |
|         |         instance:                                     |
|         |           type: OS::Nova::Server                      |
|         |           properties:                                 |
|         |             image: { get_param: image_id }            |
|         |             flavor: { get_param: instance_type }      |
|         |             key_name: { get_param: key_pair_name }    |
|         |             networks:                                 |
|         |             - network: { get_param: network_id }      |
|         |                                                       |
|         |               # my_inst:                              |
|         |               #type: OS::Nova::Server                 |
|         |               # properties:                           |
|         |               #image: { get_param: image_cent }       |
|         |               #flavor: { get_param: instance_type }   |
|         |               #key_name: { get_param: key_pair_name } |
|         |               #networks:                              |
|         |               #- network: { get_param: network_id }   |
|         |                                                       |
+---------+-------------------------------------------------------+
```
Secret types

There are a few types of secrets that are handled by barbican:

- symmetric - Used for storing byte arrays such as keys suitable for symmetric encryption.
- public - Used for storing the public key of an asymmetric keypair.
- private - Used for storing the private key of an asymmetric keypair.
- passphrase - Used for storing plain text passphrases.
- certificate - Used for storing cryptographic certificates such as X.509 certificates.
- opaque - Used for backwards compatibility with previous versions of the API without typed secrets. New applications are encouraged to specify one of the other secret types.

create test secret  passphrase
```
openstack secret store --secret-type passphrase --name "test passphrase" --payload 'aVerYSecreTTexT!'
```
                                             
--Ceilometer--

![Openstack](https://github.com/tulamelkii/openstack/blob/main/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-04-18%20114140.png)

- Create the ceilometer user
```
openstack user create --domain default --password-prompt ceilometer
```
- Add the admin role to the ceilometer user
```
openstack role add --project service --user ceilometer admin
```
- Create the ceilometer service
```
openstack service create --name ceilometer --description "Telemetry" metering
```
- Create the gnocchi user:
```
openstack user create --domain default --password-prompt gnocchi
```
- Create the gnocchi service
```
openstack service create --name gnocchi --description "Metric Service" metric
```
- Add the admin role to the gnocchi user
```
openstack role add --project service --user gnocchi admin
```
- Create the Metric service API endpoint
```
openstack endpoint create --region RegionOne metric public http://<host>:8041
openstack endpoint create --region RegionOne metric internal http://<host>:8041
openstack endpoint create --region RegionOne metric admin http://<host>:8041
```
- Install Gnocchi
```
dnf install openstack-gnocchi-api openstack-gnocchi-metricd python3-gnocchiclient
```
- create DB gnocchi
```
CREATE DATABASE gnocchi;
```
- create privileges fot user gnocchi'@'localhost
```
GRANT ALL PRIVILEGES ON gnocchi.* TO 'gnocchi'@'localhost' IDENTIFIED BY 'GNOCCHI_DBPASS';
GRANT ALL PRIVILEGES ON gnocchi.* TO 'gnocchi'@'%' IDENTIFIED BY 'GNOCCHI_DBPASS';
GRANT ALL PRIVILEGES ON gnocchi.* TO 'gnocchi'@'<host>' IDENTIFIED BY 'GNOCCHI_DBPASS';      # work access + host
```
- Change preference /etc/gnocchi/gnocchi.conf
```
[DEFAULT]
log_dir = /var/log/gnocchi


[api]
auth_mode = keystone

[keystone_authtoken]
www_authenticate_uri = http://<host>:5000/v3
auth_url = http://<host>:5000/v3
memcached_servers = <host>:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = gnocchi
password = <pass_gnoch>
interface = internalURL
region_name = RegionOne
service_token_roles_required = true

[indexer]
url = mysql+pymysql://gnocchi:<pass>@<host>:3306/gnocchi


[storage]
file_basepath = /var/lib/gnocchi
driver = file
```
- create new file /etc/httpd/conf.d/10-gnocchi_wsgi.conf
```
vim /etc/httpd/conf.d/10-gnocchi_wsgi.conf
```
- change /etc/httpd/conf.d/10-gnocchi_wsgi.conf
```
Listen 8041
<VirtualHost *:8041>
  <Directory /usr/bin>
    AllowOverride None
    Require all granted
  </Directory>

  CustomLog /var/log/httpd/gnocchi_wsgi_access.log combined
  ErrorLog /var/log/httpd/gnocchi_wsgi_error.log
  SetEnvIf X-Forwarded-Proto https HTTPS=1
  WSGIApplicationGroup %{GLOBAL}
  WSGIDaemonProcess gnocchi display-name=gnocchi_wsgi user=gnocchi group=gnocchi processes=6 threads=6
  WSGIProcessGroup gnocchi
  WSGIScriptAlias / /usr/bin/gnocchi-api
</VirtualHost>
```
- add privileges and change group
``` 
chmod 640 /etc/gnocchi/gnocchi.conf
chgrp gnocchi /etc/gnocchi/gnocchi.conf
```
upgrade gnocchi, (write db gnocchi and preference)
```
 su -s /bin/bash gnocchi -c "gnocchi-upgrade"
```
- enable and start the Gnocchi services
```
systemctl  openstack-gnocchi-metricd.service
systemctl start  openstack-gnocchi-metricd.service

```
 ## Service openstack-gnocchi-api.service disanble if you use wsgi!!!!!
 
```
```
- # CHECK
```
export OS_AUTH_TYPE=password
gnocchi resource list
```
 OK if no error is shown!!!
---

 - Install and configure components

```
dnf install openstack-ceilometer-notification openstack-ceilometer-central
```
- change yaml /etc/ceilometer/pipeline.yaml
```
publishers:
    - gnocchi://?filter_project=service&archive_policy=low   # add

```
- edit /etc/ceilometer/ceilometer.conf
```
[DEFAULT]
transport_url = rabbit://openstack:<pass>@<host>

[service_credentials]

auth_url = http://<host>:5000/v3
memcached_servers = <host>:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = ceilometer
password = <pass>
region_name = RegionOne


[keystone_authtoken]
www_authenticate_uri = http://<host>:5000/v3
auth_url = http://<host>:5000/v3
memcached_servers = <host>:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = ceilometer
password = 1qaz2wsx
service_token_roles_required = true

```
Add rulles and  ceilometer-upgrade
```
chmod 640 /etc/ceilometer/ceilometer.conf
su -s /bin/bash ceilometer -c "ceilometer-upgrade --skip-metering-database"
```
- configure nova use Telemetry
```
[DEFAULT]
instance_usage_audit = True
instance_usage_audit_period = hour
notify_on_state_change = vm_and_task_state

[oslo_messaging_notifications]
driver = messagingv2
```
edit privileges /etc/sudoers
```
ceilometer ALL = (root) NOPASSWD: /usr/bin/ceilometer-rootwrap /etc/ceilometer/rootwrap.conf *
```
- Edit the /etc/ceilometer/polling.yaml
```
- name: ipmi
  interval: 300
  meters:
    - hardware.ipmi.temperature     #add string
```
Start service and restart Nova compute
```
 systemctl enable openstack-ceilometer-compute.service
systemctl start openstack-ceilometer-compute.service
systemctl enable openstack-ceilometer-ipmi.service (optional)
systemctl start openstack-ceilometer-ipmi.service (optional)
systemctl restart openstack-nova-compute.service
```
- Change Cinder driver /etc/cinder/cinder.conf
```
[oslo_messaging_notifications]
driver = messagingv2
```
- restart service 
```
systemctl restart openstack-cinder-api.service openstack-cinder-scheduler.service
systemctl restart openstack-cinder-volume.service
```
- Edit the /etc/glance/glance-api.conf
```
[DEFAULT]
transport_url = rabbit://openstack:<password>@<host>
[oslo_messaging_notifications]
driver = messagingv2
```
- restart service glance
```
systemctl restart openstack-glance-api.service
```
- Edit the /etc/heat/heat.conf
```
[oslo_messaging_notifications]
driver = messagingv2
```
- restart services heat 
```
systemctl restart openstack-heat-api.service openstack-heat-api-cfn.service openstack-heat-engine.service
```
- Edit the /etc/neutron/neutron.conf
```
[oslo_messaging_notifications]
driver = messagingv2
```
- restart service
```
systemctl restart neutron-server.service
```
## Verify operation

```
gnocchi resource list  --type image

[root@Only ~]# gnocchi resource list


openstack service create --name gnocchi --description "Metric Service" metric

[root@Only ~]# gnocchi resource list
+--------------------------------------+----------------------+----------------------------------+----------------------------------+--------------------------------------+----------------------------------+----------+----------------------------------+--------------+----------------
| id                                   | type                 | project_id                       | user_id                          | original_resource_id                 | started_at                       | ended_at | revision_start                   | revision_end | creator        |                                      |                      |                                  |                                  |                                      |                                  |          |                                  |              |   |
+--------------------------------------+----------------------+----------------------------------+----------------------------------+--------------------------------------+----------------------------------+----------+----------------------------------+--------------+----------------
| 6246de73-cef5-5721-884f-139c8f151770 | volume_provider_pool | None                             | None                             | Only@lvm#LVM                         | 2024-04-17T08:58:54.896733+00:00 | None     | 2024-04-17T08:58:54.896741+00:00 | None         | 9 |
| aa94bf6e-3afb-5711-8f8a-0d41dae51236 | volume_provider      | None                             | None                             | Only@lvm                             | 2024-04-17T08:58:54.949741+00:00 | None     | 2024-04-17T08:58:54.949748+00:00 | None         |   |
| feaa994c-7b3f-4990-921d-4a45978da07f | instance             | ef0fc1864fe74a57bf618baa1331dddf | 8ba0260e8afc4f73856b2da075e81213 | feaa994c-7b3f-4990-921d-4a45978da07f | 2024-04-17T12:00:59.949555+00:00 | None     | 2024-04-17T12:00:59.949564+00:00 | None         |   |
| 774b132a-42a0-4949-936d-166d02e9a087 | instance             | ef0fc1864fe74a57bf618baa1331dddf | 8ba0260e8afc4f73856b2da075e81213 | 774b132a-42a0-4949-936d-166d02e9a087 | 2024-04-17T12:00:59.961904+00:00 | None     | 2024-04-17T12:00:59.961911+00:00 | None         |   |
+--------------------------------------+----------------------+----------------------------------+----------------------------------+--------------------------------------+----------------------------------+----------+----------------------------------+--------------+----------------
```
 
                                                                                                                                      --- AODH--

- create db aodh;

```
mysql -u root -p
CREATE DATABASE aodh;
```
add privilege
```
GRANT ALL PRIVILEGES ON aodh.* TO 'aodh'@'localhost' IDENTIFIED BY '<pass>';
GRANT ALL PRIVILEGES ON aodh.* TO 'aodh'@'%' IDENTIFIED BY '<pass>';
GRANT ALL PRIVILEGES ON aodh.* TO 'aodh'@'<host>' IDENTIFIED BY '<pass>';
```
- create user
```
openstack user create --domain default --password-prompt aodh
```
- create service 
```
openstack service create --name aodh --description "Telemetry" alarming
```
create endpoint
```
openstack endpoint create --region RegionOne alarming public http://Only:8042
openstack endpoint create --region RegionOne alarming internal http://Only:8042
openstack endpoint create --region RegionOne alarming admin http://Only:8042
```
- Install the packages:
```
yum install openstack-aodh-api openstack-aodh-evaluator openstack-aodh-notifier openstack-aodh-listener openstack-aodh-expirer python-aodhclient

```
- Edit the /etc/aodh/aodh.conf
```
[database]
connection = mysql+pymysql://aodh:<pass>@<Only>/aodh

[DEFAULT]
transport_url = rabbit://openstack:<only> @<only>
[api]
auth_strategy = keystone

[keystone_authtoken]
www_authenticate_uri = http://<host>:5000/v3
auth_url = http://<host>:5000/v3
memcached_servers = <host>:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = aodh
password = <pass>
service_token_roles_required = true


[service_credentials]

auth_url = http://Only:5000/v3
memcached_servers = Only:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = ceilometer
password = aodh
region_name = RegionOne
interface = internalURL

```
create wsgi
```
vim /etc/httpd/conf.d/20-aodh_wsgi.conf

Listen 8042
<VirtualHost *:8042>
    DocumentRoot "/var/www/cgi-bin/aodh"
    <Directory "/var/www/cgi-bin/aodh">
        AllowOverride None
        Require all granted
    </Directory>

    CustomLog "/var/log/httpd/aodh_wsgi_access.log" combined
    ErrorLog "/var/log/httpd/aodh_wsgi_error.log"
    SetEnvIf X-Forwarded-Proto https HTTPS=1
    WSGIApplicationGroup %{GLOBAL}
    WSGIDaemonProcess aodh display-name=aodh_wsgi user=aodh group=aodh processes=6 threads=3
    WSGIProcessGroup aodh
    WSGIScriptAlias / "/var/www/cgi-bin/aodh/app"
</VirtualHost>
```
 add privileges
 ```
 chmod 640 /etc/aodh/aodh.conf
 chgrp aodh /etc/aodh/aodh.conf
 mkdir /var/www/cgi-bin/aodh
cp /usr/lib/python3.9/site-packages/aodh/api/app.wsgi /var/www/cgi-bin/aodh/app
chown -R aodh. /var/www/cgi-bin/aodh
```
sync db 
```
su -s /bin/bash aodh -c "aodh-dbsync"
```
- restart service aodh
```
 systemctl start openstack-aodh-evaluator openstack-aodh-notifier openstack-aodh-listener
 systemctl enable openstack-aodh-evaluator openstack-aodh-notifier openstack-aodh-listener
 ```


                                                                                                                --Nova--
                                                                                                                
                                                                      
Nova - For managing amphora lifecycle and spinning up compute resources on demand.

Neutron - For network connectivity between amphorae, tenant environments, and external networks.

Barbican - For managing TLS certificates and credentials, when TLS session termination is configured on the amphorae.

Keystone - For authentication against the Octavia API, and for Octavia to authenticate with other OpenStack projects.

Glance - For storing the amphora virtual machine image.

Oslo - For communication between Octavia controller components, making Octavia work within the standard OpenStack framework and review system, and project code structure.

Taskflow - Is technically part of Oslo; however, Octavia makes extensive use of this job flow system when orchestrating back-end service configuration and management.



Check service
![Openstack](https://github.com/tulamelkii/openstack/blob/main/%D0%A1%D0%BD%D0%B8%D0%BC%D0%BE%D0%BA%20%D1%8D%D0%BA%D1%80%D0%B0%D0%BD%D0%B0%202024-04-10%20140451.png)

masakari-api
An OpenStack-native REST API that processes API requests by sending them to the masakari-engine over Remote Procedure Call (RPC).

masakari-engine
Processes the notifications received from masakari-api by executing the recovery workflow in asynchronous way.


good luck
--------------------------------------------


SESSION_ENGINE = 'django.contrib.sessions.backends.cache'

SESSION_ENGINE = 'django.contrib.sessions.backends.file'

