---
title: ✨MySQL Linux 通用版离线安装
published: 2024-04-22
pinned: false
description: "记录 MySQL 5.7 Linux 通用版离线安装流程，包含依赖安装、用户创建、初始化、配置和远程访问设置。"
tags: [MySQL, Linux]
category: 技术分享
draft: false
image: ../uploads/20260110/wl.jpg
---

# MySQL 5.7.41 安装与远程配置指南

## 一、准备工作

1. 确保依赖已安装：
   ```bash
   yum install -y libaio numactl   # CentOS / RHEL
   apt-get install -y libaio1      # Debian / Ubuntu
   ```
2. 创建 MySQL 专用用户：
   ```bash
   groupadd mysql
   useradd -r -g mysql -s /bin/false mysql
   ```

---

## 二、解压安装包

1. 上传 `mysql-5.7.41-linux-glibc2.12-x86_64.tar.gz` 到宿主机，例如 `/usr/local/src`：
   ```bash
   cd /usr/local/src
   tar -zxvf mysql-5.7.41-linux-glibc2.12-x86_64.tar.gz
   ```
2. 移动到安装目录：
   ```bash
   mv mysql-5.7.41-linux-glibc2.12-x86_64 /usr/local/mysql
   ```
3. 修改权限：
   ```bash
   chown -R mysql:mysql /usr/local/mysql
   ```

---

## 三、初始化数据库

1. 创建数据目录：

   ```bash
   mkdir -p /data/mysql
   chown -R mysql:mysql /data/mysql
   ```
2. 初始化数据库：

   ```bash
   cd /usr/local/mysql
   bin/mysqld --initialize --user=mysql --datadir=/data/mysql
   ```

   > 这里会生成临时 root 密码，注意保存。
   >

---

## 四、配置文件

> 新建 `/etc/my.cnf`：

```
[mysqld]
basedir=/usr/local/mysql
datadir=/data/mysql
socket=/tmp/mysql.sock
user=mysql
port=3306
symbolic-links=0
character-set-server=utf8mb4
max_connections=200
bind-address=0.0.0.0   # 开启远程访问

[client]
socket=/tmp/mysql.sock
default-character-set=utf8mb4
```

---

## 五、启动 MySQL

1. 注册服务脚本：
   ```bash
   cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
   chmod +x /etc/init.d/mysqld
   ```
2. 启动服务：
   ```bash
   service mysqld start
   # 或
   /etc/init.d/mysqld start
   ```
3. 开机自启：
   ```bash
   chkconfig --add mysqld     # CentOS 6
   systemctl enable mysqld    # CentOS 7+
   ```

---

## 六、登录并修改密码

1. 使用临时密码登录：
   ```bash
   /usr/local/mysql/bin/mysql -u root -p
   ```
2. 修改 root 密码：
   ```sql
   ALTER USER 'root'@'localhost' IDENTIFIED BY '你的新密码';
   ```

---

## 七、开启远程登录

### 方式一：创建新用户（推荐）

```sql
CREATE USER 'root'@'%' IDENTIFIED BY '你的密码';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

### 方式二：修改已有 root 用户

```sql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '你的密码' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

---

## 八、防火墙设置

* **CentOS / RHEL7+**：
  ```bash
  firewall-cmd --zone=public --add-port=3306/tcp --permanent
  firewall-cmd --reload
  ```
* **iptables（老系统）**：
  ```bash
  iptables -A INPUT -p tcp --dport 3306 -j ACCEPT
  service iptables save
  service iptables restart
  ```

---

## 九、远程连接测试

> 在另一台机器执行：

```bash
mysql -h 宿主机IP -P3306 -u root -p
```

---

## 十、安全建议

* 不要直接开放 `root@%`，建议创建单独的业务用户：
  ```sql
  CREATE USER 'app_user'@'%' IDENTIFIED BY '安全密码';
  GRANT ALL PRIVILEGES ON mydb.* TO 'app_user'@'%';
  FLUSH PRIVILEGES;
  ```

---

## 十一、小版本升级执行脚本

