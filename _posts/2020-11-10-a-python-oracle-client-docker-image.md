---
title: "制作一个 Oracle 数据库的 python 客户端镜像"
subtitle: ""
layout: post
author: "jlshix"
header-style: text
tags:
  - python
  - Oracle
  - 数据库
  - docker
---

`Oracle` 作为最成熟的商业数据库, 常年在 [DB Engines Ranking](https://db-engines.com/en/ranking)
榜上有名, 但是对新手来讲真的不够友好, 官方提供的 python 驱动 `cx_Oracle` 是需要额外依赖
Oracle 客户端的. 因此在使用的时候需要手动制作镜像.

首先, 在只 `pip install cx_Oracle` 的情况下, 执行数据库相关操作, 会报错, 示例如下:

```
Traceback (most recent call last):
  File "oracle_db.py", line 11, in <module>
    db = cx_Oracle.connect(ORACLE_USER, ORACLE_PASSWORD, ORACLE_URI)
cx_Oracle.DatabaseError: DPI-1047: Cannot locate a 64-bit Oracle Client library: "libclntsh.so: cannot open shared object file: No such file or directory". See https://cx-oracle.readthedocs.io/en/latest/user_guide/installation.html for help
```

根据安装指南, 必须安装 [Oracle Instant Client](https://www.oracle.com/database/technologies/instant-client.html),
选择 `Basic Light` 即可, 就这基础且轻量的, 删掉无用文件都要 100M -_-|||

根据[安装指南](https://docs.oracle.com/en/database/oracle/oracle-database/19/lnoci/instant-client.html#GUID-D0042396-7C99-4450-962C-6E3DBF6EFD41) 的说明, 
需要以下几步:

- 安装 `libaio`, 如 `apt install libaio1` 或 `yum install libaio`

- 在 [下载列表](https://www.oracle.com/database/technologies/instant-client/downloads.html) 选择需要下载的版本进行下载

- 解压后得到文件夹, 如 `instantclient_19_9`, 进入文件夹, 删除无用文件减少镜像大小: `rm -f *jdbc* *occi* *mysql* *README *jar uidrvci genezi adrci`

- 注册并更新配置, 注意必须使用绝对路径:
```shell
echo /abs/path/to/instantclient_19_9 > /etc/ld.so.conf.d/oracle-instantclient.conf
ldconfig
```

这样再执行 `cx_Oracle` 的脚本就可以成功执行了.

完整的 `Dockerfile` 如下, 假设删除无用文件的 `instantclient_19_9` 文件夹
与 `requirements.txt` 文件都在 `deploy` 文件夹下:

```Dockerfile
# python_cx_Oracle:base

# ref https://stackoverflow.com/questions/58992954/install-oracle-instant-client-into-docker-container-for-python-cx-oracle

FROM python:3.7.3

COPY ./deploy /deploy

WORKDIR /deploy

# 1. 替换国内源; 2. 安装 libaio; 3. 注册 instantclient; 4. 安装 requirements
RUN sed -i s@/deb.debian.org/@/mirrors.aliyun.com/@g /etc/apt/sources.list \
    && apt-get clean \
    && apt-get update && apt-get install -y libaio1 \
    && echo /deploy/instantclient_19_9 > /etc/ld.so.conf.d/oracle-instantclient.conf \
    && ldconfig \
    && pip install --no-cache-dir -i https://mirrors.aliyun.com/pypi/simple -r requirements.txt
```
