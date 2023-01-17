# Partie 1 : Partitionnement du serveur de stockage

🌞 **Partitionner le disque à l'aide de LVM**

- créer un *physical volume (PV)* : le nouveau disque ajouté à la VM
```
[logards@Thorfinn ~]$ sudo pvcreate /dev/sdb
[sudo] password for logards: 
  Physical volume "/dev/sdb" successfully created.

[logards@Thorfinn ~]$ sudo pvs
  Devices file sys_wwid t10.ATA_____VBOX_HARDDISK___________________________VBca23b75a-a6690a42_ PVID QFbBETJHpc4Xue81gwpxryKKnUe9pYMR last seen on /dev/sda2 not found.
  PV         VG Fmt  Attr PSize PFree
  /dev/sdb      lvm2 ---  2.00g 2.00g
```
- créer un nouveau *volume group (VG)*
```
[logards@Thorfinn ~]$ sudo vgcreate storage /dev/sdb
  Volume group "storage" successfully created
[logards@Thorfinn ~]$ sudo vgs
  Devices file sys_wwid t10.ATA_____VBOX_HARDDISK___________________________VBca23b75a-a6690a42_ PVID QFbBETJHpc4Xue81gwpxryKKnUe9pYMR last seen on /dev/sda2 not found.
  VG      #PV #LV #SN Attr   VSize  VFree 
  storage   1   0   0 wz--n- <2.00g <2.00g
```
- créer un nouveau *logical volume (LV)* : ce sera la partition utilisable
```
[logards@Thorfinn ~]$ sudo lvcreate -l 100%FREE storage -n storage
  Logical volume "storage" created.
[logards@Thorfinn ~]$ sudo lvs
  Devices file sys_wwid t10.ATA_____VBOX_HARDDISK___________________________VBca23b75a-a6690a42_ PVID QFbBETJHpc4Xue81gwpxryKKnUe9pYMR last seen on /dev/sda2 not found.
  LV      VG      Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  storage storage -wi-a----- <2.00g  
```

🌞 **Formater la partition**

- vous formaterez la partition en ext4 (avec une commande `mkfs`)
```
[logards@Thorfinn ~]$ mkfs  -t ext4 /dev/storage/storage
mke2fs 1.46.5 (30-Dec-2021)
mkfs.ext4: Permission denied while trying to determine filesystem size
[logards@Thorfinn ~]$ sudo !!
sudo mkfs  -t ext4 /dev/storage/storage
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 523264 4k blocks and 130816 inodes
Filesystem UUID: 23c50bd7-e3fc-49c5-8c8d-48056478a75e
Superblock backups stored on blocks: 
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done 
```

🌞 **Monter la partition**

- montage de la partition (avec la commande `mount`)
```
[logards@Thorfinn /]$ cd mnt/
[logards@Thorfinn mnt]$ ls
[logards@Thorfinn mnt]$ sudo mkdir storage
[logards@Thorfinn mnt]$ ls
storage
[logards@Thorfinn mnt]$ sudo mount /dev/storage/storage /mnt/storage/
[logards@Thorfinn mnt]$ df -h | grep storage
/dev/mapper/storage-storage  2.0G   24K  1.9G   1% /mnt/storage

[logards@Thorfinn storage]$ sudo vim hello
[logards@Thorfinn storage]$ cat hello
Hello little red cheat

```
- définir un montage automatique de la partition (fichier `/etc/fstab`)
```
[logards@Thorfinn ~]$ sudo vim /etc/fstab 
[logards@Thorfinn ~]$ cat /etc/fstab 

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
/dev/storage/storage /mnt/storage/ ext4 defaults 0 0

[logards@Thorfinn ~]$ 

[logards@Thorfinn ~]$ sudo umount /mnt/storage/
[logards@Thorfinn ~]$ sudo mount -av
/                        : ignored
/boot                    : already mounted
none                     : ignored
mount: /mnt/storage does not contain SELinux labels.
       You just mounted a file system that supports labels which does not
       contain labels, onto an SELinux box. It is likely that confined
       applications will generate AVC messages and not be allowed access to
       this file system.  For more details see restorecon(8) and mount(8).
/mnt/storage/            : successfully mounted
[logards@Thorfinn ~]$ ls /mnt/
storage
[logards@Thorfinn ~]$ ls /mnt/storage/
hello  lost+found
```

# Partie 2 : Serveur de partage de fichiers

🌞 **Donnez les commandes réalisées sur le serveur NFS `storage.tp4.linux`**

