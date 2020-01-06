---
layout: post
title:  "搭建Hadoop集群"
date:   2020-01-05 02:00:00 -1500
categories: hadoop 
---

手把手教你搭建Hadoop集群

---

# 前言

&emsp;目前很多企业比如银行的BI系统通常是建立在传统数据仓库上，实现载体主要是Oracle、Teradata、GreenPlum等关系型数据库。

&emsp;但是在如今信息大爆炸的时代，数据都是实时更新且数据量非常大，传统数据仓库已经不能满足BI系统的业务需求了。总的来说，传统数据仓库劣势主要有以下三点：

1. 不能满足海量数据存储需求

2. 不能处理不同类型的数据

3. 计算与处理能力差

&emsp;随着大数据技术趋于成熟，越来越多企业尝试将大数据技术应用于BI系统中。但并不是每个企业都需要打造自己的大数据平台，量力而行吧，可以自研，比如BAT，也可以采购，比如传统大企业，也可以租用，比如用阿里云和AWS。

&emsp;在众多大数据分析工具中，以开源组件Hadoop为主流。一个典型的基于Hadoop平台的数据仓库架构如下所示：

<center>
    <img style="border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\hadoop.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">基于Hadoop平台的数据仓库架构</div>
</center>

&emsp;架构分为四层：OLTP层、数据仓库层（含ODS）、数据集市层、应用层。新型的数据仓库架构与传统数据仓库架构的唯一区别就是数据仓库层的不同。新型的数据仓库架构除了使用关系型数据库，同时将大数据技术应用于数据仓库中。

&emsp;从图中可以看到，搭建Hadoop平台需要的组件包括Zookeeper、Kafka、Hadoop、Hive、Spark、Hbase以及MySQL关系型数据库等。

&emsp;一个简单的Hadoop集群至少应该有三台机器，在每台机器都安装Zookeeper、Kafka、Hadoop、Hive、Spark、Hbase，在其中一台机器上安装MySQL。好啦，话不多说，下面我们开始搭建Hadoop集群。

---

&emsp;

# 概览

