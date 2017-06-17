# concrete5 Ansible to setup Apache/Nginx/MySQL/MariaDB on an AWS EC2 Amazon Linux instance

**Work-in-Progress**: Your input is greatly appreciated.

This is a simple Ansible script to setup Apache or nginx, MariaDB or MySQL and Basic Auth into an EC2 instance.
You need to install Ansible in your PC to run.

You will connect to EC2 server and start sending commands to setup the server automatically.

-----

[日本語はこちら](README-ja.md)

## Installation

Install Ansible using Homebrew

- Homebrew

```
  $ brew install ansible
```

OR

- pip

```
  $ pip install ansible
```

## Generate SSH Key for each user

This is separate SSH key pair from `ec2-user` default SSH key.

If you want to create new user(s), and you will use this SSH key to login.

**Please start naming user different from c5juser, such as client-name, such as `CLIENT-ssh`** as we kept having the many c5juser ssh users and we may get confused.

Generate and save the SSH key inside of [./roles/add_users/files/]

File name must be `username`.pem. Default is c5juser.pem, but please change it so that it's easier to remember.

- ssh-keygen command sample

```
  ssh-keygen -f ./roles/default_setup/files/"USERNAME".pem -C "USERNAME"@localhost
```

  *It will generate [username.pem] [username.pen.pub] files.

## Execute CloudFormation script & confirms SSH access

**I haven't published CloudFormation Script yet.** Just launch an Amazon LINUX EC2 instance with public IP access, then setup DNS record to be accessible from domain name.

- Execute `ec2-simple`  CloudFormation Script to launch an EC2 Instance. (OR you could create an instance manually)
- Obtain an IP address of the EC2 instance
    - Modify the security group setting if the client requires IP address blocking from AWS Console.
- Obtain the ec2-user SSH key from AWS
- Check to see if you can access to newly created EC2 instance via SSH


## Modify the settings

- Edit host.yml, setup.yml reading the instruction below.
    - If there is a new version of concrete5, make sure to replace it
    - This package contains concrete5 package as a zip file. HOWEVER, the zip file which is distributed at concrete5.org contains a folder with `concrete5-[version]`. You must re-zip the package without the folder.

## AnsiblePlayBook

Execute Ansible, sit and wait.

```
$ cd [path/to/ansible]
$ ansible-playbook -i host.yml setup.yml
```

## Set DNS & Some Security Measure

- Go to Route 53 to set DNS
- Set additional security measure such as setting up basic auth, or IP address restriction.

-----

HOW TO SET CONFIGURATION
====================

# host.yml

Set target IP address, and SSH username (usually `ec2-user`), and SSH key

## Sample:

  ```
  [servers]
  192.168.0.1 server_name="hoge01.hogehoge.com"
  [all:vars]
  ansible_ssh_user=ec2-user
  ansible_ssh_private_key_file=/PATH/TO/ec2-usr-key.pem
  ```

## [servers] Section

- IP Address

  - EC2 intance publicIP

- server_name=

  - Indicate the host name that you would like to set.

## [all:vars]

- ansible_ssh_user=ec2-user

  - the name of sudo user, usually `ec2-user`

- ansible_ssh_private_key_file=

  - Path to your SSH key in your Mac

-----

# setup.yml

Change the parameters where necessaries.

## Instance Type

We have a few templates that optimized for each instance types.

```
  - aws_instance_type:       "t2small"
```

### Available Types

Currently we only have the following setting.

- t2micro
- t2small

## Add & Name your SSH user

- **[if_add_users] : "Yes" to create sudo users with attached SSH key. Please make sure to generate them and store it under `add_users/files` with the format of `[username].pem` secret ssh key and `[username].pem.pub` public key.
- **[add_users]** : Name your SSH username

**Please start naming user different from c5juser, such as client-name, such as `CLIENT-ssh`** as we kept having the many c5juser ssh users and we may get confused.

  ```
  - add_users:
    - "c5juser"
  ```

If you want, you can add multiple users by using the following format.
However, you also need to create additional SSH keys

```
    - add_users:
      - "user01"
      - "user02"
```

## Name user and group

Please indicate user and group of concrete5 folder owner. Apache or Nginx will run as this user and group as well.

```
- c5dir_user:        "c5juser"
- c5dir_group:       "apache"
```

## Select Web Server Type

- **[webserver_handle]** : Select Apache or Nginx

  `- webserver_handle: "apache"`

## Virtualhost Setting

- **[vhost_domain]** : The domain names to set。

  `- vhost_domain: "www.example.com"`