- contenu du fichier `/etc/exports` dans le compte-rendu notamment
```
[logards@Thorfinn ~]$ sudo dnf install nfs-utils
[sudo] password for logards: 
Last metadata expiration check: 2:16:36 ago on Tue 10 Jan 2023 02:50:08 PM CET.

[logards@Thorfinn storage]$ sudo !!
sudo mkdir site_web_1
[sudo] password for logards: 
[logards@Thorfinn storage]$ sudo mkdir site_web_2
[logards@Thorfinn storage]$ sudo mkdir /var/nfs/general -p
[logards@Thorfinn storage]$ sudo chown nobody /var/nfs/general
[logards@Thorfinn storage]$ sudo vim /etc/exports
[logards@Thorfinn storage]$ cat /etc/exports
/mnt/storage/site_web_1               10.5.1.7(rw,sync,no_root_squash,no_subtree_check)
/mnt/storage/site_web_2               10.5.1.7(rw,sync,no_root_squash,no_subtree_check)
[logards@Thorfinn storage]$ sudo systemctl enable nfs-server
[sudo] password for logards: 
Created symlink /etc/systemd/system/multi-user.target.wants/nfs-server.service → /usr/lib/systemd/system/nfs-server.service.
[logards@Thorfinn storage]$ sudo systemctl start nfs-server
[logards@Thorfinn storage]$ sudo systemctl status nfs-server
● nfs-server.service - NFS server and services
     Loaded: loaded (/usr/lib/systemd/system/nfs-server.service; enabled; vendor preset: disabled)
    Drop-In: /run/systemd/generator/nfs-server.service.d
             └─order-with-mounts.conf
     Active: active (exited) since Tue 2023-01-10 17:26:34 CET; 14s ago
    Process: 11334 ExecStartPre=/usr/sbin/exportfs -r (code=exited, status=0/SUCCESS)
    Process: 11335 ExecStart=/usr/sbin/rpc.nfsd (code=exited, status=0/SUCCESS)
    Process: 11353 ExecStart=/bin/sh -c if systemctl -q is-active gssproxy; then systemctl reload gssproxy ; fi (code=exited, status=0/SUCCESS)
   Main PID: 11353 (code=exited, status=0/SUCCESS)
        CPU: 44ms

Jan 10 17:26:34 Thorfinn.lab.ingesup systemd[1]: Starting NFS server and services...
Jan 10 17:26:34 Thorfinn.lab.ingesup systemd[1]: Finished NFS server and services.
[logards@Thorfinn storage]$ sudo !!
sudo firewall-cmd --permanent --list-all | grep services
  services: cockpit dhcpv6-client
[logards@Thorfinn storage]$ sudo firewall-cmd --permanent --add-service=nfs
success
[logards@Thorfinn storage]$ sudo firewall-cmd --permanent --add-service=mountd
success
[logards@Thorfinn storage]$ sudo firewall-cmd --permanent --add-service=rpc-bind
success
[logards@Thorfinn storage]$ sudo firewall-cmd --reload
success
[logards@Thorfinn storage]$ sudo firewall-cmd --permanent --list-all | grep services
  services: cockpit dhcpv6-client mountd nfs rpc-bind
```

🌞 **Donnez les commandes réalisées sur le client NFS `web.tp4.linux`**
```
[logards@Askeladd ~]$ sudo dnf install nfs-utils -y
Last metadata expiration check: 3:08:06 ago on Tue 10 Jan 2023 02:50:08 PM CET.
Dependencies resolved.
[logards@Askeladd ~]$ sudo mkdir -p /nfs/site_web_1
[sudo] password for logards: 
[logards@Askeladd ~]$ sudo mkdir -p /nfs/site_web_2
[logards@Askeladd ~]$ sudo mount 10.5.1.6:/mnt/storage/site_web_1 /nfs/site_web_1
[logards@Askeladd ~]$ sudo mount 10.5.1.6:/mnt/storage/site_web_2 /nfs/site_web_2
[logards@Askeladd ~]$ df -h
Filesystem                        Size  Used Avail Use% Mounted on
devtmpfs                          462M     0  462M   0% /dev
tmpfs                             481M     0  481M   0% /dev/shm
tmpfs                             193M  3.0M  190M   2% /run
/dev/mapper/rl-root               6.2G  1.2G  5.1G  19% /
/dev/sda1                        1014M  210M  805M  21% /boot
tmpfs                              97M     0   97M   0% /run/user/1000
10.5.1.6:/mnt/storage/site_web_1  2.0G     0  1.9G   0% /nfs/site_web_1
10.5.1.6:/mnt/storage/site_web_2  2.0G     0  1.9G   0% /nfs/site_web_2
[logards@Askeladd ~]$ sudo vim  /etc/fstab
[sudo] password for logards: 
[logards@Askeladd ~]$ sudo cat /etc/fstab

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
10.5.1.6:/mnt/storage/site_web_1    /nfs/site_web_1   nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
10.5.1.6:/mnt/storage/site_web_2               /nfs/site_web_2      nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
```

