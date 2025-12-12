---
title: OpenTenBase 单机部署教程
autor: Chenpi
date: 2025-12-12 19:35:00 +0800
categories: [数据库, OpenTenBase]
tags: [OpenTenbase, 教程]

math: true
mermaid: true
---

> **适用范围**：本教程只适用于RedHat系列（CentOS、RockyLinux）\
> **说明**：本文档演示单机集中式部署方案，适用于开发测试环境。

## 一、准备工作
### 1. 环境准备与依赖安装
更新系统并安装编译所需的基础依赖库。

```bash
# 更新系统及安装 EPEL 源
sudo yum update -y
sudo yum install -y epel-release

# 启用powertools源
sudo yum config-manager --set-enabled powertools
sudo yum clean all
sudo yum makecache 

# 安装编译工具链与基础库
sudo yum -y install git sudo gcc make readline-devel zlib-devel openssl-devel uuid-devel bison flex cmake postgresql-devel libssh2-devel libcurl-devel libxml2-devel
```

### 2. 修正库文件链接
部分系统 libssh2 版本命名存在差异，需建立软链接以确保兼容性。
创建一个软链接。

```bash
sudo ln -s /usr/lib64/libssh2.so.1 /usr/lib64/libssh2.so
```

检查是否存在libssh2.so

```bash
ls /usr/lib64/libssh2.so*
```

### 3. 推荐的设置
- 关闭SELinux

```bash
sudo vim /etc/selinux/config

# 将: SELINUX=enforcing 
# 修改为: SELINUX=disabled
```

- 禁用防火墙

```bash
systemctl disable firewalld && systemctl stop firewalld
```

### 4. 创建用户
1. 创建数据目录和数据库用户：

```bash
sudo mkdir -p /data # 创建目录/data
sudo useradd -d /data/opentenbase -s /bin/bash -m opentenbase # 创建用户
sudo passwd opentenbase # 设置密码，若设置分布式需所有服务器的opentenbase用户密码保持一致
```

1. 启用wheel group 的 sudo 权限

```bash
sudo usermod -aG wheel opentenbase # 添加到 WheelGroup
sudo visudo # 编辑 sudoers 文件，取消%wheel行的注释
```

然后搜索wheel，可以看到%wheel开头的两条配置，分别去掉最左边的#，保存退出。

```bash
# 取消这两行的注释
%wheel  ALL=(ALL)       ALL 
%wheel  ALL=(ALL)       NOPASSWD: ALL
```

1. 切换用户

```bash
su - opentenbase
```

## 二、源码编译
~~(不想和各种依赖打架的可以直接跳至三、安装与部署)~~
### 1. 手动编译第三方依赖
- ztsd

```bash
git clone https://github.com/facebook/zstd.git
cd zstd 
make -j$(nproc) 
sudo make install
cd ..

```

- lz4

```bash
git clone https://github.com/lz4/lz4.git
cd lz4
make -j$(nproc)
sudo make install
cd ..

```

- uuid

```bash
wget ftp://ftp.ossp.org/pkg/lib/uuid/uuid-1.6.2.tar.gz
# 下载不了就使用镜像源
wget http://www.mirrorservice.org/sites/ftp.ossp.org/pkg/lib/uuid/uuid-1.6.2.tar.gz
tar xf uuid-1.6.2.tar.gz && cd uuid-1.6.2 
./configure 
make -j$(nproc) 
sudo make install
cd ..

```

- CLI11

```bash
git clone https://github.com/CLIUtils/CLI11.git 
mkdir -p CLI11/build && cd CLI11/build 
cmake ..
make -j$(nproc) 
sudo make install
cd ..
cd ..

```

1. 清理临时文件

```bash
sudo rm -rf CLI11  lz4  uuid-1.6.2  uuid-1.6.2.tar.gz  zstd
```

### 2. 编译OpenTenBase源代码
1. 获取OpenTenBase源代码

```bash
git clone https://github.com/OpenTenBase/OpenTenBase
```

1. 设置环境变量

```bash
export SOURCECODE_PATH=/data/opentenbase/OpenTenBase
export INSTALL_PATH=/data/opentenbase/install/
```

1. 编译

