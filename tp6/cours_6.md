# I. Setup

🖥️ **VM `proxy.tp6.linux`**


🌞 **On utilisera NGINX comme reverse proxy**

```
[logards@proxy-tp6-linux ~]$ sudo dnf install nginx
[logards@proxy-tp6-linux ~]$ sudo systemctl start nginx
[logards@proxy-tp6-linux ~]$ systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
     Active: active (running) since Tue 2023-01-31 15:07:29 CET; 25s ago
    Process: 1389 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 1390 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 1391 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 1392 (nginx)
      Tasks: 2 (limit: 4638)
     Memory: 1.9M
        CPU: 37ms
     CGroup: /system.slice/nginx.service
             ├─1392 "nginx: master process /usr/sbin/nginx"
             └─1393 "nginx: worker process"

Jan 31 15:07:29 proxy-tp6-linux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Jan 31 15:07:29 proxy-tp6-linux nginx[1390]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Jan 31 15:07:29 proxy-tp6-linux nginx[1390]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Jan 31 15:07:29 proxy-tp6-linux systemd[1]: Started The nginx HTTP and reverse proxy server.
[logards@proxy-tp6-linux ~]$ sudo ss -alutnp | grep nginx | head -n 1
tcp   LISTEN 0      511          0.0.0.0:80        0.0.0.0:*    users:(("nginx",pid=1393,fd=6),("nginx",pid=1392,fd=6))
[logards@proxy-tp6-linux ~]$ sudo firewall-cmd --add-port 80/tcp --permanent 
success
[logards@proxy-tp6-linux ~]$ sudo firewall-cmd --reload
success
[logards@proxy-tp6-linux ~]$ sudo firewall-cmd --list-all | grep port | head -n 1
  ports: 80/tcp
[logards@proxy-tp6-linux ~]$ ps -ef | grep nginx
root        1392       1  0 15:07 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx       1393    1392  0 15:07 ?        00:00:00 nginx: worker process
logards     1472    1214  0 15:12 pts/0    00:00:00 grep --color=auto nginx
[logards@proxy-tp6-linux ~]$ curl http://10.105.1.13:80 | head
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
100  7620  100  7620    0     0   531k      0 --:--:-- --:--:-- --:--:--  531k
curl: (23) Failed writing body
```

🌞 **Configurer NGINX**

```
[logards@proxy-tp6-linux ~]$ sudo vim /etc/nginx/conf.d/nginx.conf
[logards@proxy-tp6-linux ~]$ cat /etc/nginx/conf.d/nginx.conf
server {
    # On indique le nom que client va saisir pour accéder au service
    # Pas d'erreur ici, c'est bien le nom de web, et pas de proxy qu'on veut ici !
    server_name www.nextcloud.tp6;

    # Port d'écoute de NGINX
    listen 80;

    location / {
        # On définit des headers HTTP pour que le proxying se passe bien
        proxy_set_header  Host $host;
        proxy_set_header  X-Real-IP $remote_addr;
        proxy_set_header  X-Forwarded-Proto https;
        proxy_set_header  X-Forwarded-Host $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;

        # On définit la cible du proxying 
        proxy_pass http://10.105.1.12:80;
    }

    # Deux sections location recommandés par la doc NextCloud
    location /.well-known/carddav {
      return 301 $scheme://$host/remote.php/dav;
    }

    location /.well-known/caldav {
      return 301 $scheme://$host/remote.php/dav;
    }
}
[logards@web-tp6-linux ~]$ sudo vim  /var/www/tp5_nextcloud/config/config.php
[sudo] password for logards:
[logards@web-tp6-linux ~]$ sudo cat   /var/www/tp5_nextcloud/config/config.php 
<?php
$CONFIG = array (
  'instanceid' => 'ocpd1ncbdxsx',
  'passwordsalt' => '2gMAdK/KDUzBsVWTkj3VRNU9dMyQrW',
  'secret' => 'zWZejPX4Kt1S/E3BJSvX3auyftpNaKbaBqOHL/lHnKnyDI8T',
  'trusted_domains' =>  
  array (
          0 => 'web.tp5.linux',
          1 => '10.105.1.13'
  ),
  'datadirectory' => '/var/www/tp5_nextcloud/data',
  'dbtype' => 'mysql',
  'version' => '25.0.0.15',
  'overwrite.cli.url' => 'http://web.tp5.linux',
  'dbname' => 'nextcloud',
  'dbhost' => '10.105.1.12',
  'dbport' => '',
  'dbtableprefix' => 'oc_',
  'mysql.utf8mb4' => true,
  'dbuser' => 'nextcloud',
  'dbpassword' => 'pewpewpew',
  'installed' => true,
);
```


