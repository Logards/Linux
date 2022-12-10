
🌞 **S'assurer que le service `sshd` est démarré**

```
[logards@Vanitas ~]$ systemctl status sshd
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2022-12-09 15:55:20 CET; 7min ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 687 (sshd)
      Tasks: 1 (limit: 5905)
     Memory: 5.6M
        CPU: 189ms
     CGroup: /system.slice/sshd.service
             └─687 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Dec 09 15:55:20 Vanitas.lab.ingesup systemd[1]: Starting OpenSSH server daemon...
Dec 09 15:55:20 Vanitas.lab.ingesup sshd[687]: Server listening on 0.0.0.0 port 22.
Dec 09 15:55:20 Vanitas.lab.ingesup sshd[687]: Server listening on :: port 22.
Dec 09 15:55:20 Vanitas.lab.ingesup systemd[1]: Started OpenSSH server daemon.
Dec 09 15:56:29 Vanitas.lab.ingesup sshd[856]: Accepted password for logards from 10.5.1.1 port 42498 ssh2
Dec 09 15:56:29 Vanitas.lab.ingesup sshd[856]: pam_unix(sshd:session): session opened for user logards(uid=1000) by (uid=0)
```

🌞 **Analyser les processus liés au service SSH**

```
[logards@Vanitas ~]$ ps -ef | grep sshd
root         687       1  0 15:55 ?        00:00:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
root         856     687  0 15:56 ?        00:00:00 sshd: logards [priv]
logards      861     856  0 15:56 ?        00:00:00 sshd: logards@pts/0
logards      937     862  0 16:08 pts/0    00:00:00 grep --color=auto sshd
```

🌞 **Déterminer le port sur lequel écoute le service SSH**
```
[logards@Vanitas ~]$ ss | grep ssh
tcp   ESTAB  0      0                         10.5.1.2:ssh           10.5.1.1:42498  
écoute sur le port 42498
```

🌞 **Consulter les logs du service SSH**

```
[logards@Vanitas log]$ journalctl| grep ssh
Dec 09 15:55:18 Vanitas.lab.ingesup systemd[1]: Created slice Slice /system/sshd-keygen.
Dec 09 15:55:19 Vanitas.lab.ingesup systemd[1]: Reached target sshd-keygen.target.
Dec 09 15:55:20 Vanitas.lab.ingesup sshd[687]: Server listening on 0.0.0.0 port 22.
Dec 09 15:55:20 Vanitas.lab.ingesup sshd[687]: Server listening on :: port 22.
Dec 09 15:56:29 Vanitas.lab.ingesup sshd[856]: Accepted password for logards from 10.5.1.1 port 42498 ssh2
Dec 09 15:56:29 Vanitas.lab.ingesup sshd[856]: pam_unix(sshd:session): session opened for user logards(uid=1000) by (uid=0)
```

## 2. Modification du service


🌞 **Identifier le fichier de configuration du serveur SSH**
```
c'est ssh_config
```
🌞 **Modifier le fichier de conf**

- exécutez un `echo $RANDOM` pour demander à votre shell de vous fournir un nombre aléatoire
  - simplement pour vous montrer la petite astuce et vous faire manipuler le shell :)
- changez le port d'écoute du serveur SSH pour qu'il écoute sur ce numéro de port
  - dans le compte-rendu je veux un `cat` du fichier de conf
  - filtré par un `| grep` pour mettre en évidence la ligne que vous avez modifié
```
[logards@Vanitas ssh]$ cat sshd_config | grep Port
   Port 13882
```
```
[logards@Vanitas ssh]$ sudo firewall-cmd --remove-service=ssh --permanent


[logards@Vanitas ssh]$ sudo firewall-cmd --add-port=13882/tcp --permanent
success

[logards@Vanitas ssh]$ sudo firewall-cmd --list-all | grep 13882
[sudo] password for logards: 
  ports: 13882/tcp
```

🌞 **Redémarrer le service**

```
[logards@Vanitas ssh]$ sudo systemctl restart sshd
```

🌞 **Effectuer une connexion SSH sur le nouveau port**

```
[bastien@fedora ~]$ ssh -p 13882 logards@10.5.1.2        
logards@10.5.1.2's password: 
Last login: Fri Dec  9 15:56:29 2022 from 10.5.1.1
```

✨ **Bonus : affiner la conf du serveur SSH**