```bash
# 进入源代码目录
cd ${SOURCECODE_PATH}

# 配置编译选项
rm -rf ${INSTALL_PATH}/opentenbase_bin_v5.0
chmod +x configure*
./configure --prefix=${INSTALL_PATH}/opentenbase_bin_v5.0 --enable-user-switch --with-libxml --disable-license --with-openssl --with-ossp-uuid CFLAGS="-g"

# 编译安装
make clean
make -sj$(nproc)
make install

# 安装contrib模块
chmod +x contrib/pgxc_ctl/make_signature
cd contrib
make -sj$(nproc)
make install
```

## 三、安装与部署
### 1. 设置环境变量

```bash
export SOURCECODE_PATH=/data/opentenbase/OpenTenBase
export INSTALL_PATH=/data/opentenbase/install/
PG_HOME=${INSTALL_PATH}/opentenbase_bin_v5.0
export PATH="$PATH:$PG_HOME/bin"
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:$PG_HOME/lib"
export LC_ALL=C
```

### 2. 准备安装包与工具
#### 分支A: 如果未进行源码编译
1. 下载opentenbase_ctl工具与二进制包

```bash
cd ${INSTALL_PATH}
wget https://github.com/Chenpi-Sakura/OpenTenBase/releases/download/v5.0/opentenbase_ctl
wget https://github.com/Chenpi-Sakura/OpenTenBase/releases/download/v5.0/opentenbase-5.21.8-i.x86_64.tar.gz
```

1. 赋予工具执行权限

```bash
chmod +x opentenbase_ctl
```

#### 分支B: 如果进行了源码编译
1. 获取部署工具opentenbase_ctl

```bash
cd ${INSTALL_PATH}
cp ${PG_HOME}/bin/opentenbase_ctl .
```

1. 将编译好的目录打包

```bash
cd ${PG_HOME} 
tar -zcf ../opentenbase-5.21.8-i.x86_64.tar.gz *
```

---

### 3. 准备好部署的配置文件

```bash
touch postgres.conf
touch config.ini
```

>postgres.conf:用户自定义配置项，如果你想节点初始化后替换某个guc配置项，可以将该配置写在这个文件中。如果没有特别的自定义配置，直接使用工具自动配置的GUC，那么这个文件为空即可。

### 4. 修改config.ini内容

```bash
vim config.ini
```

> 请务必将 10.34.159.220 替换为当前服务器的真实内网 IP 地址。

```ini
# 集中式配置

# 实例配置
[instance]
name=opentenbase_c
type=centralized
package=/data/opentenbase/install/opentenbase-5.21.8-i.x86_64.tar.gz

# 数据节点配置
[datanodes]
# 填写当前机器 IP
master=10.34.159.7
nodes-per-server=1
conf=/data/opentenbase/install/postgres.conf

[server]
ssh-user=opentenbase
# 此处更换为创建OpenTenBase用户时设置的密码
ssh-password=123456
ssh-port=22

[log]
level=DEBUG
```

### 5. 检查文件是否存在
检查config.ini，postgres.conf，opentenbase-5.21.8-i.x86_64.tar.gz，opentenbase_ctl是否都存在

```bash
[opentenbase@sc install]$ ls -l 
total 40812
-rw-rw-r--. 1 opentenbase opentenbase      294 Nov 17 20:29 config.ini
-rw-rw-r--. 1 opentenbase opentenbase 40974834 Nov 17 20:19 opentenbase-5.21.8-i.x86_64.tar.gz
drwxrwxr-x. 6 opentenbase opentenbase       56 Nov 17 20:13 opentenbase_bin_v5.0
-rwxr-xr-x. 1 opentenbase opentenbase   810544 Nov 17 20:23 opentenbase_ctl
-rw-rw-r--. 1 opentenbase opentenbase        0 Nov 17 20:19 postgres.conf
```

### 6. 开始部署

```bash
./opentenbase_ctl install -c config.ini
```

部署过程

