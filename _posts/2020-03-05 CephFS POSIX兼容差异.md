---
layout: post
title:  CephFS POSIX兼容差异
date: 2020-03-11
categories: CephFS POSIX
tags: CephFS POSIX
excerpt: CephFS POSIX
---

CephFS
---
CephFS是强一致性文件系统，比NFS弱一致性文件系统要好


1.write()写入异常时，不具有原子性
  比如write在写入文件时，open后写入了8M文件，并使用O_SYNC标志,正好这个时候失败了，那么这个操作就不具有原子性，
  可能大部分的文件系统都这个问题，包括本地文件系统。
2.write()写操作时，边界的写操作不一定是原子操作
  在写操作同时发生的情况下，超出对象边界的写操作不一定是原子操作。 
  例如，写作者A写“ aa | aa”，写作者B同时写“ bb | bb”（其中“ |”是对象边界），写“ aa | bb”而不是适当的“ aa | aa”或“ bb | bb”。
3.telldir和seekdir接口，返回值不一定有效
  telldir和seekdir会返回目录的offset，但由于cephfs的目录随时做了分片处理，所以可能不能返回一个稳定的offset值
4.sparse file，稀疏文件系统在stats统计st_blocks时，不准确
  CephFS创建文件支持稀疏文件，示例：dd if=/dev/zero of=fedora2.img  bs=1 count=0 seek=8G
  CephFS没有显示跟踪file哪些位置写入了数据，哪些位置没有写入数据，但st_blocks的值，一直是通过file size/block_size来统计的，
  所以就算CephFS支持稀疏写入，但du查看的文件仍然是创始的文件大小
5.mmap()功能同步问题
  在多个节点之间mmap CephFS文件系统内容时，节点之间的内存信息无法实时同步，会出现多节点之间的内存一致性问题
6.CephFS特殊目录.snap目录
  .snap目录排除在readdir()之外,但如果要创建一样的目录应用，则需要使用client_snapdir来修改这个名称
