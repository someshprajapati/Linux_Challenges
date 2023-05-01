### 1. Create a group called "admins"
> Results:
```
[root@centos-host ~]# groupadd admins
```

### 2.1 Create a user called "david" , change his login shell to "/bin/zsh" and set "D3vUd3raaw" password for this user.
### 2.2 Make user "david" a member of "admins" group. 
> Results:
```
[root@centos-host ~]# useradd -s /bin/zsh david

[root@centos-host ~]# echo "D3vUd3raaw" | passwd --stdin david
Changing password for user david.
passwd: all authentication tokens updated successfully.

[root@centos-host ~]# usermod -G admins david
```

### 3.1 Create a user called "natasha" , change her login shell to "/bin/zsh" and set "DwfawUd113" password for this user.
### 3.2 Make user "natasha" a member of "admins" group.
> Results:
```
[root@centos-host ~]# useradd -s /bin/zsh natasha

[root@centos-host ~]# echo "DwfawUd113" | passwd --stdin natasha
Changing password for user natasha.
passwd: all authentication tokens updated successfully.

[root@centos-host ~]# usermod -G admins natasha
```

### 4. Create a group called "devs" 
> Results:
```
[root@centos-host ~]# groupadd devs
```

### 4.1 Create a user called "ray" , change his login shell to "/bin/sh" and set "D3vU3r321" password for this user.
### 4.2 Make user "ray" a member of "devs" group. 
> Results:
```
[root@centos-host ~]# useradd -s /bin/sh ray

[root@centos-host ~]# echo "D3vU3r321" | passwd --stdin ray
Changing password for user ray.
passwd: all authentication tokens updated successfully.

[root@centos-host ~]# usermod -G devs ray
```

### 5.1 Create a user called "lisa", change her login shell to "/bin/sh" and set "D3vUd3r123" password for this user.
### 5.2 Make user "lisa" a member of "devs" group. 
> Results:
```
[root@centos-host ~]# useradd -s /bin/sh lisa

[root@centos-host ~]# echo "D3vUd3r123" | passwd --stdin lisa
Changing password for user lisa.
passwd: all authentication tokens updated successfully.

[root@centos-host ~]# usermod -G devs lisa
```

### 6. Make sure "/data" directory is owned by user "bob" and group "devs" and "user/group" owner has "full" permissions but "other" should not have any permissions. 
> Results:
```
[root@centos-host ~]# chown bob:devs /data
[root@centos-host ~]# chmod 770 /data
```

### 7. Give some additional permissions to "admins" group on "/data" directory so that any user who is the member the "admins" group has "full permissions" on this directory. 
> Results:
```
[root@centos-host ~]# setfacl -m g:admins:rwx /data

[root@centos-host ~]# getfacl /data
getfacl: Removing leading '/' from absolute path names
# file: data
# owner: bob
# group: devs
user::rwx
group::rwx
group:admins:rwx
mask::rwx
other::---
```

### 8. Make sure all users under "admins" group can run all commands with "sudo" and without entering any password. 
> Results:
```
[root@centos-host ~]# echo '%admins ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
```

### 9. Make sure all users under "devs" group can only run the "dnf" command with "sudo" and without entering any password. 
> Results:
```
[root@centos-host ~]# echo '%devs ALL=(ALL) NOPASSWD:/usr/bin/dnf' >> /etc/sudoers
```

### 10. Configure a "resource limit" for the "devs" group so that this group (members of the group) can not run more than "30 processes" in their session. This should be both a "hard limit" and a "soft limit", written in a single line.
> Results:
```
[root@centos-host ~]# echo '@devs            -       nproc           30' >> /etc/security/limits.conf

[root@centos-host ~]# tail /etc/security/limits.conf
#*               soft    core            0
#*               hard    rss             10000
#@student        hard    nproc           20
#@faculty        soft    nproc           20
#@faculty        hard    nproc           50
#ftp             hard    nproc           0
#@student        -       maxlogins       4

# End of file
@devs            -       nproc           30
```

### 11. Edit the disk quota for the group called "devs". Limit the amount of storage space it can use (not inodes). Set a "soft" limit of "100MB" and a "hard" limit of "500MB" on "/data" partition.
> Results:
```
[root@centos-host ~]# setquota -g devs 100M 500M 0 0 /dev/vdb1

[root@centos-host ~]# quota -vs
Disk quotas for user root (uid 0): 
     Filesystem   space   quota   limit   grace   files   quota   limit   grace
      /dev/vdb1      0K      0K      0K               2       0       0   

[root@centos-host ~]# quota -g devs
Disk quotas for group devs (gid 1008): 
     Filesystem  blocks   quota   limit   grace   files   quota   limit   grace
      /dev/vdb1       0  102400  512000               1       0       0      

[root@centos-host ~]# repquota -a
*** Report for user quotas on device /dev/vdb1
Block grace time: 7days; Inode grace time: 7days
                        Block limits                File limits
User            used    soft    hard  grace    used  soft  hard  grace
----------------------------------------------------------------------
root      --       0       0       0              2     0     0       
bob       --       0       0       0              1     0     0 
```
