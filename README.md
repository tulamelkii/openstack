
                                                                              --1--

## Keystone Service 

Keystone - install on the controller, this is service who create main catalog for  autinfication and authorization

Keystone needs for interactions services and users

This is fist identifity service, who use users with autinfication and authorization
- first
```
 dnf config-manager --enable crb # enable full packages for centos(this if extra package)
yum update
```
- thecond change host vim /etc/hosts.. <name vm>
- enter ip and <name vm>   in /etc/hosts

- add repo list   /etc/yum.repos.d/centos.repo
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
