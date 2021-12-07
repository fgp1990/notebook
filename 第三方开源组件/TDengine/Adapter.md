# taosAdapter

TDengine 提供了一个 API 接口，方便写入读取。但是这个东西需要单独安装。
根据官方的文档，有两种，一种是直接源码编译，把这个功能加到 server 端里面；另一种，是单独安装 adapter。
这里记录集成的方法

## 依赖准备

官方源码是没有把这个功能放进来的，需要通过 git 把相关的代码都弄到一起，然后编译。
这是更新依赖的命令：

```git
git submodule update --init --recursive
```

这个有问题，github 咱上不去，需要改 git 的 config，换成 gitee。(gitee 上也没有，这是现从 github 挪到 gitee 的)
config 内容：

```git
[core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
[remote "origin"]
        url = https://gitee.com/JfqmGw/TDengine.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "develop"]
        remote = origin
        merge = refs/heads/develop
[submodule "deps/jemalloc"]
#       url = https://github.com/jemalloc/jemalloc
        url = https://gitee.com/JfqmGw/jemalloc.git
[submodule "src/connector/go"]
        url = https://gitee.com/JfqmGw/driver-go.git
#       url = https://github.com/taosdata/driver-go.git
[submodule "src/connector/hivemq-tdengine-extension"]
        url = https://gitee.com/JfqmGw/hivemq-tdengine-extension.git
#       url = https://github.com/taosdata/hivemq-tdengine-extension.git
[submodule "src/kit/taos-tools"]
#       url = https://github.com/taosdata/taos-tools
        url = https://gitee.com/JfqmGw/taos-tools.git
[submodule "src/plugins/taosadapter"]
        url = https://gitee.com/JfqmGw/taosadapter.git
#       url = https://github.com/taosdata/taosadapter
[submodule "tests/examples/rust"]
        url = https://gitee.com/JfqmGw/tdengine-rust-bindings.git
#       url = https://github.com/songtianyi/tdengine-rust-bindings.git
```

除了这些还有个地方有依赖：

```
/home/tdEngine/TDengine/src/kit/taos-tools/.gitmodule
```

```git
[submodule "deps/avro"]
        path = deps/avro
        url = https://gitee.com/JfqmGw/avro.git
#       url = https://github.com/apache/avro
```

这样就齐活了。

执行最前面的 git 命令即可把所有代码都弄下来

## 编译

```
mkdir debug
cd debug
cmake ..
make
sudo make install
```

这里遇到几个问题：

### 1. cmake 没有

```
yum install cmake
```

然后，版本太低。要求 3.0 及以上，这装了个 2.8。
下载新版本，编译安装

```
wget "https://cmake.org/files/v3.17/cmake-3.17.0.tar.gz"
tar xvf cmmake=3.17.0.tar.gz
cd cmake-3.17.0
./bootstrap
gmake
sudo make install
```

还不行，

```shell
[root@falcon-01 cmake-3.17.0]# ./bootstrap
---------------------------------------------
CMake 3.17.0, Copyright 2000-2020 Kitware, Inc. and Contributors
C compiler on this system is: cc
---------------------------------------------
Error when bootstrapping CMake:
Cannot find a C++ compiler that supports both C++11 and the specified C++ flags.
Please specify one using environment variable CXX.
The C++ flags are "".
They can be changed using the environment variable CXXFLAGS.
See cmake_bootstrap.log for compilers attempted.
---------------------------------------------
Log of errors: /home/tdEngine/cmake-3.17.0/Bootstrap.cmk/cmake_bootstrap.log
---------------------------------------------
```

```
yum install gcc gcc-c++
```

### 2. cmake 出错

1. No package 'liblzma' found
   ```
   yum install xz-devel
   ```

## 使用

先鉴权，再取数
鉴权：
第一个'root'是用户名，第二个'root'是密码。

```shell
[root@falcon-01 ~]# curl "http://127.0.0.1:6041/rest/login/root/root"
{"status":"succ","code":0,"desc":"/KfeAzX/f9na8qdtNZmtONryp201ma04/KfeAzX/f9na8qdtNZmtONryp201ma04"}
```

使用 desc 的值，添加到后续请求的 header 中。

```shell
[root@falcon-01 ~]# curl "http://127.0.0.1:6041/rest/sql/myself" -d 'select * from myself_n1;' -H 'Authorization: Taosd /KfeAzX/f9na8qdtNZmtONryp201ma04/KfeAzX/f9na8qdtNZmtONryp201ma04'
{"status":"succ","head":["ts","current","voltage","phase"],"column_meta":[["ts",9,8],["current",6,4],["voltage",4,4],["phase",6,4]],"data":[["2021-12-06 17:49:46.000",1.1,22,2.2],["2021-12-06 18:56:26.000",2.1,42,6.2],["2021-12-06 18:57:26.000",2.1,42,6.2]],"rows":3}
```

注意 header 的格式。
这个并不好，这个从本质上还是写 sql，进行操作，并不适合规范化生产。
