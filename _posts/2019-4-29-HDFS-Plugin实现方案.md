---
layout: post
title:   HDFS Plugin实现方案
date: 2019-04-29
categories: 分布式文件系统 大数据
tags: 文件系统
excerpt: HDFS Plugin实现方案
---

前言
------


HDFS介绍
------
HDFS（Hadoop Distributed File System）是开源Hadoop的主要组件之一，适用于大规模数据集存储。HDFS容量利用率低、
成本过高问题，当前已成为Hadoop系统的首要问题所在。HDFS采用副本机制（一般为三副本）存储数据。以10PB业务数据为例，
在采用三副本机制时，实际存储到硬盘中的数据量将放大为30PB，容量利用率仅为33%。考虑硬件采购、机房空间占用、能源消
耗等多方面因素，实际存储成本是有效数据存储成本的2倍以上，且随着数据存储量的不断增大，此问题愈加突出。

虽然有厂商开始提议向开源社区提供Erasure Code技术方案，但是由此导致剧增的分布式事务复杂度、平衡性能和可靠性因素，
让此方案离成熟商用还有相当长一段时间。通过对接专业NAS存储系统，无疑是各Hadoop厂商当前快速解决此问题的良方。
(参考:华为OceanStor 9000近期推出私有HDFS Plugin软件插件，通过将其部署在Hadoop节点与客户端中，实现将HDFS协议的文件
访问请求转换为NFS协议请求。)我们则考虑实现私有的HDFS Plugin软件插件，将HDFS协议文件访问请求转换为CephFS协议请求。

HDFS Plugin实现
------
HDFS Plugin通过继承开源HDFS提供的FileSystem类和AbstractFileSystem类，对外提供如下表所示的文件访问接口。
类实现的接口如下：


