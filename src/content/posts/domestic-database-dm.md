---
title: ✨国产达梦数据库安装教程
published: 2024-04-22
pinned: false
description: "记录达梦数据库在 Linux 环境下的安装、基础配置，以及使用 dexp/dimp 在 x86 与 ARM Docker 容器间导出和导入业务模式。"
tags: [国产, DM]
category: 技术分享
draft: false
image: ../uploads/20260110/mt.png
---

# 国产达梦数据库安装教程

> 全程需要使用Linux命令进行操作

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

> dmdbms/data/DAMENG/dm.ini

## GROUP\_OPT\_FLAG = 0

> 分组项优化参数开关。

```shell
0:不优化;
1:非MYSQL兼容模式下,支持查询项不是GROUP BY表达式;
2:外层分组项下放到内层派生表中提前分组优化;
4:表示对于多级分区,并行下允许尝试不生成多个AGR;
8:进行哈希分组时，依赖分组项的个项进行分组。支持使用上述有效值的组合值
```

## COMPATIBLE\_MODE = 4

> 是否兼容其他数据库模式。配合`ENABLE_BLOB_CMP_FLAG `使`GROUP BY`可以用非查询字段

```shell
0:不兼容,
1:兼容SQL92标准,
2:兼容ORACLE,
3:兼容MS SQL SERVER,
4:兼容MYSQL,
5:兼容DM6,
```

## ENABLE\_BLOB\_CMP\_FLAG = 1

> 是否支持大字段类型的比较。

```shell
0:不支持;
1:支持,此时DISTINCT、ORDER BY、分析函数和集函数支持对大字段进行处理
```

# 3.新用户授权ALL异常

> sp\_set\_para\_value(1,'ENABLE\_DDL\_ANY\_PRIV',1);

# 4.数据库逻辑导出与导入

下面记录使用达梦 `dexp`、`dimp` 工具，将 x86 容器中的 `APP_SCHEMA_1`、`APP_SCHEMA_2` 两个业务模式迁移到 ARM 容器 `<target-container>` 的操作。

> x86 与 ARM 之间应使用逻辑导出和导入，不要直接复制数据库物理数据目录。本文只迁移两个业务模式，因此不能使用 `FULL=Y` 整库导出。

## 1.迁移前检查

源库和目标库应使用兼容的数据库初始化参数，至少确认字符集和页大小一致。之前的迁移环境中，两端均为 `UNICODE=1`、`PAGE()=32768`。

同时检查容器内的客户端字符集：

```shell
docker exec <source-container> sh -lc 'locale charmap'
docker exec <target-container> sh -lc 'locale charmap'
```

两端都应输出 `UTF-8`。如果容器中没有可用的 `en_US.UTF-8`，可以使用两端都存在的 `C.UTF-8`。执行 `dexp` 和 `dimp` 时也要显式传入相同的语言环境，避免中文内容乱码。

达梦 Docker 镜像中的工具通常位于 `/opt/dmdbms/bin`。按照本文前面步骤进行原生安装时，工具目录为 `/data/dmdba/dmdbms/bin`，请根据实际安装路径调整。

## 2.从源容器导出业务模式

将 `<source-container>` 替换为源端达梦容器名称，将 `APP_SCHEMA_1`、`APP_SCHEMA_2` 替换为实际业务模式名，并将 `your_password` 替换为源库 `SYSDBA` 的实际密码：

```shell
docker exec \
  -e LANG=C.UTF-8 \
  -e LC_ALL=C.UTF-8 \
  <source-container> sh -lc '
    mkdir -p /tmp/dm-migrate
    cd /opt/dmdbms/bin
    ./dexp \
      USERID=SYSDBA/your_password@127.0.0.1:5236 \
      DIRECTORY=/tmp/dm-migrate \
      FILE=app_schemas.dmp \
      LOG=app_schemas_dexp.log \
      SCHEMAS=APP_SCHEMA_1,APP_SCHEMA_2 \
      ROWS=Y \
      GRANTS=Y \
      INDEXES=Y \
      CONSTRAINTS=Y \
      TRIGGERS=Y
  '
```

参数说明：

- `SCHEMAS`：只导出指定模式，不导出整个数据库。
- `ROWS=Y`：导出表数据。
- `GRANTS=Y`：导出对象授权信息。
- `INDEXES=Y`：导出索引。
- `CONSTRAINTS=Y`：导出约束。
- `TRIGGERS=Y`：导出触发器。
- `DIRECTORY`：DMP 文件和日志在容器内的保存目录。

导出完成后检查日志末尾是否存在失败对象，然后将 DMP 文件复制到宿主机：