🌞 **Faites en sorte de**

- rendre le serveur `web.tp6.linux` injoignable
- sauf depuis l'IP du reverse proxy
- en effet, les clients ne doivent pas joindre en direct le serveur web : notre reverse proxy est là pour servir de serveur frontal
- **comment ?** Je vous laisser là encore chercher un peu par vous-mêmes (hint : firewall)

🌞 **Une fois que c'est en place**

- faire un `ping` manuel vers l'IP de `proxy.tp6.linux` fonctionne
- faire un `ping` manuel vers l'IP de `web.tp6.linux` ne fonctionne pas

![Not sure](../pics/reverse_proxy.png)

# II. HTTPS

Le but de cette section est de permettre une connexion chiffrée lorsqu'un client se connecte. Avoir le ptit HTTPS :)

Le principe :

- on génère une paire de clés sur le serveur `proxy.tp6.linux`
  - une des deux clés sera la clé privée : elle restera sur le serveur et ne bougera jamais
  - l'autre est la clé publique : elle sera stockée dans un fichier appelé *certificat*
    - le *certificat* est donné à chaque client qui se connecte au site
- on ajuste la conf NGINX
  - on lui indique le chemin vers le certificat et la clé privée afin qu'il puisse les utiliser pour chiffrer le trafic
  - on lui demande d'écouter sur le port convetionnel pour HTTPS : 443 en TCP

Je vous laisse Google vous-mêmes "nginx reverse proxy nextcloud" ou ce genre de chose :)

🌞 **Faire en sorte que NGINX force la connexion en HTTPS plutôt qu'HTTP**

🌞 Faites en sorte que :

- si quelqu'un se plante 3 fois de password pour une co SSH en moins de 1 minute, il est ban
```
[logards@db-tp6-linux fail2ban]$ cat jail.local | grep 'maxretry = 3'
maxretry = 3
[logards@db-tp6-linux fail2ban]$ cat jail.local | grep 'bantime  = 1m'
bantime  = 1m
```
```
[logards@db-tp6-linux fail2ban]$ sudo fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed: 1
|  |- Total failed:     6
|  `- Journal matches:  _SYSTEMD_UNIT=sshd.service + _COMM=sshd
`- Actions
   |- Currently banned: 1
   |- Total banned:     2
   `- Banned IP list:   10.105.1.11
```
```
[logards@db-tp6-linux fail2ban]$ sudo iptables -vnL | grep '10.105.1.11'
   18  2364 REJECT     all  --  *      *       10.105.1.11          0.0.0.0/0            reject-with icmp-port-unreachable
```
```
[logards@db-tp6-linux fail2ban]$ sudo fail2ban-client unban '10.105.1.11'
1
[logards@db-tp6-linux fail2ban]$ sudo fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed: 1
|  |- Total failed:     12
|  `- Journal matches:  _SYSTEMD_UNIT=sshd.service + _COMM=sshd
`- Actions
   |- Currently banned: 0
   |- Total banned:     5
   `- Banned IP list:
```

🌞 **Installer Netdata**

