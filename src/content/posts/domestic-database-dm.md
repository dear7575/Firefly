---
title: ✨国产达梦数据库安装教程
published: 2024-04-22
pinned: false
description: ""
tags: [国产, DM]
category: 技术分享
draft: false
image: ./images/firefly1.avif
---

# 国产达梦数据库安装教程

> 全程需要使用Linux命令进行操作

```markdown
# 1.安装流程

## 1.创建安装用户组 dinstall -- 删除用户组 groupdel

```shell
root 用户操作 groupadd dinstall
```

## 2.创建安装用户 dmdba -- 删除用户 userdel

```shell
root 用户操作 useradd -g dinstall -m -d /data/dmdba -s /bin/bash dmdba 
```

## 3.初始化用户密码

```shell
root 用户操作 passwd dmdba
```

## 4.创建挂载目录

```shell
root 用户操作 mkdir /data/DM8
```

## 5.挂载驱动 -- 卸载驱动 umount /挂载路径

```shell
root 用户操作 mount dm8_setup_arm64_ent_8.1.1.56_20200113.iso /data/DM8/
```

## 6.文件夹授权

```shell
root 用户操作 chown -R dmdba:dinstall /data
```

## 7.切换用户

```shell
su dmdba
```

## 8.进入挂载目录并执行DMInstall.bin

```shell
1.进入 cd /data/DM8/
2.执行 ./DMInstall.bin -i
```

## 9.创建数据库

```shell
dmdba 用户操作
1.进入 cd /data/dmdba/dmdbms/bin
2.执行 ./dminit path=/data/dmdba/dmdbms/data db_name=DAMENG instance_name=DMSERVER port_num=5236
```

## 10.创建服务

```shell
root 用户操作
1.进入 cd /data/dmdba/dmdbms/script/root
2.执行 ./dm_service_installer.sh -t dmserver -p DMSERVER -dm_ini /data/dmdba/dmdbms/data/DAMENG/dm.ini
```

## 10.开机启动

```shell
root 用户操作
1.开机启动 systemctl enable DmServiceDMSERVER
2.启动 systemctl start DmServiceDMSERVER
3.状态 systemctl status DmServiceDMSERVER
```

# 2.配置路径

dmdbms/data/DAMENG/dm.ini

## GROUP\_OPT\_FLAG = 0

分组项优化参数开关。

```shell
0:不优化;
1:非MYSQL兼容模式下,支持查询项不是GROUP BY表达式;
2:外层分组项下放到内层派生表中提前分组优化;
4:表示对于多级分区,并行下允许尝试不生成多个AGR;
8:进行哈希分组时，依赖分组项的个项进行分组。支持使用上述有效值的组合值
```

## COMPATIBLE\_MODE = 4

是否兼容其他数据库模式。配合`ENABLE_BLOB_CMP_FLAG `使`GROUP BY`可以用非查询字段

```shell
0:不兼容,
1:兼容SQL92标准,
2:兼容ORACLE,
3:兼容MS SQL SERVER,
4:兼容MYSQL,
5:兼容DM6,
```

## ENABLE\_BLOB\_CMP\_FLAG = 1

是否支持大字段类型的比较。

```shell
0:不支持;
1:支持,此时DISTINCT、ORDER BY、分析函数和集函数支持对大字段进行处理
```

# 3.新用户授权ALL异常

sp\_set\_para\_value(1,'ENABLE\_DDL\_ANY\_PRIV',1);