# Partie 3 : Serveur web

## 1. Intro NGINX

## 2. Install

🖥️ **VM web.tp4.linux**

🌞 **Installez NGINX**

```
[bastien@UWU ~]$ ssh logards@10.5.1.6
ssh: connect to host 10.5.1.6 port 22: No route to host
[bastien@UWU ~]$ ssh logards@10.5.1.7
logards@10.5.1.7's password: 
Last login: Mon Jan 16 14:07:34 2023
[logards@Askeladd ~]$ sudo dnf install nginx
[sudo] password for logards: 
Rocky Linux 9 - BaseOS                                                                                                                                                              7.8 kB/s | 3.6 kB     00:00    
Rocky Linux 9 - AppStream                                                                                                                                                           6.5 kB/s | 4.1 kB     00:00    
Rocky Linux 9 - AppStream                                                                                                                                                           5.7 MB/s | 6.4 MB     00:01    
Rocky Linux 9 - Extras                                                                                                                                                              5.0 kB/s | 2.9 kB     00:00    
Package nginx-1:1.20.1-13.el9.x86_64 is already installed.
Dependencies resolved.
Nothing to do.
Complete!
```

## 3. Analyse

Avant de config des truks 2 ouf étou, on va lancer à l'aveugle et inspecter ce qu'il se passe, inspecter avec les outils qu'on connaît ce que fait NGINX à notre OS.

Commencez donc par démarrer le service NGINX :

```bash
$ sudo systemctl start nginx
$ sudo systemctl status nginx
```

🌞 **Analysez le service NGINX**


```
[logards@Askeladd ~]$ sudo ss -alutnpH | grep nginx
tcp LISTEN 0      511      0.0.0.0:80 0.0.0.0:* users:(("nginx",pid=844,fd=6),("nginx",pid=842,fd=6))
```
```
[logards@Askeladd ~]$ cat /etc/nginx/nginx.conf | grep root
#        root         /usr/share/nginx/html;
```
```
[logards@Askeladd ~]$ ls -l /usr/share/nginx/html
total 12
-rw-r--r--. 1 root root 3332 Oct 31 16:35 404.html
-rw-r--r--. 1 root root 3404 Oct 31 16:35 50x.html
drwxr-xr-x. 2 root root   27 Dec  9 17:36 icons
lrwxrwxrwx. 1 root root   25 Oct 31 16:37 index.html -> ../../testpage/index.html
-rw-r--r--. 1 root root  368 Oct 31 16:35 nginx-logo.png
lrwxrwxrwx. 1 root root   14 Oct 31 16:37 poweredby.png -> nginx-logo.png
lrwxrwxrwx. 1 root root   37 Oct 31 16:37 system_noindex_logo.png -> ../../pixmaps/system-noindex-logo.png
```
```
[logards@Askeladd ~]$ ps -ef | grep nginx
root         842       1  0 14:07 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx        844     842  0 14:07 ?        00:00:00 nginx: worker process
logards     1121     927  0 14:46 pts/0    00:00:00 grep --color=auto nginx
```
## 4. Visite du service web

**Et ça serait bien d'accéder au service non ?** Genre c'est un serveur web. On veut voir un site web !

🌞 **Configurez le firewall pour autoriser le trafic vers le service NGINX**

- vous avez reperé avec `ss` dans la partie d'avant le port à ouvrir
```
[logards@Askeladd log]$ sudo firewall-cmd --add-port 80/tcp
success
[logards@Askeladd log]$ sudo firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 
  services: cockpit dhcpv6-client
  ports: 22/tcp 10897/tcp 23125/tcp 80/tcp
  protocols: 
  forward: yes
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 

```

🌞 **Accéder au site web**