```bash
#!/bin/bash
# ==============================================
# MySQL 小版本升级脚本（例如 5.7.41 -> 5.7.43）
# 用法:
#   ./upgrade_mysql.sh <mysql_tar_path> <install_dir> <root_password>
#
# 示例:
#   ./upgrade_mysql.sh /opt/mysql-5.7.43-linux-glibc2.12-x86_64.tar.gz /data/mysql 123456
# ==============================================

set -e

MYSQL_TAR=$1
INSTALL_DIR=$2
MYSQL_PASS=$3
BACKUP_DIR=/backup/mysql_backup_$(date +%F_%H%M%S)

if [ -z "$MYSQL_TAR" ] || [ -z "$INSTALL_DIR" ] || [ -z "$MYSQL_PASS" ]; then
  echo "❌ 用法: $0 <mysql_tar_path> <install_dir> <root_password>"
  exit 1
fi

if [ ! -f "$MYSQL_TAR" ]; then
  echo "❌ 找不到安装包: $MYSQL_TAR"
  exit 1
fi

# Step 1: 逻辑备份数据库
echo "📦 导出所有数据库到 $BACKUP_DIR/dump.sql ..."
mkdir -p $BACKUP_DIR
$INSTALL_DIR/bin/mysqldump -uroot -p"$MYSQL_PASS" --all-databases --single-transaction --quick --routines --events > $BACKUP_DIR/dump.sql

# Step 2: 物理备份整个安装目录
echo "📂 备份 MySQL 安装目录到 $BACKUP_DIR/mysql_dir ..."
cp -a $INSTALL_DIR $BACKUP_DIR/mysql_dir

# Step 3: 停止 MySQL
echo "⏹ 停止 MySQL..."
systemctl stop mysql || pkill -9 mysqld || true

# Step 4: 解压新版本到临时目录
echo "📦 解压新版本 MySQL ..."
TMP_DIR=$(mktemp -d)
tar -xf $MYSQL_TAR -C $TMP_DIR
MYSQL_NEW_DIR=$(tar -tf $MYSQL_TAR | head -1 | cut -f1 -d"/")

# Step 5: 替换二进制文件（保留 data、配置文件）
echo "🔄 替换二进制文件..."
rsync -av --delete --exclude=data --exclude=mysql.sock --exclude=mysql.pid \\
  $TMP_DIR/$MYSQL_NEW_DIR/ $INSTALL_DIR/

chown -R mysql:mysql $INSTALL_DIR

# Step 6: 启动 MySQL
echo "🚀 启动 MySQL..."
systemctl start mysql
sleep 8

# Step 7: 运行 mysql_upgrade
echo "🔧 执行 mysql_upgrade ..."
$INSTALL_DIR/bin/mysql_upgrade -uroot -p"$MYSQL_PASS"

# Step 8: 显示版本
$INSTALL_DIR/bin/mysql -uroot -p"$MYSQL_PASS" -e "SELECT VERSION();"

echo "✅ MySQL 升级完成！"
echo "   原安装目录已备份到: $BACKUP_DIR"
echo "   当前安装目录: $INSTALL_DIR"

```

## 十二、自动安装脚本

