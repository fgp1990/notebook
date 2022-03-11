# Windows下的linux
win10自带一套linux系统，系统是Ubuntu。
## 1、安装
现在windows功能里面，把linux功能打开。就跟telnet client一样。
然后到Microsoft Store里面搜索linux，就能看到windows目前支持的linux发行版本，基本上就只有Ubuntu你晓得。选不带版本号的，这个是跟着发布走，随时更新。当然具体用哪个版本没有什么太大区别。
选择安装，时间有点长。Microsoft Store网络挺差的。
安装好后，再装一个Windows Terminal，这个就是个美化过后的ssh终端界面，还可以。
## 2、配置
装好以后，启动。第一次启动会配置一些数据，时间稍微长一点。
配置好后，会要求你输入用户名、密码，这里可以不输入。不输入的话，下次启动，就直接是root用户，你要是输入了，就是你输入的用户。比较而言，root方便一点。
~~你要是不小心输入了，可以切换到root，把你创建的用户删掉即可。sudo passwd root就可以改root密码，改好就能切换了~~
这样做根本不行，你切换过去，原用户处于登录状态，压根没法删除。你应该进到cmd里面，执行
```shell
ubuntu config --default-user root
```
## 3、挂载、环境变量和默认目录
默认情况下，会自动把你windows下的磁盘都挂载上，这个多半是用不到的。当然放上去也行
具体配置方法：
创建/etc/wsl.conf文件，默认没有的。
```conf
[automount]
enabled = true
root = /mnt/
options = "metadata,umask=22,fmask=11"
mountFsTab = false

[interop]
enabled = true
appendWindowsPath = false
```

如果不想挂载，就把automount里面的true改为false。下面的options是控制挂载时的权限，默认情况挂载目录权限是777，其实没有什么，就是看起来很难受。
interop里面是控制linux里面的环境变量的，如果不配置，就会把windows里面的环境变量全带进来，很讨厌。
再个就是启动时默认打开的目录，如果你挂载windows目录的话，就是windows当前登录的用户目录，我希望是root目录。
在Windows Terminal终端的设置中，打开setting.json文件，找到Ubuntu，在list里面，搞成下面的样子：
```json
 {
                "cursorShape": "vintage",
                "fontFace": "Consolas",
                "guid": "{2c4de342-38b7-51cf-b940-2309a097f518}",
                "hidden": false,
                "name": "Ubuntu",
                "source": "Windows.Terminal.Wsl",
                "startingDirectory": "//wsl$/Ubuntu/root"
},
```
这就好了，注意上一行末尾是有个逗号的
<font color="red">重点：
这些配置完了，不一定生效。需要在cmd里面执行</font>
```
C:\Users\fgp>wsl --list
适用于 Linux 的 Windows 子系统分发版:
Ubuntu (默认)

C:\Users\fgp>wsl --terminate Ubuntu
```
然后重启，才会生效。主要是环境变量和挂载。
## 4、更换源
```
sudo cp -a /etc/apt/sources.list /etc/apt/sources.list.bak
sudo sed -i "s@http://.*archive.ubuntu.com@http://repo.huaweicloud.com@g" /etc/apt/sources.list
sudo sed -i "s@http://.*security.ubuntu.com@http://repo.huaweicloud.com@g" /etc/apt/sources.list
apt-get update
```