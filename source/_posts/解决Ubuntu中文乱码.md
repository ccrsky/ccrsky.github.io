---
title: 解决Ubuntu中文乱码
date: 2018-05-07 01:52:56
tags:
---

最近使用ubuntu系统搭建CI环境，发现不支持中文，中文显示？？？，解决步骤如下：

1. 安装中文支持包language-pack-zh-hans
    ```sh
    sudo apt-get install language-pack-zh-hans
    ```
2. 修改/etc/environment
    ```sh
    # 文件追加如下内容
    LANG="zh_CN.UTF-8"
    LANGUAGE="zh_CN:zh:en_US:en"
    ```
3. 修改/var/lib/locales/supported.d/local
    ```sh
    en_US.UTF-8 UTF-8
    zh_CN.UTF-8 UTF-8
    zh_CN.GBK GBK
    zh_CN GB2312
    ```
4. 执行命令
    ```sh
    sudo locale-gen
    ```

至此中文乱码问题即刻解决~