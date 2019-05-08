---
layout: post
title:  cephfs与spark
date: 2019-04-30
categories: 分布式文件系统 大数据
tags: 文件系统
excerpt: cephfs与spark
---

前言
------
在Spark中使用CephFS

对接步骤
------

1.安装和部署好Ceph集群和Spark集群，并创建好CephFS文件系统实例

2.创建client客户端auth权限信息

3.安装相关软件包，包括下面部分

yum install ceph-fuse
yum install libcephfs_jni1
cp /usr/lib64/libcephfs_jni.so.1.0.0 /opt/ZDH/parcels/lib/hadoop/lib/native
cd /opt/ZDH/parcels/lib/hadoop/lib/native
ln -s libcephfs_jni.so.1.0.0 libcephfs_jni.so
cp hadoop-cephfs.jar /opt/ZDH/parcels/lib/hadoop/lib/
cp libcephfs.jar /opt/ZDH/parcels/lib/hadoop/lib/

4.修改hbase配置

hbase-env.sh的高级配置代码段（安全阀）：
hbase-env.sh 的 HBASE 客户端环境高级配置代码段（安全阀）：
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
hbase-site.xml高级配置代码段（安全阀）：
hbase-site.xml 的 HBASE 客户端高级配置代码段（安全阀）：
<property>
<name>fs.defaultFS</name>
<value>ceph://100.100.100.115:6789/</value>
</property>
<property>
<name>hbase.rootdir</name>
<value>ceph://100.100.100.115:6789/hbase</value>
</property>
<property>
<name>fs.ceph.impl</name>
<value>org.apache.hadoop.fs.ceph.CephFileSystem</value>
</property>
<property>
<name>ceph.conf.file</name>
<value>/etc/ceph/ceph.conf</value>
</property>
<property>
<name>ceph.mon.address</name>
<value>100.100.100.115:6789</value>
</property>
<property>
<name>ceph.auth.id</name>
<value>mr</value>
</property>
<property>
<name>ceph.auth.keyring</name>
<value>/etc/ceph/mr.keyring</value>
</property>
<property>
<name>ceph.data.pools</name>
<value>cephfs_data</value>
</property>
<property>
<name>ceph.replication</name>
<value>3</value>
</property>
<property>
<name>ceph.object.size</name>
<value>2097152</value>
</property>

5.修改spark配置

spark-env.sh 的 Spark 客户端高级配置代码段（安全阀）：
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
hive-site.xml文件自定义配置：
<property>
<name>fs.defaultFS</name>
<value>ceph://100.100.100.115:6789/</value>
</property>
<property>
<name>fs.ceph.impl</name>
<value>org.apache.hadoop.fs.ceph.CephFileSystem</value>
</property>
<property>
<name>ceph.conf.file</name>
<value>/etc/ceph/ceph.conf</value>
</property>
<property>
<name>ceph.mon.address</name>
<value>100.100.100.115:6789</value>
</property>
<property>
<name>ceph.auth.id</name>
<value>mr</value>
</property>
<property>
<name>ceph.auth.keyring</name>
<value>/etc/ceph/mr.keyring</value>
</property>
<property>
<name>ceph.data.pools</name>
<value>cephfs_data</value>
</property>
<property>
<name>ceph.replication</name>
<value>3</value>
</property>
<property>
<name>ceph.object.size</name>
<value>2097152</value>
</property>

6.配置环境变量

编辑文件vi /etc/profile，在文件最后添加如下命令：
export LD_LIBRARY_PATH=/opt/ZDH/parcels/lib/hadoop/lib/native
注：存储集群中每个节点都需要配置此环境变量


遇到问题
------

Q：其中一台服务器，插入光模块（ZTE光模块）后不识别

A：查看系统日志，显示failed to load because an unsupported SFP+ or QSFP
82599EB只支持与intel自家的光模块对接
执行以下命令规避：
modprobe -r ixgbe;modprobe ixgbe allow_unsupported_sfp=1

Q：vmax DAP在存储节点上安装salt服务失败

A：ceph的管理系统使用的是salt 2017，vmax使用salt 2015，将ceph管理系统卸载，重新通过DAP安装salt，还是失败
查看日志或salt安装脚本，只支持cgsl v3/v4、suse 11、CentOS release 5/6系统，rpm包通过scp拷贝到
其它节点安装的，每次安装前，会先将原来的salt卸载掉。
先将salt版本安装好，再将DAP中的salt安装脚本中卸载部分去掉，执行安装检测成功。
通过systemctl start salt-minion启动salt客户端后问题解决

Q：vmax DAP增加主机时，报ERROR: dynamic port configuration error,need 30100-50000. Please check manually.

A：编辑文件vi /etc/sysctl.conf，在文件最后增加：net.ipv4.ip_local_port_range = 30100 50000
然后执行sysctl -p命令

Q：其中一台存储节点，vmax DAP接管不了，提示连接超时

A：通过其中一个节点ssh到存储节点，正常。
查看系统日志，Unable to negotiate with 100.100.100.48 port 38083: no matching key exchange method found.
当前ssh服务端版本openssh-server-7.4p1-11，与DAP ssh客户端不匹配，卸载后，安装openssh-server-6.6.1p1-25解决

Q：重启系统后，ssh连不上，可以ping通

