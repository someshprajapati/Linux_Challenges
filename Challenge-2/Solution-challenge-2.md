### Q. The app server called centos-host is running a Go app on the 8081 port. You have been asked to troubleshoot some issues with yum/dnf on this system, Install Nginx server, configure Nginx as a reverse proxy for this Go app, install firewalld package and then configure some firewall rules.


### 1. Install "nginx" and "firewalld" package. Troubleshoot the issues with "yum/dnf" and make sure you are able to install the packages on "centos-host".
> Results:
```
[bob@centos-host ~]# yum install nginx firewalld -y
CentOS Stream 8 - AppStream                                                                                                                          0.0  B/s |   0  B     00:00    
Errors during downloading metadata for repository 'appstream':
  - Curl error (6): Couldn't resolve host name for http://mirrorlist.centos.org/?release=8-stream&arch=x86_64&repo=AppStream&infra=stock [Could not resolve host: mirrorlist.centos.org]
Error: Failed to download metadata for repo 'appstream': Cannot prepare internal mirrorlist: Curl error (6): Couldn't resolve host name for http://mirrorlist.centos.org/?release=8-stream&arch=x86_64&repo=AppStream&infra=stock [Could not resolve host: mirrorlist.centos.org]

As above error, we are not able to resolve the hostname. Need to fix the resolv.conf

[bob@centos-host ~]# cat /etc/resolv.conf
search us-central1-a.c.kk-lab-prod.internal c.kk-lab-prod.internal google.internal
options ndots:0

[bob@centos-host ~]# vi /etc/resolv.conf 

Add the `nameserver 8.8.8.8` entry in /etc/resolv.conf file and try again to install the package.

[bob@centos-host ~]# cat /etc/resolv.conf
search us-central1-a.c.kk-lab-prod.internal c.kk-lab-prod.internal google.internal
options ndots:0
nameserver 8.8.8.8

[bob@centos-host ~]# yum install nginx firewalld -y
```

### 2.1. Start and Enable "firewalld" service.
> Results:
```
[bob@centos-host ~]# systemctl enable firewalld

[bob@centos-host ~]# systemctl start firewalld

[bob@centos-host ~]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) since Mon 2023-05-01 07:38:12 UTC; 57s ago
     Docs: man:firewalld(1)
 Main PID: 10893 (firewalld)
    Tasks: 2 (limit: 1340692)
   Memory: 32.1M
   CGroup: /system.slice/firewalld.service
           └─10893 /usr/libexec/platform-python -s /usr/sbin/firewalld --nofork --nopid
```

### 2.2. Add firewall rules to allow only incoming port "22", "80" and "8081".
> Results:
```
[bob@centos-host ~]# firewall-cmd --zone=public --add-port=80/tcp --permanent
success

[bob@centos-host ~]# firewall-cmd --zone=public --add-port=22/tcp --permanent
success

[bob@centos-host ~]# firewall-cmd --zone=public --add-port=8081/tcp --permanent
success
```

### 2.3. The firewall rules must be permanent and effective immediately.
> Results:
```
[bob@centos-host ~]# firewall-cmd --reload
success

[bob@centos-host ~]# firewall-cmd --list-all
public
  target: default
  icmp-block-inversion: no
  interfaces: 
  sources: 
  services: cockpit dhcpv6-client ssh
  ports: 80/tcp 22/tcp 8081/tcp
  protocols: 
  forward: no
  masquerade: no
  forward-ports: 
  source-ports: 
  icmp-blocks: 
  rich rules: 
```


