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

|类 | 函数名称 |  描述|  
|:-|:-|:-|
|FileSystem | initialize              | 初始化HDFS Plugin。 |
|FileSystem | getFileBlockLocations   | 获取文件偏移信息。 |
|FileSystem | append                  | 追加写文件。 |
|FileSystem | create                  | 创建文件。 |
|FileSystem | delete                  | 删除文件或者文件夹。 |
|FileSystem | getFileStatus | 获取文件的信息。 |
|FileSystem | listStatus | 列举文件的信息。 |
|FileSystem | open | 打开文件。 |
|FileSystem | rename | 重命名文件。 |
|FileSystem | mkdirs | 创建文件夹。 |
|FileSystem | setOwner | 设置文件或者文件夹的属主与属组。 |
|FileSystem | setPermission | 设置文件或者文件夹的权限。 |
|FileSystem | createSymlink | 创建一个软连接。 |
|FileSystem | getFileLinkStatus | 获取给定软连接对应的文件信息。 |

数据存储与任务调度机制对比

|对比项 | 开源HDFS |  HDFS Plugin+| 
|-|-|-|
|数据块存储机制 | 文件按所定义的块大小和副本机制存储在多个DataNode中 | 文件被切分为Strip，通过矩阵运算生成若干个校验Strip，然后将这些Strip存储在多个节点中 |
|数据块存储机制 | $默认数据块大小为64MB，一般设置为64MB～128MB；默认采用三副本机制 | Strip大小可设置为256KB、128KB、32KB或16KB，保护级别可设置为N+1、N+2、N+3、N+4、N+2:1和N+3:1模式 |
|任务调度策略 | 采用本地优先原则、数据就近获取原则，尽可能将任务分配给靠近数据所在节点中执行，减少网络中的数据搬迁量 | 计算与存储物理分离，所有计算节点到存储节点的路径开销相等 |


