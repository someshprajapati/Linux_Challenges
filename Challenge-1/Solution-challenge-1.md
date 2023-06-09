### Q. The database server called centos-host is running short on space! You have been asked to add an LVM volume for the Database team using some of the existing disks on this server.


### 1. Create a group called "dba_users" and add the user called 'bob' to this group
> Results:
```
[bob@centos-host ~]$ sudo groupadd dba_users
[bob@centos-host ~]$ sudo usermod -a -G dba_users bob
```

### 2. Install the correct packages that will allow the use of "lvm" on the centos machine.
> Results:
```
[bob@centos-host ~]$ sudo yum -y install lvm2

[bob@centos-host ~]$ sudo yum makecache --refresh
```


### 2.1. Check the available partition and disk
> Results:
```
[bob@centos-host ~]$ lsblk
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vda    253:0    0  11G  0 disk 
└─vda1 253:1    0  10G  0 part /
vdb    253:16   0   1G  0 disk 
vdc    253:32   0   1G  0 disk 
vdd    253:48   0   1G  0 disk 
vde    253:64   0   1G  0 disk 

[bob@centos-host ~]$ sudo fdisk -l
Disk /dev/vda: 11 GiB, 11811160064 bytes, 23068672 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xef431952

Device     Boot Start      End  Sectors Size Id Type
/dev/vda1  *     2048 20971519 20969472  10G 83 Linux

Disk /dev/vdb: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/vdc: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/vdd: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

Disk /dev/vde: 1 GiB, 1073741824 bytes, 2097152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

### 3. Create a Physical Volume for "/dev/vdb"
> Results:
```
[bob@centos-host ~]$ sudo pvcreate /dev/vdb
  Physical volume "/dev/vdb" successfully created.
```

### 4. Create a Physical Volume for "/dev/vdc"
> Results:
```
[bob@centos-host ~]$ sudo pvcreate /dev/vdc
  Physical volume "/dev/vdc" successfully created.
```

### 5. Create a volume group called "dba_storage" using the physical volumes "/dev/vdb" and "/dev/vdc"
> Results:
```
[bob@centos-host ~]$ sudo vgcreate dba_storage /dev/vdb /dev/vdc
  Volume group "dba_storage" successfully created
```

### 6. Create an "lvm" called "volume_1" from the volume group called "dba_storage". Make use of the entire space available in the volume
> Results:
```
[bob@centos-host ~]$ sudo lvcreate -n volume_1 -l 100%FREE dba_storage
  Logical volume "volume_1" created.
```

### 7.1. Format the lvm volume "volume_1" as an "XFS" filesystem
> Results:
```
[bob@centos-host ~]$ sudo mkfs.xfs /dev/dba_storage/volume_1 
meta-data=/dev/dba_storage/volume_1 isize=512    agcount=4, agsize=130560 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=522240, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

### 7.2. Mount the filesystem at the path "/mnt/dba_storage".
> Results:
```
[bob@centos-host ~]$ sudo mkdir /mnt/dba_storage
[bob@centos-host ~]$ sudo mount /dev/dba_storage/volume_1 /mnt/dba_storage
```

### 7.3. Make sure that this mount point is persistent across reboots with the correct default options.
> Results:
```
[bob@centos-host ~]$ sudo vi /etc/fstab 
[bob@centos-host ~]$ cat /etc/fstab 

#
# /etc/fstab
# Created by anaconda on Fri Dec  4 17:37:32 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=a62c5b49-755e-41b0-9d36-de3d95e17232 /                       xfs     defaults        0 0
/dev/mapper/dba_storage-volume_1         /mnt/dba_storage         xfs     defaults        0 0
/swapfile none swap defaults 0 0
#VAGRANT-BEGIN
# The contents below are automatically generated by Vagrant. Do not modify.
#VAGRANT-END
```

### 8.1. Ensure that the mountpoint "/mnt/dba_storage" has the group ownership set to the "dba_users" group
> Results:
```
[bob@centos-host ~]$ ls -l /mnt/
total 0
drwxr-xr-x. 2 root root 6 May  1 06:21 dba_storage

[bob@centos-host ~]$ sudo chown -R bob:dba_users /mnt/dba_storage/

[bob@centos-host ~]$ ls -l /mnt/
total 0
drwxr-xr-x. 2 bob dba_users 6 May  1 06:21 dba_storage
```

### 8.2. Ensure that the mount point "/mnt/dba_storage" has "read/write" and execute permissions for the owner and group and no permissions for anyone else.
> Results:
```
[bob@centos-host ~]$ sudo chmod 770 /mnt/dba_storage/

[bob@centos-host ~]$ ls -l /mnt/
total 0
drwxrwx---. 2 bob dba_users 6 May  1 06:21 dba_storage
```