```bash
#!/bin/bash
# ==============================================
# 通用 MySQL 安装脚本（离线安装 + 自动生成 my.cnf + systemd + 免临时密码）
# 用法:
#   ./install_mysql.sh <mysql_tar_path> <root_password> [install_dir]
#
# 示例:
#   ./install_mysql.sh /opt/mysql-5.7.41-linux-glibc2.12-x86_64.tar.gz 123456
#   ./install_mysql.sh /opt/mysql-5.7.43-linux-glibc2.12-x86_64.tar.gz Abc@1234 /data/mysql
# ==============================================

set -e

MYSQL_TAR=$1
MYSQL_PASS=$2
INSTALL_DIR=${3:-/usr/local/mysql}   # 默认安装目录
MYSQL_DATADIR="$INSTALL_DIR/data"
MYSQL_USER="mysql"

if [ -z "$MYSQL_TAR" ] || [ -z "$MYSQL_PASS" ]; then
  echo "❌ 用法: $0 <mysql_tar_path> <root_password> [install_dir]"
  exit 1
fi

if [ ! -f "$MYSQL_TAR" ]; then
  echo "❌ 找不到安装包: $MYSQL_TAR"
  exit 1
fi

# Step 1: 停止旧 MySQL
echo "⏹ 停止 MySQL..."
pkill -9 mysqld 2>/dev/null || true
systemctl stop mysqld 2>/dev/null || true

# Step 2: 删除旧目录
echo "🧹 删除旧 MySQL 目录 $INSTALL_DIR ..."
rm -rf $INSTALL_DIR

# Step 3: 创建 mysql 用户
id -u $MYSQL_USER &>/dev/null || useradd -r -s /sbin/nologin $MYSQL_USER

# Step 4: 解压指定版本
echo "📦 解压 MySQL 包..."
tar -xvf $MYSQL_TAR -C /usr/local/
MYSQL_NEW_DIR=$(tar -tf $MYSQL_TAR | head -1 | cut -f1 -d"/")
mv /usr/local/$MYSQL_NEW_DIR $INSTALL_DIR

# Step 5: 初始化数据目录（使用 --initialize-insecure，root 密码为空）
echo "📂 初始化数据目录..."
rm -rf $MYSQL_DATADIR
mkdir -p $MYSQL_DATADIR
chown -R $MYSQL_USER:$MYSQL_USER $INSTALL_DIR
chmod 750 $INSTALL_DIR
chmod 750 $MYSQL_DATADIR

$INSTALL_DIR/bin/mysqld --initialize-insecure --user=$MYSQL_USER \\
  --basedir=$INSTALL_DIR --datadir=$MYSQL_DATADIR

# Step 5.1: 创建日志文件，确保可写
touch $INSTALL_DIR/mysql.err
chown $MYSQL_USER:$MYSQL_USER $INSTALL_DIR/mysql.err

# Step 6: 生成 /etc/my.cnf
echo "📝 生成 /etc/my.cnf ..."
cat > /etc/my.cnf <<EOF
[client]
port = 3306
socket = $INSTALL_DIR/mysql.sock

[mysqld]
basedir = $INSTALL_DIR
datadir = $MYSQL_DATADIR
socket = $INSTALL_DIR/mysql.sock
pid-file = $INSTALL_DIR/mysql.pid
log-error = $INSTALL_DIR/mysql.err
symbolic-links = 0
sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
EOF

# Step 6.1: 创建 systemd 服务文件
echo "📝 生成 /etc/systemd/system/mysql.service ..."
cat > /etc/systemd/system/mysql.service <<EOF
[Unit]
Description=MySQL Server
After=network.target

[Service]
User=$MYSQL_USER
Group=$MYSQL_USER
ExecStart=$INSTALL_DIR/bin/mysqld_safe --defaults-file=/etc/my.cnf
ExecStop=$INSTALL_DIR/bin/mysqladmin --defaults-file=/etc/my.cnf -uroot -p$MYSQL_PASS shutdown
LimitNOFILE=5000
Restart=on-failure

[Install]
WantedBy=multi-user.target
EOF

# 重新加载 systemd 并开机自启
systemctl daemon-reexec
systemctl enable mysql

# Step 7: 启动 MySQL
echo "🚀 启动 MySQL..."
systemctl start mysql
sleep 8

# Step 8: 设置 root 密码
echo "🔧 设置 root 密码..."
$INSTALL_DIR/bin/mysql -uroot --socket=$INSTALL_DIR/mysql.sock \\
  -e "ALTER USER 'root'@'localhost' IDENTIFIED BY '$MYSQL_PASS'; FLUSH PRIVILEGES;"

# Step 9: 开启远程访问
echo "🌐 开启远程访问..."
$INSTALL_DIR/bin/mysql -uroot -p"$MYSQL_PASS" --socket=$INSTALL_DIR/mysql.sock \\
  -e "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '$MYSQL_PASS' WITH GRANT OPTION; FLUSH PRIVILEGES;"

# Step 10: 显示版本
$INSTALL_DIR/bin/mysql -uroot -p"$MYSQL_PASS" --socket=$INSTALL_DIR/mysql.sock -e "SELECT VERSION();"

echo "✅ MySQL 安装完成！"
echo "   版本包: $MYSQL_TAR"
echo "   安装目录: $INSTALL_DIR"
echo "   root 密码: $MYSQL_PASS"
echo "   使用 systemctl 管理 MySQL: systemctl start|stop|restart mysql"

```

## 十三、自动重置脚本

```bash
#!/bin/bash
# ==============================================
# MySQL 重置脚本
# 用法:
#   ./reset_mysql.sh [install_dir]
#
# 示例:
#   ./reset_mysql.sh              # 默认 /usr/local/mysql
#   ./reset_mysql.sh /data/mysql  # 自定义安装目录
# ==============================================

set -e

INSTALL_DIR=${1:-/usr/local/mysql}
MYSQL_USER="mysql"

echo "⚠️ 警告: 该操作将删除 MySQL 安装目录及数据目录!"
read -p "确定要清空 $INSTALL_DIR 吗? (yes/no): " CONFIRM
if [ "$CONFIRM" != "yes" ]; then
  echo "❌ 已取消操作"
  exit 1
fi

# Step 1: 停止 MySQL
echo "⏹ 停止 MySQL..."
pkill -9 mysqld 2>/dev/null || true
systemctl stop mysqld 2>/dev/null || true

# Step 2: 删除安装目录
if [ -d "$INSTALL_DIR" ]; then
  echo "🧹 删除目录: $INSTALL_DIR"
  rm -rf "$INSTALL_DIR"
fi


# Step 3: 删除 socket 和 PID 文件（防止残留）
rm -f /tmp/mysql.sock
rm -f /tmp/mysql.pid
rm -f /var/run/mysqld/mysqld.pid
rm -f /var/run/mysqld/mysqld.sock

# Step 4: 保证 mysql 用户存在
id -u $MYSQL_USER &>/dev/null || useradd -r -s /sbin/nologin $MYSQL_USER

echo "✅ MySQL 已重置完成，$INSTALL_DIR 已清空"

```