```shell
mkdir -p ./dm-migrate

docker cp \
  <source-container>:/tmp/dm-migrate/app_schemas.dmp \
  ./dm-migrate/app_schemas.dmp

docker cp \
  <source-container>:/tmp/dm-migrate/app_schemas_dexp.log \
  ./dm-migrate/app_schemas_dexp.log
```

## 3.将 DMP 文件复制到目标容器

```shell
docker exec <target-container> mkdir -p /tmp/dm-migrate

docker cp \
  ./dm-migrate/app_schemas.dmp \
  <target-container>:/tmp/dm-migrate/app_schemas.dmp
```

导入前，应确认目标库已经创建 `APP_SCHEMA_1`、`APP_SCHEMA_2` 用户及其表空间，并为用户设置已知密码。使用 `DBMS_METADATA.GET_DDL('USER', ...)` 获取用户定义时，密码会被隐藏为 `<PASSWORD>`，不能依靠该结果恢复原密码。

## 4.导入到 ARM 目标容器

将 `<target-container>` 替换为目标端达梦容器名称，并将 `your_password` 替换为目标库 `SYSDBA` 的实际密码：

```shell
docker exec \
  -e LANG=C.UTF-8 \
  -e LC_ALL=C.UTF-8 \
  <target-container> sh -lc '
    cd /opt/dmdbms/bin
    ./dimp \
      USERID=SYSDBA/your_password@127.0.0.1:5236 \
      FILE=/tmp/dm-migrate/app_schemas.dmp \
      DIRECTORY=/tmp/dm-migrate \
      LOG=app_schemas_dimp.log \
      SCHEMAS=APP_SCHEMA_1,APP_SCHEMA_2 \
      ROWS=Y \
      GRANTS=Y \
      INDEXES=Y \
      CONSTRAINTS=Y \
      TRIGGERS=Y
  '
```

如果目标库已有同名表，先备份现有数据，再根据需要增加以下参数：

```shell
TABLE_EXISTS_ACTION=REPLACE
```

`REPLACE` 会删除并重建已存在的表，属于破坏性操作。首次导入空库时不需要设置；重复导入时还可以根据需求选择 `SKIP`、`APPEND` 或 `TRUNCATE`。

导入结束后，将日志复制到宿主机保存：

```shell
docker cp \
  <target-container>:/tmp/dm-migrate/app_schemas_dimp.log \
  ./dm-migrate/app_schemas_dimp.log
```

## 5.导入结果检查

进入目标库：

```shell
docker exec -it <target-container> \
  /opt/dmdbms/bin/disql SYSDBA/your_password@127.0.0.1:5236
```

检查两个模式的表和对象数量：

```sql
SELECT OWNER, COUNT(*) AS TABLE_COUNT
FROM DBA_TABLES
WHERE OWNER IN ('APP_SCHEMA_1', 'APP_SCHEMA_2')
GROUP BY OWNER;

SELECT OWNER, COUNT(*) AS OBJECT_COUNT
FROM DBA_OBJECTS
WHERE OWNER IN ('APP_SCHEMA_1', 'APP_SCHEMA_2')
GROUP BY OWNER;

SELECT OWNER, OBJECT_NAME, OBJECT_TYPE, STATUS
FROM DBA_OBJECTS
WHERE OWNER IN ('APP_SCHEMA_1', 'APP_SCHEMA_2')
  AND STATUS <> 'VALID';
```

还需要抽查业务表行数、中文字段、视图、存储过程、触发器和应用连接，确认导入日志中没有失败对象。

## 6.中文乱码处理

如果导入后中文乱码，不要继续复用原 DMP 文件，应先修复源端和目标端的 locale，再重新导出和导入：

```shell
docker exec \
  -e LANG=C.UTF-8 \
  -e LC_ALL=C.UTF-8 \
  <container> sh -lc 'locale charmap'
```

确认输出为 `UTF-8` 后，再使用相同的 `LANG=C.UTF-8`、`LC_ALL=C.UTF-8` 执行 `dexp` 和 `dimp`。如果问题仍存在，再使用高权限账号核对两端的 `UNICODE`、`PAGE`、`COMPATIBLE_MODE`、`BLANK_PAD_MODE` 和 `LENGTH_IN_CHAR` 等初始化参数。

## 7.参考资料

- [达梦技术文档：dexp 逻辑导出](https://eco.dameng.com/document/dm/zh-cn/pm/dexp-logical-export.html)
- [达梦技术文档：dimp 逻辑导入](https://eco.dameng.com/document/dm/zh-cn/pm/dimp-logical%20import.html)
