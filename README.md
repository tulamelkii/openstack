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

```
[database]
# ...
connection = mysql+pymysql://keystone:<password_for_db_ceystone>@controller/keystone
``` 
- edit token provider

[token]
# ...
provider = fernet
```
run sync mariadb and module keystone 

su -s /bin/sh -c "keystone-manage db_sync" keystone
  
