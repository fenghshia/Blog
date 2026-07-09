---
title: "Vaultwarden迁移到Cloudflare"
description: 
date: 2026-06-18T15:01:36+08:00
image: 
math: 
license: 
comments: true
draft: true
categories:
    - Cloudflare
    - Vaultwarden
tags:
    - Cloudflare
    - Vaultwarden
build:
    list: always    # Change to "never" to hide the page from the list
---

本文章讨论利用Cloudflare D1来构建Vaultwarden的Server服务

## 将原来Vaultwarden连接的Postgres库迁移到Cloudflare D1

1. 将 PostgreSQL 数据迁移至本地 SQLite

```shell
pgloader postgresql://用户:密码@IP:端口/数据库名 sqlite://本地生成的文件名.db
```

2. 将 SQLite 导出为 D1 兼容的 SQL 文本文件

```shell
apt -y install sqlite3
sqlite3 vaultwarden_temp.db .dump > vaultwarden_d1_import.sql
```

打开生成的 vaultwarden_d1_import.sql, 必须删除文件开头带有 `PRAGMA` 的行, 删除文件开头的 `BEGIN TRANSACTION;` 和文件结尾的 `COMMIT;`

因为 Wrangler 会自己管理事务，嵌套事务会导致 D1 导入失败

3. 将表定义语句与数据插入语句分离

复制 vaultwarden_d1_import.sql 中所有 `create table` 语句到 vaultwarden_d1_schema.sql

复制 vaultwarden_d1_import.sql 中所有 `insert into` 语句到 vaultwarden_d1_data.sql, 在首行加一句`PRAGMA foreign_keys = OFF;`无视外键关联

4. 导入到 Cloudflare D1

```shell
# 登录 Cloudflare
npx wrangler login
# 导入表定义
npx wrangler d1 execute vaultwarden-db --file=./vaultwarden_d1_schema.sql --remote
# 导入表数据
npx wrangler d1 execute vaultwarden-db --file=./vaultwarden_d1_data.sql --remote
```

## 迁移附件到 Cloudflare R2

1. 创建 R2 桶

```shell
# 登录 Cloudflare
npx wrangler login
# 创建一个用于存附件的 R2 桶
npx wrangler r2 bucket create vw-attachments
```

2. 创建 Cloudflare R2 的 S3 兼容凭证

R2 -> 右侧的 Manage R2 API Tokens（管理 R2 API 令牌）

点击 Create API Token（创建 API 令牌）

选择 Admin Read & Write（管理员读写权限）

点击创建，安全保存以下三个关键信息 : `Access Key ID（访问键 ID）` `Secret Access Key（机密访问键）` `Jurisdiction-specific Endpoint（S3 客户端终结点 URL，格式通常为 https://<您的账户ID>.r2.cloudflarestorage.com）`

3. 配置 rclone 一键同步

```shell
# 安装
apt -y install rclone
# 配置
rclone config
NOTICE: Config file "./rclone.conf" not found - using defaults
No remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
# 创建新配置
n/s/q> n
# 填写配置名
name> r2
# 存储类型填s3
Storage> s3
# 填Other其他
provider> Other
# 填1手动填写凭据
Choose a number from below, or type in your own value
 1 / Enter AWS credentials in the next step
   \ "false"
 2 / Get AWS credentials from the environment (env vars or IAM)
   \ "true"
env_auth> 1
# 填写`Access Key ID（访问键 ID）`
AWS Access Key ID.
Leave blank for anonymous access or runtime credentials.
Enter a string value. Press Enter for the default ("").
access_key_id> XXX
# 填写`Secret Access Key（机密访问键）`
AWS Secret Access Key (password)
Leave blank for anonymous access or runtime credentials.
Enter a string value. Press Enter for the default ("").
secret_access_key> XXX
# 区域空
region>
# 填写`Jurisdiction-specific Endpoint（S3 客户端终结点 URL，格式通常为 https://<您的账户ID>.r2.cloudflarestorage.com）`
endpoint> https://<您的账户ID>.r2.cloudflarestorage.com
# 空
location_constraint>
# 空
acl>
# 编辑高级配置填n否
Edit advanced config? (y/n)
y) Yes
n) No (default)
y/n> n
# 填y保存配置
y) Yes this is OK (default)
e) Edit this remote
d) Delete this remote
y/e/d> y
# 填q退出配置编辑
e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q
```

4. 执行同步操作

```shell
rclone sync ./attachments r2:vw-attachments --progress
```
