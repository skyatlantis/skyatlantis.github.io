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

<table border=0 cellpadding=0 cellspacing=0 width=509 style='border-collapse:
 collapse;table-layout:fixed;width:382pt;empty-cells: show;border-spacing: 0px;
 max-width: 800px !important;font-variant-ligatures: normal;font-variant-caps: normal;
 orphans: 2;text-align:start;widows: 2;-webkit-text-stroke-width: 0px;
 text-decoration-style: initial;text-decoration-color: initial;border-color:
 initial'>
 <col width=137 style='mso-width-source:userset;mso-width-alt:4384;width:103pt'>
 <col width=147 style='mso-width-source:userset;mso-width-alt:4704;width:110pt'>
 <col width=225 style='mso-width-source:userset;mso-width-alt:7200;width:169pt'>
 <tr height=19 style='height:14.25pt'>
  <td height=19 class=xl6919427 width=137 style='height:14.25pt;width:103pt;
  overflow-wrap: break-word;max-width: 800px'><span style='white-space:inherit !important'>&#31867;</span></td>
  <td class=xl6919427 width=147 style='border-left:none;width:110pt;overflow-wrap: break-word;
  max-width: 800px;border-left-color:initial'><span style='white-space:inherit !important'>&#20989;&#25968;&#21517;&#31216;</span></td>
  <td class=xl6919427 width=225 style='border-left:none;width:169pt;overflow-wrap: break-word;
  max-width: 800px;border-left-color:initial'><span style='white-space:inherit !important'>&#25551;&#36848;</span></td>
 </tr>
 <tr height=19 style='height:14.25pt'>
  <td rowspan=12 height=244 class=xl7019427 width=137 style='height:183.0pt;
  border-top:none;width:103pt;overflow-wrap: break-word;max-width: 800px;
  border-top-color:initial'><span style='white-space:inherit !important'>FileSystem</span></td>
  <td class=xl7019427 width=147 style='border-top:none;border-left:none;
  width:110pt;overflow-wrap: break-word;max-width: 800px;border-left-color:
  initial;border-top-color:initial'><span style='white-space:inherit !important'>initialize</span></td>
  <td class=xl7119427 width=225 style='border-top:none;border-left:none;
  width:169pt;overflow-wrap: break-word;max-width: 800px;border-left-color:
  initial;border-top-color:initial'><span style='white-space:inherit !important'>&#21021;&#22987;&#21270;<font
  class="font519427">HDFS Plugin</font><font class="font619427">&#12290;</font></span></td>
 </tr>
 <tr height=19 style='height:14.25pt'>
  <td height=19 class=xl7019427 width=147 style='height:14.25pt;border-top:
  none;border-left:none;width:110pt;overflow-wrap: break-word;max-width: 800px;
  border-left-color:initial;border-top-color:initial'><span style='white-space:
  inherit !important'>getFileBlockLocations</span></td>
  <td class=xl7119427 width=225 style='border-top:none;border-left:none;
  width:169pt;overflow-wrap: break-word;max-width: 800px;border-left-color:
  initial;border-top-color:initial'><span style='white-space:inherit !important'>&#33719;&#21462;&#25991;&#20214;&#20559;&#31227;&#20449;&#24687;&#12290;</span></td>
 </tr>
 <tr height=19 style='height:14.25pt'>
  <td height=19 class=xl7019427 width=147 style='height:14.25pt;border-top:
  none;border-left:none;width:110pt;overflow-wrap: break-word;max-width: 800px;
  border-left-color:initial;border-top-color:initial'><span style='white-space:
  inherit !important'>append</span></td>
  <td class=xl7119427 width=225 style='border-top:none;border-left:none;
  width:169pt;overflow-wrap: break-word;max-width: 800px;border-left-color:
  initial;border-top-color:initial'><span style='white-space:inherit !important'>&#36861;&#21152;&#20889;&#25991;&#20214;&#12290;</span></td>
 </tr>
 <tr height=19 style='height:14.25pt'>
  <td height=19 class=xl7019427 width=147 style='height:14.25pt;border-top:
  none;border-left:none;width:110pt;overflow-wrap: break-word;max-width: 800px;
  border-left-color:initial;border-top-color:initial'><span style='white-space:
  inherit !important'>create</span></td>
  <td class=xl7119427 width=225 style='border-top:none;border-left:none;
  width:169pt;overflow-wrap: break-word;max-width: 800px;border-left-color:
  initial;border-top-color:initial'><span style='white-space:inherit !important'>&#21019;&#24314;&#25991;&#20214;&#12290;</span></td>
 </tr>
 <tr height=19 style='height:14.25pt'>
  <td height=19 class=xl7019427 width=147 style='height:14.25pt;border-top:
  none;border-left:none;width:110pt;overflow-wrap: break-word;max-width: 800px;
  border-left-color:initial;border-top-color:initial'><span style='white-space:
  inherit !important'>delete</span></td>
  <td class=xl7119427 width=225 style='border-top:none;border-left:none;
  width:169pt;overflow-wrap: break-word;max-width: 800px;border-left-color:
  initial;border-top-color:initial'><span style='white-space:inherit !important'>&#21024;&#38500;&#25991;&#20214;&#25110;&#32773;&#25991;&#20214;&#22841;&#12290;</span></td>
 </tr>
 <tr height=19 style='height:14.25pt'>
  <td height=19 class=xl7019427 width=147 style='height:14.25pt;border-top:
  none;border-left:none;width:110pt;overflow-wrap: break-word;max-width: 800px;
  border-left-color:initial;border-top-color:initial'><span style='white-space:
  inherit !important'>getFileStatus</span></td>
  <td class=xl7119427 width=225 style='border-top:none;border-left:none;
  width:169pt;overflow-wrap: break-word;max-width: 800px;border-left-color:
  initial;border-top-color:initial'><span style='white-space:inherit !important'>&#33719;&#21462;&#25991;&#20214;&#30340;&#20449;&#24687;&#12290;</span></td>
 </tr>
 <tr height=19 style='height:14.25pt'>
  <td height=19 class=xl7019427 width=147 style='height:14.25pt;border-top:
  none;border-left:none;width:110pt;overflow-wrap: break-word;max-width: 800px;
  border-left-color:initial;border-top-color:initial'><span style='white-space:
  inherit !important'>listStatus</span></td>
  <td class=xl7119427 width=225 style='border-top:none;border-left:none;
  width:169pt;overflow-wrap: break-word;max-width: 800px;border-left-color:
  initial;border-top-color:initial'><span style='white-space:inherit !important'>&#21015;&#20030;&#25991;&#20214;&#30340;&#20449;&#24687;&#12290;</span></td>
 </tr>
 <tr height=19 style='height:14.25pt'>
  <td height=19 class=xl7019427 width=147 style='height:14.25pt;border-top:
  none;border-left:none;width:110pt;overflow-wrap: break-word;max-width: 800px;
  border-left-color:initial;border-top-color:initial'><span style='white-space:
  inherit !important'>open</span></td>
  <td class=xl7119427 width=225 style='border-top:none;border-left:none;
  width:169pt;overflow-wrap: break-word;max-width: 800px;border-left-color:
  initial;border-top-color:initial'><span style='white-space:inherit !important'>&#25171;&#24320;&#25991;&#20214;&#12290;</span></td>
 </tr>
 <tr height=19 style='height:14.25pt'>
  <td height=19 class=xl7019427 width=147 style='height:14.25pt;border-top:
  none;border-left:none;width:110pt;overflow-wrap: break-word;max-width: 800px;
  border-left-color:initial;border-top-color:initial'><span style='white-space:
  inherit !important'>rename</span></td>
  <td class=xl7119427 width=225 style='border-top:none;border-left:none;
  width:169pt;overflow-wrap: break-word;max-width: 800px;border-left-color:
  initial;border-top-color:initial'><span style='white-space:inherit !important'>&#37325;&#21629;&#21517;&#25991;&#20214;&#12290;</span></td>
 </tr>
 <tr height=19 style='height:14.25pt'>
  <td height=19 class=xl7019427 width=147 style='height:14.25pt;border-top:
  none;border-left:none;width:110pt;overflow-wrap: break-word;max-width: 800px;
  border-left-color:initial;border-top-color:initial'><span style='white-space:
  inherit !important'>mkdirs</span></td>
  <td class=xl7119427 width=225 style='border-top:none;border-left:none;
  width:169pt;overflow-wrap: break-word;max-width: 800px;border-left-color:
  initial;border-top-color:initial'><span style='white-space:inherit !important'>&#21019;&#24314;&#25991;&#20214;&#22841;&#12290;</span></td>
 </tr>
 <tr height=35 style='height:26.25pt'>
  <td height=35 class=xl7019427 width=147 style='height:26.25pt;border-top:
  none;border-left:none;width:110pt;overflow-wrap: break-word;max-width: 800px;
  border-left-color:initial;border-top-color:initial'><span style='white-space:
  inherit !important'>setOwner</span></td>
  <td class=xl7119427 width=225 style='border-top:none;border-left:none;
  width:169pt;overflow-wrap: break-word;max-width: 800px;border-left-color:
  initial;border-top-color:initial'><span style='white-space:inherit !important'>&#35774;&#32622;&#25991;&#20214;&#25110;&#32773;&#25991;&#20214;&#22841;&#30340;&#23646;&#20027;&#19982;&#23646;&#32452;&#12290;</span></td>
 </tr>
 <tr height=19 style='height:14.25pt'>
  <td height=19 class=xl7019427 width=147 style='height:14.25pt;border-top:
  none;border-left:none;width:110pt;overflow-wrap: break-word;max-width: 800px;
  border-left-color:initial;border-top-color:initial'><span style='white-space:
  inherit !important'>setPermission</span></td>
  <td class=xl7119427 width=225 style='border-top:none;border-left:none;
  width:169pt;overflow-wrap: break-word;max-width: 800px;border-left-color:
  initial;border-top-color:initial'><span style='white-space:inherit !important'>&#35774;&#32622;&#25991;&#20214;&#25110;&#32773;&#25991;&#20214;&#22841;&#30340;&#26435;&#38480;&#12290;</span></td>
 </tr>
 <tr height=19 style='height:14.25pt'>
  <td rowspan=2 height=38 class=xl6719427 width=137 style='border-bottom:1.0pt solid black;
  height:28.5pt;border-top:none;width:103pt;overflow-wrap: break-word;
  max-width: 800px;border-top-color:initial'><span style='white-space:inherit !important'>AbstractFileSystem</span></td>
  <td class=xl6519427 width=147 style='width:110pt;overflow-wrap: break-word;
  max-width: 800px;border-left-color:initial;border-top-color:initial'><span
  style='white-space:inherit !important'>createSymlink</span></td>
  <td class=xl6619427 width=225 style='width:169pt;overflow-wrap: break-word;
  max-width: 800px;border-left-color:initial;border-top-color:initial'><span
  style='white-space:inherit !important'>&#21019;&#24314;&#19968;&#20010;&#36719;&#36830;&#25509;&#12290;</span></td>
 </tr>
 <tr height=19 style='height:14.25pt'>
  <td height=19 class=xl6519427 width=147 style='height:14.25pt;width:110pt;
  overflow-wrap: break-word;max-width: 800px;border-left-color:initial;
  border-top-color:initial'><span style='white-space:inherit !important'>getFileLinkStatus</span></td>
  <td class=xl6619427 width=225 style='width:169pt;overflow-wrap: break-word;
  max-width: 800px;border-left-color:initial;border-top-color:initial'><span
  style='white-space:inherit !important'>&#33719;&#21462;&#32473;&#23450;&#36719;&#36830;&#25509;&#23545;&#24212;&#30340;&#25991;&#20214;&#20449;&#24687;&#12290;</span></td>
 </tr>
 <![if supportMisalignedColumns]>
 <tr height=0 style='display:none'>
  <td width=137 style='width:103pt'></td>
  <td width=147 style='width:110pt'></td>
  <td width=225 style='width:169pt'></td>
 </tr>
 <![endif]>