- **[vhost_docroot]** : Set the web document root path

  `- vhost_docroot: "/var/www/vhosts/"`

  With the setting above, it will set `www.example.com` as virtual host, and the document root of that domain will be "/var/www/vhosts/www.example.com"

## DB Environment

Choose your DB environment(mariadb / mariadb-client / mysql / mysql-client / none )
    - mariadb: Install MariaDB Server & Client
    - mariadb-client: Install MariaDB Client
    - mysql: Install MySQL Server & Client
    - mysql-client: Install only MySQL Client
    - none: do nothing

```
  - db_environment:         "mariadb"
```

## MySQL or MariaDB Master Setting

- **[mysql_rootpass]** : MySQL's root password

  `- mysql_rootpass: "mysql_root_pass"`

- **[mysql_loginhost]** : MySQL Host address, you don't change it usually.

  `- mysql_loginhost: "127.0.0.1"`

- **[mysql_loginuser]** : MySQL root user password. Please keep it as "root" usually.

  `- mysql_loginuser: "root"`

- **[mysql_loginpass]** : Please keep it same as [mysql_rootpass]

  `- mysql_loginpass: "mysql_root_pass"`

## MySQL Setting for concrete5

It will login as [mysql_loginuser] user and create database and additional user based on the following information.

- **[mysql_dbname]** : Set the name of concrete5 MySQL database. Leave it as default unless instructed.

  `- mysql_dbname: "dev-c5db"`

- **[mysql_username]** : Set concrete5's MySQL DB username. Leave it as default unless instructed.

  `- mysql_username: "dev-c5dbuser"`

- **[mysql_userpass]** : Set concrete5's MySQL User password. Make sure to make random and secure password that fits MySQL password restriction.

  `- mysql_userpass: "db-password"`

- **[mysql_userpermitedhost]** : Setting of MySQL host names, you usually don't change it.

  ```
  - mysql_userpermitedhost:

    - "127.0.0.1"
    - "localhost"
  ```

## concrete5 Installation Setting

- **[c5_sitename]** : Enter the name of concrete5 sitename.

  `- c5_sitename: "concrete5 Demo Site"`

- **[c5dir_user]** : Enter the username of directory owner.

  `- c5dir_user: "c5juser"`

- **[c5dir_user]** : Enter the groupname of directory owner. Our convention is "apache" for apache server and "nginx" for nginx server.

  `- c5dir_group: "apache"`  

- **[c5_starting_point]** : Select the starting point, it's either "elemental_blank" or "elemental_full".

  `- c5_starting_point: "elemental_blank"`

- **[c5_admin_email]** : Set concrete5 Admin user email address. Make it as server@concrete5.co.jp unless instructed.

  `- c5_admin_email: "info@example.com"`

- **[c5_admin_pass]** : Set concrete5 Admin user password. Please generate the secure password, and make a note.

  `- c5_admin_pass: "site_password"`


  `- c5_locale:         "ja_JP"`

  - **[c5_locale]** : Set concrete5 default site locale

-----

## NewRelic Setting

If you're not using, it please ignore it.
NewRelic is server monitoring service, if you're using it, you must set the setting.

- **[use_newrelic]** : Set `yes`, if you want to install it

  `- use_newrelic:           "no"`

  - **[use_newrelicphp]** : Set `yes`, if you're going to use newrelic.php

  `- use_newrelicphp:        "no"`

  - **[newrelic_licensekey]** : Enter your Newrelic license key.

  `- newrelic_licensekey:　"xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"`


  - **[newrelic_appname]** : If you're using NewRelic-php, set the name of application

  `- newrelic_appname: "PHP Application_name"`

  -----

# Role Settings

- **[roles]** : Role contains the set up commands for Ansible. This package contains group set of commands. According to `roles`, it will execute each role in order, also you can use a little condition as well.

 ```
   roles:
   - role: default_setup
     when: server_environment=="prod"
   - role: add_users
     when: if_add_users=="yes"
   - role: apache
     when: webserver_handle=="apache"
   - role: nginx
     when: webserver_handle=="nginx"
   - role: web_dummy
   - role: mysql_client
     when: db_environment in ['mysql', 'mysql-client', 'mariadb-client']
   - role: mysql_server
     when: db_environment=="mysql"
   - role: mariadb_repo
     when: db_environment in ['mariadb', 'mariadb-client']
 #  - role: mariadb_client
 #    when: db_environment in ['mariadb', 'mariadb-client']
   - role: mariadb_server
     when: db_environment=="mariadb"
   - role: mysql_appdb
   - role: concrete5
   - role: newrelic
     when: use_newrelic=="yes"
   - role: newrelic-php
     when: use_newrelicphp=="yes"
  ```