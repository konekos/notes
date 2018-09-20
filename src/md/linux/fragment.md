### Fragment

#### 固定IP

```
ifconfig
网卡enpxxx
vim /etc/sysconfig/network-scripts/ifcfg-enpxxx

BOOTPROTO="static"
IPADDR=...
NETMASK=255.255.255.0
GATEWAY=...
NM_CONTROLLED="no"
```

#### Git 安装

http://blog.51cto.com/xiong51/2088755

1. 环境准备 

   ```
   yum -y install gcc openssl openssl-devel curl curl-devel unzip perl perl-devel expat expat-devel zlib zlib-devel asciidoc xmlto gettext-devel openssh-clients
   ```

2. 编译安装 

   ```bash
   [root@testss git-2.9.5]# tar xf git-2.9.5.tar.xz 
   [root@testss git-2.9.5]# cd git-2.9.5
   [root@testss git-2.9.5]# ./configure --prefix=/usr/local/git --with-openssl --with-libpcre 
   [root@testss git-2.9.5]# make -j 2 && make -j 2 install
   ```

3. 环境及用户权限配置

   ```bash
   [root@testss git-2.9.5]# echo "export PATH=/usr/local/git/bin/:$PATH" > /etc/profile.d/git.sh
   [root@testss git-2.9.5]# source !$
   
   [root@testss git-2.9.5]# cat /etc/passwd  将/bin/bash改为如下 
       git:x:496:496::/home/git:/usr/local/git/bin/git-shell
   
   // 说明：git用户可以正常通过ssh使用git，但无法登录shell，因为我们为git 用户指定的git-shell每次一登录就自动退出
   ```

### JDK卸载与安装

```shell
rpm -qa | grep java
rpm -e #卸载openJDK

tar -zxvf jdk-7u67-linux-i586.tar.gz -C /usr/local
vim /etc/profile

export JAVA_HOME=/usr/local/jdk1.7.0_67      #这里换成你的JDK解压路径
export PATH=$PATH:$JAVA_HOME/bin

source /etc/profile

```