### 3. Start GoApp by running the "nohup go run main.go &" command from "/home/bob/go-app/" directory, it can take few seconds to start.
> Results:
```
[bob@centos-host ~]# cd /home/bob/go-app/

[bob@centos-host go-app]# ls -l
total 43304
-rw-r--r-- 1 bob bob     1194 May  1 07:24 CONTRIBUTING.md
-rw-r--r-- 1 bob bob      118 May  1 07:24 Dockerfile
-rw-r--r-- 1 bob bob     1068 May  1 07:24 LICENSE
-rw-r--r-- 1 bob bob     7376 May  1 07:24 README.md
-rw-r--r-- 1 bob bob      503 May  1 07:24 application.develop.yml
-rw-r--r-- 1 bob bob      482 May  1 07:24 application.docker.yml
-rw-r--r-- 1 bob bob      581 May  1 07:24 application.k8s.yml
drwxr-xr-x 2 bob bob     4096 May  1 07:24 config
drwxr-xr-x 2 bob bob     4096 May  1 07:24 container
drwxr-xr-x 2 bob bob     4096 May  1 07:24 controller
drwxr-xr-x 2 bob bob     4096 May  1 07:24 docs
-rwxr-xr-x 1 bob bob 44143008 May  1 07:24 go-webapp-sample
-rw-r--r-- 1 bob bob     3153 May  1 07:24 go.mod
-rw-r--r-- 1 bob bob    78153 May  1 07:24 go.sum
drwxr-xr-x 2 bob bob     4096 May  1 07:24 logger
-rw-r--r-- 1 bob bob     1432 May  1 07:24 main.go
drwxr-xr-x 2 bob bob     4096 May  1 07:24 middleware
drwxr-xr-x 2 bob bob     4096 May  1 07:24 migration
drwxr-xr-x 3 bob bob     4096 May  1 07:24 model
drwxr-xr-x 5 bob bob     4096 May  1 07:24 public
drwxr-xr-x 2 bob bob     4096 May  1 07:24 repository
drwxr-xr-x 2 bob bob     4096 May  1 07:24 router
drwxr-xr-x 2 bob bob     4096 May  1 07:24 service
drwxr-xr-x 2 bob bob     4096 May  1 07:24 session
drwxr-xr-x 2 bob bob     4096 May  1 07:24 test
drwxr-xr-x 2 bob bob     4096 May  1 07:24 util
-rw-r--r-- 1 bob bob      451 May  1 07:24 zaplogger.develop.yml
-rw-r--r-- 1 bob bob      496 May  1 07:24 zaplogger.docker.yml
-rw-r--r-- 1 bob bob      496 May  1 07:24 zaplogger.k8s.yml

[root@centos-host go-app]# nohup go run main.go &
[1] 9275
[root@centos-host go-app]# nohup: ignoring input and appending output to 'nohup.out'

[root@centos-host go-app]# ps -ef| grep 9275
root        9275    4570  0 08:40 pts/0    00:00:08 go run main.go
root       10931    9275  0 08:40 pts/0    00:00:00 /usr/bin/gcc -I /root/go/pkg/mod/github.com/mattn/go-sqlite3@v2.0.3+incompatible -fPIC -m64 -pthread -Wl,--no-gc-sections -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build4055533116/b214=/tmp/go-build -gno-record-gcc-switches -I /tmp/go-build4055533116/b214/ -g -O2 -std=gnu99 -DSQLITE_ENABLE_RTREE -DSQLITE_THREADSAFE=1 -DHAVE_USLEEP=1 -DSQLITE_ENABLE_FTS3 -DSQLITE_ENABLE_FTS3_PARENTHESIS -DSQLITE_ENABLE_FTS4_UNICODE61 -DSQLITE_TRACE_SIZE_LIMIT=15 -DSQLITE_OMIT_DEPRECATED -DSQLITE_DISABLE_INTRINSIC -DSQLITE_DEFAULT_WAL_SYNCHRONOUS=1 -DSQLITE_ENABLE_UPDATE_DELETE_LIMIT -Wno-deprecated-declarations -DHAVE_PREAD64=1 -DHAVE_PWRITE64=1 -I/root/go/pkg/mod/github.com/mattn/go-sqlite3@v2.0.3+incompatible -o /tmp/go-build4055533116/b214/_x011.o -c sqlite3-binding.c
root       10934    4570  0 08:40 pts/0    00:00:00 grep --color=auto 9275
```