```bash
 ====== Start to Install  instance opentenbase_c  ====== 

step 1: Make *.tar.gz pkg ...
    Make opentenbase-5.21.8-i.x86_64.tar.gz successfully.

step 2: Transfer and extract pkg to servers ...
    Package_path: /data/opentenbase/install/opentenbase-5.21.8-i.x86_64.tar.gz
    Transfer and extract pkg to servers successfully.

step 3: Install cn/dn master node ...
    Install dn0001(10.34.159.220) ...
    Install dn0001(10.34.159.220) successfully
    Success to install all cn/dn master nodes. 

step 4: Install slave nodes ...
    Success to install all slave nodes. 

step 5:Create node group ...
    Create node group successfully. 

 ====== Installation completed successfully  ====== 
```

### 7. 检查

```bash
./opentenbase_ctl status
```

此时的结果应当为...

```bash
 ------------- Instance status -----------  
Instance name: opentenbase_c
Version: v5.21.8

 -------------- Node status --------------  
Node dn0001(10.34.159.220:11000) is Running 
[Result] Total: 1, Running: 1, Stopped: 0, Unknown: 0

 ------- Master DN Connection Info -------  
Environment variable: export LD_LIBRARY_PATH=/data/opentenbase/install/opentenbase/5.21.8/lib  && export PATH=/data/opentenbase/install/opentenbase/5.21.8/bin:${PATH} 
PSQL connection: psql -h 10.34.159.220 -p 11000 -U opentenbase postgres
```

**无报错，恭喜你部署成功！**

还可以验证一下：
复制并粘贴Environment variable环境变量和PSQL connection后面的内容

```bash
[opentenbase@sc install]$ export LD_LIBRARY_PATH=/data/opentenbase/install/opentenbase/5.21.8/lib  && export PATH=/data/opentenbase/install/opentenbase/5.21.8/bin:${PATH} 
[opentenbase@sc install]$ psql -h 10.34.159.220 -p 11000 -U opentenbase postgres
psql (PostgreSQL 10.0 @ OpenTenBase_v5.0 (commit: b612d77cb) 2025-11-17 20:11:26)
Type "help" for help.

postgres=# CREATE TABLE hello_opentenbase(id INT);
CREATE TABLE
postgres=# \dt
          List of relations
 Schema |        Name       | Type  |    Owner    
--------+-------------------+-------+-------------
 public | hello_opentenbase | table | opentenbase
(1 row)
```

---

## 四、注意事项
1. 假如服务器的IP发生了变化，一定要更新配置文件中的IP地址，否则会发生连接超时！

2. 查看工具支持的功能，可以执行命令 `./opentenbase_ctl -h `来看详细支持的功能。
- 查看支持的基本功能

``` bash
[opentenbase@sc install]$ ./opentenbase_ctl -h
Opentenbase cluster management tool 
 
 
opentenbase_ctl [OPTIONS] [SUBCOMMAND]
 
 
OPTIONS:
  -h,     --help              Print this help message and exit 
          --version           Display program version information and exit 
 
SUBCOMMANDS:
  install                     Install a new Opentenbase cluster 
  delete                      Delete an existing Opentenbase cluster 
  start                       Start a Opentenbase cluster 
  stop                        Stop a Opentenbase cluster 
  status                      Show Opentenbase cluster status 
  expand                      Expand a Opentenbase cluster 
  shrink                      Shrink a Opentenbase cluster 
```

- 查看具体某个功能（比如 stop）的命令参数： 

``` bash
[opentenbase@sc install]$ ./opentenbase_ctl stop -h
Stop a Opentenbase cluster 
 
 
opentenbase_ctl stop [OPTIONS]
 
 
OPTIONS:
  -h,     --help              Print this help message and exit 
  -c,     --config TEXT       Path to configuration file 
          --instance-name TEXT 
                              Instance name 
          --package-path TEXT Package path 
          --node-name TEXT    Node name 
          --node-ip TEXT      Node IP 
          --ssh-user TEXT     SSH user 
          --ssh-password TEXT SSH password 
          --ssh-port TEXT     SSH port 
```

---

参考资料
[OpenTenBase/README_ZH.md](https://github.com/OpenTenBase/OpenTenBase/blob/master/README_ZH.md)
[快速入门 - OpenTenBase Documentation](https://docs.opentenbase.org/guide/01-quickstart/)
[opentenbase_ctl](https://github.com/OpenTenBase/OpenTenBase/tree/master/contrib/opentenbase_ctl)
