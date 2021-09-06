# history
保留操作记录
编辑/etc/profile（或者/root/.bashrc），添加：
```
USER_IP=`who -u am i 2>/dev/null| awk '{print $NF}'|sed -e 's/[()]//g'`  
export HISTTIMEFORMAT="[%F %T][`whoami`][${USER_IP}] " 
```

方案三：实现登陆过系统的用户、IP地址、操作命令以及操作时间一一对应
通过在/etc/profile里面加入以下代码就可以实现：
```
#history
PS1="[\u@\h \W]\$"
history
USER_IP=`who -u am i 2>/dev/null| awk '{print $NF}'|sed -e 's/[()]//g'`
export HISTTIMEFORMAT="[%F %T][`whoami`][${USER_IP}] " 
if [ "$USER_IP" = "" ]
then
USER_IP=`hostname`
fi
if [ ! -d /tmp/dbasky ]
then
mkdir /tmp/dbasky
chmod 777 /tmp/dbasky
fi
if [ ! -d /tmp/dbasky/${LOGNAME} ]
then
mkdir /tmp/dbasky/${LOGNAME}
chmod 300 /tmp/dbasky/${LOGNAME}
fi
export HISTSIZE=4096
DT=`date -d '0 day' +\%Y\%m\%d_\%H\%M\%S`
export HISTFILE="/tmp/dbasky/${LOGNAME}/dbasky@${USER_IP}_$DT"
chmod 600 /tmp/dbasky/${LOGNAME}/*dbasky* 2>/dev/null
```
 
其实通过上面的代码不难看出，在系统的/tmp新建个dbasky目录，在目录中记录了所有的登陆过系统的用户和IP地址。
 
实现后效果如下：
```
[root@XXXXXX nxuser]$ll
total 8
-rw-------  1 nxuser nxgroup 15 Sep  6 23:04 dbasky@122.234.54.174_20110907_230314
-rw-------  1 nxuser nxgroup 19 Sep  6 23:05 dbasky@122.234.54.174_20110907_230439
[root@ZJ-WAP-SNMP nxuser]$more dbasky@122.234.54.174_20110907_230439
ls
ll
free -m
exit
[root@XXXXXXX nxuser]$pwd
/tmp/dbasky/nxuser
[root@XXXXXXX nxuser]$
```

但是方案三配置后，使用history命令只能查看到本次登录所操作的命令；如需查看之前登录的历史命令，可通过查看~/.bash_History文件。