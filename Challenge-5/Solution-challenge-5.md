### Q. We got a couple of tasks that need to be done on centos-host server. Most of these tasks are dependent on each other but not all of them.

### 1. Add a local DNS entry for the database hostname "mydb.kodekloud.com" so that it can resolve to "10.0.0.50" IP address. 
> Results:
```
[root@centos-host ~]# echo "10.0.0.50    mydb.kodekloud.com" >> /etc/hosts

[root@centos-host ~]# ping mydb.kodekloud.com
PING mydb.kodekloud.com (10.0.0.50) 56(84) bytes of data.
```

### 2. Add an extra IP to "eth1" interface on this system: 10.0.0.50/24
> Results:
```
[root@centos-host ~]# ip a s eth1
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:8d:dc:f3 brd ff:ff:ff:ff:ff:ff
    inet 172.28.128.109/24 brd 172.28.128.255 scope global dynamic noprefixroute eth1
       valid_lft 2924sec preferred_lft 2924sec
    inet6 fe80::5054:ff:fe8d:dcf3/64 scope link 
       valid_lft forever preferred_lft forever

[root@centos-host ~]# ip address add 10.0.0.50/24 dev eth1

[root@centos-host ~]# ip a s eth1
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:8d:dc:f3 brd ff:ff:ff:ff:ff:ff
    inet 172.28.128.109/24 brd 172.28.128.255 scope global dynamic noprefixroute eth1
       valid_lft 2917sec preferred_lft 2917sec
    inet 10.0.0.50/24 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe8d:dcf3/64 scope link 
       valid_lft forever preferred_lft forever
```

### 3. Install "mariadb" database server on this server and "start/enable" its service.
> Results:
```
[root@centos-host ~]# yum install mariadb-server -y

[root@centos-host ~]# systemctl enable mariadb
Created symlink /etc/systemd/system/mysql.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/mysqld.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/multi-user.target.wants/mariadb.service → /usr/lib/systemd/system/mariadb.service.

[root@centos-host ~]# systemctl start mariadb

[root@centos-host ~]# systemctl status mariadb
● mariadb.service - MariaDB 10.3 database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2023-05-01 11:24:52 UTC; 6s ago
     Docs: man:mysqld(8)
           https://mariadb.com/kb/en/library/systemd/
  Process: 29423 ExecStartPost=/usr/libexec/mysql-check-upgrade (code=exited, status=0/SUCCESS)
  Process: 29288 ExecStartPre=/usr/libexec/mysql-prepare-db-dir mariadb.service (code=exited, status=0/SUCCESS)
  Process: 29264 ExecStartPre=/usr/libexec/mysql-check-socket (code=exited, status=0/SUCCESS)
 Main PID: 29391 (mysqld)
   Status: "Taking your SQL requests now..."
    Tasks: 30 (limit: 5970)
   Memory: 77.0M
   CGroup: /system.slice/mariadb.service
           └─29391 /usr/libexec/mysqld --basedir=/usr
```

### 4. Set a password for mysql root user to "S3cure#321"
> Results:
```
[root@centos-host ~]# mysqladmin -u root password 'S3cure#321'
```

### 5. The "root" account is currently locked on "centos-host", please unlock it. Make user "root" a member of "wheel" group
> Results:
```
[root@centos-host ~]# usermod -U root
[root@centos-host ~]# usermod -G wheel root
```

### 6. Pull "nginx" docker image.
> Results:
```
[root@centos-host ~]# docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
26c5c85e47da: Pull complete 
4f3256bdf66b: Pull complete 
2019c71d5655: Pull complete 
8c767bdbc9ae: Pull complete 
78e14bb05fd3: Pull complete 
75576236abf5: Pull complete 
Digest: sha256:63b44e8ddb83d5dd8020327c1f40436e37a6fffd3ef2498a6204df23be6e7e94
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest

[root@centos-host ~]# docker run -d -p 80:80 --name myapp nginx
120a33f94df75299a87df31ba22a79d8e053157555de1ccfd0b5a5f831a629bd

[root@centos-host ~]# docker ps -a
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                               NAMES
120a33f94df7   nginx     "/docker-entrypoint.…"   6 seconds ago   Up 5 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp   myapp
```

