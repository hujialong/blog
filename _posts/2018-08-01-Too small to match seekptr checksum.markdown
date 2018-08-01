---
layout: post
title:  "Too small to match seekptr checksum"
date:   2018-08-01 10:08:54
categories: Splunk
tags: Tips
excerpt:
mathjax: true
---

用Splunk Monitor文件（xml、html等）的时候，如果文件开始的256bytes是一样的话，Splunk默认会认为是一个文件，从而不会input进index。




* 目录
{:toc}
## 问题描述
用Splunk Monitor文件（xml、html等）的时候，如果文件开始的256bytes是一样的话，Splunk默认会认为是一个文件，从而不会input进index。我们会在`$SPLUNK_HOME/var/log/splunkd.log`中看到如下的报错：
```
splunkd.log:09-22-2017 01:30:04.522 +0000 ERROR TailReader - File will not be read, is too small to match seekptr checksum (file=/home/ubuntu/logs/json-bowman-<myserver>1-bowman-worker_search-1.log). Last time we saw this initcrc, filename was different. You may wish to use larger initCrcLen for this sourcetype, or a CRC salt on this source. Consult the documentation or file a support case online at http://www.splunk.com/page/submit_issue for more info.
```
### 分析
Splunk默认通过校验文件头的256bytes，来判断文件是否是新文件，其控制Splunk对文件进行校验的参数在`inputs.conf`文件中：
```
initCrcLength = 256     # default value:256,     
```

## Workaround
在`inputs.conf`文件中加入`crcSalt = <SOURCE>`,这样`inputs.conf`文件看起来会是这样：
```
[monitor://\\dhcpsrv\dhcp$]
disabled = 0
followTail = 0
host = dhcpsrv
index = default
sourcetype = ms_dhcpd
_whitelist = DhcpSrvLog.(Sun|Mon|Tue|Wed|Thu|Fri|Sat)$
crcSalt = <SOURCE>
```