```
[logards@db-tp6-linux ~]$ sudo wget -O /tmp/netdata-kickstart.sh https://my-netdata.io/kickstart.sh && sh /tmp/netdata-kickstart.sh
[logards@db-tp6-linux ~]$ sudo systemctl start netdata
[logards@db-tp6-linux ~]$ sudo systemctl enable netdata
[logards@db-tp6-linux ~]$ systemctl status netdata
● netdata.service - Real time performance monitoring
     Loaded: loaded (/usr/lib/systemd/system/netdata.service; enabled; vendor preset: disabled)
     Active: active (running) since Tue 2023-02-07 11:44:54 CET; 1min 23s ago
   Main PID: 4773 (netdata)
      Tasks: 70 (limit: 4638)
     Memory: 154.4M
        CPU: 6.227s
     CGroup: /system.slice/netdata.service
             ├─4773 /usr/sbin/netdata -P /run/netdata/netdata.pid -D
             ├─4777 /usr/sbin/netdata --special-spawn-server
             ├─5000 bash /usr/libexec/netdata/plugins.d/tc-qos-helper.sh 1
             ├─5006 /usr/libexec/netdata/plugins.d/apps.plugin 1
             ├─5008 /usr/libexec/netdata/plugins.d/ebpf.plugin 1
             └─5009 /usr/libexec/netdata/plugins.d/go.d.plugin 1

Feb 07 11:44:56 db-tp6-linux ebpf.plugin[5008]: thread created with task id 5078
Feb 07 11:44:56 db-tp6-linux ebpf.plugin[5008]: set name of thread 5078 to EBPF SHM
Feb 07 11:44:56 db-tp6-linux ebpf.plugin[5008]: thread created with task id 5076
Feb 07 11:44:56 db-tp6-linux ebpf.plugin[5008]: set name of thread 5076 to EBPF SOFTIRQ
Feb 07 11:44:56 db-tp6-linux ebpf.plugin[5008]: thread created with task id 5075
Feb 07 11:44:56 db-tp6-linux ebpf.plugin[5008]: set name of thread 5075 to EBPF HARDIRQ
Feb 07 11:44:56 db-tp6-linux ebpf.plugin[5008]: thread created with task id 5073
Feb 07 11:44:56 db-tp6-linux ebpf.plugin[5008]: set name of thread 5073 to EBPF MOUNT
Feb 07 11:44:56 db-tp6-linux ebpf.plugin[5008]: thread with task id 5077 finished
Feb 07 11:44:57 db-tp6-linux apps.plugin[5006]: Using now_boottime_usec() for uptime (dt is 9 ms)
[logards@db-tp6-linux ~]$ sudo firewall-cmd --permanent --add-port=19999/tcp
success
[logards@db-tp6-linux ~]$ sudo firewall-cmd --reload
success
[logards@db-tp6-linux ~]$ 

[logards@web-tp6-linux ~]$ sudo wget -O /tmp/netdata-kickstart.sh https://my-netdata.io/kickstart.sh && sh /tmp/netdata-kickstart.sh
[logards@web-tp6-linux ~]$ systemctl start netdata
Failed to start netdata.service: Access denied
See system logs and 'systemctl status netdata.service' for details.
[logards@web-tp6-linux ~]$ sudo !!
sudo systemctl start netdata
[logards@web-tp6-linux ~]$ sudo systemctl enable netdata
[logards@web-tp6-linux ~]$ ~systemctl status netdata
-bash: ~systemctl: command not found
[logards@web-tp6-linux ~]$ systemctl status netdata
● netdata.service - Real time performance monitoring
     Loaded: loaded (/usr/lib/systemd/system/netdata.service; enabled; vendor preset: disabled)
     Active: active (running) since Tue 2023-02-07 11:44:43 CET; 59s ago
   Main PID: 4876 (netdata)
      Tasks: 71 (limit: 4638)
     Memory: 140.3M
        CPU: 5.727s
     CGroup: /system.slice/netdata.service
             ├─4876 /usr/sbin/netdata -P /run/netdata/netdata.pid -D
             ├─4903 /usr/sbin/netdata --special-spawn-server
             ├─5103 bash /usr/libexec/netdata/plugins.d/tc-qos-helper.sh 1
             ├─5108 /usr/libexec/netdata/plugins.d/apps.plugin 1
             ├─5111 /usr/libexec/netdata/plugins.d/ebpf.plugin 1
             └─5112 /usr/libexec/netdata/plugins.d/go.d.plugin 1

Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: set name of thread 5175 to EBPF SWAP
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: thread created with task id 5179
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: set name of thread 5179 to EBPF SOFTIRQ
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: thread created with task id 5173
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: set name of thread 5173 to EBPF CACHESTAT
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: thread created with task id 5181
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: set name of thread 5181 to EBPF SHM
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: thread created with task id 5177
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: set name of thread 5177 to EBPF FD
Feb 07 11:44:46 web-tp6-linux apps.plugin[5108]: Using now_boottime_usec() for uptime (dt is 5 ms)
[logards@web-tp6-linux ~]$ sudo firewall-cmd --permanent --add-port=19999/tcp
success
[logards@web-tp6-linux ~]$ sudo firewall-cmd --reload
success
[logards@web-tp6-linux ~]$ 

```

