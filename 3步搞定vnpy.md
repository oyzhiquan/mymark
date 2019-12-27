---
title: 3步搞定vnpy
date: 2019-07-26 08:55:11
tags: vnpy
---

**前提**

Anaconda要先安装好，注意选择Python3.7的最新版。

从github上下载最新的vnpy

**创建vnpy环境**

Anaconda Prompt下进入vnpy所在目录，执行：

```
conda create --name test-vnpy python=3.7
```

激活：

```
conda activate vnpy-test
```

**编译vnpy**

vnpy提供了一键安装依赖和编译的脚本，在Anaconda Prompt下执行：

```
install.bat
```

**安装vnpy-test的Spyder**

打开Anaconda Navigator工具，选择到vnpy-test，安装Spyder。

**测试**

打开Spyder(vnpy-test)，新建Python文件，执行以下代码：

```python
# -*- coding: utf-8 -*-
"""
Created on Fri Jul 26 08:44:53 2019

@author: Administrator
"""

from vnpy.event import EventEngine
from vnpy.trader.engine import MainEngine
from vnpy.trader.ui import MainWindow, create_qapp
from vnpy.gateway.ctp import CtpGateway
from vnpy.app.cta_strategy import CtaStrategyApp

def main():
    """Start VN Trader"""
    qapp = create_qapp()

    event_engine = EventEngine()
    main_engine = MainEngine(event_engine)

    main_engine.add_gateway(CtpGateway)
    main_engine.add_app(CtaStrategyApp)

    main_window = MainWindow(main_engine, event_engine)
    main_window.showMaximized()

    qapp.exec()

if __name__ == "__main__":
    main()
```



