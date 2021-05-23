---
title: PostgreSQL常用命令记录
date: 2020-09-12 15:00:15
tags: [PostgreSQL]
---

PostgreSQL版本是12.2，**psql** 是 PostgreSQL 中的一个命令行交互式客户端工具，它允许你交互地键入 SQL 命令，然后把它们发送给 PostgreSQL 服务器，再显示 SQL 或命令的结果。输入的内容允许来自一个文件，此外它还提供了一些元命令和多种类似 shell 的特性来实现书写脚本，以及对大量任务的自动化工作。约定[]表示其中的内容可选，{}和|表示需选择一个候选，…表示前面的元素可以重复。中文官方链接，地址[指路](http://www.postgres.cn/docs/12/index.html)。<!--more-->

## 角色

### 登录

1. 使用用默认超级用户`postgres`登录，输入`psql -U postgres`，再输入安装时设置的超级用户密码，回车即可登入。 
2. 假设现有拥有登录权限用户为user和数据库exampledb，则登录命令为`psql -U user -d exampledb`，-U参数指定登录用户，-d参数指定待操作的数据库。
3. 假设现有数据库exampledb和登录权限的用户为user，且**user正好为当前操作系统用户**，此时可以省略-U参数，登录命令为`psql -d exampledb`。

### 创建角色

1. 在超级用户登录状态下，创建新角色，`CREATE ROLE name`，创建名称为name的角色，没有任何权限。

2. 在超级用户登录状态下，创建新角色，`CREATE USER name`，创建名称为name的角色，除了登录权限外，没有任何权限，该行命令等价于`CREATE ROLE name LOGIN`。

3. 在超级用户登录状态下，创建一个新超级用户，使用`CREATE ROLE name SUPERUSER`。一个数据库超级用户除了登录会绕开所有的权限检查，**使用超级用户操作是危险的操作，推荐用受限的角色完成大部分工作**。

4. 创建超级用户的详细命令以及对应的解释如下。

```sql
   CREATE ROLE name [ [ WITH ] option [ ... ] ]
   
   where option：
   
         SUPERUSER | NOSUPERUSER --该角色是否是超级用户
       | CREATEDB | NOCREATEDB --该角色是否可以创建数据库
       | CREATEROLE | NOCREATEROLE --该角色是否可以创建新角色
       | INHERIT | NOINHERIT --该角色是否继承自其他角色
       | LOGIN | NOLOGIN --该角色是否具有登录权限
       | REPLICATION | NOREPLICATION --决定一个角色是否为复制角色，能以复制模式（物理复制或者逻辑复制）连接到服务器以及创建或者删除复制槽，这是非常高特权，默认为否
       | BYPASSRLS | NOBYPASSRLS --是否一个角色可以绕过每一条行级安全性（RLS）策略
       | CONNECTION LIMIT connlimit --如果角色能登录，这指定该角色能建立多少并发连接
       | [ ENCRYPTED ] PASSWORD 'password' | PASSWORD NULL --设置角色的口令（口令只对具有LOGIN属性的角色有用，如果没有指定口令，口令将被设置为空并且该用户的口令认证总是会失败。也可以用PASSWORD NULL显式地写出一个空口令。
       | VALID UNTIL 'timestamp' --设置一个日期和时间，在该时间点之后角色的口令将会失效
       | IN ROLE role_name [, ...] --IN ROLE子句列出一个或多个现有的角色，新角色将被立即作为新成员加入到这些角色中
       | IN GROUP role_name [, ...] --是IN ROLE的废弃写法
       | ROLE role_name [, ...] --ROLE子句列出一个或者多个现有角色，它们会被自动作为成员加入到新角色中
       | ADMIN role_name [, ...] --略
       | USER role_name [, ...] --是ROLE子句的废弃写法
       | SYSID uid --向后兼容
```
###    修改角色权限
在创建角色时未指定的权限，都可以在超级用户登录状态下得以修改，比如更改角色名称，更改/移除一个角色的口令，更改一个角色的失效日期，使角色具有创建数据库的权限等等。
```sql
--示例
--更改一个角色的口令
ALTER ROLE davide WITH PASSWORD 'hu8jmn3';

--移除一个角色的口令
ALTER ROLE davide WITH PASSWORD NULL;

--让一个角色能够创建新的数据库：
ALTER ROLE davide CREATEDB;

--更改一个口令的失效日期，指定该口令应该在2020年5月4日中午（在一个比UTC快 1 小时的时区）过期
ALTER ROLE davide VALID UNTIL 'May 4 12:00:00 2020 +1';

--更改一个角色的名称
ALTER ROLE davide RENAME TO new_davide;
```

完整的角色权限修改语句和参数，其中大部分与创建角色时的参数相同，含义也一致，[参考地址](http://www.postgres.cn/docs/12/sql-alterrole.html )。

### 删除角色

由于角色可以拥有数据库对象并且能持有访问其他对象的特权，删除一个角色 常常并非一次`DROP ROLE`就能解决。该用户所拥有的对象必须被删除或转移给其他角色。

```sql
ALTER TABLE bobs_table OWNER TO alice;
--类似的数据完全安排给其他角色后
DROP ROLE davide;
```

## 数据库

### 创建数据库

1. 超级用户或普通角色登录状态下，`CREATE DATABASE dbname`，创建数据库默认为当前登录用户所有。
2. 超级用户或普通角色登录状态下，`CREATE DATABASE dbname OWNER rolename` ，创建数据库并指定数据库拥有者。

### 删除数据库

```sql
DROP DATABASE dbname;
```

## 表

### 创建表

```sql
DROP TABLE tablename;
```

### 删除表

```sql
DROP TABLE tablename;
```

## 其余常用命令

`\l`查看已经存在的数据库

`\c dbname` 进入数据库

`\d`查看所有表

`\d tablename`查看指定表信息

`\du`查看所有角色