```
[logards@Askeladd log]$ curl http://10.5.1.7:80
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
      /*<![CDATA[*/
      
      html {
        height: 100%;
        width: 100%;
      }  
        body {
  background: rgb(20,72,50);
  background: -moz-linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%)  ;
  background: -webkit-linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%) ;
  background: linear-gradient(180deg, rgba(23,43,70,1) 30%, rgba(0,0,0,1) 90%);
  background-repeat: no-repeat;
  background-attachment: fixed;
  filter: progid:DXImageTransform.Microsoft.gradient(startColorstr="#3c6eb4",endColorstr="#3c95b4",GradientType=1); 
        color: white;
        font-size: 0.9em;
        font-weight: 400;
        font-family: 'Montserrat', sans-serif;
        margin: 0;
        padding: 10em 6em 10em 6em;
        box-sizing: border-box;      
        
      }

   
  h1 {
    text-align: center;
    margin: 0;
    padding: 0.6em 2em 0.4em;
    color: #fff;
    font-weight: bold;
    font-family: 'Montserrat', sans-serif;
    font-size: 2em;
  }
  h1 strong {
    font-weight: bolder;
    font-family: 'Montserrat', sans-serif;
  }
  h2 {
    font-size: 1.5em;
    font-weight:bold;
  }
  
  .title {
    border: 1px solid black;
    font-weight: bold;
    position: relative;
    float: right;
    width: 150px;
    text-align: center;
    padding: 10px 0 10px 0;
    margin-top: 0;
  }
  
  .description {
    padding: 45px 10px 5px 10px;
    clear: right;
    padding: 15px;
  }
  
  .section {
    padding-left: 3%;
   margin-bottom: 10px;
  }
  
  img {
  
    padding: 2px;
    margin: 2px;
  }
  a:hover img {
    padding: 2px;
    margin: 2px;
  }

  :link {
    color: rgb(199, 252, 77);
    text-shadow:
  }
  :visited {
    color: rgb(122, 206, 255);
  }
  a:hover {
    color: rgb(16, 44, 122);
  }
  .row {
    width: 100%;
    padding: 0 10px 0 10px;
  }
  
  footer {
    padding-top: 6em;
    margin-bottom: 6em;
    text-align: center;
    font-size: xx-small;
    overflow:hidden;
    clear: both;
  }
 
  .summary {
    font-size: 140%;
    text-align: center;
  }

  #rocky-poweredby img {
    margin-left: -10px;
  }

  #logos img {
    vertical-align: top;
  }
  
  /* Desktop  View Options */
 
  @media (min-width: 768px)  {
  
    body {
      padding: 10em 20% !important;
    }
    
    .col-md-1, .col-md-2, .col-md-3, .col-md-4, .col-md-5, .col-md-6,
    .col-md-7, .col-md-8, .col-md-9, .col-md-10, .col-md-11, .col-md-12 {
      float: left;
    }
  
    .col-md-1 {
      width: 8.33%;
    }
    .col-md-2 {
      width: 16.66%;
    }
    .col-md-3 {
      width: 25%;
    }
    .col-md-4 {
      width: 33%;
    }
    .col-md-5 {
      width: 41.66%;
    }
    .col-md-6 {
      border-left:3px ;
      width: 50%;
      

    }
    .col-md-7 {
      width: 58.33%;
    }
    .col-md-8 {
      width: 66.66%;
    }
    .col-md-9 {
      width: 74.99%;
    }
    .col-md-10 {
      width: 83.33%;
    }
    .col-md-11 {
      width: 91.66%;
    }
    .col-md-12 {
      width: 100%;
    }
  }
  
  /* Mobile View Options */
  @media (max-width: 767px) {
    .col-sm-1, .col-sm-2, .col-sm-3, .col-sm-4, .col-sm-5, .col-sm-6,
    .col-sm-7, .col-sm-8, .col-sm-9, .col-sm-10, .col-sm-11, .col-sm-12 {
      float: left;
    }
  
    .col-sm-1 {
      width: 8.33%;
    }
    .col-sm-2 {
      width: 16.66%;
    }
    .col-sm-3 {
      width: 25%;
    }
    .col-sm-4 {
      width: 33%;
    }
    .col-sm-5 {
      width: 41.66%;
    }
    .col-sm-6 {
      width: 50%;
    }
    .col-sm-7 {
      width: 58.33%;
    }
    .col-sm-8 {
      width: 66.66%;
    }
    .col-sm-9 {
      width: 74.99%;
    }
    .col-sm-10 {
      width: 83.33%;
    }
    .col-sm-11 {
      width: 91.66%;
    }
    .col-sm-12 {
      width: 100%;
    }
    h1 {
      padding: 0 !important;
    }
  }
        
  
  </style>
  </head>
  <body>
    <h1>HTTP Server <strong>Test Page</strong></h1>

    <div class='row'>
    
      <div class='col-sm-12 col-md-6 col-md-6 '></div>
          <p class="summary">This page is used to test the proper operation of
            an HTTP server after it has been installed on a Rocky Linux system.
            If you can read this page, it means that the software is working
            correctly.</p>
      </div>
      
      <div class='col-sm-12 col-md-6 col-md-6 col-md-offset-12'>
     
       
        <div class='section'>
          <h2>Just visiting?</h2>

          <p>This website you are visiting is either experiencing problems or
          could be going through maintenance.</p>

          <p>If you would like the let the administrators of this website know
          that you've seen this page instead of the page you've expected, you
          should send them an email. In general, mail sent to the name
          "webmaster" and directed to the website's domain should reach the
          appropriate person.</p>

          <p>The most common email address to send to is:
          <strong>"webmaster@example.com"</strong></p>
    
          <h2>Note:</h2>
          <p>The Rocky Linux distribution is a stable and reproduceable platform
          based on the sources of Red Hat Enterprise Linux (RHEL). With this in
          mind, please understand that:

        <ul>
          <li>Neither the <strong>Rocky Linux Project</strong> nor the
          <strong>Rocky Enterprise Software Foundation</strong> have anything to
          do with this website or its content.</li>
          <li>The Rocky Linux Project nor the <strong>RESF</strong> have
          "hacked" this webserver: This test page is included with the
          distribution.</li>
        </ul>
        <p>For more information about Rocky Linux, please visit the
          <a href="https://rockylinux.org/"><strong>Rocky Linux
          website</strong></a>.
        </p>
        </div>
      </div>
      <div class='col-sm-12 col-md-6 col-md-6 col-md-offset-12'>
        <div class='section'>
         
          <h2>I am the admin, what do I do?</h2>

        <p>You may now add content to the webroot directory for your
        software.</p>

        <p><strong>For systems using the
        <a href="https://httpd.apache.org/">Apache Webserver</strong></a>:
        You can add content to the directory <code>/var/www/html/</code>.
        Until you do so, people visiting your website will see this page. If
        you would like this page to not be shown, follow the instructions in:
        <code>/etc/httpd/conf.d/welcome.conf</code>.</p>

        <p><strong>For systems using
        <a href="https://nginx.org">Nginx</strong></a>:
        You can add your content in a location of your
        choice and edit the <code>root</code> configuration directive
        in <code>/etc/nginx/nginx.conf</code>.</p>
        
        <div id="logos">
          <a href="https://rockylinux.org/" id="rocky-poweredby"><img src="icons/poweredby.png" alt="[ Powered by Rocky Linux ]" /></a> <!-- Rocky -->
          <img src="poweredby.png" /> <!-- webserver -->
        </div>       
      </div>
      </div>
      
      <footer class="col-sm-12">
      <a href="https://apache.org">Apache&trade;</a> is a registered trademark of <a href="https://apache.org">the Apache Software Foundation</a> in the United States and/or other countries.<br />
      <a href="https://nginx.org">NGINX&trade;</a> is a registered trademark of <a href="https://">F5 Networks, Inc.</a>.
      </footer>
      
  </body>
</html>
```

