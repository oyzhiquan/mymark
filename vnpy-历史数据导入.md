---
title: vnpy-历史数据导入
date: 2019-07-26 10:17:56
tags:vnpy
---

## 前提

安装MongoDB，新建vnpy库。

rq是收费的，可以先申请试用。

也可以从tushare获取数据，然后插入到MongoDB（需要重写代码）。

## 配置

在全局配置界面，配置MongoDB相关项。

配置示例如下：

```
database.driver<str> mongodb
database.database<str> vnpy
database.host<str> localhost
database.port<int> 27017
```

## 导入数据

出现问题重新安装

```
pip install --user --extra-index-url https://rquser:ricequant99@py.ricequant.com/simple/ rqdatac
```

rq试用（3个月有效），修改配置为license中用户、密码：

```
rqdata.username<str> license
rqdata.password<str> L2EqqpIGSMoiH9ta0fQ1nwx3B96_IBtASTKwEfJQiVo746fqQQqTBZayfs57VF3CSmRjHwstrrobEhQCBfZ6JEa60gXa8WCSSLhXzregnoW0MrAKXu1VhcWKAr2I-l2iNyl3YKmNVDzg4QuI837l6Wd8CdWL3AI25JN2uhuzumo=epTSLj3hxrGKrHgpdkwwUYMItPfmIBeFtEB9t8TgBwpkdpIREBS9Omq_dPAsHXSnp1DZ77zRsl7O4lzVftrQykmUA3ITxwO2MJSw22UgCg-wlWAVmBWDQ9YR9zsXf84GnpeN9RBksh_hLuOJrEexAZT7wWLbirVno_vCvMXGYLw=
```

数据格式：

```
1min数据
{
    "_id" : ObjectId("5d3a6430ccdd0b2177854bb8"),
    "datetime" : ISODate("2019-07-26T10:21:00.000Z"),
    "interval" : "1m",
    "symbol" : "IF1912",
    "close_price" : 3827.4,
    "exchange" : "CFFEX",
    "high_price" : 3827.4,
    "low_price" : 3827.4,
    "open_interest" : 9854.0,
    "open_price" : 3827.4,
    "volume" : 0.0
}
1d数据
{
    "_id" : ObjectId("5d3a98c3ccdd0b21778b2868"),
    "datetime" : ISODate("2016-07-26T00:00:00.000Z"),
    "interval" : "d",
    "symbol" : "IF88",
    "close_price" : 3247.0,
    "exchange" : "CFFEX",
    "high_price" : 3247.0,
    "low_price" : 3196.8,
    "open_interest" : 0.0,
    "open_price" : 3199.4,
    "volume" : 11772.0
}
1h数据
{
    "_id" : ObjectId("5d3a9a17ccdd0b21778b2f48"),
    "datetime" : ISODate("2019-06-26T09:30:00.000Z"),
    "interval" : "1h",
    "symbol" : "IF88",
    "close_price" : 3771.6,
    "exchange" : "CFFEX",
    "high_price" : 3775.0,
    "low_price" : 3748.6,
    "open_interest" : 0.0,
    "open_price" : 3753.0,
    "volume" : 24169.0
}

```