🌞 **Une fois Netdata installé et fonctionnel, déterminer :**
```
[logards@web-tp6-linux ~]$ ps -ef | grep netdata
root        4637       1  0 11:44 ?        00:00:00 gpg-agent --homedir /var/cache/dnf/netdata-edge-a383c484584e0b14/pubring --use-standard-socket --daemon
root        4639    4637  0 11:44 ?        00:00:00 scdaemon --multi-server --homedir /var/cache/dnf/netdata-edge-a383c484584e0b14/pubring
root        4682       1  0 11:44 ?        00:00:00 gpg-agent --homedir /var/cache/dnf/netdata-repoconfig-3ca68ffb39611f32/pubring --use-standard-socket --daemon
root        4684    4682  0 11:44 ?        00:00:00 scdaemon --multi-server --homedir /var/cache/dnf/netdata-repoconfig-3ca68ffb39611f32/pubring
netdata     4876       1  1 11:44 ?        00:00:23 /usr/sbin/netdata -P /run/netdata/netdata.pid -D
netdata     4903    4876  0 11:44 ?        00:00:00 /usr/sbin/netdata --special-spawn-server
netdata     5103    4876  0 11:44 ?        00:00:01 bash /usr/libexec/netdata/plugins.d/tc-qos-helper.sh 1
netdata     5108    4876  1 11:44 ?        00:00:22 /usr/libexec/netdata/plugins.d/apps.plugin 1
root        5111    4876  0 11:44 ?        00:00:01 /usr/libexec/netdata/plugins.d/ebpf.plugin 1
netdata     5112    4876  0 11:44 ?        00:00:08 /usr/libexec/netdata/plugins.d/go.d.plugin 1
logards     5860    4482  0 12:14 pts/0    00:00:00 grep --color=auto netdata
[logards@web-tp6-linux ~]$ sudo ss -alutnpH | grep netdata | grep 1999
tcp LISTEN 0      4096     0.0.0.0:19999 0.0.0.0:* users:(("netdata",pid=4876,fd=7))                                                                                          
tcp LISTEN 0      4096        [::]:19999    [::]:* users:(("netdata",pid=4876,fd=8)) 
[logards@web-tp6-linux ~]$ journalctl -xeu netdata
░░ 
░░ A start job for unit netdata.service has finished successfully.
░░ 
░░ The job identifier is 1126.
Feb 07 11:44:43 web-tp6-linux netdata[4876]: CONFIG: cannot load cloud config '/var/lib/netdata/cloud.d/cloud.conf'. Running with internal defaults.
Feb 07 11:44:43 web-tp6-linux netdata[4876]: 2023-02-07 11:44:43: netdata INFO  : MAIN : CONFIG: cannot load cloud config '/var/lib/netdata/cloud.d/cloud.conf'. Running with internal defaults.
Feb 07 11:44:43 web-tp6-linux netdata[4876]: Found 0 legacy dbengines, setting multidb diskspace to 256MB
Feb 07 11:44:43 web-tp6-linux netdata[4876]: 2023-02-07 11:44:43: netdata INFO  : MAIN : Found 0 legacy dbengines, setting multidb diskspace to 256MB
Feb 07 11:44:43 web-tp6-linux netdata[4876]: 2023-02-07 11:44:43: netdata INFO  : MAIN : Created file '/var/lib/netdata/dbengine_multihost_size' to store the computed value
Feb 07 11:44:43 web-tp6-linux netdata[4876]: Created file '/var/lib/netdata/dbengine_multihost_size' to store the computed value
Feb 07 11:44:45 web-tp6-linux perf.plugin[5114]: no charts enabled - nothing to do.
Feb 07 11:44:45 web-tp6-linux apps.plugin[5108]: PROCFILE: Cannot open file '/etc/netdata/apps_groups.conf'
Feb 07 11:44:45 web-tp6-linux apps.plugin[5108]: Cannot read process groups configuration file '/etc/netdata/apps_groups.conf'. Will try '/usr/lib/netdata/conf.d/apps_groups.conf'
Feb 07 11:44:45 web-tp6-linux apps.plugin[5108]: Loaded config file '/usr/lib/netdata/conf.d/apps_groups.conf'
Feb 07 11:44:45 web-tp6-linux ebpf.plugin[5111]: Does not have a configuration file inside `/etc/netdata/ebpf.d.conf. It will try to load stock file.
Feb 07 11:44:45 web-tp6-linux apps.plugin[5108]: started on pid 5108
Feb 07 11:44:45 web-tp6-linux apps.plugin[5108]: set name of thread 5142 to APPS_READER
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: Cannot read process groups configuration file '/etc/netdata/apps_groups.conf'. Will try '/usr/lib/netdata/conf.d/apps_groups.conf'
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: thread created with task id 5172
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: set name of thread 5172 to EBPF PROCESS
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: thread created with task id 5174
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: set name of thread 5174 to EBPF SYNC
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: thread created with task id 5176
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: set name of thread 5176 to EBPF MOUNT
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: thread created with task id 5178
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: set name of thread 5178 to EBPF HARDIRQ
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: thread created with task id 5180
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: set name of thread 5180 to EBPF OOMKILL
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: thread with task id 5180 finished
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: thread created with task id 5175
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: set name of thread 5175 to EBPF SWAP
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: thread created with task id 5179
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: set name of thread 5179 to EBPF SOFTIRQ
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: thread created with task id 5173
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: set name of thread 5173 to EBPF CACHESTAT
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: thread created with task id 5181
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: set name of thread 5181 to EBPF SHM
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: thread created with task id 5177
Feb 07 11:44:46 web-tp6-linux ebpf.plugin[5111]: set name of thread 5177 to EBPF FD
Feb 07 11:44:46 web-tp6-linux apps.plugin[5108]: Using now_boottime_usec() for uptime (dt is 5 ms)
lines 13-52/52 (END)
```
🌞 **Configurer Netdata pour qu'il vous envoie des alertes** 

```
[logards@web-tp6-linux ~]$ cat /etc/netdata/health_alarm_notify.conf
###############################################################################
# sending discord notifications

