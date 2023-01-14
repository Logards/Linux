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