A：登录系统查看，sshd进程启动失败，报sshd配置文件错误，将sshd配置中最后一行PubkeyAcceptedKeyTypes=+ssh-dss注释掉，
重新启动sshd服务，可以正常启动。尝试用客户端连接，可以正常连接。
再次重启系统，发现/etc/ssh/sshd_config配置文件中最后一行又增加了PubkeyAcceptedKeyTypes=+ssh-dss，sshd进程启动失败。
发现在开机启动过程中，执行了 /usr/local/bin/os_enforce.sh 脚本，并强制修改了/etc/ssh/sshd_config配置文件，注释掉添加
PubkeyAcceptedKeyTypes=+ssh-dss的命令后，重启系统可以正常连接ssh

Q：执行java -version命令，显示：
Java HotSpot(TM) 64-Bit Server VM warning: INFO:
os::commit_memory(0x0000000540000000, 1073741824, 1073741824, 0) failed;
error='Cannot allocate memory' (errno=12); Cannot allocate large pages, falling back to regular pages

A：Hugepagesize被设置为1G，默认值为2048K
cat /proc/meminfo | grep Huge
AnonHugePages: 2818048 kB
HugePages_Total: 2
HugePages_Free: 1
HugePages_Rsvd: 0
HugePages_Surp: 0
Hugepagesize: 1048576 kB
编辑vi /boot/grub2/grub.cfg，将配置文件中default_hugepagesz=1G hugepagesz=1G hugepages=0 去掉，
重启系统java -version命令正常

Q：cephfs对接spark后启动服务，报错：Caused by: java.lang.NoClassDefFoundError:
Could not initialize class com.ceph.fs.CephMount

A：spark找不到ceph的动态库，配置环境变量LD_LIBRARY_PATH=/opt/ZDH/parcels/lib/hadoop/lib/native后问题解决
（在/etc/profile最后一行增加此环境变量）
使用前，需要重启服务；如果是使用shell命令，如sparksql，需要执行source /etc/profile或者重启登录ssh

Q：通过sparksql进行数据访问时，报无权限错误
A：之前从hdfs拷贝数据到cephfs时，使用的是root用户，所以所有文件的owner都是root，而vmax中访问数据时使用的是mr用户。
将vmax源数据目录及文件owner全部修改为mr后问题解决

Q：通过sparksql读数据时正常，写数据时，报以下错误：
most recent failure: Lost task 1.3 in stage 1.0 (TID 2140, clove117): java.io.IOException: Permission denied
at com.ceph.fs.CephMount.native_ceph_lstat(Native Method)
at com.ceph.fs.CephMount.lstat(CephMount.java:457)
at org.apache.hadoop.fs.ceph.CephTalker.lstat(CephTalker.java:163)
at org.apache.hadoop.fs.ceph.CephFileSystem.getFileStatus(CephFileSystem.java:208)
at org.apache.hadoop.fs.FileSystem.exists(FileSystem.java:1412)
at org.apache.hadoop.fs.ceph.CephFileSystem.create(CephFileSystem.java:373)

A：查看cephfs jni日志，lstat接口返回-13，可以确认是权限问题。
上层应用传下来的mode是416，用的是root用户创建的目录，在这个目录下再用mr用户创建目录或者新建文件都会报权限错误，
可以确认上层的mode参数传错了。上层传mode参数时，把x权限加上去就没问题了。
由于涉及到修改spark的代码，暂时将cephfs client配置项client_permissions修改为false，关闭权限检测规避。

Q：启动hbase服务失败，报错如下：
2017-12-19 13:19:41,295 FATAL org.apache.hadoop.hbase.master.HMaster: Failed to become active master
java.lang.NullPointerException
at org.apache.hadoop.fs.Globber.doGlob(Globber.java:238)
at org.apache.hadoop.fs.Globber.glob(Globber.java:151)
at org.apache.hadoop.fs.FileSystem.globStatus(FileSystem.java:1637)
at org.apache.hadoop.hbase.util.FSUtils.getTableDirs(FSUtils.java:1372)
at org.apache.hadoop.hbase.master.MasterFileSystem.checkTempDir(MasterFileSystem.java:515)
at org.apache.hadoop.hbase.master.MasterFileSystem.createInitialFileSystemLayout(MasterFileSystem.java:158)
at org.apache.hadoop.hbase.master.MasterFileSystem.<init>(MasterFileSystem.java:130)
at org.apache.hadoop.hbase.master.HMaster.finishActiveMasterInitialization(HMaster.java:649)
at org.apache.hadoop.hbase.master.HMaster.access$500(HMaster.java:187)
at org.apache.hadoop.hbase.master.HMaster$1.run(HMaster.java:1756)
at java.lang.Thread.run(Thread.java:745)
  
A：hadoop common中liststatus接口，当路径 不存在时，要求返回FileNotFoundException异常，
cephfs实现的liststatus接口中，当路径 不存在时，返回的是NULL
用最新的hadoop-cephfs.jar包替换掉原来的，关闭vmax所有服务，再启动后正常

Q：开始执行spark任务，发现没有对ceph的读写流量，ceph运行正常

A：按我们目前的配置，现在这些卡住的任务要10%，也就是一个核就够了，但是因为这些任务的配置文件里，
有一句set spark.executor.cores=2, 限制了最低资源要求，导致在我们这种CPU很少的环境里出现了10%的核不够2个情况的，
然后就一直waiting不执行了
要想办法从4000多个任务里，把这种类型的任务都挑出来
