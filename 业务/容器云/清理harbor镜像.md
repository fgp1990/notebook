
# 清理harbor镜像
## 一、安装Python的pip环境
### 1、获取：
```
[root@k8s-habor harbor-script]$wget https://bootstrap.pypa.io/get-pip.py --no-check-certificate
--2019-08-07 10:49:14--  https://bootstrap.pypa.io/get-pip.py
Resolving bootstrap.pypa.io (bootstrap.pypa.io)... 151.101.228.175, 2a04:4e42:36::175
Connecting to bootstrap.pypa.io (bootstrap.pypa.io)|151.101.228.175|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1775087 (1.7M) [text/x-python]
Saving to: ‘get-pip.py’
100%
[==========================================================================================================
1,775,087 277KB/s in 27s 
[18:49:42]
[18:49:42]2019-08-07 10:49:43 (63.7 KB/s) - ‘get-pip.py’ saved [1775087/1775087]
```
### 2、安装
```
[root@k8s-habor harbor-script]$python get-pip.py
DEPRECATION: Python 2.7 will reach the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 won't be maintained after that
date. A future version of pip will drop support for Python 2.7. More details about Python 2 support in pip, can be found at
https://pip.pypa.io/en/latest/development/release-process/#python-2-support
Collecting pip
Downloading https://files.pythonhosted.org/packages/62/ca/94d32a6516ed197a491d17d46595ce58a83cbb2fca280414e57cd86b84dc/pip-19.2.1-py2.py3-none-any.whl(1.4MB)
|████████████████████████████████| 1.4MB 288kB/s
Collecting setuptools
Downloading https://files.pythonhosted.org/packages/ec/51/f45cea425fd5cb0b0380f5b0f048ebc1da5b417e48d304838c02d6288a1e/setuptools-41.0.1-py2.py3-none-any.whl(575kB)
|████████████████████████████████| 583kB 596kB/s
Collecting wheel
Downloading https://files.pythonhosted.org/packages/bb/10/44230dd6bf3563b8f227dbf344c908d412ad2ff48066476672f3a72e174e/wheel-0.33.4-py2.py3-none-any.whl
Installing collected packages: pip, setuptools, wheel
Successfully installed pip-19.2.1 setuptools-41.0.1 wheel-0.33.4
[root@k8s-habor harbor-script]$pip install --ignore-installed requests
DEPRECATION: Python 2.7 will reach the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 won't be maintained after that
date. A future version of pip will drop support for Python 2.7. More details about Python 2 support in pip, can be found at
https://pip.pypa.io/en/latest/development/release-process/#python-2-support
Collecting requests
Downloading https://files.pythonhosted.org/packages/51/bd/23c926cd341ea6b7dd0b2a00aba99ae0f828be89d72b2190f27c11d4b7fb/requests-2.22.0-py2.py3-none-any.whl(57kB)
|████████████████████████████████| 61kB 43kB/s
Collecting chardet<3.1.0,>=3.0.2 (from requests)
Downloading https://files.pythonhosted.org/packages/bc/a9/01ffebfb562e4274b6487b4bb1ddec7ca55ec7510b22e4c51f14098443b8/chardet-3.0.4-py2.py3-none-any.whl(133kB)
|████████████████████████████████| 143kB 10kB/s
Collecting idna<2.9,>=2.5 (from requests)
Downloading https://files.pythonhosted.org/packages/14/2c/cd551d81dbe15200be1cf41cd03869a46fe7226e7450af7a6545bfc474c9/idna-2.8-py2.py3-none-any.whl(58kB)
|████████████████████████████████| 61kB 22kB/s
Collecting urllib3!=1.25.0,!=1.25.1,<1.26,>=1.21.1 (from requests)
Downloading https://files.pythonhosted.org/packages/e6/60/247f23a7121ae632d62811ba7f273d0e58972d75e58a94d329d51550a47d/urllib3-1.25.3-py2.py3-none-any.whl(150kB)
|████████████████████████████████| 153kB 11kB/s
Collecting certifi>=2017.4.17 (from requests)
Downloading https://files.pythonhosted.org/packages/69/1b/b853c7a9d4f6a6d00749e94eb6f3a041e342a885b87340b79c1ef73e3a78/certifi-2019.6.16-py2.py3-none-any.whl(157kB)
|████████████████████████████████| 163kB 12kB/s
Installing collected packages: chardet, idna, urllib3, certifi, requests
Successfully installed certifi-2019.6.16 chardet-3.0.4 idna-2.8 requests-2.22.0 urllib3-1.25.3
```
## 二、harbor环境准备
##### 1、修改harbor配置信息：
get_image.sh和del_image.sh文件内定义了harbor地址，需修改为实际值，比如HARBOR='10.18.210.178:40080'
get_repo_to_del.py和del_image.py定义了harbor admin用户的密码，需修改为实际值
##### 2、 从lvs机器上，拷贝/etc/route_nginx/kubecfg_rnwatcher.cfg文件到harbor机器/etc/kubernetes/kube-watcher.conf
##### 3、执行get_image.sh脚本，会导出要删除的镜像到imagedel.log
##### 4、 执行del_image.sh脚本，会根据上一步导出的imagedel.log结果执行删除
##### 5、 重启harbor释放磁盘空间：
```
systemctl stop harbor
docker load -i registry-master.tar
```
需确认线上harbor镜像库存储目录（物理机目录）
```
docker run --net=none --rm --entrypoint registry -e "REGISTRY_STORAGE_DELETE_ENABLED=true" -v /data/registry:/var/lib/registry -v /data/registry-master-config-for-clean.yml:/etc/docker/registry/config.yml docker.io/distribution/registry:master garbage-collect /etc/docker/registry/config.yml -m
systemctl start harbor
```
systemctl start harbor——只是重启，如果说你修改了docker的配置文件，这么做是没有用的
/usr/local/fountain/docker/docker-harbor/harbor/harbor-install.sh——这是开发给的脚本，要用这个


相关文件路径：/home/20190807
重启后验证：
1、docker ps 共有6个docker
2、ui 服务docker确认
3、浏览器访问确认；
harbor服务重启以后，mysql进程的CPU 负载较高，需要cpu恢复正常后，再查看是否能正常登陆打开；



附：
在11.54的/home/20190807/里面

## 二、清理docker分区(dockervg)
```
docker ps -a | grep -E "Exited|Dead|Created" | awk -F ' ' '{print $1}' | xargs docker rm -f
docker images | grep months | awk '{print $1":"$2}' | xargs -r docker rmi
```
## 三、如果删多了怎么办
在master上导出pod镜像列表来
```
kubectl --all-namespaces=true get pods -o custom-columns='IMAGES:.spec.containers.*.image' --no-headers=true | tr ',' '\n' | sort -u > all_pods_images.txt
```
在harbor上pull一下试试有没有
```
for image in `cat all_pods_images.txt`; do docker pull $image; done
```
可能略费harbor本地空间……

要查deployments当前镜像就
```
kubectl --all-namespaces=true get deployments -o custom-columns='IMAGES:.spec.template.spec.containers.*.image' --no-headers=true | tr ',' '\n' | sort -u
```
可以把所有当前正在使用的镜像都统计出来，然后重新拉