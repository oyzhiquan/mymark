---
title: vnpy-行情数据导入到MongoDB
date: 2019-07-26 10:17:56
tags:vnpy
---

## 前提

安装MongoDB，新建vnpy库。

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

打开行情记录窗口，设置需要记录的合约行情。