🌞 **Vérifier les logs d'accès**

```
[logards@Askeladd log]$ sudo cat /var/log/nginx/access.log | tail -n 3
10.5.1.1 - - [16/Jan/2023:15:16:49 +0100] "GET /icons/poweredby.png HTTP/1.1" 200 15443 "http://10.5.1.7/" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36" "-"
10.5.1.1 - - [16/Jan/2023:15:16:49 +0100] "GET /poweredby.png HTTP/1.1" 200 368 "http://10.5.1.7/" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36" "-"
10.5.1.1 - - [16/Jan/2023:15:16:49 +0100] "GET /favicon.ico HTTP/1.1" 404 3332 "http://10.5.1.7/" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36" "-"
```

## 5. Modif de la conf du serveur web

🌞 **Changer le port d'écoute**


```
[logards@Askeladd log]$ sudo vim /etc/nginx/nginx.conf
[logards@Askeladd log]$ sudo systemctl restart nginx
[logards@Askeladd log]$ systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
     Active: active (running) since Mon 2023-01-16 15:25:41 CET; 1min 59s ago
    Process: 1696 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 1697 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 1698 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 1699 (nginx)
      Tasks: 2 (limit: 5905)
     Memory: 1.9M
        CPU: 15ms
     CGroup: /system.slice/nginx.service
             ├─1699 "nginx: master process /usr/sbin/nginx"
             └─1700 "nginx: worker process"

Jan 16 15:25:41 Askeladd.lab.ingesup systemd[1]: nginx.service: Deactivated successfully.
Jan 16 15:25:41 Askeladd.lab.ingesup systemd[1]: Stopped The nginx HTTP and reverse proxy server.
Jan 16 15:25:41 Askeladd.lab.ingesup systemd[1]: Starting The nginx HTTP and reverse proxy server...
Jan 16 15:25:41 Askeladd.lab.ingesup nginx[1697]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Jan 16 15:25:41 Askeladd.lab.ingesup nginx[1697]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Jan 16 15:25:41 Askeladd.lab.ingesup systemd[1]: Started The nginx HTTP and reverse proxy server.
[logards@Askeladd log]$ sudo ss -alutnpH | grep nginx
tcp LISTEN 0      511      0.0.0.0:8080 0.0.0.0:* users:(("nginx",pid=1700,fd=6),("nginx",pid=1699,fd=6)) 
tcp LISTEN 0      511         [::]:8080    [::]:* users:(("nginx",pid=1700,fd=7),("nginx",pid=1699,fd=7)) 
[logards@Askeladd log]$ sudo firewall-cmd --remove-port 80/tcp
success
[logards@Askeladd log]$ sudo firewall-cmd --list-all
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
[logards@Askeladd log]$ curl http://10.5.1.7:8080 | tail
      </div>
      </div>
      
      <footer class="col-sm-12">
      <a href="https://apache.org">Apache&trade;</a> is a registered trademark of <a href="https://apache.org">the Apache Software Foundation</a> in the United States and/or other countries.<br />
      <a href="https://nginx.org">NGINX&trade;</a> is a registered trademark of <a href="https://">F5 Networks, Inc.</a>.
      </footer>
      
  </body>
</html>

```

