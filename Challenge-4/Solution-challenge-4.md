### Q. Some of our apps generate some raw data and store the same in /home/bob/preserved directory. We want to clean and manipulate some data and then want to create an archive of that data.

> NOTE: The validation will verify the final processed data so some of the tests might fail till all data is processed as asked.


### 1. Find the "hidden" files in "/home/bob/preserved" directory and copy them in "/opt/appdata/hidden/" directory (create the destination directory if doesn't exist).
> Results:
```
[root@centos-host preserved]# mkdir -p /opt/appdata/hidden

[root@centos-host preserved]# find /home/bob/preserved -type f -name ".*" -exec cp "{}" /opt/appdata/hidden/ \;
```

### 2. Find the "non-hidden" files in "/home/bob/preserved" directory and copy them in "/opt/appdata/files/" directory (create the destination directory if doesn't exist).


> Results:
```
[root@centos-host preserved]# mkdir -p /opt/appdata/files

[root@centos-host preserved]# find /home/bob/preserved -type f -not -name ".*" -exec cp "{}" /opt/appdata/files/ \;
```

### 3. Find and delete the files in "/opt/appdata" directory that contain a word ending with the letter "t" (case sensitive).
> Results:
```
[root@centos-host preserved]# rm -f $(find /opt/appdata/ -type f  -exec grep -l 't\>' "{}"  \; )
```

### 4. Change all the occurrences of the word "yes" to "no" in all files present under "/opt/appdata/" directory.
> Results:
```
[root@centos-host preserved]# find /opt/appdata -type f -name "*" -exec sed -i 's/\byes\b/no/g' "{}" \;
```

### 5. Change all the occurrences of the word "raw" to "processed" in all files present under "/opt/appdata/" directory. It must be a "case-insensitive" replacement, means all words must be replaced like "raw , Raw , RAW" etc. 
> Results:
```
[root@centos-host preserved]# find /opt/appdata -type f -name "*" -exec sed -i 's/\braw\b/processed/ig' "{}" \;
```

### 6. Create a "tar.gz" archive of "/opt/appdata" directory and save the archive to this file: "/opt/appdata.tar.gz". The "appdata.tar.gz" archive should have the final processed data.
> Results:
```
[root@centos-host preserved]# cd /opt

[root@centos-host opt]# tar -zcf appdata.tar.gz appdata

[root@centos-host opt]# ls
appdata  appdata.tar.gz  bucket.txt

```

### 7. Add the "sticky bit" special permission on "/opt/appdata" directory (keep the other permissions as it is).
> Results:
```
[root@centos-host opt]# ls -l
total 116
drwxr-xr-x 4 root    root               4096 May  1 10:51 appdata
-rw-r--r-- 1 root    root             108070 May  1 11:00 appdata.tar.gz
-rw-rw-r-- 1 gluster systemd-coredump     45 Mar 24 18:15 bucket.txt

[root@centos-host opt]# chmod +t /opt/appdata

[root@centos-host opt]# ls -l
total 116
drwxr-xr-t 4 root    root               4096 May  1 10:51 appdata
-rw-r--r-- 1 root    root             108070 May  1 11:00 appdata.tar.gz
-rw-rw-r-- 1 gluster systemd-coredump     45 Mar 24 18:15 bucket.txt

```

### 8. Make "bob" the "user" and the "group" owner of "/opt/appdata.tar.gz" file.
> Results:
```
[root@centos-host opt]# chown bob:bob /opt/appdata.tar.gz

[root@centos-host opt]# ls -l
total 116
drwxr-xr-t 4 root    root               4096 May  1 10:51 appdata
-rw-r--r-- 1 bob     bob              108070 May  1 11:00 appdata.tar.gz
-rw-rw-r-- 1 gluster systemd-coredump     45 Mar 24 18:15 bucket.txt
```

### 9. The "user/group" owner should have "read only" permissions on "/opt/appdata.tar.gz" file and "others" should not have any permissions.  
> Results:
```
[root@centos-host opt]# chmod 440 appdata.tar.gz

[root@centos-host opt]# ls -l
total 116
drwxr-xr-t 4 root    root               4096 May  1 10:51 appdata
-r--r----- 1 bob     bob              108070 May  1 11:00 appdata.tar.gz
-rw-rw-r-- 1 gluster systemd-coredump     45 Mar 24 18:15 bucket.txt
```

### 10. Create a "softlink" called "/home/bob/appdata.tar.gz" of "/opt/appdata.tar.gz" file.
> Results:
```
[root@centos-host opt]# ln -s /opt/appdata.tar.gz /home/bob/appdata.tar.gz

[root@centos-host opt]# ls -l /home/bob/appdata.tar.gz
lrwxrwxrwx 1 root root 19 May  1 11:07 /home/bob/appdata.tar.gz -> /opt/appdata.tar.gz
```

### 11. Create a script called "/home/bob/filter.sh". The script should filter the lines from "/opt/appdata.tar.gz" file which contain the word "processed", and save the filtered output in "/home/bob/filtered.txt" file. It must "overwrite" the existing contents of "/home/bob/filtered.txt" file  
> Results:
```
[root@centos-host opt]# cat <<'EOF' > /home/bob/filter.sh
> #!/bin/bash
> 
> tar -xzOf /opt/appdata.tar.gz | grep processed > /home/bob/filtered.txt
> EOF


[root@centos-host opt]# cat /home/bob/filter.sh
#!/bin/bash

tar -xzOf /opt/appdata.tar.gz | grep processed > /home/bob/filtered.txt

[root@centos-host opt]# chmod +x /home/bob/filter.sh

[root@centos-host opt]# ls -l /home/bob/filter.sh
-rwxr-xr-x 1 root root 85 May  1 11:10 /home/bob/filter.sh

[root@centos-host opt]# /home/bob/filter.sh

[root@centos-host opt]# head /home/bob/filtered.txt
My processed data
My processed data
My processed data
My processed data
My processed data
My processed data
My processed data
My processed data
My processed data
My processed data
```