### 7. Create a bash script called "container-start.sh" under "/home/bob/" which should be able to "start" the "myapp" container. It should also display a message "myapp container started!"
> Results:
```
[root@centos-host ~]# cat <<EOF > /home/bob/container-start.sh
> #!/usr/bin/env bash
> 
> docker start myapp
> echo "myapp container started!"
> EOF


[root@centos-host ~]# cat /home/bob/container-start.sh
#!/usr/bin/env bash

docker start myapp
echo "myapp container started!"

[root@centos-host ~]# chmod +x /home/bob/container-start.sh
```


### 8. Create a bash script called "container-stop.sh" under "/home/bob/" which should be able to stop the "myapp" container. It should also display a message "myapp container stopped!" 
> Results:
```
[root@centos-host ~]# cat <<EOF > /home/bob/container-stop.sh
> #!/usr/bin/env bash
> 
> docker stop myapp
> echo "myapp container stopped!"
> EOF


[root@centos-host ~]# cat /home/bob/container-stop.sh
#!/usr/bin/env bash

docker stop myapp
echo "myapp container stopped!"

[root@centos-host ~]# chmod +x /home/bob/container-stop.sh
```


### 9. Add a cron job for the "root" user which should run "container-stop.sh" script at "12am" everyday.
> Results:
```
[root@centos-host ~]# crontab -l
no crontab for root

[root@centos-host ~]# (crontab -l 2>/dev/null; echo "0 0 * * * /home/bob/container-stop.sh") | crontab -

[root@centos-host ~]# crontab -l
0 0 * * * /home/bob/container-stop.sh
```

### 10. Add a cron job for the "root" user which should run "container-start.sh" script at "8am" everyday.  
> Results:
```
[root@centos-host ~]# (crontab -l 2>/dev/null; echo "0 8 * * * /home/bob/container-start.sh") | crontab -

[root@centos-host ~]# crontab -l
0 0 * * * /home/bob/container-stop.sh
0 8 * * * /home/bob/container-start.sh
```


### 11. Edit the PAM configuration file for the "su" utility so that this utility only accepts the requests from the users that are part of the "wheel" group and the requests from the users should be accepted immediately, without asking for any password.
> Results:
```
[root@centos-host ~]# cat /etc/pam.d/su
#%PAM-1.0
auth            required        pam_env.so
auth            sufficient      pam_rootok.so
# Uncomment the following line to implicitly trust users in the "wheel" group.
#auth           sufficient      pam_wheel.so trust use_uid
# Uncomment the following line to require a user to be in the "wheel" group.
#auth           required        pam_wheel.so use_uid
auth            substack        system-auth
auth            include         postlogin
account         sufficient      pam_succeed_if.so uid = 0 use_uid quiet
account         [success=1 default=ignore] \
                                pam_succeed_if.so user = vagrant use_uid quiet
account         required        pam_succeed_if.so user notin root:vagrant
account         include         system-auth
password        include         system-auth
session         include         system-auth
session         include         postlogin
session         optional        pam_xauth.so


[root@centos-host ~]# sed -i 's/#auth/auth/' /etc/pam.d/su


[root@centos-host ~]# cat /etc/pam.d/su
#%PAM-1.0
auth            required        pam_env.so
auth            sufficient      pam_rootok.so
# Uncomment the following line to implicitly trust users in the "wheel" group.
auth            sufficient      pam_wheel.so trust use_uid
# Uncomment the following line to require a user to be in the "wheel" group.
auth            required        pam_wheel.so use_uid
auth            substack        system-auth
auth            include         postlogin
account         sufficient      pam_succeed_if.so uid = 0 use_uid quiet
account         [success=1 default=ignore] \
                                pam_succeed_if.so user = vagrant use_uid quiet
account         required        pam_succeed_if.so user notin root:vagrant
account         include         system-auth
password        include         system-auth
session         include         system-auth
session         include         postlogin
session         optional        pam_xauth.so
[root@centos-host ~]# 
```