```
[logards@Vanitas ~]$ sudo cat /etc/ssh/sshd_config | grep Max
MaxAuthTries 2
MaxSessions 2

[logards@Vanitas ~]$ sudo cat /etc/ssh/sshd_config | grep PermitR
PermitRootLogin No

```

# II. Service HTTP

## 1. Mise en place


🌞 **Installer le serveur NGINX**

```
[logards@Vanitas ~]$ sudo dnf install nginx
```
🌞 **Démarrer le service NGINX**
```
[logards@Vanitas ~]$ sudo systemctl enable nginx
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /usr/lib/systemd/system/nginx.service.
[logards@Vanitas ~]$ sudo systemctl start nginx
```

🌞 **Déterminer sur quel port tourne NGINX**
```
[logards@Vanitas ~]$ sudo ss -lutnp | grep nginx
tcp   LISTEN 0      511          0.0.0.0:80        0.0.0.0:*    users:(("nginx",pid=10721,fd=6),("nginx",pid=10720,fd=6))
tcp   LISTEN 0      511             [::]:80           [::]:*    users:(("nginx",pid=10721,fd=7),("nginx",pid=10720,fd=7))
écoute sur le port 80
```

> **NB : c'est la dernière fois que je signale explicitement la nécessité d'ouvrir un port dans le firewall.** Vous devrez vous-mêmes y penser lorsque nécessaire. **Toutes les commandes liées au firewall doivent malgré tout figurer dans le compte-rendu.**

🌞 **Déterminer les processus liés à l'exécution de NGINX**
```
[logards@Vanitas ~]$ sudo ps -ef | grep nginx
root       10720       1  0 17:37 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx      10721   10720  0 17:37 ?        00:00:00 nginx: worker process
logards    10888     856  0 17:55 pts/0    00:00:00 grep --color=auto nginx
```

🌞 **Euh wait**
```
[logards@Vanitas ~]$ curl http://10.5.1.2:80 | head -n 7
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
100  7620  100  7620    0     0   496k      0 --:--:-- --:--:-- --:--:--  531k
curl: (23) Failed writing body
```

## 2. Analyser la conf de NGINX

🌞 **Déterminer le path du fichier de configuration de NGINX**

```
[logards@Vanitas ~]$ ls -al /etc/nginx/ | grep nginx.conf
-rw-r--r--.  1 root root 2334 Oct 31 16:37 nginx.conf

```

🌞 **Trouver dans le fichier de conf**

```[logards@Vanitas nginx]$ cat nginx.conf | grep server -A 15
    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
```

## 3. Déployer un nouveau site web

🌞 **Créer un site web**

```
[logards@Vanitas www]$ sudo !!
sudo mkdir tp2_linux
[logards@Vanitas www]$ ls
tp2_linux
[logards@Vanitas tp2_linux]$ sudo touch index.html
[logards@Vanitas tp2_linux]$ ls
index.html
[logards@Vanitas tp2_linux]$ sudo vim index.html
[logards@Vanitas tp2_linux]$ cat index.html 
<h1>MEOW Mon premier serveur web UWU <h1>
```

🌞 **Adapter la conf NGINX**

- dans le fichier de conf principal
  - vous supprimerez le bloc `server {}` repéré plus tôt pour que NGINX ne serve plus le site par défaut
  - redémarrez NGINX pour que les changements prennent effet
- créez un nouveau fichier de conf
  - il doit être nommé correctement
  - il doit être placé dans le bon dossier
  - c'est quoi un "nom correct" et "le bon dossier" ?
    - bah vous avez repéré dans la partie d'avant les fichiers qui sont inclus par le fichier de conf principal non ?
    - créez votre fichier en conséquence
  - redémarrez NGINX pour que les changements prennent effet
  - le contenu doit être le suivant :

```nginx
server {
  # le port choisi devra être obtenu avec un 'echo $RANDOM' là encore
  listen <PORT>;

  root /var/www/tp2_linux;
}
```

```
[logards@Vanitas ~]$ sudo cat /etc/nginx/nginx.conf | grep server -A 10
# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2;
#        listen       [::]:443 ssl http2;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers PROFILE=SYSTEM;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }

[logards@Vanitas ~]$ sudo systemctl restart nginx
```

🌞 **Visitez votre super site web**

- toujours avec une commande `curl` depuis votre PC (ou un navigateur)

