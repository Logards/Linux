# Partie 1 : Mise en place et maîtrise du serveur Web

🌞 **Installer le serveur Apache**
```
[logards@Nezuko ~]$ sudo dnf install httpd
[sudo] password for logards: 
Last metadata expiration check: 0:04:02 ago on Tue 17 Jan 2023 10:12:15 AM CET.
Dependencies resolved.
[logards@Nezuko ~]$ sudo vim /etc/httpd/conf/httpd.conf
[sudo] password for logards: 
[logards@Nezuko ~]$ cat /etc/httpd/conf/httpd.conf

ServerRoot "/etc/httpd"

Listen 80

Include conf.modules.d/*.conf

User apache
Group apache


ServerAdmin root@localhost


<Directory />
    AllowOverride none
    Require all denied
</Directory>


DocumentRoot "/var/www/html"

<Directory "/var/www">
    AllowOverride None
    Require all granted
</Directory>

<Directory "/var/www/html">
    Options Indexes FollowSymLinks

    AllowOverride None

    Require all granted
</Directory>

<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>

<Files ".ht*">
    Require all denied
</Files>

ErrorLog "logs/error_log"

LogLevel warn

<IfModule log_config_module>
    LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
    LogFormat "%h %l %u %t \"%r\" %>s %b" common

    <IfModule logio_module>
      LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio
    </IfModule>


    CustomLog "logs/access_log" combined
</IfModule>

<IfModule alias_module>


    ScriptAlias /cgi-bin/ "/var/www/cgi-bin/"

</IfModule>

<Directory "/var/www/cgi-bin">
    AllowOverride None
    Options None
    Require all granted
</Directory>

<IfModule mime_module>
    TypesConfig /etc/mime.types

    AddType application/x-compress .Z
    AddType application/x-gzip .gz .tgz



    AddType text/html .shtml
    AddOutputFilter INCLUDES .shtml
</IfModule>

AddDefaultCharset UTF-8

<IfModule mime_magic_module>
    MIMEMagicFile conf/magic
</IfModule>


EnableSendfile on

IncludeOptional conf.d/*.conf
```

```
[logards@Nezuko ~]$ sudo systemctl start httpd.service
[logards@Nezuko ~]$ systemctl status httpd.service
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
     Active: active (running) since Tue 2023-01-17 10:25:54 CET; 11s ago
       Docs: man:httpd.service(8)
   Main PID: 10746 (httpd)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes served/sec:   0 B/sec"
      Tasks: 213 (limit: 5905)
     Memory: 23.1M
        CPU: 72ms
     CGroup: /system.slice/httpd.service
             ├─10746 /usr/sbin/httpd -DFOREGROUND
             ├─10747 /usr/sbin/httpd -DFOREGROUND
             ├─10748 /usr/sbin/httpd -DFOREGROUND
             ├─10749 /usr/sbin/httpd -DFOREGROUND
             └─10750 /usr/sbin/httpd -DFOREGROUND

Jan 17 10:25:21 Nezuko systemd[1]: Starting The Apache HTTP Server...
Jan 17 10:25:35 Nezuko httpd[10746]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using fe80::a00:27ff:fe09:c699%enp0s3. Set the 'ServerName' directive globally to suppr>
Jan 17 10:25:54 Nezuko systemd[1]: Started The Apache HTTP Server.
Jan 17 10:25:54 Nezuko httpd[10746]: Server configured, listening on: port 80
[logards@Nezuko ~]$ sudo systemctl enable httpd.service
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
[logards@Nezuko ~]$ sudo ss -alutnp |grep httpd
tcp   LISTEN 0      511                *:80              *:*    users:(("httpd",pid=10750,fd=4),("httpd",pid=10749,fd=4),("httpd",pid=10748,fd=4),("httpd",pid=10746,fd=4))
[logards@Nezuko ~]$ sudo firewall-cmd --add-port 80/tcp --permanent
success
[logards@Nezuko ~]$ sudo firewall-cmd --reload
success
[logards@Nezuko ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 80/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```


```bash
# Demander à systemd les logs relatifs au service httpd
$ sudo journalctl -xe -u httpd

# Consulter le fichier de logs d'erreur d'Apache
$ sudo cat /var/log/httpd/error_log

# Il existe aussi un fichier de log qui enregistre toutes les requêtes effectuées sur votre serveur
$ sudo cat /var/log/httpd/access_log
```

🌞 **TEST**

```
[logards@Nezuko ~]$ systemctl status httpd.service
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
     Active: active (running) since Tue 2023-01-17 10:25:54 CET; 4min 50s ago
       Docs: man:httpd.service(8)
   Main PID: 10746 (httpd)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes served/sec:   0 B/sec"
      Tasks: 213 (limit: 5905)
     Memory: 23.1M
        CPU: 261ms
     CGroup: /system.slice/httpd.service
             ├─10746 /usr/sbin/httpd -DFOREGROUND
             ├─10747 /usr/sbin/httpd -DFOREGROUND
             ├─10748 /usr/sbin/httpd -DFOREGROUND
             ├─10749 /usr/sbin/httpd -DFOREGROUND
             └─10750 /usr/sbin/httpd -DFOREGROUND

Jan 17 10:25:21 Nezuko systemd[1]: Starting The Apache HTTP Server...
Jan 17 10:25:35 Nezuko httpd[10746]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using fe80::a00:27ff:fe09:c699%enp0s3. Set the 'ServerName' directive globally to suppr>
Jan 17 10:25:54 Nezuko systemd[1]: Started The Apache HTTP Server.
Jan 17 10:25:54 Nezuko httpd[10746]: Server configured, listening on: port 80
[logards@Nezuko ~]$ systemctl status httpd.service | grep enabled
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
[logards@Nezuko ~]$ curl localhost | head 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/
      
      html {
100  7620  100  7620    0     0   676k      0 --:--:-- --:--:-- --:--:--  676k
curl: (23) Failed writing body


[bastien@UWU ~]$ curl http://10.105.1.11:80 | head
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 <!doctype html>0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
 <html>
7  <head>
6    <meta charset='utf-8'>
2    <meta name='viewport' content='width=device-width, initial-scale=1'>
0    <title>HTTP Server Test Page powered by: Rocky Linux</title>
     <style type="text/css">
       /*<![CDATA[*/
1      
0      html {
0  7620    0     0  3823k      0 --:--:-- --:--:-- --:--:-- 7441k
curl: (23) Failed writing body
```

## 2. Avancer vers la maîtrise du service

🌞 **Le service Apache...**

```
[logards@Nezuko ~]$ cat /usr/lib/systemd/system/httpd.service
# See httpd.service(8) for more information on using the httpd service.

# Modifying this file in-place is not recommended, because changes
# will be overwritten during package upgrades.  To customize the
# behaviour, run "systemctl edit httpd" to create an override unit.

# For example, to pass additional options (such as -D definitions) to
# the httpd binary at startup, create an override unit (as is done by
# systemctl edit) and enter the following:

#       [Service]
#       Environment=OPTIONS=-DMY_DEFINE

[Unit]
Description=The Apache HTTP Server
Wants=httpd-init.service
After=network.target remote-fs.target nss-lookup.target httpd-init.service
Documentation=man:httpd.service(8)

[Service]
Type=notify
Environment=LANG=C

ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
# Send SIGWINCH for graceful stop
KillSignal=SIGWINCH
KillMode=mixed
PrivateTmp=true
OOMPolicy=continue

[Install]
WantedBy=multi-user.target
```

🌞 **Déterminer sous quel utilisateur tourne le processus Apache**


```
[logards@Nezuko ~]$ cat /etc/httpd/conf/httpd.conf | grep User | head -n 1
User apache
[logards@Nezuko ~]$ ps -ef | grep apache
apache     10747   10746  0 10:25 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache     10748   10746  0 10:25 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache     10749   10746  0 10:25 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache     10750   10746  0 10:25 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
logards    11078     850  0 10:42 pts/0    00:00:00 grep --color=auto apache
[logards@Nezuko ~]$ ls -al /usr/share/testpage/
total 12
drwxr-xr-x.  2 root root   24 Jan 17 10:16 .
drwxr-xr-x. 82 root root 4096 Jan 17 10:16 ..
-rw-r--r--.  1 root root 7620 Jul 27 20:05 index.html
```

🌞 **Changer l'utilisateur utilisé par Apache**
```
[logards@Nezuko ~]$ sudo useradd Web -d /usr/share/httpd -s /sbin/nologin
useradd: warning: the home directory /usr/share/httpd already exists.
useradd: Not copying any file from skel directory into it.
[logards@Nezuko ~]$ cat /etc/passwd | grep Web
Web:x:1001:1001::/usr/share/httpd:/sbin/nologin
[logards@Nezuko ~]$ cat /etc/httpd/conf/httpd.conf | grep Web
User Web
[logards@Nezuko ~]$ sudo systemctl restart httpd.service
[logards@Nezuko ~]$ systemctl status httpd.service
● httpd.service - The Apache HTTP Server
     Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
     Active: active (running) since Tue 2023-01-17 10:55:59 CET; 58s ago
       Docs: man:httpd.service(8)
   Main PID: 11138 (httpd)
     Status: "Total requests: 0; Idle/Busy workers 100/0;Requests/sec: 0; Bytes served/sec:   0 B/sec"
      Tasks: 213 (limit: 5905)
     Memory: 22.7M
        CPU: 118ms
     CGroup: /system.slice/httpd.service
             ├─11138 /usr/sbin/httpd -DFOREGROUND
             ├─11139 /usr/sbin/httpd -DFOREGROUND
             ├─11140 /usr/sbin/httpd -DFOREGROUND
             ├─11141 /usr/sbin/httpd -DFOREGROUND
             └─11142 /usr/sbin/httpd -DFOREGROUND

Jan 17 10:55:14 Nezuko systemd[1]: Starting The Apache HTTP Server...
Jan 17 10:55:39 Nezuko httpd[11138]: AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using fe80::a00:27ff:fe09:c699%enp0s3. Set the 'ServerName' directive globally to suppr>
Jan 17 10:55:59 Nezuko systemd[1]: Started The Apache HTTP Server.
Jan 17 10:55:59 Nezuko httpd[11138]: Server configured, listening on: port 80
[logards@Nezuko ~]$ ps -ef | grep Web
Web        11139   11138  0 10:55 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
Web        11140   11138  0 10:55 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
Web        11141   11138  0 10:55 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
Web        11142   11138  0 10:55 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
logards    11359     850  0 10:57 pts/0    00:00:00 grep --color=auto Web
```