# note: multiple recipients can be given like this:
#                  "CHANNEL1 CHANNEL2 ..."

# enable/disable sending discord notifications
SEND_DISCORD="YES"

# Create a webhook by following the official documentation -
# https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks
DISCORD_WEBHOOK_URL="https://discordapp.com/api/webhooks/1072478674034626652/moolBTtTmLZaJd-Ea3myF4LbJ_U19Pk012mF6w1AdD_VkfexBc-prNIf5SIPscJBZCKV"

# if a role's recipients are not configured, a notification will be send to
# this discord channel (empty = do not send a notification for unconfigured
# roles):
DEFAULT_RECIPIENT_DISCORD="alarms"
```

🌞 **Vérifier que les alertes fonctionnent**

```
[logards@web-tp6-linux ~]$ stress -c 2 -i 1 -m 1 --vm-bytes 128M -t 60s
stress: info: [6352] dispatching hogs: 2 cpu, 1 io, 1 vm, 0 hdd
stress: info: [6352] successful run completed in 60s
```

🌞 **Ecrire le script `bash`**

```
[logards@web-tp6-linux srv]$ cat tp6_backup.sh 
#!/bin/bash
# 13/02/23
# Logards
# système de backup

netdata='/etc/netdata/'
backup='/srv/backup/'

