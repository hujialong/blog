---
layout: post
title:  "ldapsearch result collect to index"
date:   2017-07-06 11:48:54
categories: Splunk
tags: Tips
excerpt:
mathjax: true
---

当我用collect命令将ldapsearch的结果存入到index的时候遇到一些问题，collect命令并不能很好切分ldapsearch的结果。在collect之前使用table命令将能很好解决这个问题。



* 目录
{:toc}

## 问题描述
我使用以下SPL从ldap上将用户账号信息存放在ldap_summary上，以便日后通过查询使用。（也可以输出为csv，通过lookup来使用，这个不在这里讨论）
```
| ldapsearch domain="xxx-xx.in" search="(&(sAMAccoutName=*))" basedn="OU=AD Account,DC=xxx-xx,DC=in" attrs="sAMAccountName,cn,sn,giveName,emplyeeID,title,department,description,mail,telephoneNumber,memberOt,distinguishedName"
| collect index=ldap_summary
```
结果有一部分按行切分为独立的event，大部分，随机的多行（最多有3000行）合并为一个event，😢

## Workaround
通过多次测试，发现只要在collect命令之前，使用table命令，可以很好的解决这个问题。具体为什么会这样目前还不太清楚。

需改后的SPL如下：
```
| ldapsearch domain="xxx-xx.in" search="(&(sAMAccoutName=*))" basedn="OU=AD Account,DC=xxx-xx,DC=in" attrs="sAMAccountName,cn,sn,giveName,emplyeeID,title,department,description,mail,telephoneNumber,memberOt,distinguishedName"
| table sAMAccountName,cn,sn,giveName,emplyeeID,title,department,description,mail,telephoneNumber,memberOt,distinguishedName
| collect index=ldap_summary
```