1. [准备工作](#anchor1)

   + [准备虚拟机](#anchor1_1)

   + [网络配置](#anchor1_2)

      - [网卡配置](#anchor1_2_1)

      - [关闭防火墙](#anchor1_2_2)

      - [修改hosts](#anchor1_2_3)

   + [下载基础工具](#anchor1_3)

   + [创建hadoop用户](#anchor1_4)

   + [分发密钥](#anchor1_5)

   + [安装ntp时钟同步服务](#anchor1_6)

2. [安装jdk](#anchor2)

2. [在cluster2上安装MySQL](#anchor3)

2. [安装Zookeeper](#anchor4)

2. [安装Kafka](#anchor5)

2. [安装Hadoop](#anchor6)

2. [安装HBase](#anchor7)

2. [安装Hive](#anchor8)

2. [安装Spark](#anchor9)

   + [安装Scala](#anchor9_1)

   + [安装Spark](#anchor9_2)

2. [安装Storm](#anchor10)

&emsp;

---

<span id = "anchor1">&emsp;</span>

# 准备工作

因为要安装的工具比较多，所以在此先列出安装清单。

<table width="90%" border="1">
    <thead>
        <tr>
            <th width="60%" align="center">软件包文件名</th>
            <th width="10%" align="center">软件名称</th>
            <th width="20%" align="center">软件版本</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>jdk-8u201-linux-x64.tar.gz</td>
            <td align="center">JDK</td>
            <td align="center">1.8</td>
        </tr>
        <tr>
            <td>mysql-8.0.15-linux-glibc2.12-x86_64.tar.xz</td>
            <td align="center">MySQL</td>
            <td align="center">8.0.15</td>
        </tr>
        <tr>
            <td>zookeeper-3.4.13.tar.gz</td>
            <td align="center">Zookeeper</td>
            <td align="center">3.4.13</td>
        </tr>
        <tr>
            <td>kafka_2.12-2.1.1.tgz</td>
            <td align="center">Kafka</td>
            <td align="center">2.1.1</td>
        </tr>
        <tr>
            <td>hadoop-3.1.2.tar.gz</td>
            <td align="center">Hadoop</td>
            <td align="center">3.1.2</td>
        </tr>
        <tr>
            <td>hbase-2.1.3-bin.tar.gz</td>
            <td align="center">HBase</td>
            <td align="center">2.1.3</td>
        </tr>
        <tr>
            <td>apache-hive-2.3.4-bin.tar.gz</td>
            <td align="center">Hive</td>
            <td align="center">2.3.4</td>
        </tr>
        <tr>
            <td>scala-2.12.8.tgz</td>
            <td align="center">Scala</td>
            <td align="center">2.12.8</td>
        </tr>
        <tr>
            <td>spark-2.4.0-bin-hadoop2.7.tgz</td>
            <td align="center">Spark</td>
            <td align="center">2.4.0</td>
        </tr>
        <tr>
            <td>apache-storm-1.2.2.tar.gz</td>
            <td align="center">Storm</td>
            <td align="center">1.2.2</td>
        </tr>
        <tr>
            <td>CentOS-7-x86_64-Minimal-1708.iso</td>
            <td align="center">CentOS7</td>
            <td align="center">最小安装版镜像</td>
        </tr>
        <tr>
            <td>MobaXterm_Portable_v12.1.zip</td>
            <td align="center">MobaXterm</td>
            <td align="center">12.1</td>
        </tr>
    </tbody>
</table>

<span id = "anchor1_1">&emsp;</span>

## 准备虚拟机

&emsp;虚拟机可以在电脑上自行安装，常用的工具有VMware和VirtualBox。也可以租3台云服务器来搭建集群。

&emsp;如果在电脑上自己安装 3 台 linux 虚拟机，推荐使用CentOS 7系统镜像，文件名“CentOS-7-x86_64-Minimal-1708.iso”。

&emsp;如果购买云服务器，比如阿里云服务器，需要注意一点，因为端口是默认关闭的，为使集群之间正常通信，要将服务器的部分端口开放。

<span id = "anchor1_2">&emsp;</span>

## 网络配置

<span id = "anchor1_2_1">&emsp;</span>

### 虚拟机网卡配置

&emsp;以Vmware安装虚拟机为例，为使虚拟机能与主机通信，网卡应按如下设置：

&emsp;&emsp;1) 虚拟机的网络设置选择“NAT”模式，也就是使用VMnet8虚拟网卡。

<center>
    <img style="width:50%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\vmware.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">设置虚拟机网络适配器</div>
</center>

&emsp;&emsp;2) 配置VMnet8虚拟网卡的IP地址，如下图所示。比如我这里设置地址为“192.168.61.0”，那么接下来三台虚拟机的IP应在网段“192.168.61.128-192.168.61.254”内选择。

<center>
    <img style="width:50%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\vmware2.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">设置VMnet8虚拟网卡IP地址</div>
</center>

&emsp;&emsp;3) 开启虚拟机，输入“ip addr”查看网卡名称。比如我的 cluster1 虚拟机网卡名称为 ens33，那么输入“vi /etc/sysconfig/network-scripts/ifcfg-ens33”编辑网络配置文件。

<center>
    <img style="width:70%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\ipaddr2.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">查看网卡信息</div>
</center>

<center>
    <img style="width:36%;
    border-radius: 0.3125em;
    box-shadow: 0 2px 4px 0 rgba(34,36,38,.12),0 2px 10px 0 rgba(34,36,38,.08);" 
    src="\assets\ens332.png">
    <br>
    <div style="color:orange; border-bottom: 1px solid #d9d9d9;
    display: inline-block;
    color: #999;
    padding: 2px;">配置虚拟机网卡</div>
</center>

&emsp;&emsp;&emsp;编辑网卡配置文件。设置BOOTPROTO为 static 或 none，IP地址范围为“192.168.61.128-192.168.61.254”，VMnet8网关地址为“192.168.61.2”，掩码可设可不设。设置开机启动网卡“ONBOOT=yes”。
```
vi /etc/sysconfig/network-scripts/ifcfg-ens33

# 设置BOOTPROTO为 static 或 none
BOOTPROTO=static
ONBOOT=yes
# 地址范围“192.168.61.128-192.168.61.254”
IPADDR=192.168.61.130
# 网关选择VMnet8网关
GATEWAY=192.168.61.2
# 掩码可不设，默认为255.255.255.0
NETMASK=255.255.255.0
```
&emsp;&emsp;&emsp;然后重启网络：
```
service network restart
```

&emsp;&emsp;4) 因为静态地址无法上网，需要添加DNS服务器才能上网，常用的DNS服务器有Google的“8.8.8.8” 服务器和“8.8.4.4” 服务器。
```
vi /etc/resolv.conf
nameserver 8.8.8.8
```

<span id = "anchor1_2_2">&emsp;</span>

### 关闭防火墙

```
#关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

#禁用selinux
vi /etc/selinux/config
SELINUX=disabled

#查看selinux状态，显示“Disabled”表示已被禁用
getenforce
```

<span id = "anchor1_2_2">&emsp;</span>

### 修改hosts

&emsp;在hosts文件中添加三台机器的ip地址与域名的映射关系。
```
192.168.61.130 cluster1
192.168.61.131 cluster2
192.168.61.132 cluster3
```
&emsp;如果是阿里云服务器或其他云服务器，要注意云服务器的外网和内网的关系，外网地址是云服务器厂商提供给用户使用的地址，在万维网中可访问；内网地址实际上是厂商的私有局域网中的网址，用于云服务器之间的内部通信。所以使用云服务器搭建集群的时候，配置hosts需要将本机映射到内网地址，其他机器映射到外网地址。下面给出示范：

&emsp;&emsp;cluster1的外网地址为47.98.176.164，内网地址为172.16.136.37

&emsp;&emsp;cluster2的外网地址为47.98.47.81，内网地址为172.16.206.43

&emsp;&emsp;cluster2的外网地址为116.62.119.79，内网地址为172.16.29.7

&emsp;&emsp;那么，三台机器的hosts配置应该是这样的：

&emsp;&emsp;cluster1
```
172.16.136.37 cluster1
47.98.47.81 cluster2
116.62.119.79 cluster3
```
&emsp;&emsp;cluster2
```
47.98.176.164 cluster1
172.16.206.43 cluster2
116.62.119.79 cluster3
```
&emsp;&emsp;cluster3
```
47.98.176.164 cluster1
47.98.47.81 cluster2
172.16.29.7 cluster3
```

<span id = "anchor1_3">&emsp;</span>

## 下载基础工具
```
yum install perl*
yum install ntpdate
yum install libaio
yum install screen
```

<span id = "anchor1_4">&emsp;</span>

## 创建hadoop用户
```
groupadd hadoop
useradd -s /bin/bash -g hadoop -d /home/hadoop -m hadoop
```

<span id = "anchor1_5">&emsp;</span>

## 分发密钥

&emsp;每个节点生成私钥和公钥，然后将公钥分发给其他节点，这样能避免机器之间的一些互相信任问题和访问权限问题。
```
# 切换到 hadoop 用户，生成ssh密钥（私钥和公钥）
ssh-keygen -t rsa

# 将ssh公钥拷贝给其他节点，一路回车
ssh-copy-id cluster1
ssh-copy-id cluster2
ssh-copy-id cluster3
```

<span id = "anchor1_6">&emsp;</span>

## 安装ntp时钟同步服务

&emsp;分布式并行操作往往会在集群时间不同步时出现问题，所以这里需要安装ntp时钟同步服务。
```
# ntpdate服务器每个节点都要安装：
yum install ntpdate
```
&emsp;以下操作只在cluster1上进行：
```
yum install ntp

# 配置ntp
vi /etc/ntp.conf
#注释掉
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
#在最后面添加
restrict default ignore
restrict 192.168.0.0 mask 255.255.0.0 nomodify notrap
server 127.127.1.0

#重启cluster1的ntp服务器
service ntpd restart
chkconfig ntpd on
```
&emsp;以下操作在cluster2、cluster3上进行：
```
# 设定每天00:00向服务器同步时间，并写入日志
# “crontab -e”命令会自动打开一个空文件，然后添加内容“0 0 * * * /usr/sbin/ntpdate cluster1>> /root/ntpd.log”
crontab -e
0 0 * * * /usr/sbin/ntpdate cluster1>> /root/ntpd.log

# 手动同步时间。在cluster2、cluster3上，输入“ntpdate cluster1”命令，将时间与cluster1同步。
ntpdate cluster1
```

<span id = "anchor2">&emsp;</span>

## 安装jdk


<span id = "anchor3">&emsp;</span>

## 在cluster2上安装MySQL


<span id = "anchor4">&emsp;</span>

## 安装Zookeeper


<span id = "anchor5">&emsp;</span>

## 安装Kafka


<span id = "anchor6">&emsp;</span>

## 安装Hadoop


<span id = "anchor7">&emsp;</span>

## 安装HBase


<span id = "anchor8">&emsp;</span>

## 安装Hive


<span id = "anchor9">&emsp;</span>

## 安装Spark


<span id = "anchor9_1">&emsp;</span>

### 安装Scala


<span id = "anchor9_2">&emsp;</span>

### 安装Spark


<span id = "anchor10">&emsp;</span>

## 安装Storm