rsync -avz ${netdata} ${backup}/nextcloud_`date +"%Y%m%d"`/
cd ${backup}
zip -r nextcloud_`date +"%Y%m%d"`.zip ${backup}nextcloud_`date +"%Y%m%d"`
rm -rf ${backup}nextcloud_`date +"%Y%m%d"`
```

```
#!/bin/bash
```

- pour apprendre quels dossiers il faut sauvegarder dans tout le bordel de NextCloud, [il existe une page de la doc officielle qui vous informera](https://docs.nextcloud.com/server/latest/admin_manual/maintenance/backup.html)
- vous devez compresser les dossiers importants
  - au format `.zip` ou `.tar.gz`
  - le fichier produit sera stocké dans le dossier `/srv/backup/`
  - il doit comporter la date, l'heure la minute et la seconde où a été effectué la sauvegarde
    - par exemple : `nextcloud_2211162108.tar.gz`
```
[logards@web-tp6-linux srv]$ sudo ./tp6_backup.sh 
sending incremental file list
created directory /srv/backup//nextcloud_20230213
./
.install-type
edit-config
health_alarm_notify.conf
netdata.conf
charts.d/
custom-plugins.d/
ebpf.d/
go.d/
health.d/
python.d/
ssl/
statsd.d/

sent 3,673 bytes  received 185 bytes  7,716.00 bytes/sec
total size is 9,659  speedup is 2.50
  adding: srv/backup/nextcloud_20230213/ (stored 0%)
  adding: srv/backup/nextcloud_20230213/charts.d/ (stored 0%)
  adding: srv/backup/nextcloud_20230213/custom-plugins.d/ (stored 0%)
  adding: srv/backup/nextcloud_20230213/ebpf.d/ (stored 0%)
  adding: srv/backup/nextcloud_20230213/go.d/ (stored 0%)
  adding: srv/backup/nextcloud_20230213/health.d/ (stored 0%)
  adding: srv/backup/nextcloud_20230213/python.d/ (stored 0%)
  adding: srv/backup/nextcloud_20230213/ssl/ (stored 0%)
  adding: srv/backup/nextcloud_20230213/statsd.d/ (stored 0%)
  adding: srv/backup/nextcloud_20230213/.install-type (deflated 10%)
  adding: srv/backup/nextcloud_20230213/edit-config (deflated 74%)
  adding: srv/backup/nextcloud_20230213/health_alarm_notify.conf (deflated 42%)
  adding: srv/backup/nextcloud_20230213/netdata.conf (deflated 45%)

[logards@web-tp6-linux srv]$ ls /srv/backup/
nextcloud_20230213.zip

```

```
[logards@web-tp6-linux srv]$ sudo useradd -d /srv/backup/ -s /usr/bin/nologin backup
[sudo] password for logards: 
useradd: Warning: missing or non-executable shell '/usr/bin/nologin'
useradd: warning: the home directory /srv/backup/ already exists.
useradd: Not copying any file from skel directory into it.
[logards@web-tp6-linux srv]$ cat /etc/passwd | grep backup
backup:x:1002:1002::/srv/backup/:/usr/bin/nologin
```
```
[logards@web-tp6-linux /]$ sudo chown  backup /srv/backup/ 
[logards@web-tp6-linux /]$ ls -l /srv/
total 4
drwxr-xr-x. 2 backup root  36 Feb 13 19:19 backup
-rwxr-xr-x. 1 root   root 323 Feb 13 19:28 tp6_backup.sh

[logards@web-tp6-linux srv]$ sudo -u backup /srv/tp6_backup.sh
sending incremental file list
created directory /srv/backup//nextcloud_20230214
./
.install-type
edit-config
health_alarm_notify.conf
netdata.conf
charts.d/
custom-plugins.d/
ebpf.d/
go.d/
health.d/
python.d/
ssl/
statsd.d/

sent 3,673 bytes  received 185 bytes  7,716.00 bytes/sec
total size is 9,659  speedup is 2.50
  adding: srv/backup/nextcloud_20230214/ (stored 0%)
  adding: srv/backup/nextcloud_20230214/charts.d/ (stored 0%)
  adding: srv/backup/nextcloud_20230214/custom-plugins.d/ (stored 0%)
  adding: srv/backup/nextcloud_20230214/ebpf.d/ (stored 0%)
  adding: srv/backup/nextcloud_20230214/go.d/ (stored 0%)
  adding: srv/backup/nextcloud_20230214/health.d/ (stored 0%)
  adding: srv/backup/nextcloud_20230214/python.d/ (stored 0%)
  adding: srv/backup/nextcloud_20230214/ssl/ (stored 0%)
  adding: srv/backup/nextcloud_20230214/statsd.d/ (stored 0%)
  adding: srv/backup/nextcloud_20230214/.install-type (deflated 10%)
  adding: srv/backup/nextcloud_20230214/edit-config (deflated 74%)
  adding: srv/backup/nextcloud_20230214/health_alarm_notify.conf (deflated 42%)
  adding: srv/backup/nextcloud_20230214/netdata.conf (deflated 45%)

