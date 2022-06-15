rsync 同步
1、在主上创建用户，并配置配置文件/etc/rsyncd.conf
创建用户：

```
useradd htmlBak
passwd htmlBak
```

`htmlBak` 输入两次

配置文件，/etc/rsyncd.conf：

```conf
# /etc/rsyncd: configuration file for rsync daemon mode
# See rsyncd.conf man page for more options.
# configuration example:

# uid = nobody
# gid = nobody
# use chroot = yes
# max connections = 4
# pid file = /var/run/rsyncd.pid
# exclude = lost+found/
# transfer logging = yes
# timeout = 900
# ignore nonreadable = yes
# dont compress   = *.gz *.tgz *.zip *.z *.Z *.rpm *.deb *.bz2

# [ftp]
#        path = /home/ftp
#        comment = ftp export area
uid = htmlBak
read only = true
use chroot = true
transfer logging = true
log format = %h %o %f %l %b
log file = /var/log/rsyncd.log
pid file = /var/run/rsyncd.pid
hosts allow = 192.168.6.186 192.168.6.102 192.168.11.5 192.168.15.21 192.168.12.113 192.168.15.64
slp refresh = 300

[htmlBak]
path = /data/html
comment = htmlBak
auth users = htmlBak
secrets file = /etc/rsyncd.secrets

[nginxBak]
path = /var/log/nginx/
comment = nginxBak
auth users = htmlBak
secrets file = /etc/rsyncd.secrets
```

配置密码文件，/etc/rsyncd.secrets

```
htmlBak:htmlBak
```

在从上，配置密码文件，直接同步即可：
1、配置密码文件，/etc/htmlbak.rsync

```
htmlBak
```

2、直接同步

```
rsync -vzrt --progress --delete htmlBak@192.168.6.113::htmlBak --password-file=/etc/htmlbak.rsync /data/html
```

注意点：
1、密码文件的权限必须是 600
2、所同步的文件必须有读权限