🌞 **Changer l'utilisateur qui lance le service**

```
[logards@Askeladd ~]$ sudo cat /etc/nginx/nginx.conf | grep web
user web;
[logards@Askeladd ~]$ ps -ef | grep web
web         1677    1676  0 16:46 ?        00:00:00 nginx: worker process
logards     1681    1417  0 16:47 pts/0    00:00:00 grep --color=auto web
```

🌞 **Changer l'emplacement de la racine Web**

```
[logards@Askeladd nfs]$ cat /etc/nginx/nginx.conf | grep site_
        root         /var/www/site_web_1/index.html;
[bastien@UWU ~]$ curl http://10.5.1.7:8080
<h1>salut à tous</h1>
```

## 6. Deux sites web sur un seul serveur

🌞 **Repérez dans le fichier de conf**

```
[logards@Askeladd /]$ cat /etc/nginx/nginx.conf | grep /etc/nginx/conf.d/*
    # Load modular configuration files from the /etc/nginx/conf.d directory.
    include /etc/nginx/conf.d/*.conf;
```

🌞 **Créez le fichier de configuration pour le premier site**

```
[logards@Askeladd /]$ cat /etc/nginx/nginx.conf | grep server
[logards@Askeladd /]$ cat  /etc/nginx/conf.d/site_web_1.conf
    server {
        listen       8080;
        root         /var/www/site_web_1;
    }
}

```

🌞 **Créez le fichier de configuration pour le deuxième site**

```
[logards@Askeladd /]$ sudo vim /etc/nginx/conf.d/site_web_2.conf 
[logards@Askeladd /]$ cat /etc/nginx/conf.d/site_web_2.conf 
    server {
        listen       8888;
        root         /var/www/site_web_2;
    }
}
```


> N'oubliez pas d'ouvrir le port 8888 dans le firewall. Vous pouvez constater si vous le souhaitez avec un `ss` que NGINX écoute bien sur ce nouveau port.

🌞 **Prouvez que les deux sites sont disponibles**

```
[bastien@UWU ~]$ curl http://10.5.1.7:8080
<h1>salut à tous</h1>
[bastien@UWU ~]$ curl http://10.5.1.7:8888
<h1>salut à tous</h1>
```