```
[logards@Vanitas nginx]$ cat nginx.conf
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;


# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2;
#        listen       [::]:443 ssl http2;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers PROFILE=SYSTEM;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }

}
[logards@Vanitas conf.d]$ sudo firewall-cmd --remove-service=http --permanent
success
[logards@Vanitas conf.d]$ sudo firewall-cmd --reload
success
[logards@Vanitas conf.d]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 
  services: cockpit dhcpv6-client
  ports: 22/tcp 10897/tcp 23125/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
[logards@Vanitas conf.d]$ 

[logards@Vanitas conf.d]$ sudo firewall-cmd --add-port=23125/tcp --permanent
success
[logards@Vanitas conf.d]$ sudo firewall-cmd --reload
success
[logards@Vanitas conf.d]$ sudo systemctl restart nginx
[logards@Vanitas conf.d]$ curl 10.5.1.2:23125
<h1>MEOW Mon premier serveur web UWU <h1>
```



# III. Your own services


🌞 **Afficher le fichier de service SSH**

```
[logards@Vanitas ~]$ systemctl status sshd
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2022-12-10 10:47:10 CET; 7min ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 688 (sshd)
      Tasks: 1 (limit: 5905)
     Memory: 5.6M
        CPU: 188ms
     CGroup: /system.slice/sshd.service
             └─688 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Dec 10 10:47:10 Vanitas.lab.ingesup systemd[1]: Starting OpenSSH server daemon...
Dec 10 10:47:10 Vanitas.lab.ingesup sshd[688]: Server listening on 0.0.0.0 port 22.
Dec 10 10:47:10 Vanitas.lab.ingesup sshd[688]: Server listening on :: port 22.
Dec 10 10:47:10 Vanitas.lab.ingesup systemd[1]: Started OpenSSH server daemon.
Dec 10 10:48:06 Vanitas.lab.ingesup sshd[854]: Accepted password for logards from 10.5.1.1 port 49052 ssh2
Dec 10 10:48:06 Vanitas.lab.ingesup sshd[854]: pam_unix(sshd:session): session opened for user logards(uid=1000) by (uid=0)

[logards@Vanitas ~]$ sudo cat /usr/lib/systemd/system/sshd.service | grep ExecStart
ExecStart=/usr/sbin/sshd -D $OPTIONS

[logards@Vanitas ~]$ sudo !!
sudo systemctl start sshd
```

🌞 **Afficher le fichier de service NGINX**

- mettez en évidence la ligne qui commence par `ExecStart=`
```
[logards@Vanitas ~]$ systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
     Active: active (running) since Sat 2022-12-10 10:47:12 CET; 13min ago
    Process: 792 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 797 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 803 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 808 (nginx)
      Tasks: 2 (limit: 5905)
     Memory: 3.6M
        CPU: 38ms
     CGroup: /system.slice/nginx.service
             ├─808 "nginx: master process /usr/sbin/nginx"
             └─812 "nginx: worker process"

Dec 10 10:47:12 Vanitas.lab.ingesup systemd[1]: Starting The nginx HTTP and reverse proxy server...
Dec 10 10:47:12 Vanitas.lab.ingesup nginx[797]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Dec 10 10:47:12 Vanitas.lab.ingesup nginx[797]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Dec 10 10:47:12 Vanitas.lab.ingesup systemd[1]: Started The nginx HTTP and reverse proxy server.

[logards@Vanitas ~]$ sudo cat /usr/lib/systemd/system/nginx.service | grep ExecStart
ExecStartPre=/usr/bin/rm -f /run/nginx.pid
ExecStartPre=/usr/sbin/nginx -t
ExecStart=/usr/sbin/nginx
```

## 3. Création de service

🌞 **Créez le fichier `/etc/systemd/system/tp2_nc.service`**
```
[logards@Vanitas ~]$ sudo cat /etc/systemd/system/tp2_nc.service
[Unit]
Description=Super netcat tout fou

[Service]
ExecStart=/usr/bin/nc -l 10897

[logards@Vanitas ~]$ sudo firewall-cmd --add-port=10897/tcp --permanent
success

```
🌞 **Indiquer au système qu'on a modifié les fichiers de service**

```
[logards@Vanitas ~]$ sudo systemctl daemon-reload
```

🌞 **Démarrer notre service de ouf**
```
[logards@Vanitas ~]$ sudo !!
sudo systemctl start tp2_nc.service
```