```

### 3. Service et timer

🌞 **Créez un *service*** système qui lance le script

```
[logards@web-tp6-linux srv]$ sudo vim /etc/systemd/system/backup.service
[logards@web-tp6-linux srv]$ cat /etc/systemd/system/backup.service 
[Unit]
Description=lancement du systeme de backup

[Service]
Type=oneshot
ExecStart=/srv/tp6_backup.sh
User=backup


[Install]
WantedBy=multi-user.target

[logards@web-tp6-linux srv]$ sudo systemctl start backup.service
[logards@web-tp6-linux srv]$ systemctl status backup.service
○ backup.service - lancement du systeme de backup
     Loaded: loaded (/etc/systemd/system/backup.service; disabled; vendor preset: disabled)
     Active: inactive (dead)

Feb 14 11:09:06 web-tp6-linux tp6_backup.sh[2255]: updating: srv/backup/nextcloud_20230214/health.d/ (stored 0%)
Feb 14 11:09:06 web-tp6-linux tp6_backup.sh[2255]: updating: srv/backup/nextcloud_20230214/python.d/ (stored 0%)
Feb 14 11:09:06 web-tp6-linux tp6_backup.sh[2255]: updating: srv/backup/nextcloud_20230214/ssl/ (stored 0%)
Feb 14 11:09:06 web-tp6-linux tp6_backup.sh[2255]: updating: srv/backup/nextcloud_20230214/statsd.d/ (stored 0%)
Feb 14 11:09:06 web-tp6-linux tp6_backup.sh[2255]: updating: srv/backup/nextcloud_20230214/.install-type (deflated 10%)
Feb 14 11:09:06 web-tp6-linux tp6_backup.sh[2255]: updating: srv/backup/nextcloud_20230214/edit-config (deflated 74%)
Feb 14 11:09:06 web-tp6-linux tp6_backup.sh[2255]: updating: srv/backup/nextcloud_20230214/health_alarm_notify.conf (deflated 42%)
Feb 14 11:09:06 web-tp6-linux tp6_backup.sh[2255]: updating: srv/backup/nextcloud_20230214/netdata.conf (deflated 45%)
Feb 14 11:09:06 web-tp6-linux systemd[1]: backup.service: Deactivated successfully.
Feb 14 11:09:06 web-tp6-linux systemd[1]: Finished lancement du systeme de backup.

```

🌞 **Créez un *timer*** système qui lance le *service* à intervalles réguliers

```
[logards@web-tp6-linux /]$ sudo vim /etc/systemd/system/backup.timer
[sudo] password for logards: 
[logards@web-tp6-linux /]$ cat /etc/systemd/system/backup.timer
[Unit]
Description=Run backup.service 

[Timer]
OnCalendar=*-*-* 4:00:00

[Install]
WantedBy=timers.target
```

🌞 Activez l'utilisation du *timer*

```
[logards@web-tp6-linux /]$ sudo systemctl daemon-reload
[logards@web-tp6-linux /]$ sudo systemctl start backup.timer
[logards@web-tp6-linux /]$ sudo systemctl enable backup.timer
Created symlink /etc/systemd/system/timers.target.wants/backup.timer → /etc/systemd/system/backup.timer.
[logards@web-tp6-linux /]$ sudo systemctl status backup.timer
● backup.timer - Run backup.service
     Loaded: loaded (/etc/systemd/system/backup.timer; enabled; vendor preset: disabled)
     Active: active (waiting) since Tue 2023-02-14 11:31:54 CET; 13s ago
      Until: Tue 2023-02-14 11:31:54 CET; 13s ago
    Trigger: Wed 2023-02-15 04:00:00 CET; 16h left
   Triggers: ● backup.service