🌞 **Faites en sorte que Apache tourne sur un autre port**

```
[logards@Nezuko ~]$ cat /etc/httpd/conf/httpd.conf | grep Listen
Listen 8888
[logards@Nezuko ~]$ sudo firewall-cmd --add-port 8888/tcp --permanent
success
[logards@Nezuko ~]$ sudo firewall-cmd --reload
success
[logards@Nezuko ~]$ sudo firewall-cmd --remove-port 80/tcp --permanent
success
[logards@Nezuko ~]$ sudo firewall-cmd --reload
success
[logards@Nezuko ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 8888/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[logards@Nezuko ~]$ sudo ss -alutnp | grep httpd
[sudo] password for logards: 
tcp   LISTEN 0      511                *:8888            *:*    users:(("httpd",pid=11446,fd=4),("httpd",pid=11445,fd=4),("httpd",pid=11444,fd=4),("httpd",pid=11442,fd=4))
[logards@Nezuko ~]$ curl localhost:8888 | head 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
<!doctype html>    0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/
      
      html {
100  7620  100  7620    0     0   930k      0 --:--:-- --:--:-- --:--:--  930k
curl: (23) Failed writing body
```


🌞 **Install de MariaDB sur `db.tp5.linux`**

- déroulez [la doc d'install de Rocky](https://docs.rockylinux.org/guides/database/database_mariadb-server/)
- je veux dans le rendu **toutes** les commandes réalisées
- faites en sorte que le service de base de données démarre quand la machine s'allume
  - pareil que pour le serveur web, c'est une commande `systemctl` fiez-vous au mémo

```
[logards@Tanjiro ~]$ sudo dnf install mariadb-server
[sudo] password for logards: 
Rocky Linux 9 - BaseOS                                                                                                                                                              9.0 kB/s | 3.6 kB     00:00    
Rocky Linux 9 - BaseOS                                                                                                                                                              2.3 MB/s | 1.7 MB     00:00    
Rocky Linux 9 - AppStream                                                                                                                                                            14 kB/s | 4.1 kB     00:00    
Rocky Linux 9 - AppStream                                                                                                                                                           3.9 MB/s | 6.4 MB     00:01    
Rocky Linux 9 - Extras                                                                                                                                                               10 kB/s | 2.9 kB     00:00    
Rocky Linux 9 - Extras                                                                                                                                                               23 kB/s | 8.5 kB     00:00    
Dependencies resolved.
[logards@Tanjiro ~]$ sudo systemctl enable mariadb
Created symlink /etc/systemd/system/mysql.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/mysqld.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/multi-user.target.wants/mariadb.service → /usr/lib/systemd/system/mariadb.service.
[logards@Tanjiro ~]$ sudo systemctl start mariadb
[logards@Tanjiro ~]$ sudo mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] Y
Enabled successfully!
Reloading privilege tables..
 ... Success!


You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] Y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] Y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] Y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] 
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
```

🌞 **Port utilisé par MariaDB**

```
[logards@Tanjiro ~]$ sudo ss -alutnp | grep maria
tcp   LISTEN 0      80                 *:3306            *:*    users:(("mariadbd",pid=777,fd=19))
[logards@Tanjiro ~]$ sudo firewall-cmd --add-port 3306/tcp --permanent
success
[logards@Tanjiro ~]$ sudo firewall-cmd --reload
success
[logards@Tanjiro ~]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 3306/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```

🌞 **Processus liés à MariaDB**

```
[logards@Tanjiro ~]$ ps -ef | grep mariadb
mysql        777       1  0 11:42 ?        00:00:00 /usr/libexec/mariadbd --basedir=/usr
logards     1079     949  0 11:50 pts/0    00:00:00 grep --color=auto mariadb
```

🌞 **Préparation de la base pour NextCloud**

```
[logards@Tanjiro ~]$ sudo mysql -u root -p
[sudo] password for logards: 
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 16
Server version: 10.5.16-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE USER 'nextcloud'@'10.105.1.11' IDENTIFIED BY 'pewpewpew';
Query OK, 0 rows affected (0.007 sec)

MariaDB [(none)]> CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
Query OK, 1 row affected (0.001 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'10.105.1.11';
Query OK, 0 rows affected (0.003 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.001 sec)
```

🌞 **Exploration de la base de données**

- afin de tester le bon fonctionnement de la base de données, vous allez essayer de vous connecter, **comme NextCloud le fera plus tard** :
  - depuis la machine `web.tp5.linux` vers l'IP de `db.tp5.linux`
  - utilisez la commande `mysql` pour vous connecter à une base de données depuis la ligne de commande
    - par exemple `mysql -u <USER> -h <IP_DATABASE> -p`
    - si vous ne l'avez pas, installez-là
    - vous pouvez déterminer dans quel paquet est disponible la commande `mysql` en saisissant `dnf provides mysql`
- **donc vous devez effectuer une commande `mysql` sur `web.tp5.linux`**
- une fois connecté à la base, utilisez les commandes SQL fournies ci-dessous pour explorer la base
```
[logards@Nezuko ~]$ mysql -u nextcloud -h 10.105.1.12 -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 20
Server version: 5.5.5-10.5.16-MariaDB MariaDB Server

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.


mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| nextcloud          |
+--------------------+
2 rows in set (0.00 sec)

mysql> USE nextcloud
Database changed
```



🌞 **Trouver une commande SQL qui permet de lister tous les utilisateurs de la base de données**

```
MariaDB [(none)]> SELECT user FROM mysql.user;
+-------------+
| User        |
+-------------+
| nextcloud   |
| mariadb.sys |
| mysql       |
| root        |
+-------------+
4 rows in set (0.003 sec)
```

🌞 **Install de PHP**


```
[logards@Nezuko ~]$ sudo dnf config-manager --set-enabled crb
[logards@Nezuko ~]$ sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm -y
Rocky Linux 9 - BaseOS                                                                                                                                                              7.3 kB/s | 3.6 kB     00:00    
Rocky Linux 9 - AppStream                                                                                                                                                           9.9 kB/s | 4.1 kB     00:00    
Rocky Linux 9 - AppStream                                                                                                                                                           5.6 MB/s | 6.4 MB     00:01    
Rocky Linux 9 - CRB                                                                                                                                                                 2.5 MB/s | 2.0 MB     00:00    
remi-release-9.rpm                                                                                                                                                                  252 kB/s |  28 kB     00:00    
Dependencies resolved.
====================================================================================================================================================================================================================
 Package                                                      Architecture                               Version                                             Repository                                        Size
====================================================================================================================================================================================================================
Installing:
 remi-release                                                 noarch                                     9.1-2.el9.remi                                      @commandline                                      28 k
 yum-utils                                                    noarch                                     4.1.0-3.el9                                         baseos                                            36 k
Upgrading:
 dnf                                                          noarch                                     4.12.0-4.el9                                        baseos                                           451 k
 dnf-data                                                     noarch                                     4.12.0-4.el9                                        baseos                                            39 k
 dnf-plugins-core                                             noarch                                     4.1.0-3.el9                                         baseos                                            33 k
 libdnf                                                       x86_64                                     0.67.0-3.el9                                        baseos                                           654 k
 libsolv                                                      x86_64                                     0.7.22-1.el9                                        baseos                                           392 k
 python3-dnf                                                  noarch                                     4.12.0-4.el9                                        baseos                                           409 k
 python3-dnf-plugins-core                                     noarch                                     4.1.0-3.el9                                         baseos                                           221 k
 python3-hawkey                                               x86_64                                     0.67.0-3.el9                                        baseos                                           105 k
 python3-libdnf                                               x86_64                                     0.67.0-3.el9                                        baseos                                           774 k
 rocky-release                                                noarch                                     9.1-1.11.el9                                        baseos                                            22 k
 rocky-repos                                                  noarch                                     9.1-1.11.el9                                        baseos                                            12 k
 yum                                                          noarch                                     4.12.0-4.el9                                        baseos                                            91 k
Installing dependencies:
 epel-release                                                 noarch                                     9-4.el9                                             extras                                            19 k

Transaction Summary
====================================================================================================================================================================================================================
Install   3 Packages
Upgrade  12 Packages

Total size: 3.2 M
Total download size: 3.2 M
Downloading Packages:
(1/14): yum-utils-4.1.0-3.el9.noarch.rpm                                                                                                                                            164 kB/s |  36 kB     00:00    
(2/14): epel-release-9-4.el9.noarch.rpm                                                                                                                                              78 kB/s |  19 kB     00:00    
(3/14): dnf-data-4.12.0-4.el9.noarch.rpm                                                                                                                                            793 kB/s |  39 kB     00:00    
(4/14): yum-4.12.0-4.el9.noarch.rpm                                                                                                                                                 281 kB/s |  91 kB     00:00    
(5/14): python3-dnf-4.12.0-4.el9.noarch.rpm                                                                                                                                         3.4 MB/s | 409 kB     00:00    
(6/14): dnf-plugins-core-4.1.0-3.el9.noarch.rpm                                                                                                                                     868 kB/s |  33 kB     00:00    
(7/14): rocky-release-9.1-1.11.el9.noarch.rpm                                                                                                                                       453 kB/s |  22 kB     00:00    
(8/14): python3-dnf-plugins-core-4.1.0-3.el9.noarch.rpm                                                                                                                             2.0 MB/s | 221 kB     00:00    
(9/14): dnf-4.12.0-4.el9.noarch.rpm                                                                                                                                                 3.0 MB/s | 451 kB     00:00    
(10/14): rocky-repos-9.1-1.11.el9.noarch.rpm                                                                                                                                        369 kB/s |  12 kB     00:00    
(11/14): python3-hawkey-0.67.0-3.el9.x86_64.rpm                                                                                                                                     1.9 MB/s | 105 kB     00:00    
(12/14): libsolv-0.7.22-1.el9.x86_64.rpm                                                                                                                                            3.8 MB/s | 392 kB     00:00    
(13/14): python3-libdnf-0.67.0-3.el9.x86_64.rpm                                                                                                                                     7.0 MB/s | 774 kB     00:00    
(14/14): libdnf-0.67.0-3.el9.x86_64.rpm                                                                                                                                             8.8 MB/s | 654 kB     00:00    
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                               3.0 MB/s | 3.2 MB     00:01     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                                            1/1 
  Upgrading        : rocky-repos-9.1-1.11.el9.noarch                                                                                                                                                           1/27 
  Upgrading        : rocky-release-9.1-1.11.el9.noarch                                                                                                                                                         2/27 
  Upgrading        : libsolv-0.7.22-1.el9.x86_64                                                                                                                                                               3/27 
  Upgrading        : libdnf-0.67.0-3.el9.x86_64                                                                                                                                                                4/27 
  Upgrading        : python3-libdnf-0.67.0-3.el9.x86_64                                                                                                                                                        5/27 
  Upgrading        : python3-hawkey-0.67.0-3.el9.x86_64                                                                                                                                                        6/27 
  Upgrading        : dnf-data-4.12.0-4.el9.noarch                                                                                                                                                              7/27 
  Upgrading        : python3-dnf-4.12.0-4.el9.noarch                                                                                                                                                           8/27 
  Upgrading        : dnf-4.12.0-4.el9.noarch                                                                                                                                                                   9/27 
  Running scriptlet: dnf-4.12.0-4.el9.noarch                                                                                                                                                                   9/27 
  Upgrading        : python3-dnf-plugins-core-4.1.0-3.el9.noarch                                                                                                                                              10/27 
  Upgrading        : dnf-plugins-core-4.1.0-3.el9.noarch                                                                                                                                                      11/27 
  Installing       : epel-release-9-4.el9.noarch                                                                                                                                                              12/27 
  Running scriptlet: epel-release-9-4.el9.noarch                                                                                                                                                              12/27 
Many EPEL packages require the CodeReady Builder (CRB) repository.
It is recommended that you run /usr/bin/crb enable to enable the CRB repository.

  Installing       : remi-release-9.1-2.el9.remi.noarch                                                                                                                                                       13/27 
  Installing       : yum-utils-4.1.0-3.el9.noarch                                                                                                                                                             14/27 
  Upgrading        : yum-4.12.0-4.el9.noarch                                                                                                                                                                  15/27 
  Cleanup          : rocky-release-9.0-2.3.el9.noarch                                                                                                                                                         16/27 
  Cleanup          : dnf-plugins-core-4.0.24-4.el9_0.noarch                                                                                                                                                   17/27 
  Cleanup          : python3-dnf-plugins-core-4.0.24-4.el9_0.noarch                                                                                                                                           18/27 
  Cleanup          : yum-4.10.0-5.el9_0.noarch                                                                                                                                                                19/27 
  Running scriptlet: dnf-4.10.0-5.el9_0.noarch                                                                                                                                                                20/27 
  Cleanup          : dnf-4.10.0-5.el9_0.noarch                                                                                                                                                                20/27 
  Running scriptlet: dnf-4.10.0-5.el9_0.noarch                                                                                                                                                                20/27 
  Cleanup          : python3-dnf-4.10.0-5.el9_0.noarch                                                                                                                                                        21/27 
  Cleanup          : dnf-data-4.10.0-5.el9_0.noarch                                                                                                                                                           22/27 
  Cleanup          : rocky-repos-9.0-2.3.el9.noarch                                                                                                                                                           23/27 
  Cleanup          : python3-hawkey-0.65.0-5.1.el9.x86_64                                                                                                                                                     24/27 
  Cleanup          : python3-libdnf-0.65.0-5.1.el9.x86_64                                                                                                                                                     25/27 
  Cleanup          : libdnf-0.65.0-5.1.el9.x86_64                                                                                                                                                             26/27 
  Cleanup          : libsolv-0.7.20-2.el9.x86_64                                                                                                                                                              27/27 
  Running scriptlet: libsolv-0.7.20-2.el9.x86_64                                                                                                                                                              27/27 
  Verifying        : yum-utils-4.1.0-3.el9.noarch                                                                                                                                                              1/27 
  Verifying        : epel-release-9-4.el9.noarch                                                                                                                                                               2/27 
  Verifying        : remi-release-9.1-2.el9.remi.noarch                                                                                                                                                        3/27 
  Verifying        : yum-4.12.0-4.el9.noarch                                                                                                                                                                   4/27 
  Verifying        : yum-4.10.0-5.el9_0.noarch                                                                                                                                                                 5/27 
  Verifying        : python3-dnf-4.12.0-4.el9.noarch                                                                                                                                                           6/27 
  Verifying        : python3-dnf-4.10.0-5.el9_0.noarch                                                                                                                                                         7/27 
  Verifying        : dnf-data-4.12.0-4.el9.noarch                                                                                                                                                              8/27 
  Verifying        : dnf-data-4.10.0-5.el9_0.noarch                                                                                                                                                            9/27 
  Verifying        : dnf-4.12.0-4.el9.noarch                                                                                                                                                                  10/27 
  Verifying        : dnf-4.10.0-5.el9_0.noarch                                                                                                                                                                11/27 
  Verifying        : python3-dnf-plugins-core-4.1.0-3.el9.noarch                                                                                                                                              12/27 
  Verifying        : python3-dnf-plugins-core-4.0.24-4.el9_0.noarch                                                                                                                                           13/27 
  Verifying        : dnf-plugins-core-4.1.0-3.el9.noarch                                                                                                                                                      14/27 
  Verifying        : dnf-plugins-core-4.0.24-4.el9_0.noarch                                                                                                                                                   15/27 
  Verifying        : rocky-release-9.1-1.11.el9.noarch                                                                                                                                                        16/27 
  Verifying        : rocky-release-9.0-2.3.el9.noarch                                                                                                                                                         17/27 
  Verifying        : rocky-repos-9.1-1.11.el9.noarch                                                                                                                                                          18/27 
  Verifying        : rocky-repos-9.0-2.3.el9.noarch                                                                                                                                                           19/27 
  Verifying        : libsolv-0.7.22-1.el9.x86_64                                                                                                                                                              20/27 
  Verifying        : libsolv-0.7.20-2.el9.x86_64                                                                                                                                                              21/27 
  Verifying        : python3-libdnf-0.67.0-3.el9.x86_64                                                                                                                                                       22/27 
  Verifying        : python3-libdnf-0.65.0-5.1.el9.x86_64                                                                                                                                                     23/27 
  Verifying        : python3-hawkey-0.67.0-3.el9.x86_64                                                                                                                                                       24/27 
  Verifying        : python3-hawkey-0.65.0-5.1.el9.x86_64                                                                                                                                                     25/27 
  Verifying        : libdnf-0.67.0-3.el9.x86_64                                                                                                                                                               26/27 
  Verifying        : libdnf-0.65.0-5.1.el9.x86_64                                                                                                                                                             27/27 

Upgraded:
  dnf-4.12.0-4.el9.noarch               dnf-data-4.12.0-4.el9.noarch                      dnf-plugins-core-4.1.0-3.el9.noarch       libdnf-0.67.0-3.el9.x86_64               libsolv-0.7.22-1.el9.x86_64            
  python3-dnf-4.12.0-4.el9.noarch       python3-dnf-plugins-core-4.1.0-3.el9.noarch       python3-hawkey-0.67.0-3.el9.x86_64        python3-libdnf-0.67.0-3.el9.x86_64       rocky-release-9.1-1.11.el9.noarch      
  rocky-repos-9.1-1.11.el9.noarch       yum-4.12.0-4.el9.noarch                          
Installed:
  epel-release-9-4.el9.noarch                                        remi-release-9.1-2.el9.remi.noarch                                        yum-utils-4.1.0-3.el9.noarch                                       

Complete!
[logards@Nezuko ~]$ dnf module list php
Last metadata expiration check: 0:00:08 ago on Tue 17 Jan 2023 12:36:37 PM CET.
Rocky Linux 9 - AppStream
Name                                       Stream                                        Profiles                                                        Summary                                                    
php                                        8.1                                           common [d], devel, minimal                                      PHP scripting language                                     

Remi's Modular repository for Enterprise Linux 9 - x86_64
Name                                       Stream                                        Profiles                                                        Summary                                                    
php                                        remi-7.4                                      common [d], devel, minimal                                      PHP scripting language                                     
php                                        remi-8.0                                      common [d], devel, minimal                                      PHP scripting language                                     
php                                        remi-8.1                                      common [d], devel, minimal                                      PHP scripting language                                     
php                                        remi-8.2                                      common [d], devel, minimal                                      PHP scripting language                                     

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
[logards@Nezuko ~]$  sudo dnf module enable php:remi-8.1 -y
Extra Packages for Enterprise Linux 9 - x86_64                                                                                                                                      6.1 MB/s |  13 MB     00:02    
Remi's Modular repository for Enterprise Linux 9 - x86_64                                                                                                                           3.8 kB/s | 833  B     00:00    
Remi's Modular repository for Enterprise Linux 9 - x86_64                                                                                                                           3.0 MB/s | 3.1 kB     00:00    
Importing GPG key 0x478F8947:
 Userid     : "Remi's RPM repository (https://rpms.remirepo.net/) <remi@remirepo.net>"
 Fingerprint: B1AB F71E 14C9 D748 97E1 98A8 B195 27F1 478F 8947
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-remi.el9
Remi's Modular repository for Enterprise Linux 9 - x86_64                                                                                                                           2.4 MB/s | 835 kB     00:00    
Safe Remi's RPM repository for Enterprise Linux 9 - x86_64                                                                                                                          4.6 kB/s | 833  B     00:00    
Safe Remi's RPM repository for Enterprise Linux 9 - x86_64                                                                                                                          3.0 MB/s | 3.1 kB     00:00    
Importing GPG key 0x478F8947:
 Userid     : "Remi's RPM repository (https://rpms.remirepo.net/) <remi@remirepo.net>"
 Fingerprint: B1AB F71E 14C9 D748 97E1 98A8 B195 27F1 478F 8947
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-remi.el9
Safe Remi's RPM repository for Enterprise Linux 9 - x86_64                                                                                                                          2.6 MB/s | 886 kB     00:00    
Dependencies resolved.
====================================================================================================================================================================================================================
 Package                                            Architecture                                      Version                                              Repository                                          Size
====================================================================================================================================================================================================================
Enabling module streams:
 php                                                                                                  remi-8.1                                                                                                     

Transaction Summary
====================================================================================================================================================================================================================

Complete!
[logards@Nezuko ~]$ sudo dnf install -y php81-php
Last metadata expiration check: 0:00:31 ago on Tue 17 Jan 2023 12:37:26 PM CET.
Dependencies resolved.
====================================================================================================================================================================================================================
 Package                                                         Architecture                              Version                                               Repository                                    Size
====================================================================================================================================================================================================================
Installing:
 php81-php                                                       x86_64                                    8.1.14-1.el9.remi                                     remi-safe                                    1.7 M
Upgrading:
 audit                                                           x86_64                                    3.0.7-103.el9                                         baseos                                       252 k
 audit-libs                                                      x86_64                                    3.0.7-103.el9                                         baseos                                       116 k
 libselinux                                                      x86_64                                    3.4-3.el9                                             baseos                                        85 k
 libselinux-utils                                                x86_64                                    3.4-3.el9                                             baseos                                       158 k
 libsemanage                                                     x86_64                                    3.4-2.el9                                             baseos                                       118 k
 libsepol                                                        x86_64                                    3.4-1.1.el9                                           baseos                                       315 k
 policycoreutils                                                 x86_64                                    3.4-4.el9                                             baseos                                       202 k
 python3-libselinux                                              x86_64                                    3.4-3.el9                                             appstream                                    185 k
Installing dependencies:
 checkpolicy                                                     x86_64                                    3.4-1.el9                                             appstream                                    346 k
 environment-modules                                             x86_64                                    5.0.1-1.el9                                           baseos                                       481 k
 libsodium                                                       x86_64                                    1.0.18-8.el9                                          epel                                         161 k
 libxslt                                                         x86_64                                    1.1.34-9.el9                                          appstream                                    240 k
 oniguruma5php                                                   x86_64                                    6.9.8-1.el9.remi                                      remi-safe                                    219 k
 php81-php-common                                                x86_64                                    8.1.14-1.el9.remi                                     remi-safe                                    667 k
 php81-runtime                                                   x86_64                                    8.1-2.el9.remi                                        remi-safe                                    1.1 M
 policycoreutils-python-utils                                    noarch                                    3.4-4.el9                                             appstream                                     69 k
 python3-audit                                                   x86_64                                    3.0.7-103.el9                                         appstream                                     83 k
 python3-libsemanage                                             x86_64                                    3.4-2.el9                                             appstream                                     80 k
 python3-policycoreutils                                         noarch                                    3.4-4.el9                                             appstream                                    2.0 M
 python3-setools                                                 x86_64                                    4.4.0-5.el9                                           baseos                                       546 k
 python3-setuptools                                              noarch                                    53.0.0-10.el9                                         baseos                                       841 k
 scl-utils                                                       x86_64                                    1:2.0.3-2.el9                                         appstream                                     37 k
 tcl                                                             x86_64                                    1:8.6.10-7.el9                                        baseos                                       1.1 M
Installing weak dependencies:
 php81-php-cli                                                   x86_64                                    8.1.14-1.el9.remi                                     remi-safe                                    3.5 M
 php81-php-fpm                                                   x86_64                                    8.1.14-1.el9.remi                                     remi-safe                                    1.8 M
 php81-php-mbstring                                              x86_64                                    8.1.14-1.el9.remi                                     remi-safe                                    475 k
 php81-php-opcache                                               x86_64                                    8.1.14-1.el9.remi                                     remi-safe                                    378 k
 php81-php-pdo                                                   x86_64                                    8.1.14-1.el9.remi                                     remi-safe                                     86 k
 php81-php-sodium                                                x86_64                                    8.1.14-1.el9.remi                                     remi-safe                                     41 k
 php81-php-xml                                                   x86_64                                    8.1.14-1.el9.remi                                     remi-safe                                    141 k

Transaction Summary
====================================================================================================================================================================================================================
Install  23 Packages
Upgrade   8 Packages

Total download size: 17 M
Downloading Packages:
(1/31): libsodium-1.0.18-8.el9.x86_64.rpm                                                                                                                                           1.0 MB/s | 161 kB     00:00    
(2/31): oniguruma5php-6.9.8-1.el9.remi.x86_64.rpm                                                                                                                                   1.3 MB/s | 219 kB     00:00    
(3/31): php81-php-common-8.1.14-1.el9.remi.x86_64.rpm                                                                                                                               5.0 MB/s | 667 kB     00:00    
(4/31): php81-php-8.1.14-1.el9.remi.x86_64.rpm                                                                                                                                      5.0 MB/s | 1.7 MB     00:00    
(5/31): php81-php-mbstring-8.1.14-1.el9.remi.x86_64.rpm                                                                                                                             5.1 MB/s | 475 kB     00:00    
(6/31): php81-php-opcache-8.1.14-1.el9.remi.x86_64.rpm                                                                                                                              4.8 MB/s | 378 kB     00:00    
(7/31): php81-php-fpm-8.1.14-1.el9.remi.x86_64.rpm                                                                                                                                  6.9 MB/s | 1.8 MB     00:00    
(8/31): php81-php-pdo-8.1.14-1.el9.remi.x86_64.rpm                                                                                                                                  1.7 MB/s |  86 kB     00:00    
(9/31): php81-php-sodium-8.1.14-1.el9.remi.x86_64.rpm                                                                                                                               1.2 MB/s |  41 kB     00:00    
(10/31): php81-php-xml-8.1.14-1.el9.remi.x86_64.rpm                                                                                                                                 3.3 MB/s | 141 kB     00:00    
(11/31): php81-php-cli-8.1.14-1.el9.remi.x86_64.rpm                                                                                                                                 5.8 MB/s | 3.5 MB     00:00    
(12/31): php81-runtime-8.1-2.el9.remi.x86_64.rpm                                                                                                                                    6.8 MB/s | 1.1 MB     00:00    
(13/31): python3-setuptools-53.0.0-10.el9.noarch.rpm                                                                                                                                2.7 MB/s | 841 kB     00:00    
(14/31): tcl-8.6.10-7.el9.x86_64.rpm                                                                                                                                                6.3 MB/s | 1.1 MB     00:00    
(15/31): environment-modules-5.0.1-1.el9.x86_64.rpm                                                                                                                                 1.3 MB/s | 481 kB     00:00    
(16/31): python3-setools-4.4.0-5.el9.x86_64.rpm                                                                                                                                     1.5 MB/s | 546 kB     00:00    
(17/31): python3-audit-3.0.7-103.el9.x86_64.rpm                                                                                                                                     733 kB/s |  83 kB     00:00    
(18/31): policycoreutils-python-utils-3.4-4.el9.noarch.rpm                                                                                                                          1.8 MB/s |  69 kB     00:00    
(19/31): checkpolicy-3.4-1.el9.x86_64.rpm                                                                                                                                           1.7 MB/s | 346 kB     00:00    
(20/31): python3-libsemanage-3.4-2.el9.x86_64.rpm                                                                                                                                   2.1 MB/s |  80 kB     00:00    
(21/31): scl-utils-2.0.3-2.el9.x86_64.rpm                                                                                                                                           608 kB/s |  37 kB     00:00    
(22/31): libxslt-1.1.34-9.el9.x86_64.rpm                                                                                                                                            1.7 MB/s | 240 kB     00:00    
(23/31): audit-libs-3.0.7-103.el9.x86_64.rpm                                                                                                                                        2.9 MB/s | 116 kB     00:00    
(24/31): policycoreutils-3.4-4.el9.x86_64.rpm                                                                                                                                       3.9 MB/s | 202 kB     00:00    
(25/31): audit-3.0.7-103.el9.x86_64.rpm                                                                                                                                             3.5 MB/s | 252 kB     00:00    
(26/31): libsepol-3.4-1.1.el9.x86_64.rpm                                                                                                                                            5.2 MB/s | 315 kB     00:00    
(27/31): libsemanage-3.4-2.el9.x86_64.rpm                                                                                                                                           2.0 MB/s | 118 kB     00:00    
(28/31): libselinux-utils-3.4-3.el9.x86_64.rpm                                                                                                                                      3.6 MB/s | 158 kB     00:00    
(29/31): libselinux-3.4-3.el9.x86_64.rpm                                                                                                                                            2.0 MB/s |  85 kB     00:00    
(30/31): python3-libselinux-3.4-3.el9.x86_64.rpm                                                                                                                                    1.7 MB/s | 185 kB     00:00    
(31/31): python3-policycoreutils-3.4-4.el9.noarch.rpm                                                                                                                               3.1 MB/s | 2.0 MB     00:00    
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                               5.7 MB/s |  17 MB     00:03     
Extra Packages for Enterprise Linux 9 - x86_64                                                                                                                                      1.6 MB/s | 1.6 kB     00:00    
Importing GPG key 0x3228467C:
 Userid     : "Fedora (epel9) <epel@fedoraproject.org>"
 Fingerprint: FF8A D134 4597 106E CE81 3B91 8A38 72BF 3228 467C
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-9
Key imported successfully
Safe Remi's RPM repository for Enterprise Linux 9 - x86_64                                                                                                                          3.0 MB/s | 3.1 kB     00:00    
Importing GPG key 0x478F8947:
 Userid     : "Remi's RPM repository (https://rpms.remirepo.net/) <remi@remirepo.net>"
 Fingerprint: B1AB F71E 14C9 D748 97E1 98A8 B195 27F1 478F 8947
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-remi.el9
Key imported successfully
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                                            1/1 
  Upgrading        : libsepol-3.4-1.1.el9.x86_64                                                                                                                                                               1/39 
  Upgrading        : libselinux-3.4-3.el9.x86_64                                                                                                                                                               2/39 
  Running scriptlet: libselinux-3.4-3.el9.x86_64                                                                                                                                                               2/39 
  Upgrading        : audit-libs-3.0.7-103.el9.x86_64                                                                                                                                                           3/39 
  Upgrading        : libsemanage-3.4-2.el9.x86_64                                                                                                                                                              4/39 
  Upgrading        : libselinux-utils-3.4-3.el9.x86_64                                                                                                                                                         5/39 
  Upgrading        : python3-libselinux-3.4-3.el9.x86_64                                                                                                                                                       6/39 
  Installing       : python3-libsemanage-3.4-2.el9.x86_64                                                                                                                                                      7/39 
  Upgrading        : policycoreutils-3.4-4.el9.x86_64                                                                                                                                                          8/39 
  Running scriptlet: policycoreutils-3.4-4.el9.x86_64                                                                                                                                                          8/39 
  Installing       : python3-audit-3.0.7-103.el9.x86_64                                                                                                                                                        9/39 
  Installing       : libxslt-1.1.34-9.el9.x86_64                                                                                                                                                              10/39 
  Installing       : checkpolicy-3.4-1.el9.x86_64                                                                                                                                                             11/39 
  Installing       : tcl-1:8.6.10-7.el9.x86_64                                                                                                                                                                12/39 
  Installing       : environment-modules-5.0.1-1.el9.x86_64                                                                                                                                                   13/39 
  Running scriptlet: environment-modules-5.0.1-1.el9.x86_64                                                                                                                                                   13/39 
  Installing       : scl-utils-1:2.0.3-2.el9.x86_64                                                                                                                                                           14/39 
  Installing       : python3-setuptools-53.0.0-10.el9.noarch                                                                                                                                                  15/39 
  Installing       : python3-setools-4.4.0-5.el9.x86_64                                                                                                                                                       16/39 
  Installing       : python3-policycoreutils-3.4-4.el9.noarch                                                                                                                                                 17/39 
  Installing       : policycoreutils-python-utils-3.4-4.el9.noarch                                                                                                                                            18/39 
  Installing       : php81-runtime-8.1-2.el9.remi.x86_64                                                                                                                                                      19/39 
  Running scriptlet: php81-runtime-8.1-2.el9.remi.x86_64                                                                                                                                                      19/39 
  Installing       : php81-php-common-8.1.14-1.el9.remi.x86_64                                                                                                                                                20/39 
  Installing       : php81-php-cli-8.1.14-1.el9.remi.x86_64                                                                                                                                                   21/39 
  Installing       : php81-php-fpm-8.1.14-1.el9.remi.x86_64                                                                                                                                                   22/39 
  Running scriptlet: php81-php-fpm-8.1.14-1.el9.remi.x86_64                                                                                                                                                   22/39 
  Installing       : php81-php-opcache-8.1.14-1.el9.remi.x86_64                                                                                                                                               23/39 
  Installing       : php81-php-pdo-8.1.14-1.el9.remi.x86_64                                                                                                                                                   24/39 
  Installing       : php81-php-xml-8.1.14-1.el9.remi.x86_64                                                                                                                                                   25/39 
  Installing       : oniguruma5php-6.9.8-1.el9.remi.x86_64                                                                                                                                                    26/39 
  Installing       : php81-php-mbstring-8.1.14-1.el9.remi.x86_64                                                                                                                                              27/39 
  Installing       : libsodium-1.0.18-8.el9.x86_64                                                                                                                                                            28/39 
  Installing       : php81-php-sodium-8.1.14-1.el9.remi.x86_64                                                                                                                                                29/39 
  Installing       : php81-php-8.1.14-1.el9.remi.x86_64                                                                                                                                                       30/39 
  Upgrading        : audit-3.0.7-103.el9.x86_64                                                                                                                                                               31/39 
  Running scriptlet: audit-3.0.7-103.el9.x86_64                                                                                                                                                               31/39 
  Running scriptlet: policycoreutils-3.3-6.el9_0.x86_64                                                                                                                                                       32/39 
  Cleanup          : policycoreutils-3.3-6.el9_0.x86_64                                                                                                                                                       32/39 
  Cleanup          : libsemanage-3.3-2.el9.x86_64                                                                                                                                                             33/39 
  Cleanup          : libselinux-utils-3.3-2.el9.x86_64                                                                                                                                                        34/39 
  Cleanup          : python3-libselinux-3.3-2.el9.x86_64                                                                                                                                                      35/39 
  Running scriptlet: audit-3.0.7-101.el9_0.2.x86_64                                                                                                                                                           36/39 
  Cleanup          : audit-3.0.7-101.el9_0.2.x86_64                                                                                                                                                           36/39 
  Running scriptlet: audit-3.0.7-101.el9_0.2.x86_64                                                                                                                                                           36/39 
  Cleanup          : libselinux-3.3-2.el9.x86_64                                                                                                                                                              37/39 
  Cleanup          : libsepol-3.3-2.el9.x86_64                                                                                                                                                                38/39 
  Cleanup          : audit-libs-3.0.7-101.el9_0.2.x86_64                                                                                                                                                      39/39 
  Running scriptlet: audit-libs-3.0.7-101.el9_0.2.x86_64                                                                                                                                                      39/39 
  Verifying        : libsodium-1.0.18-8.el9.x86_64                                                                                                                                                             1/39 
  Verifying        : oniguruma5php-6.9.8-1.el9.remi.x86_64                                                                                                                                                     2/39 
  Verifying        : php81-php-8.1.14-1.el9.remi.x86_64                                                                                                                                                        3/39 
  Verifying        : php81-php-cli-8.1.14-1.el9.remi.x86_64                                                                                                                                                    4/39 
  Verifying        : php81-php-common-8.1.14-1.el9.remi.x86_64                                                                                                                                                 5/39 
  Verifying        : php81-php-fpm-8.1.14-1.el9.remi.x86_64                                                                                                                                                    6/39 
  Verifying        : php81-php-mbstring-8.1.14-1.el9.remi.x86_64                                                                                                                                               7/39 
  Verifying        : php81-php-opcache-8.1.14-1.el9.remi.x86_64                                                                                                                                                8/39 
  Verifying        : php81-php-pdo-8.1.14-1.el9.remi.x86_64                                                                                                                                                    9/39 
  Verifying        : php81-php-sodium-8.1.14-1.el9.remi.x86_64                                                                                                                                                10/39 
  Verifying        : php81-php-xml-8.1.14-1.el9.remi.x86_64                                                                                                                                                   11/39 
  Verifying        : php81-runtime-8.1-2.el9.remi.x86_64                                                                                                                                                      12/39 
  Verifying        : python3-setuptools-53.0.0-10.el9.noarch                                                                                                                                                  13/39 
  Verifying        : environment-modules-5.0.1-1.el9.x86_64                                                                                                                                                   14/39 
  Verifying        : python3-setools-4.4.0-5.el9.x86_64                                                                                                                                                       15/39 
  Verifying        : tcl-1:8.6.10-7.el9.x86_64                                                                                                                                                                16/39 
  Verifying        : checkpolicy-3.4-1.el9.x86_64                                                                                                                                                             17/39 
  Verifying        : python3-audit-3.0.7-103.el9.x86_64                                                                                                                                                       18/39 
  Verifying        : python3-policycoreutils-3.4-4.el9.noarch                                                                                                                                                 19/39 
  Verifying        : policycoreutils-python-utils-3.4-4.el9.noarch                                                                                                                                            20/39 
  Verifying        : libxslt-1.1.34-9.el9.x86_64                                                                                                                                                              21/39 
  Verifying        : python3-libsemanage-3.4-2.el9.x86_64                                                                                                                                                     22/39 
  Verifying        : scl-utils-1:2.0.3-2.el9.x86_64                                                                                                                                                           23/39 
  Verifying        : audit-libs-3.0.7-103.el9.x86_64                                                                                                                                                          24/39 
  Verifying        : audit-libs-3.0.7-101.el9_0.2.x86_64                                                                                                                                                      25/39 
  Verifying        : audit-3.0.7-103.el9.x86_64                                                                                                                                                               26/39 
  Verifying        : audit-3.0.7-101.el9_0.2.x86_64                                                                                                                                                           27/39 
  Verifying        : policycoreutils-3.4-4.el9.x86_64                                                                                                                                                         28/39 
  Verifying        : policycoreutils-3.3-6.el9_0.x86_64                                                                                                                                                       29/39 
  Verifying        : libsepol-3.4-1.1.el9.x86_64                                                                                                                                                              30/39 
  Verifying        : libsepol-3.3-2.el9.x86_64                                                                                                                                                                31/39 
  Verifying        : libsemanage-3.4-2.el9.x86_64                                                                                                                                                             32/39 
  Verifying        : libsemanage-3.3-2.el9.x86_64                                                                                                                                                             33/39 
  Verifying        : libselinux-utils-3.4-3.el9.x86_64                                                                                                                                                        34/39 
  Verifying        : libselinux-utils-3.3-2.el9.x86_64                                                                                                                                                        35/39 
  Verifying        : libselinux-3.4-3.el9.x86_64                                                                                                                                                              36/39 
  Verifying        : libselinux-3.3-2.el9.x86_64                                                                                                                                                              37/39 
  Verifying        : python3-libselinux-3.4-3.el9.x86_64                                                                                                                                                      38/39 
  Verifying        : python3-libselinux-3.3-2.el9.x86_64                                                                                                                                                      39/39 

Upgraded:
  audit-3.0.7-103.el9.x86_64          audit-libs-3.0.7-103.el9.x86_64        libselinux-3.4-3.el9.x86_64    libselinux-utils-3.4-3.el9.x86_64    libsemanage-3.4-2.el9.x86_64    libsepol-3.4-1.1.el9.x86_64   
  policycoreutils-3.4-4.el9.x86_64    python3-libselinux-3.4-3.el9.x86_64   
Installed:
  checkpolicy-3.4-1.el9.x86_64                       environment-modules-5.0.1-1.el9.x86_64               libsodium-1.0.18-8.el9.x86_64                       libxslt-1.1.34-9.el9.x86_64                           
  oniguruma5php-6.9.8-1.el9.remi.x86_64              php81-php-8.1.14-1.el9.remi.x86_64                   php81-php-cli-8.1.14-1.el9.remi.x86_64              php81-php-common-8.1.14-1.el9.remi.x86_64             
  php81-php-fpm-8.1.14-1.el9.remi.x86_64             php81-php-mbstring-8.1.14-1.el9.remi.x86_64          php81-php-opcache-8.1.14-1.el9.remi.x86_64          php81-php-pdo-8.1.14-1.el9.remi.x86_64                
  php81-php-sodium-8.1.14-1.el9.remi.x86_64          php81-php-xml-8.1.14-1.el9.remi.x86_64               php81-runtime-8.1-2.el9.remi.x86_64                 policycoreutils-python-utils-3.4-4.el9.noarch         
  python3-audit-3.0.7-103.el9.x86_64                 python3-libsemanage-3.4-2.el9.x86_64                 python3-policycoreutils-3.4-4.el9.noarch            python3-setools-4.4.0-5.el9.x86_64                    
  python3-setuptools-53.0.0-10.el9.noarch            scl-utils-1:2.0.3-2.el9.x86_64                       tcl-1:8.6.10-7.el9.x86_64                          

Complete!

```

🌞 **Install de tous les modules PHP nécessaires pour NextCloud**

```bash
[logards@Nezuko ~]$ sudo dnf install -y libxml2 openssl php81-php php81-php-ctype php81-php-curl php81-php-gd php81-php-iconv php81-php-json php81-php-libxml php81-php-mbstring php81-php-openssl php81-php-posix php81-php-session php81-php-xml php81-php-zip php81-php-zlib php81-php-pdo php81-php-mysqlnd php81-php-intl php81-php-bcmath php81-php-gmp
Last metadata expiration check: 0:02:30 ago on Tue 17 Jan 2023 12:37:26 PM CET.
Package libxml2-2.9.13-1.el9_0.1.x86_64 is already installed.
Package openssl-1:3.0.1-41.el9_0.x86_64 is already installed.
Package php81-php-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-mbstring-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-xml-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-common-8.1.14-1.el9.remi.x86_64 is already installed.
Package php81-php-pdo-8.1.14-1.el9.remi.x86_64 is already installed.
Dependencies resolved.
====================================================================================================================================================================================================================
 Package                                                 Architecture                                Version                                                   Repository                                      Size
====================================================================================================================================================================================================================
Installing:
 php81-php-bcmath                                        x86_64                                      8.1.14-1.el9.remi                                         remi-safe                                       38 k
 php81-php-gd                                            x86_64                                      8.1.14-1.el9.remi                                         remi-safe                                       45 k
 php81-php-gmp                                           x86_64                                      8.1.14-1.el9.remi                                         remi-safe                                       35 k
 php81-php-intl                                          x86_64                                      8.1.14-1.el9.remi                                         remi-safe                                      155 k
 php81-php-mysqlnd                                       x86_64                                      8.1.14-1.el9.remi                                         remi-safe                                      147 k
 php81-php-pecl-zip                                      x86_64                                      1.21.1-1.el9.remi                                         remi-safe                                       58 k
 php81-php-process                                       x86_64                                      8.1.14-1.el9.remi                                         remi-safe                                       44 k
Upgrading:
 libxml2                                                 x86_64                                      2.9.13-2.el9                                              baseos                                         747 k
 openssl                                                 x86_64                                      1:3.0.1-43.el9_0                                          baseos                                         1.1 M
 openssl-libs                                            x86_64                                      1:3.0.1-43.el9_0                                          baseos                                         2.1 M
Installing dependencies:
 fontconfig                                              x86_64                                      2.14.0-1.el9                                              appstream                                      275 k
 fribidi                                                 x86_64                                      1.0.10-6.el9.2                                            appstream                                       84 k
 gd3php                                                  x86_64                                      2.3.3-9.el9.remi                                          remi-safe                                      136 k
 gdk-pixbuf2                                             x86_64                                      2.42.6-2.el9                                              appstream                                      466 k
 highway                                                 x86_64                                      1.0.2-1.el9                                               epel                                           361 k
 jbigkit-libs                                            x86_64                                      2.1-23.el9                                                appstream                                       52 k
 libX11                                                  x86_64                                      1.7.0-7.el9                                               appstream                                      652 k
 libX11-common                                           noarch                                      1.7.0-7.el9                                               appstream                                      152 k
 libXau                                                  x86_64                                      1.0.9-8.el9                                               appstream                                       30 k
 libXpm                                                  x86_64                                      3.5.13-7.el9                                              appstream                                       58 k
 libaom                                                  x86_64                                      3.5.0-2.el9                                               epel                                           1.7 M
 libavif                                                 x86_64                                      0.11.1-4.el9                                              epel                                            81 k
 libdav1d                                                x86_64                                      1.0.0-2.el9                                               epel                                           562 k
 libicu71                                                x86_64                                      71.1-2.el9.remi                                           remi-safe                                       10 M
 libimagequant                                           x86_64                                      2.17.0-1.el9                                              epel                                            62 k
 libjpeg-turbo                                           x86_64                                      2.0.90-5.el9                                              appstream                                      175 k
 libjxl                                                  x86_64                                      0.7.0-1.el9                                               epel                                           957 k
 libraqm                                                 x86_64                                      0.8.0-1.el9                                               epel                                            19 k
 libtiff                                                 x86_64                                      4.4.0-2.el9                                               appstream                                      195 k
 libvmaf                                                 x86_64                                      2.3.0-2.el9                                               epel                                           177 k
 libwebp                                                 x86_64                                      1.2.0-3.el9                                               appstream                                      276 k
 libxcb                                                  x86_64                                      1.13.1-9.el9                                              appstream                                      224 k
 rav1e-libs                                              x86_64                                      0.5.1-5.el9                                               epel                                           913 k
 remi-libzip                                             x86_64                                      1.9.2-3.el9.remi                                          remi-safe                                       66 k
 shared-mime-info                                        x86_64                                      2.1-4.el9                                                 baseos                                         373 k
 svt-av1-libs                                            x86_64                                      0.9.0-1.el9                                               epel                                           1.7 M
 xml-common                                              noarch                                      0.6.3-58.el9                                              appstream                                       31 k
Installing weak dependencies:
 jxl-pixbuf-loader                                       x86_64                                      0.7.0-1.el9                                               epel                                            53 k

Transaction Summary
====================================================================================================================================================================================================================
Install  35 Packages
Upgrade   3 Packages

Total download size: 24 M
Downloading Packages:
(1/38): jxl-pixbuf-loader-0.7.0-1.el9.x86_64.rpm                                                                                                                                    327 kB/s |  53 kB     00:00    
(2/38): highway-1.0.2-1.el9.x86_64.rpm                                                                                                                                              1.3 MB/s | 361 kB     00:00    
(3/38): libavif-0.11.1-4.el9.x86_64.rpm                                                                                                                                             750 kB/s |  81 kB     00:00    
(4/38): libimagequant-2.17.0-1.el9.x86_64.rpm                                                                                                                                       850 kB/s |  62 kB     00:00    
(5/38): libdav1d-1.0.0-2.el9.x86_64.rpm                                                                                                                                             2.9 MB/s | 562 kB     00:00    
(6/38): libraqm-0.8.0-1.el9.x86_64.rpm                                                                                                                                              353 kB/s |  19 kB     00:00    
(7/38): libaom-3.5.0-2.el9.x86_64.rpm                                                                                                                                               3.1 MB/s | 1.7 MB     00:00    
(8/38): libvmaf-2.3.0-2.el9.x86_64.rpm                                                                                                                                              3.2 MB/s | 177 kB     00:00    
(9/38): libjxl-0.7.0-1.el9.x86_64.rpm                                                                                                                                               3.2 MB/s | 957 kB     00:00    
(10/38): rav1e-libs-0.5.1-5.el9.x86_64.rpm                                                                                                                                          3.7 MB/s | 913 kB     00:00    
(11/38): gd3php-2.3.3-9.el9.remi.x86_64.rpm                                                                                                                                         596 kB/s | 136 kB     00:00    
(12/38): php81-php-bcmath-8.1.14-1.el9.remi.x86_64.rpm                                                                                                                              1.0 MB/s |  38 kB     00:00    
(13/38): svt-av1-libs-0.9.0-1.el9.x86_64.rpm                                                                                                                                        4.6 MB/s | 1.7 MB     00:00    
(14/38): php81-php-gd-8.1.14-1.el9.remi.x86_64.rpm                                                                                                                                  1.0 MB/s |  45 kB     00:00    
(15/38): php81-php-intl-8.1.14-1.el9.remi.x86_64.rpm                                                                                                                                2.1 MB/s | 155 kB     00:00    
(16/38): php81-php-mysqlnd-8.1.14-1.el9.remi.x86_64.rpm                                                                                                                             2.3 MB/s | 147 kB     00:00    
(17/38): php81-php-pecl-zip-1.21.1-1.el9.remi.x86_64.rpm                                                                                                                            1.2 MB/s |  58 kB     00:00    
(18/38): php81-php-process-8.1.14-1.el9.remi.x86_64.rpm                                                                                                                             1.2 MB/s |  44 kB     00:00    
(19/38): remi-libzip-1.9.2-3.el9.remi.x86_64.rpm                                                                                                                                    1.6 MB/s |  66 kB     00:00    
(20/38): php81-php-gmp-8.1.14-1.el9.remi.x86_64.rpm                                                                                                                                  56 kB/s |  35 kB     00:00    
(21/38): libicu71-71.1-2.el9.remi.x86_64.rpm                                                                                                                                         12 MB/s |  10 MB     00:00    
(22/38): fribidi-1.0.10-6.el9.2.x86_64.rpm                                                                                                                                          277 kB/s |  84 kB     00:00    
(23/38): libXpm-3.5.13-7.el9.x86_64.rpm                                                                                                                                             137 kB/s |  58 kB     00:00    
(24/38): shared-mime-info-2.1-4.el9.x86_64.rpm                                                                                                                                      295 kB/s | 373 kB     00:01    
(25/38): fontconfig-2.14.0-1.el9.x86_64.rpm                                                                                                                                         329 kB/s | 275 kB     00:00    
(26/38): libX11-common-1.7.0-7.el9.noarch.rpm                                                                                                                                       637 kB/s | 152 kB     00:00    
(27/38): libXau-1.0.9-8.el9.x86_64.rpm                                                                                                                                              201 kB/s |  30 kB     00:00    
(28/38): libjpeg-turbo-2.0.90-5.el9.x86_64.rpm                                                                                                                                      859 kB/s | 175 kB     00:00    
(29/38): jbigkit-libs-2.1-23.el9.x86_64.rpm                                                                                                                                         376 kB/s |  52 kB     00:00    
(30/38): libxcb-1.13.1-9.el9.x86_64.rpm                                                                                                                                             821 kB/s | 224 kB     00:00    
(31/38): xml-common-0.6.3-58.el9.noarch.rpm                                                                                                                                         662 kB/s |  31 kB     00:00    
(32/38): libtiff-4.4.0-2.el9.x86_64.rpm                                                                                                                                             1.1 MB/s | 195 kB     00:00    
(33/38): libwebp-1.2.0-3.el9.x86_64.rpm                                                                                                                                             672 kB/s | 276 kB     00:00    
(34/38): gdk-pixbuf2-2.42.6-2.el9.x86_64.rpm                                                                                                                                        1.5 MB/s | 466 kB     00:00    
(35/38): libX11-1.7.0-7.el9.x86_64.rpm                                                                                                                                              1.4 MB/s | 652 kB     00:00    
(36/38): openssl-libs-3.0.1-43.el9_0.x86_64.rpm                                                                                                                                     4.3 MB/s | 2.1 MB     00:00    
(37/38): libxml2-2.9.13-2.el9.x86_64.rpm                                                                                                                                            1.1 MB/s | 747 kB     00:00    
(38/38): openssl-3.0.1-43.el9_0.x86_64.rpm                                                                                                                                          2.1 MB/s | 1.1 MB     00:00    
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                                               4.7 MB/s |  24 MB     00:05     
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                                                                                                            1/1 
  Upgrading        : libxml2-2.9.13-2.el9.x86_64                                                                                                                                                               1/41 
  Installing       : libjpeg-turbo-2.0.90-5.el9.x86_64                                                                                                                                                         2/41 
  Installing       : shared-mime-info-2.1-4.el9.x86_64                                                                                                                                                         3/41 
  Running scriptlet: shared-mime-info-2.1-4.el9.x86_64                                                                                                                                                         3/41 
  Upgrading        : openssl-libs-1:3.0.1-43.el9_0.x86_64                                                                                                                                                      4/41 
  Installing       : libwebp-1.2.0-3.el9.x86_64                                                                                                                                                                5/41 
  Installing       : remi-libzip-1.9.2-3.el9.remi.x86_64                                                                                                                                                       6/41 
  Installing       : gdk-pixbuf2-2.42.6-2.el9.x86_64                                                                                                                                                           7/41 
  Running scriptlet: xml-common-0.6.3-58.el9.noarch                                                                                                                                                            8/41 
  Installing       : xml-common-0.6.3-58.el9.noarch                                                                                                                                                            8/41 
  Installing       : fontconfig-2.14.0-1.el9.x86_64                                                                                                                                                            9/41 
  Running scriptlet: fontconfig-2.14.0-1.el9.x86_64                                                                                                                                                            9/41 
  Installing       : jbigkit-libs-2.1-23.el9.x86_64                                                                                                                                                           10/41 
  Installing       : libtiff-4.4.0-2.el9.x86_64                                                                                                                                                               11/41 
  Installing       : libXau-1.0.9-8.el9.x86_64                                                                                                                                                                12/41 
  Installing       : libxcb-1.13.1-9.el9.x86_64                                                                                                                                                               13/41 
  Installing       : libX11-common-1.7.0-7.el9.noarch                                                                                                                                                         14/41 
  Installing       : libX11-1.7.0-7.el9.x86_64                                                                                                                                                                15/41 
  Installing       : libXpm-3.5.13-7.el9.x86_64                                                                                                                                                               16/41 
  Installing       : fribidi-1.0.10-6.el9.2.x86_64                                                                                                                                                            17/41 
  Installing       : libraqm-0.8.0-1.el9.x86_64                                                                                                                                                               18/41 
  Installing       : libicu71-71.1-2.el9.remi.x86_64                                                                                                                                                          19/41 
  Installing       : svt-av1-libs-0.9.0-1.el9.x86_64                                                                                                                                                          20/41 
  Installing       : rav1e-libs-0.5.1-5.el9.x86_64                                                                                                                                                            21/41 
  Installing       : libvmaf-2.3.0-2.el9.x86_64                                                                                                                                                               22/41 
  Installing       : libimagequant-2.17.0-1.el9.x86_64                                                                                                                                                        23/41 
  Installing       : libdav1d-1.0.0-2.el9.x86_64                                                                                                                                                              24/41 
  Installing       : highway-1.0.2-1.el9.x86_64                                                                                                                                                               25/41 
  Installing       : jxl-pixbuf-loader-0.7.0-1.el9.x86_64                                                                                                                                                     26/41 
  Installing       : libjxl-0.7.0-1.el9.x86_64                                                                                                                                                                27/41 
  Installing       : libaom-3.5.0-2.el9.x86_64                                                                                                                                                                28/41 
  Installing       : libavif-0.11.1-4.el9.x86_64                                                                                                                                                              29/41 
  Installing       : gd3php-2.3.3-9.el9.remi.x86_64                                                                                                                                                           30/41 
  Installing       : php81-php-gd-8.1.14-1.el9.remi.x86_64                                                                                                                                                    31/41 
  Installing       : php81-php-intl-8.1.14-1.el9.remi.x86_64                                                                                                                                                  32/41 
  Installing       : php81-php-pecl-zip-1.21.1-1.el9.remi.x86_64                                                                                                                                              33/41 
  Upgrading        : openssl-1:3.0.1-43.el9_0.x86_64                                                                                                                                                          34/41 
  Installing       : php81-php-process-8.1.14-1.el9.remi.x86_64                                                                                                                                               35/41 
  Installing       : php81-php-mysqlnd-8.1.14-1.el9.remi.x86_64                                                                                                                                               36/41 
  Installing       : php81-php-gmp-8.1.14-1.el9.remi.x86_64                                                                                                                                                   37/41 
  Installing       : php81-php-bcmath-8.1.14-1.el9.remi.x86_64                                                                                                                                                38/41 
  Cleanup          : openssl-1:3.0.1-41.el9_0.x86_64                                                                                                                                                          39/41 
  Cleanup          : openssl-libs-1:3.0.1-41.el9_0.x86_64                                                                                                                                                     40/41 
  Cleanup          : libxml2-2.9.13-1.el9_0.1.x86_64                                                                                                                                                          41/41 
  Running scriptlet: fontconfig-2.14.0-1.el9.x86_64                                                                                                                                                           41/41 
  Running scriptlet: libxml2-2.9.13-1.el9_0.1.x86_64                                                                                                                                                          41/41 
  Verifying        : highway-1.0.2-1.el9.x86_64                                                                                                                                                                1/41 
  Verifying        : jxl-pixbuf-loader-0.7.0-1.el9.x86_64                                                                                                                                                      2/41 
  Verifying        : libaom-3.5.0-2.el9.x86_64                                                                                                                                                                 3/41 
  Verifying        : libavif-0.11.1-4.el9.x86_64                                                                                                                                                               4/41 
  Verifying        : libdav1d-1.0.0-2.el9.x86_64                                                                                                                                                               5/41 
  Verifying        : libimagequant-2.17.0-1.el9.x86_64                                                                                                                                                         6/41 
  Verifying        : libjxl-0.7.0-1.el9.x86_64                                                                                                                                                                 7/41 
  Verifying        : libraqm-0.8.0-1.el9.x86_64                                                                                                                                                                8/41 
  Verifying        : libvmaf-2.3.0-2.el9.x86_64                                                                                                                                                                9/41 
  Verifying        : rav1e-libs-0.5.1-5.el9.x86_64                                                                                                                                                            10/41 
  Verifying        : svt-av1-libs-0.9.0-1.el9.x86_64                                                                                                                                                          11/41 
  Verifying        : gd3php-2.3.3-9.el9.remi.x86_64                                                                                                                                                           12/41 
  Verifying        : libicu71-71.1-2.el9.remi.x86_64                                                                                                                                                          13/41 
  Verifying        : php81-php-bcmath-8.1.14-1.el9.remi.x86_64                                                                                                                                                14/41 
  Verifying        : php81-php-gd-8.1.14-1.el9.remi.x86_64                                                                                                                                                    15/41 
  Verifying        : php81-php-gmp-8.1.14-1.el9.remi.x86_64                                                                                                                                                   16/41 
  Verifying        : php81-php-intl-8.1.14-1.el9.remi.x86_64                                                                                                                                                  17/41 
  Verifying        : php81-php-mysqlnd-8.1.14-1.el9.remi.x86_64                                                                                                                                               18/41 
  Verifying        : php81-php-pecl-zip-1.21.1-1.el9.remi.x86_64                                                                                                                                              19/41 
  Verifying        : php81-php-process-8.1.14-1.el9.remi.x86_64                                                                                                                                               20/41 
  Verifying        : remi-libzip-1.9.2-3.el9.remi.x86_64                                                                                                                                                      21/41 
  Verifying        : shared-mime-info-2.1-4.el9.x86_64                                                                                                                                                        22/41 
  Verifying        : fribidi-1.0.10-6.el9.2.x86_64                                                                                                                                                            23/41 
  Verifying        : fontconfig-2.14.0-1.el9.x86_64                                                                                                                                                           24/41 
  Verifying        : libXpm-3.5.13-7.el9.x86_64                                                                                                                                                               25/41 
  Verifying        : libX11-common-1.7.0-7.el9.noarch                                                                                                                                                         26/41 
  Verifying        : libXau-1.0.9-8.el9.x86_64                                                                                                                                                                27/41 
  Verifying        : libxcb-1.13.1-9.el9.x86_64                                                                                                                                                               28/41 
  Verifying        : libjpeg-turbo-2.0.90-5.el9.x86_64                                                                                                                                                        29/41 
  Verifying        : jbigkit-libs-2.1-23.el9.x86_64                                                                                                                                                           30/41 
  Verifying        : libtiff-4.4.0-2.el9.x86_64                                                                                                                                                               31/41 
  Verifying        : libwebp-1.2.0-3.el9.x86_64                                                                                                                                                               32/41 
  Verifying        : xml-common-0.6.3-58.el9.noarch                                                                                                                                                           33/41 
  Verifying        : libX11-1.7.0-7.el9.x86_64                                                                                                                                                                34/41 
  Verifying        : gdk-pixbuf2-2.42.6-2.el9.x86_64                                                                                                                                                          35/41 
  Verifying        : libxml2-2.9.13-2.el9.x86_64                                                                                                                                                              36/41 
  Verifying        : libxml2-2.9.13-1.el9_0.1.x86_64                                                                                                                                                          37/41 
  Verifying        : openssl-libs-1:3.0.1-43.el9_0.x86_64                                                                                                                                                     38/41 
  Verifying        : openssl-libs-1:3.0.1-41.el9_0.x86_64                                                                                                                                                     39/41 
  Verifying        : openssl-1:3.0.1-43.el9_0.x86_64                                                                                                                                                          40/41 
  Verifying        : openssl-1:3.0.1-41.el9_0.x86_64                                                                                                                                                          41/41 

Upgraded:
  libxml2-2.9.13-2.el9.x86_64                                       openssl-1:3.0.1-43.el9_0.x86_64                                       openssl-libs-1:3.0.1-43.el9_0.x86_64                                      
Installed:
  fontconfig-2.14.0-1.el9.x86_64         fribidi-1.0.10-6.el9.2.x86_64           gd3php-2.3.3-9.el9.remi.x86_64             gdk-pixbuf2-2.42.6-2.el9.x86_64             highway-1.0.2-1.el9.x86_64                
  jbigkit-libs-2.1-23.el9.x86_64         jxl-pixbuf-loader-0.7.0-1.el9.x86_64    libX11-1.7.0-7.el9.x86_64                  libX11-common-1.7.0-7.el9.noarch            libXau-1.0.9-8.el9.x86_64                 
  libXpm-3.5.13-7.el9.x86_64             libaom-3.5.0-2.el9.x86_64               libavif-0.11.1-4.el9.x86_64                libdav1d-1.0.0-2.el9.x86_64                 libicu71-71.1-2.el9.remi.x86_64           
  libimagequant-2.17.0-1.el9.x86_64      libjpeg-turbo-2.0.90-5.el9.x86_64       libjxl-0.7.0-1.el9.x86_64                  libraqm-0.8.0-1.el9.x86_64                  libtiff-4.4.0-2.el9.x86_64                
  libvmaf-2.3.0-2.el9.x86_64             libwebp-1.2.0-3.el9.x86_64              libxcb-1.13.1-9.el9.x86_64                 php81-php-bcmath-8.1.14-1.el9.remi.x86_64   php81-php-gd-8.1.14-1.el9.remi.x86_64     
  php81-php-gmp-8.1.14-1.el9.remi.x86_64 php81-php-intl-8.1.14-1.el9.remi.x86_64 php81-php-mysqlnd-8.1.14-1.el9.remi.x86_64 php81-php-pecl-zip-1.21.1-1.el9.remi.x86_64 php81-php-process-8.1.14-1.el9.remi.x86_64
  rav1e-libs-0.5.1-5.el9.x86_64          remi-libzip-1.9.2-3.el9.remi.x86_64     shared-mime-info-2.1-4.el9.x86_64          svt-av1-libs-0.9.0-1.el9.x86_64             xml-common-0.6.3-58.el9.noarch            

Complete!

```

🌞 **Récupérer NextCloud**

- créez le dossier `/var/www/tp5_nextcloud/`
  - ce sera notre *racine web* (ou *webroot*)
  - l'endroit où le site est stocké quoi, on y trouvera un `index.html` et un tas d'autres marde, tout ce qui constitue NextCloud :D
- récupérer le fichier suivant avec une commande `curl` ou `wget` : https://download.nextcloud.com/server/prereleases/nextcloud-25.0.0rc3.zip
- extrayez tout son contenu dans le dossier `/var/www/tp5_nextcloud/` en utilisant la commande `unzip`
  - installez la commande `unzip` si nécessaire
  - vous pouvez extraire puis déplacer ensuite, vous prenez pas la tête
  - contrôlez que le fichier `/var/www/tp5_nextcloud/index.html` existe pour vérifier que tout est en place
- **assurez-vous que le dossier `/var/www/tp5_nextcloud/` et tout son contenu appartient à l'utilisateur qui exécute le service Apache**
  - utilisez une commande `chown` si nécessaire

> A chaque fois que vous faites ce genre de trucs, assurez-vous que c'est bien ok. Par exemple, vérifiez avec un `ls -al` que tout appartient bien à l'utilisateur qui exécute Apache.
```
[logards@Nezuko ~]$ curl https://download.nextcloud.com/server/prereleases/nextcloud-25.0.0rc3.zip --output /tmp/tp5_nextcloud
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  168M  100  168M    0     0  11.5M      0  0:00:14  0:00:14 --:--:-- 11.0M

[logards@Nezuko ~]$ sudo unzip /tmp/next_cloud -d /var/www/tp5_nextcloud/
[logards@Nezuko ~]$ cat /var/www/tp5_nextcloud/index.html
<!DOCTYPE html>
<html>
<head>
        <script> window.location.href="index.php"; </script>
        <meta http-equiv="refresh" content="0; URL=index.php">
</head>
</html>

```

🌞 **Adapter la configuration d'Apache**

```apache
[logards@Nezuko ~]$ cat /etc/httpd/conf.d/apache.conf
<VirtualHost *:80>
  DocumentRoot /var/www/tp5_nextcloud/
  ServerName  web.tp5.linux

  <Directory /var/www/tp5_nextcloud/> 
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews
    <IfModule mod_dav.c>
      Dav off
    </IfModule>
  </Directory>
</VirtualHost>
```

🌞 **Redémarrer le service Apache** pour qu'il prenne en compte le nouveau fichier de conf

![NextCloud error](../pics/nc_install.png)

## 3. Finaliser l'installation de NextCloud

➜ **Sur votre PC**

```
[bastien@fedora /]$ sudo vim /etc/hosts
[bastien@fedora /]$ curl http://web.tp5.linux | head 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!DOCTYPE html>
<html class="ng-csp" data-placeholder-focus="false" lang="en" data-locale="en" >
        <head
 data-requesttoken="xQp+29XO9qtbwsjDL7lxCC+LaT6GnRIvZS5LAgZ3JwM=:7nsOn7eBrMoopoqPXtBDckfNA2ro7CdZC10+NTMYH2c=">
                <meta charset="utf-8">
                <title>
                        Nextcloud               </title>
                <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0">
                                <meta name="apple-itunes-app" content="app-id=1125420102">
                                <meta name="theme-color" content="#0082c9">
100  6421    0  6421    0     0   130k      0 --:--:-- --:--:-- --:--:--  130k
curl: (23) Failed writing body
```



🌞 **Exploration de la base de données**

```
[logards@Tanjiro ~]$ sudo mysql -u root -p
[sudo] password for logards: 
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 128
Server version: 10.5.16-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> SELECT count(*) AS number FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = 'nextcloud';
+--------+
| number |
+--------+
|     95 |
+--------+
1 row in set (0.001 sec)
```