</table>

数据存储与任务调度机制对比

<table border=0 cellpadding=0 cellspacing=0 width=724 style='border-collapse:
 collapse;table-layout:fixed;width:543pt;empty-cells: show;border-spacing: 0px;
 max-width: 800px !important;font-variant-ligatures: normal;font-variant-caps: normal;
 orphans: 2;text-align:start;widows: 2;-webkit-text-stroke-width: 0px;
 text-decoration-style: initial;text-decoration-color: initial;border-color:
 initial'>
 <col width=260 style='mso-width-source:userset;mso-width-alt:8320;width:195pt'>
 <col width=273 style='mso-width-source:userset;mso-width-alt:8736;width:205pt'>
 <col width=191 style='mso-width-source:userset;mso-width-alt:6112;width:143pt'>
 <tr height=39 style='height:29.25pt'>
  <td height=39 class=xl654712 width=260 style='height:29.25pt;width:195pt;
  overflow-wrap: break-word;max-width: 800px'><span style='white-space:inherit !important'>&#23545;&#27604;&#39033;</span></td>
  <td class=xl654712 width=273 style='border-left:none;width:205pt;overflow-wrap: break-word;
  max-width: 800px;border-left-color:initial'><span style='white-space:inherit !important'>&#24320;&#28304;<font
  class="font54712">HDFS</font></span></td>
  <td class=xl684712 width=191 style='border-left:none;width:143pt;overflow-wrap: break-word;
  max-width: 800px;border-left-color:initial'><span style='white-space:inherit !important'>HDFS
  Plugin+OceanStor 9000</span></td>
 </tr>
 <tr height=71 style='height:53.25pt'>
  <td rowspan=2 height=144 class=xl674712 width=260 style='height:108.0pt;
  border-top:none;width:195pt;overflow-wrap: break-word;max-width: 800px;
  border-top-color:initial'><span style='white-space:inherit !important'>&#25968;&#25454;&#22359;&#23384;&#20648;&#26426;&#21046;</span></td>
  <td class=xl674712 width=273 style='border-top:none;border-left:none;
  width:205pt;overflow-wrap: break-word;max-width: 800px;border-left-color:
  initial;border-top-color:initial'><span style='white-space:inherit !important'>&#25991;&#20214;&#25353;&#25152;&#23450;&#20041;&#30340;&#22359;&#22823;&#23567;&#21644;&#21103;&#26412;&#26426;&#21046;&#23384;&#20648;&#22312;&#22810;&#20010;<font
  class="font64712">DataNode</font><font class="font74712">&#20013;&#12290;</font></span></td>
  <td class=xl674712 width=191 style='border-top:none;border-left:none;
  width:143pt;overflow-wrap: break-word;max-width: 800px;border-left-color:
  initial;border-top-color:initial'><span style='white-space:inherit !important'>&#22914;&#19979;&#22270;&#25152;&#31034;&#65292;&#25991;&#20214;&#34987;&#20999;&#20998;&#20026;<font
  class="font64712">Strip</font><font class="font74712">&#65292;&#36890;&#36807;&#30697;&#38453;&#36816;&#31639;&#29983;&#25104;&#33509;&#24178;&#20010;&#26657;&#39564;</font><font
  class="font64712">Strip</font><font class="font74712">&#65292;&#28982;&#21518;&#23558;&#36825;&#20123;</font><font
  class="font64712">Strip</font><font class="font74712">&#23384;&#20648;&#22312;&#22810;&#20010;&#33410;&#28857;&#20013;&#12290;</font></span></td>
 </tr>
 <tr height=73 style='height:54.75pt'>
  <td height=73 class=xl674712 width=273 style='height:54.75pt;border-top:none;
  border-left:none;width:205pt'><span style='white-space:inherit !important'>&#40664;&#35748;&#25968;&#25454;&#22359;&#22823;&#23567;&#20026;<font
  class="font64712">64MB</font><font class="font74712">&#65292;&#19968;&#33324;&#35774;&#32622;&#20026;</font><font
  class="font64712">64MB</font><font class="font74712">&#65374;</font><font
  class="font64712">128MB</font><font class="font74712">&#65307;&#40664;&#35748;&#37319;&#29992;&#19977;&#21103;&#26412;&#26426;&#21046;&#12290;</font></span></td>
  <td class=xl664712 width=191 style='border-top:none;border-left:none;
  width:143pt'><span style='white-space:inherit !important'>Strip<font
  class="font74712">&#22823;&#23567;&#21487;&#35774;&#32622;&#20026;</font><font
  class="font64712">256KB</font><font class="font74712">&#12289;</font><font
  class="font64712">128KB</font><font class="font74712">&#12289;</font><font
  class="font64712">32KB</font><font class="font74712">&#25110;</font><font
  class="font64712">16KB</font><font class="font74712">&#65292;&#20445;&#25252;&#32423;&#21035;&#21487;&#35774;&#32622;&#20026;</font><font
  class="font64712">N+1</font><font class="font74712">&#12289;</font><font
  class="font64712">N+2</font><font class="font74712">&#12289;</font><font
  class="font64712">N+3</font><font class="font74712">&#12289;</font><font
  class="font64712">N+4</font><font class="font74712">&#12289;</font><font
  class="font64712">N+2:1</font><font class="font74712">&#21644;</font><font
  class="font64712">N+3:1</font><font class="font74712">&#27169;&#24335;&#12290;</font></span></td>
 </tr>
 <tr height=52 style='height:39.0pt'>
  <td height=52 class=xl674712 width=260 style='height:39.0pt;border-top:none;
  width:195pt;overflow-wrap: break-word;max-width: 800px;border-top-color:initial'><span
  style='white-space:inherit !important'>&#20219;&#21153;&#35843;&#24230;&#31574;&#30053;</span></td>
  <td class=xl674712 width=273 style='border-top:none;border-left:none;
  width:205pt;overflow-wrap: break-word;max-width: 800px;border-left-color:
  initial;border-top-color:initial'><span style='white-space:inherit !important'>&#37319;&#29992;&#26412;&#22320;&#20248;&#20808;&#21407;&#21017;&#12289;&#25968;&#25454;&#23601;&#36817;&#33719;&#21462;&#21407;&#21017;&#65292;&#23613;&#21487;&#33021;&#23558;&#20219;&#21153;&#20998;&#37197;&#32473;&#38752;&#36817;&#25968;&#25454;&#25152;&#22312;&#33410;&#28857;&#20013;&#25191;&#34892;&#65292;&#20943;&#23569;&#32593;&#32476;&#20013;&#30340;&#25968;&#25454;&#25644;&#36801;&#37327;&#12290;</span></td>
  <td class=xl674712 width=191 style='border-top:none;border-left:none;
  width:143pt;overflow-wrap: break-word;max-width: 800px;border-left-color:
  initial;border-top-color:initial'><span style='white-space:inherit !important'>&#35745;&#31639;&#19982;&#23384;&#20648;&#29289;&#29702;&#20998;&#31163;&#65292;&#25152;&#26377;&#35745;&#31639;&#33410;&#28857;&#21040;&#23384;&#20648;&#33410;&#28857;&#30340;&#36335;&#24452;&#24320;&#38144;&#30456;&#31561;&#12290;</span></td>
 </tr>
 <![if supportMisalignedColumns]>
 <tr height=0 style='display:none'>
  <td width=260 style='width:195pt'></td>
  <td width=273 style='width:205pt'></td>
  <td width=191 style='width:143pt'></td>
 </tr>
 <![endif]>
</table>