🌞 **Vérifier que ça fonctionne**

```
[logards@Vanitas ~]$ systemctl status tp2_nc.service
● tp2_nc.service - Super netcat tout fou
     Loaded: loaded (/etc/systemd/system/tp2_nc.service; static)
     Active: active (running) since Sat 2022-12-10 11:15:37 CET; 11s ago
   Main PID: 985 (nc)
      Tasks: 1 (limit: 5905)
     Memory: 772.0K
        CPU: 6ms
     CGroup: /system.slice/tp2_nc.service
             └─985 /usr/bin/nc -l 10897

Dec 10 11:15:37 Vanitas.lab.ingesup systemd[1]: Started Super netcat tout fou.

[logards@Vanitas ~]$ sudo !!
sudo ss -lutnp | grep nc
tcp   LISTEN 0      10           0.0.0.0:10897      0.0.0.0:*    users:(("nc",pid=985,fd=4))     
tcp   LISTEN 0      10              [::]:10897         [::]:*    users:(("nc",pid=985,fd=3)) 

[logards@Vanitas ~]$ sudo firewall-cmd --add-port=10897/tcp --permanent
Warning: ALREADY_ENABLED: 10897:tcp
success

[logards@Noe ~]$ nc 10.5.1.2 10897
:nbf
```


🌞 **Les logs de votre service**
```
[logards@Vanitas ~]$ sudo journalctl -xe -u tp2_nc
[sudo] password for logards: 
Dec 10 11:15:37 Vanitas.lab.ingesup systemd[1]: Started Super netcat tout fou.
░░ Subject: A start job for unit tp2_nc.service has finished successfully
░░ Defined-By: systemd
░░ Support: https://access.redhat.com/support
░░ 
░░ A start job for unit tp2_nc.service has finished successfully.
░░ 
░░ The job identifier is 872.
Dec 10 11:21:42 Vanitas.lab.ingesup nc[985]: :nbf

[logards@Vanitas ~]$ sudo journalctl -xe -u tp2_nc -f 
Dec 10 11:15:37 Vanitas.lab.ingesup systemd[1]: Started Super netcat tout fou.
░░ Subject: A start job for unit tp2_nc.service has finished successfully
░░ Defined-By: systemd
░░ Support: https://access.redhat.com/support
░░ 
░░ A start job for unit tp2_nc.service has finished successfully.
░░ 
░░ The job identifier is 872.
Dec 10 11:21:42 Vanitas.lab.ingesup nc[985]: :nbf
Dec 10 11:35:41 Vanitas.lab.ingesup nc[985]: salur
Dec 10 11:35:44 Vanitas.lab.ingesup nc[985]: ça va
Dec 10 11:35:48 Vanitas.lab.ingesup nc[985]: MIAAAAAOUUUUU
Dec 10 11:36:05 Vanitas.lab.ingesup nc[985]: UWU

[logards@Vanitas ~]$ journalctl | grep start | grep tp2
Dec 10 11:15:37 Vanitas.lab.ingesup sudo[981]:  logards : TTY=pts/0 ; PWD=/home/logards ; USER=root ; COMMAND=/bin/systemctl start tp2_nc.service

[logards@Vanitas ~]$ journalctl | grep nc | grep 985
Dec 10 11:21:42 Vanitas.lab.ingesup nc[985]: :nbf
Dec 10 11:35:41 Vanitas.lab.ingesup nc[985]: salur
Dec 10 11:35:44 Vanitas.lab.ingesup nc[985]: ça va
Dec 10 11:35:48 Vanitas.lab.ingesup nc[985]: MIAAAAAOUUUUU
Dec 10 11:36:05 Vanitas.lab.ingesup nc[985]: UWU

[logards@Vanitas ~]$ journalctl | grep tp2 | grep stop
Dec 10 12:05:52 Vanitas.lab.ingesup sudo[1122]:  logards : TTY=pts/0 ; PWD=/home/logards ; USER=root ; COMMAND=/bin/systemctl stop tp2_nc.service

```

🌞 **Affiner la définition du service**
```
[logards@Vanitas ~]$ sudo cat /etc/systemd/system/tp2_nc.service 
[Unit]
Description=Super netcat tout fou

[Service]
ExecStart=/usr/bin/nc -l 10897
Restart=always

[logards@Vanitas ~]$ sudo systemctl restart tp2_nc.service
```