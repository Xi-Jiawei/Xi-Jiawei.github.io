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

&emsp;架构分为四层：OLTP层、数据仓库层（含ODS）、数据集市层、应用层。新型的数据仓库架构与传统数据仓库架构的唯一区别就是数据仓库层的不同。新型的数据仓库架构除了使用关系型数据库，同时将大数据技术应用于数据仓库中。

&emsp;从图中可以看到，搭建Hadoop平台需要的组件包括Zookeeper、Kafka、Hadoop、Hive、Spark、Hbase以及MySQL关系型数据库等。

&emsp;一个简单的Hadoop集群至少应该有三台机器，在每台机器都安装Zookeeper、Kafka、Hadoop、Hive、Spark、Hbase，在其中一台机器上安装MySQL。好啦，话不多说，下面我们开始搭建Hadoop集群。

---

&emsp;

# 概览

1. [准备工作](#anchor1)

   + [准备虚拟机](#anchor1_1)

   + [网络配置](#anchor1_2)

      - [关闭防火墙](#anchor1_2_1)

      - [网卡配置](#anchor1_2_2)

      - [修改host](#anchor1_2_3)

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

2. [安装Storm](#anchor2)

&emsp;

---

<span id = "anchor1">&emsp;</span>

# 准备工作


