---
layout: post
title:  "在Splunk中对数据进行脱敏"
date:   2018-08-01 15:25:34
categories: Splunk
tags: Tips
excerpt:
mathjax: true
---
在我的很多客户都问过，如何在Splunk中对数据进行脱敏。通常这么做的目的是为了保护个人隐私信息或者符合安全法规的要求。在这篇博客中，我们会讨论在Splunk中对数据进行脱敏的一些方法。




* 目录
{:toc}
## 实验数据

在这篇博客中，我们所有操作都基于以下数据来实现：

```
sourcetype=vendor_sales
sample_log:
[15/Dec/2016:07:36:56] VendorID=5069 Code=A AcctID=4397877757846973
[15/Dec/2016:07:21:15] VendorID=3108 Code=M AcctID=6347267702474393
```
其中我们需要将AcctID的前12为隐藏起来，只保留最后4位置

## SEDCMD

Splunk可以在index phase的时候使用SEDCMD对数据进行脱敏，基于以上实验数据，我们可以用以下配置来实现：

`props.conf`
```
[vendor_sales]
SEDCMD-lacct = s/AcctID=\d{12}(\d{4})/AcctID=xxxxxxxxxxxx\1/g
```
这样Splunk在index数据之前会对AcctID进行脱敏，脱敏后的结果为：
```
sample_log-Masked:
[15/Dec/2016:07:36:56] VendorID=5069 Code=A AcctID=xxxxxxxxxxxx6973
[15/Dec/2016:07:21:15] VendorID=3108 Code=M AcctID=xxxxxxxxxxxx4393
```

## TRANSFORMS
在Splunk中同时可以通过transform来实现：

在`props.conf`中定义transform：
`props.conf`
```
[vendor_sales]
TRANSFORMS-anonymize = AcctID-anonymizer
```
同时在`transforms.conf`文件中定义脱敏的正则：

`transforms.conf`
```
[AcctID-anonymizer]
REGEX = (?m)^(.*)AcctID=\d{12}+(\d{4}.*)$
FORMAT = $1AcctID=xxxxxxxxxxxx$2
DEST_KEY = _raw
```
这两种办法都是在index之前对数据进行脱敏，在Splunk平台是看到的是脱敏后的结果。可是往往有时候用户会有其他的一下情况出现：
- 一部分有权限的人能够看到敏感数据
- 没有权限的人员只能看到脱敏后的数据
这个时候，如果在index之前就脱敏，那就无法实现了。下面将介绍在Splunk平台如何给予用户权限对数据进行脱敏。

## 给予用户权限的数据脱敏

- 同时转发两份数据到index，其中一份是没有脱敏的数据，另外一个脱敏后的数据，分别放在不通的index里面。（这种方式会消耗License）然后使用权限控制，让不通权限的用户访问不通的index。
- 运行一个schedule search，对数据进行脱敏，并生产summary index。同时让没有权限查看敏感数据的用户只能看到summary index的数据。

脱敏SPL样例：
```
index="sales" AcctID=* | rex field=AcctID mode=sed s/(\d{12})\d{4}/\1xxxx/g | table Vendor VendorID VendorCountry VendorCity Code AcctID productId product_name price sale_price
```

## 参考正则表达式
- 信用卡/银行卡
```
[creditcard-anonymizer]
REGEX=(?ms)(.*)\b(?:4[0-9]{8}(?:[0-9]{3})?|5[1-5][0-9]{10}|6(?:011|5[0-9]{2})[0-9]{8}|3[47][0-9]{9}|3(?:0[0-5]|[68][0-9])[0-9]{7}|(?:2131|1800|35\d{3})\d{7})(\d{4}\b.*)
FORMAT= $1###SCRUBBED###$2
DEST_KEY = _raw
```
ccpartner
```
(?:4[0-9]{12}(?:[0-9]{3})?|5[1-5][0-9]{14}|6(?:011|5[0-9][0-9])[0-9]{12}|3[47][0-9]{13}|3(?:0[0-5]|[68][0-9])[0-9]{11}|(?:2131|1800|35\d{3})\d{11})
```