Feb 14 11:31:54 web-tp6-linux systemd[1]: Started Run backup.service.
[logards@web-tp6-linux /]$ sudo systemctl list-timers | grep backup
Wed 2023-02-15 04:00:00 CET 16h left      n/a                         n/a       backup.timer                 backup.service
```

## II. NFS

🌞 **Préparer un dossier à partager sur le réseau** (sur la machine `storage.tp6.linux`)


```
[logards@storage-tp6-linux ~]$ sudo mkdir /srv/nfs_shares
[sudo] password for logards: 
[logards@storage-tp6-linux ~]$ sudo mkdir /srv/nfs_shares/web.tp6.linux
```

🌞 **Installer le serveur NFS** (sur la machine `storage.tp6.linux`)

```
[logards@storage-tp6-linux ~]$ sudo touch /etc/exports
[logards@storage-tp6-linux ~]$ sudo vim /etc/exports
[logards@storage-tp6-linux ~]$ cat /etc/exports
/srv/nfs_shares/web.tp6.linux    10.105.1.11(rw,sync,no_subtree_check)
[logards@storage-tp6-linux ~]$ sudo systemctl enable nfs-server
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
[logards@storage-tp6-linux ~]$ sudo systemctl start nfs-server
[logards@storage-tp6-linux ~]$ sudo systemctl status nfs-server
● nfs-server.service - NFS server and services
     Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; vendor preset: disabled)
    Drop-In: /run/systemd/generator/nfs-server.service.d
             └─order-with-mounts.conf
     Active: active (exited) since Tue 2023-02-14 12:17:51 CET; 17s ago
    Process: 44901 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
    Process: 44902 ExecStart=/usr/sbin/rpc.nfsd (code=exited, status=0/SUCCESS)
    Process: 44920 ExecStart=/bin/sh -c if systemctl -q is-active gssproxy; then systemctl reload gssproxy ; fi (code=exited, status=0/SUCCESS)
   Main PID: 44920 (code=exited, status=0/SUCCESS)
        CPU: 43ms

Feb 14 12:17:51 storage-tp6-linux systemd[1]: Starting NFS server and services...
Feb 14 12:17:51 storage-tp6-linux systemd[1]: Finished NFS server and services.
[logards@storage-tp6-linux ~]$ sudo firewall-cmd --permanent --add-service=nfs
success
[logards@storage-tp6-linux ~]$ sudo firewall-cmd --permanent --add-service=mountd
success
[logards@storage-tp6-linux ~]$ sudo firewall-cmd --permanent --add-service=rpc-bind
success
[logards@storage-tp6-linux ~]$ sudo firewall-cmd --reload
success
[logards@storage-tp6-linux ~]$ sudo firewall-cmd --permanent --list-all | grep services
  services: cockpit dhcpv6-client mountd nfs rpc-bind ssh
```

### 2. Client NFS

🌞 **Installer un client NFS sur `web.tp6.linux`**

```
[logards@web-tp6-linux /]$ sudo mount 10.105.1.15:/srv/nfs_shares/web.tp6.linux /srv/backup/
[logards@web-tp6-linux /]$ df -h | grep backup
10.105.1.15:/srv/nfs_shares/web.tp6.linux  6.2G  1.3G  5.0G  21% /srv/backup
[logards@web-tp6-linux /]$ sudo vim /etc/fstab 
[logards@web-tp6-linux /]$ cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Thu Oct 13 09:03:39 2022
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/rl-root     /                       xfs     defaults        0 0
UUID=28b0b6f0-be9b-4740-b395-3d286c366458 /boot                   xfs     defaults        0 0
/dev/mapper/rl-swap     none                    swap    defaults        0 0

```

🌞 **Tester la restauration des données** sinon ça sert à rien :)

- livrez-moi la suite de commande que vous utiliseriez pour restaurer les données dans une version antérieure
```
[logards@web-tp6-linux ~]$ sudo rm -rf /var/www/tp5_nexcloud
[logards@web-tp6-linux ~]$ sudo unzip /srv/backup/nextcloud_20230214.zip /var/www
```

