Keystone modules - install on the controller, this is service who create main catalog for  autinfication and authorization

Keystone needs for interactions services and users

This is fist identifity service, who use users with autinfication and authorization
- first  yum update
- Thecond install maria db and add repo list   /etc/yum.repos.d/centos.repo
```
[mariadb]
name = MariaDB
baseurl = https://rpm.mariadb.org/10.6/rhel/$releasever/$basearch
gpgkey= https://rpm.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```
second istall server and client Maria db

- sudo yum install MariaDB-server MariaDB-client
(and how watch package yum whatprovides "Maria*"

Start db
 - systemctl start mariadb

   check status active and go to maria

-  mysql -u root -p                                 # login for db  and press enter  )
-  MariaDB [(none)]> CREATE DATABASE keystone;      # Create db keystone
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
- edit config  /etc/keystone/keystone.conf add password
- create coonection who connected 
```
[database]
# ...
connection = mysql+pymysql://keystone:<password_for_db_ceystone>@<ip:port>/keystone   #change password and ip port controller
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
  --bootstrap-admin-url http://<ip_controller>:5000/v3/ \
  --bootstrap-internal-url http://ip :5000/v3/ \
  --bootstrap-public-url http://10.11.213.200:5000/v3/ \
  --bootstrap-region-id RegionOn
```
add server name for apcahe
change preference   /etc/httpd/conf/httpd.conf
```
servername