### 4.1. Configure Nginx as a reverse proxy for the GoApp so that we can access the GoApp on port "80"
> Results:
```
[root@centos-host ~]# vi /etc/nginx/nginx.conf

# Configure Nginx as a reverse proxy for the GoApp so that we can access the GoApp on port "80"
# Do this by inserting a proxy_pass line after "location / {" at line 48
sed -i '48i\            proxy_pass  http://localhost:8081;' /etc/nginx/nginx.conf

[root@centos-host ~]# grep proxy /etc/nginx/nginx.conf
        proxy_pass  http://localhost:8081;

[bob@centos-host go-app]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

### 4.2. Start and Enable "nginx" service.
> Results:
```
[bob@centos-host go-app]# systemctl enable nginx
Created symlink /etc/systemd/system/multi-user.target.wants/nginx.service → /usr/lib/systemd/system/nginx.service.

[bob@centos-host go-app]# systemctl start nginx

[bob@centos-host go-app]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2023-05-01 07:52:13 UTC; 5s ago
  Process: 16209 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 16189 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 16182 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 16222 (nginx)
    Tasks: 37 (limit: 1340692)
   Memory: 55.0M
   CGroup: /system.slice/nginx.service
           ├─16222 nginx: master process /usr/sbin/nginx
           ├─16223 nginx: worker process
           ├─16224 nginx: worker process
           ├─16225 nginx: worker process
           ├─16226 nginx: worker process
           ├─16227 nginx: worker process
           ├─16228 nginx: worker process
           ├─16229 nginx: worker process
           ├─16230 nginx: worker process
           ├─16231 nginx: worker process
           ├─16232 nginx: worker process
           ├─16233 nginx: worker process
           ├─16234 nginx: worker process
           ├─16235 nginx: worker process


[root@centos-host go-app]# curl -u test:test http://localhost:80 
<!DOCTYPE html><html lang="en"><head><meta charset="utf-8"><meta http-equiv="X-UA-Compatible" content="IE=edge"><meta name="viewport" content="width=device-width,initial-scale=1"><!--[if IE]><link rel="icon" href="/favicon.ico"><![endif]--><title>vuejs-webapp-sample</title><link rel="stylesheet" href="//fonts.googleapis.com/css?family=Roboto:300,400,500,700,400italic"><link rel="stylesheet" href="//fonts.googleapis.com/icon?family=Material+Icons"><link href="/css/app.750b60b0.css" rel="preload" as="style"><link href="/css/chunk-vendors.533831d3.css" rel="preload" as="style"><link href="/js/app.dbc5a974.js" rel="preload" as="script"><link href="/js/chunk-vendors.0cedba66.js" rel="preload" as="script"><link href="/css/chunk-vendors.533831d3.css" rel="stylesheet"><link href="/css/app.750b60b0.css" rel="stylesheet"><link rel="icon" type="image/png" sizes="32x32" href="/img/icons/favicon-32x32.png"><link rel="icon" type="image/png" sizes="16x16" href="/img/icons/favicon-16x16.png"><link rel="manifest" href="/manifest.json"><meta name="theme-color" content="#4DBA87"><meta name="apple-mobile-web-app-capable" content="no"><meta name="apple-mobile-web-app-status-bar-style" content="default"><meta name="apple-mobile-web-app-title" content="vuejs-webapp-sample"><link rel="apple-touch-icon" href="/img/icons/apple-touch-icon-152x152.png"><link rel="mask-icon" href="/img/icons/safari-pinned-tab.svg" color="#4DBA87"><meta name="msapplication-TileImage" content="/img/icons/msapplication-icon-144x144.png"><meta name="msapplication-TileColor" content="#000000"></head><body><noscript><strong>We're sorry but vuejs-webapp-sample doesn't work properly without JavaScript enabled. Please enable it to continue.</strong></noscript><div id="app"></div><script src="/js/chunk-vendors.0cedba66.js"></script><script src="/js/app.dbc5a974.js"></script></body></html>[root@centos-host go-app]# 
```
