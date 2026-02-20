---
title: SQL 快速拾遗
published: 2026-01-25
updated: 2026-01-25
description: 'Learn SQL in Y minutes'
image: ''
tags: [SQL]
category: 'SQL'
draft: false
---

# SQL

## 🛠 Quick Start

By the way, I use Arch Linux. 🤓

```zsh
sudo pacman -Syu
sudo pacman -S mysql
sudo mysqld --initialize --user=mysql --basedir=/usr --datadir=/var/lib/mysql
sudo systemctl start mysqld.service
sudo systemctl status mysqld.service
sudo systemctl enable mysqld.service
```

:::warning
`mysqld` 的时候要留意，输出的内容含有 `root` 用户的默认密码．

别 `clear` 把密码吃了，小馋猫 😋
:::

:::tip
不想用图形界面的话，`mycli` 可以提供语法高亮和命令补全．

::github{repo="dbcli/mycli"}

```zsh
yay -S mycli
```
:::

登录 `root` 账户，修改密码．

```zsh
mysql -u root -p [-h localhost -P 3306]
```
```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'xxxxxx';
```

> 「与其登录一个 `root`，不如创造一个 `root` 😁」

```sql
CREATE USER 'nisemono'@'%' IDENTIFIED BY 'yyyyyy';
GRANT ALL PRIVILEGES ON *.* TO 'nisemono'@'%' WITH GRANT OPTION;
EXIT;
```
```zsh
mysql -u nisemono -p [-h localhost -P 3306]
```

## ✨ Examples

### 展示

假设现在我的数据库情况是：

```sql
SHOW DATABASES;
USE study_sql;
SHOW TABLES;
SELECT * FROM hello;
```

```text
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| study_sql          |
| sys                |
+--------------------+

5 rows in set
Time: 0.002s

You are now connected to database "shy_db" as user "nisemono"
Time: 0.000s

+---------------------+
| Tables_in_study_sql |
+---------------------+
| hello               |
+---------------------+

1 row in set
Time: 0.004s

+----+-----------+------------+-----+--------+
| id | user_name | real_name  | age | gender |
+----+-----------+------------+-----+--------+
| 1  | nisemono  | shy-vector | 20  | 女     |
| 2  | ASTERIAX  | asteriax   | 21  | 男     |
+----+-----------+------------+-----+--------+

2 rows in set
Time: 0.003s
```

### `database` 和 `table`

我想把 `study_sql` 这个数据库名称改成 `shy_db`．

```sql
CREATE DATABASE IF NOT EXISTS shy_db;
EXIT;
```

```zsh
mysqldump -u root -p -h localhost -P 3306 --set-gtid-purged=OFF study_sql > /tmp/study_sql_backup.sql
mysql -u root -p -h localhost -P 3306 shy_db < /tmp/study_sql_backup.sql
mysql -u nisemono -p -h localhost -P 3306
```

```sql
DROP DATABASE IF EXISTS study_sql;
SHOW DATABASES;
USE shy_db;
SHOW TABLES;
SELECT * FROM hello;
```

```text
You're about to run a destructive command.
Do you want to proceed? (y/n): y
Your call!
Query OK, 1 row affected
Time: 0.042s

+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| shy_db             |
| sys                |
+--------------------+

5 rows in set
Time: 0.003s

You are now connected to database "shy_db" as user "nisemono"
Time: 0.000s

+------------------+
| Tables_in_shy_db |
+------------------+
| hello            |
+------------------+

1 row in set
Time: 0.004s

+----+-----------+------------+-----+--------+
| id | user_name | real_name  | age | gender |
+----+-----------+------------+-----+--------+
| 1  | nisemono  | shy-vector | 20  | 女     |
| 2  | ASTERIAX  | asteriax   | 21  | 男     |
+----+-----------+------------+-----+--------+

2 rows in set
Time: 0.002s
```

与此同时，我又想把表名 `hello` 改成 `hello_tb`．

```sql
RENAME TABLE hello TO hello_tb;
SHOW TABLES;
```

```text
Query OK, 0 rows affected
Time: 0.031s

+------------------+
| Tables_in_shy_db |
+------------------+
| hello_tb         |
+------------------+

1 row in set
Time: 0.002s
```

### 字段

新建表 `user_tb`

```sql
CREATE TABLE IF NOT EXISTS user_tb (
    id INT PRIMARY KEY AUTO_INCREMENT COMMENT '编号(主键,唯一,自增)',
    user_name VARCHAR(30) NOT null UNIQUE COMMENT '用户名(非空,唯一)',
    create_time DATETIME NOT null DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间(非空,默认为当前时间戳)',
    update_time DATETIME NOT null DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '最近修改时间',
    real_name VARCHAR(20) NOT null COMMENT '姓名(非空)',
    real_id CHAR(18) NOT null UNIQUE COMMENT '身份证号(非空,18字符,唯一)',
    age TINYINT UNSIGNED COMMENT '年龄(非负)',
    gender TINYINT UNSIGNED COMMENT '性别(0:男,1:女)',
    birthday DATE COMMENT '生日'
) COMMENT '用户列表';

DESC user_tb;
```

```text
user_tb
+-------------+------------------+------+-----+-------------------+-----------------------------------------------+
| Field       | Type             | Null | Key | Default           | Extra                                         |
+-------------+------------------+------+-----+-------------------+-----------------------------------------------+
| id          | int              | NO   | PRI | <null>            | auto_increment                                |
| user_name   | varchar(30)      | NO   | UNI | <null>            |                                               |
| create_time | datetime         | NO   |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED                             |
| update_time | datetime         | NO   |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED on update CURRENT_TIMESTAMP |
| real_name   | varchar(20)      | NO   |     | <null>            |                                               |
| real_id     | char(18)         | NO   | UNI | <null>            |                                               |
| age         | tinyint unsigned | YES  |     | <null>            |                                               |
| gender      | tinyint unsigned | YES  |     | <null>            |                                               |
| birthday    | date             | YES  |     | <null>            |                                               |
+-------------+------------------+------+-----+-------------------+-----------------------------------------------+

9 rows in set
Time: 0.005s
```

修改字段

```sql
ALTER TABLE user_tb
    DROP COLUMN age,
    DROP COLUMN birthday;
    ADD COLUMN deadline DATETIME AFTER update_time,
    ADD COLUMN qq VARCHAR(20) COMMENT 'QQ号码',
    ADD COLUMN tt INT,
    CHANGE COLUMN real_id id_card VARCHAR(18) COMMENT '身份证号码';

    -- ADD COLUMN ss FLOAT AFTER tt,    -- `tt` not exists before exec.
    -- MODIFY COLUMN deadline ...,      -- `deadline` not exists before exec.
    -- CHANGE COLUMN tt wx ...;         -- `tt` not exists before exec.

ALTER TABLE user_tb
    MODIFY COLUMN deadline
        DATETIME NOT null DEFAULT '1970-01-01 00:00:00' COMMENT '截止日期',
    MODIFY COLUMN qq
        VARCHAR(13) UNIQUE COMMENT 'QQ号',
    CHANGE COLUMN tt
        wx VARCHAR(25) UNIQUE COMMENT 'WX号';

DESC user_tb;
```

```text
user_tb
+-------------+------------------+------+-----+---------------------+-----------------------------------------------+
| Field       | Type             | Null | Key | Default             | Extra                                         |
+-------------+------------------+------+-----+---------------------+-----------------------------------------------+
| id          | int              | NO   | PRI | <null>              | auto_increment                                |
| user_name   | varchar(30)      | NO   | UNI | <null>              |                                               |
| create_time | datetime         | NO   |     | CURRENT_TIMESTAMP   | DEFAULT_GENERATED                             |
| update_time | datetime         | NO   |     | CURRENT_TIMESTAMP   | DEFAULT_GENERATED on update CURRENT_TIMESTAMP |
| deadline    | datetime         | NO   |     | 1970-01-01 00:00:00 |                                               |
| real_name   | varchar(20)      | NO   |     | <null>              |                                               |
| id_card     | varchar(18)      | YES  | UNI | <null>              |                                               |
| gender      | tinyint unsigned | YES  |     | <null>              |                                               |
| qq          | varchar(13)      | YES  | UNI | <null>              |                                               |
| wx          | varchar(25)      | YES  | UNI | <null>              |                                               |
+-------------+------------------+------+-----+---------------------+-----------------------------------------------+

10 rows in set
Time: 0.004s
```

### 增删改并

添加初始数据

```sql
INSERT INTO user_tb
    (user_name, real_name, id_card, gender, wx)
VALUES
    ('nisemono', '张三', '123456789123456789', 0, 'wx_qwerty'),
    ('shy-vector', '李四', '987654321987654321', 1, 'shy_vector');

SELECT * FROM user_tb;
```

```text
user_tb
+----+------------+---------------------+---------------------+---------------------+-----------+--------------------+--------+--------+------------+
| id | user_name  | create_time         | update_time         | deadline            | real_name | id_card            | gender | qq     | wx         |
+----+------------+---------------------+---------------------+---------------------+-----------+--------------------+--------+--------+------------+
| 1  | nisemono   | 2026-01-25 20:06:34 | 2026-01-25 20:06:34 | 1970-01-01 00:00:00 | 张三      | 123456789123456789 | 0      | <null> | wx_qwerty  |
| 2  | shy-vector | 2026-01-25 20:06:34 | 2026-01-25 20:06:34 | 1970-01-01 00:00:00 | 李四      | 987654321987654321 | 1      | <null> | shy_vector |
+----+------------+---------------------+---------------------+---------------------+-----------+--------------------+--------+--------+------------+
```

创建另一张表

```sql
CREATE TABLE IF NOT EXISTS user2_tb AS SELECT * FROM user_tb;
SELECT * FROM user2_tb;

UPDATE user2_tb
SET
    user_name = 'ice',
    create_time = NOW(),
    update_time = NOW(),
    deadline = NOW() + INTERVAL 2 MONTH,
    real_name = '王五',
    id_card = '1234xxxxx1234xxxxx',
    qq = '0D000721',
    wx = null
WHERE id % 2 = 1;

UPDATE user2_tb
SET
    user_name = 'aha',
    create_time = NOW(),
    update_time = NOW(),
    deadline = NOW() + INTERVAL 2 MONTH,
    real_name = '赵六',
    id_card = '5678xxxxx5678xxxxx',
    qq = '1145141919',
    wx = null
WHERE id % 2 = 0;

SELECT * FROM user2_tb;
```

```text
user2_tb
+----+-----------+---------------------+---------------------+---------------------+-----------+--------------------+--------+------------+--------+
| id | user_name | create_time         | update_time         | deadline            | real_name | id_card            | gender | qq         | wx     |
+----+-----------+---------------------+---------------------+---------------------+-----------+--------------------+--------+------------+--------+
| 1  | ice       | 2026-01-25 20:39:45 | 2026-01-25 20:39:45 | 2026-03-25 20:39:45 | 王五      | 1234xxxxx1234xxxxx | 0      | 0D000721   | <null> |
| 2  | aha       | 2026-01-25 20:42:10 | 2026-01-25 20:42:10 | 2026-03-25 20:42:10 | 赵六      | 5678xxxxx5678xxxxx | 1      | 1145141919 | <null> |
+----+-----------+---------------------+---------------------+---------------------+-----------+--------------------+--------+------------+--------+
```

:::warning
这里直接使用了 `CREATE TABLE tb1 AS SELECT * FROM tb2` 后，字段类型保留，但**约束丢失**．
应该使用 `CREATE TABLE tb1(...) AS SELECT * FROM tb2`，手动确定约束．
经测试，从 `FLOAT(3,2)` 到 `DOUBLE` 的精度损失情况：

```text
9.99 -> 9.989999771118164
1.23 -> 1.2300000190734863
1.0  -> 1.0
1.78 -> 1.7799999713897705
```
:::

**纵向合并**两表，相当于批量插入

```sql
INSERT IGNORE INTO user_tb
    (user_name, create_time, update_time, deadline, real_name, id_card, gender, qq, wx)
SELECT
    user_name, create_time, update_time, deadline, real_name, id_card, gender, qq, wx
FROM user2_tb;

SELECT * FROM user_tb;
```

```text
user_tb
+----+------------+---------------------+---------------------+---------------------+-----------+--------------------+--------+------------+------------+
| id | user_name  | create_time         | update_time         | deadline            | real_name | id_card            | gender | qq         | wx         |
+----+------------+---------------------+---------------------+---------------------+-----------+--------------------+--------+------------+------------+
| 1  | nisemono   | 2026-01-25 20:06:34 | 2026-01-25 20:06:34 | 1970-01-01 00:00:00 | 张三      | 123456789123456789 | 0      | <null>     | wx_qwerty  |
| 2  | shy-vector | 2026-01-25 20:06:34 | 2026-01-25 20:06:34 | 1970-01-01 00:00:00 | 李四      | 987654321987654321 | 1      | <null>     | shy_vector |
| 3  | ice        | 2026-01-25 20:39:45 | 2026-01-25 20:39:45 | 2026-03-25 20:39:45 | 王五      | 1234xxxxx1234xxxxx | 0      | 0D000721   | <null>     |
| 4  | aha        | 2026-01-25 20:42:10 | 2026-01-25 20:42:10 | 2026-03-25 20:42:10 | 赵六      | 5678xxxxx5678xxxxx | 1      | 1145141919 | <null>     |
+----+------------+---------------------+---------------------+---------------------+-----------+--------------------+--------+------------+------------+
```

小插曲：由于 `qq` 和 `wx` 有 `UNIQUE` 约束，下面的更新失败

```sql
UPDATE user_tb
SET wx = 'AAAAAAAAA', qq = 'BBBBBB'
WHERE id > 1;
```

```text
(1062, "Duplicate entry 'BBBBBB' for key 'user_tb.qq'")
```

:::warning
1. 如果执行 `INSERT` 但最终回滚（比如主键冲突、唯一键冲突），计数器 `AUTO_INCREMENT` 仍会自增，导致 `user_tb` 后面的 `id` 跳号．
2. 执行 `DELETE` 不会使计数器清零．
3. 除此之外也有很多隐性消耗导致跳号，比如批量插入的自增 ID 预分配．

自增 ID 的设计目标是保证**唯一性**，而非连续性，所以无法完全避免跳号．
:::

:::tip[`INSERT`-like 语句]
1. **simple insert**: 如 `INSERT INTO tb () VALUES ()`，可预先确定插入行数．
2. **bulk insert**: 如 `INSERT INTO tb SELECT ... FROM ...`、`LOAD data`，插入行数（需要申请的自增值数目）不可预期．
3. **mixed-mode insert**: 如 `INSERT INTO tb (id,name) VALUES (1,'a'), (NULL,'b'), (5,'c'), (NULL,'d');` 自增值被指定．
:::

:::tip[`innodb_autoinc_lock_mode`]
`innodb_autoinc_lock_mode` 是 MySQL InnoDB 存储引擎的一个系统变量，它控制着自增（`AUTO_INCREMENT`）值的锁定机制．这个设置对于高并发的数据库操作，特别是插入（`INSERT`）操作，有着显著的性能影响．

1. `0`，传统锁模式．整个表 `auto_inc` 的锁在同一时刻只会被一个 `INSERT`-like 语句持有，直至语句结束．保证自增值连续．
2. `1`，连续锁模式．对于 simple insert，持有相应数量的自增值互斥锁来避免使用表锁，这个锁仅在分配过程中持有，不会持续到语句结束，可以保证自增值的连续．对于 bulk insert 仍然使用表锁（源表使用共享锁，目标表使用自增锁）．mixed-mode insert 会分配数量多于所需的自增锁，自动分配，过量舍弃．
3. `2`，交叉锁模式（MySQL 8.0+ 默认）．不会使用表锁，并发性良好．**单个 simple insert 语句生成的自增值连续，bulk insert 则无法保证．**
:::

重新调整表格字段

```sql
ALTER TABLE user_tb
    DROP COLUMN update_time,
    DROP COLUMN deadline,
    DROP COLUMN id_card;

ALTER TABLE user2_tb
    -- 补上约束
    MODIFY COLUMN id INT PRIMARY KEY AUTO_INCREMENT,
    MODIFY COLUMN user_name VARCHAR(30) NOT null UNIQUE;
    DROP COLUMN create_time,
    DROP COLUMN update_time,
    DROP COLUMN deadline,
    DROP COLUMN real_name,
    DROP COLUMN id_card,
    DROP COLUMN gender,
    DROP COLUMN qq,
    DROP COLUMN wx,
    ADD COLUMN height DECIMAL(5,2),
    ADD COLUMN weight DOUBLE(6,3),
    ADD COLUMN rating FLOAT(3,2),

DESC user_tb;
DESC user2_tb; 
```

```text
user_tb
+-------------+------------------+------+-----+-------------------+-------------------+
| Field       | Type             | Null | Key | Default           | Extra             |
+-------------+------------------+------+-----+-------------------+-------------------+
| id          | int              | NO   | PRI | <null>            | auto_increment    |
| user_name   | varchar(30)      | NO   | UNI | <null>            |                   |
| create_time | datetime         | NO   |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED |
| real_name   | varchar(20)      | NO   |     | <null>            |                   |
| gender      | tinyint unsigned | YES  |     | <null>            |                   |
| qq          | varchar(13)      | YES  | UNI | <null>            |                   |
| wx          | varchar(25)      | YES  | UNI | <null>            |                   |
+-------------+------------------+------+-----+-------------------+-------------------+

user2_tb
+-----------+--------------+------+-----+---------+----------------+
| Field     | Type         | Null | Key | Default | Extra          |
+-----------+--------------+------+-----+---------+----------------+
| id        | int          | NO   | PRI | <null>  | auto_increment |
| user_name | varchar(30)  | NO   | UNI | <null>  |                |
| height    | decimal(5,2) | YES  |     | <null>  |                |
| weight    | double(6,3)  | YES  |     | <null>  |                |
| rating    | float(3,2)   | YES  |     | <null>  |                |
+-----------+--------------+------+-----+---------+----------------+
```

:::warning
```text
+---------+------+------------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                          |
+---------+------+------------------------------------------------------------------------------------------------------------------+
| Warning | 1681 | Specifying number of digits for floating point data types is deprecated and will be removed in a future release. |
| Warning | 1681 | Specifying number of digits for floating point data types is deprecated and will be removed in a future release. |
+---------+------+------------------------------------------------------------------------------------------------------------------+

```
:::

```sql
INSERT INTO user_tb
    (user_name, real_name, gender, qq, wx)
VALUES
    ('tourist', '孙七', 0, '810114514', 'wx_TOURIST');


INSERT INTO user2_tb
    (user_name, height, weight, rating)
VALUES
    ('nisemono', 1.77, 60, 1.00),
    ('Not Found', 1.63, 53, 0.83),
    ('ice', 1.75, 62, 1.23),
    ('tourist', 1.80, 68.2, 1.78);

SELECT * FROM user_tb;
SELECT * FROM user2_tb;
```

```text
user_tb
+----+------------+---------------------+-----------+--------+------------+------------+
| id | user_name  | create_time         | real_name | gender | qq         | wx         |
+----+------------+---------------------+-----------+--------+------------+------------+
| 1  | nisemono   | 2026-01-25 20:06:34 | 张三      | 0      | <null>     | wx_qwerty  |
| 2  | shy-vector | 2026-01-25 20:06:34 | 李四      | 1      | <null>     | shy_vector |
| 3  | ice        | 2026-01-25 20:39:45 | 王五      | 0      | 0D000721   | <null>     |
| 4  | aha        | 2026-01-25 20:42:10 | 赵六      | 1      | 1145141919 | <null>     |
| 6  | tourist    | 2026-01-25 21:56:24 | 孙七      | 0      | 810114514  | wx_TOURIST |
+----+------------+---------------------+-----------+--------+------------+------------+
  ⬆ 跳号（前面 INSERT 失败回滚过一次）

user2_tb
+----+-----------+--------+--------+--------+
| id | user_name | height | weight | rating |
+----+-----------+--------+--------+--------+
| 1  | nisemono  | 1.77   | 60.0   | 1.0    |
| 2  | Not Found | 1.63   | 53.0   | 0.83   |
| 3  | ice       | 1.75   | 62.0   | 1.23   |
| 4  | tourist   | 1.80   | 68.2   | 1.78   |
+----+-----------+--------+--------+--------+
```

我想让这两张表**横向合并**至新的表．先创建新表．

```sql
CREATE TABLE user3_tb (
    id INT PRIMARY KEY AUTO_INCREMENT,
    user_name VARCHAR(30) NOT null UNIQUE,
    real_name VARCHAR(20) NOT null,
    gender TINYINT UNSIGNED,
    height DECIMAL(5,2),
    weight DOUBLE(6,3),
    rating FLOAT(3,2),
    qq VARCHAR(13) UNIQUE,
    wx VARCHAR(25) UNIQUE,
    create_time DATETIME NOT null DEFAULT CURRENT_TIMESTAMP
);

DESC user3_tb;
```

```text
user3_tb
+-------------+------------------+------+-----+-------------------+-------------------+
| Field       | Type             | Null | Key | Default           | Extra             |
+-------------+------------------+------+-----+-------------------+-------------------+
| id          | int              | NO   | PRI | <null>            | auto_increment    |
| user_name   | varchar(30)      | NO   | UNI | <null>            |                   |
| real_name   | varchar(20)      | NO   |     | <null>            |                   |
| gender      | tinyint unsigned | YES  |     | <null>            |                   |
| height      | decimal(5,2)     | YES  |     | <null>            |                   |
| weight      | double(6,3)      | YES  |     | <null>            |                   |
| rating      | float(3,2)       | YES  |     | <null>            |                   |
| qq          | varchar(13)      | YES  | UNI | <null>            |                   |
| wx          | varchar(25)      | YES  | UNI | <null>            |                   |
| create_time | datetime         | NO   |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED |
+-------------+------------------+------+-----+-------------------+-------------------+
```

:::tip[`JOIN`]
1. `INNER JOIN` 合并时，两张表的失匹项都将被舍弃．
2. `LEFT JOIN` 合并时，左表完全保留，失匹项来自右表的字段将被设为默认值；右表的失匹项将被舍弃．
3. `RIGHT JOIN` 合并时，右表完全保留，失匹项来自左表的字段将被设为默认值；左表的失匹项将被舍弃．
:::

内合并：

```sql
INSERT INTO user3_tb
    (user_name, create_time, real_name, gender, qq, wx,
     height, weight, rating)
SELECT
    u.user_name, u.create_time, u.real_name, u.gender, u.qq, u.wx,
    v.height, v.weight, v.rating
FROM user_tb u
INNER JOIN user2_tb v
ON u.user_name = v.user_name;

SELECT * FROM user3_tb;
```

```text
注：这里漏了一段操作．
user2_tb 的 `Not Found` 原本是别的值，
合并后的 user3_tb 原本有 4 个项，
后面改过来的时候需要 DELETE 整个 user3_tb，
计数器不归零，但本应从 5 开始，而下面从 8 开始．

user3_tb (INNER JOIN)
+----+-----------+-----------+--------+--------+--------+--------+-----------+------------+---------------------+
| id | user_name | real_name | gender | height | weight | rating | qq        | wx         | create_time         |
+----+-----------+-----------+--------+--------+--------+--------+-----------+------------+---------------------+
| 8  | nisemono  | 张三      | 0      | 1.77   | 60.0   | 1.0    | <null>    | wx_qwerty  | 2026-01-25 20:06:34 |
| 9  | ice       | 王五      | 0      | 1.75   | 62.0   | 1.23   | 0D000721  | <null>     | 2026-01-25 20:39:45 |
| 10 | tourist   | 孙七      | 0      | 1.80   | 68.2   | 1.78   | 810114514 | wx_TOURIST | 2026-01-25 21:56:24 |
+----+-----------+-----------+--------+--------+--------+--------+-----------+------------+---------------------+
  ⬆（跳号，bulk insert 不保证连续）
```

左合并：

```sql
DELETE FROM user3_tb;

INSERT INTO user3_tb
    (user_name, create_time, real_name, gender, qq, wx,
     height, weight, rating)
SELECT
    u.user_name, u.create_time, u.real_name, u.gender, u.qq, u.wx,
    v.height, v.weight, v.rating
FROM user_tb u
LEFT JOIN user2_tb v
ON u.user_name = v.user_name;

SELECT * FROM user3_tb;
```

```txt
user3_tb (LEFT JOIN)
+----+------------+-----------+--------+--------+--------+--------+------------+------------+---------------------+
| id | user_name  | real_name | gender | height | weight | rating | qq         | wx         | create_time         |
+----+------------+-----------+--------+--------+--------+--------+------------+------------+---------------------+
| 11 | nisemono   | 张三      | 0      | 1.77   | 60.0   | 1.0    | <null>     | wx_qwerty  | 2026-01-25 20:06:34 |
| 12 | shy-vector | 李四      | 1      | <null> | <null> | <null> | <null>     | shy_vector | 2026-01-25 20:06:34 |
| 13 | ice        | 王五      | 0      | 1.75   | 62.0   | 1.23   | 0D000721   | <null>     | 2026-01-25 20:39:45 |
| 14 | aha        | 赵六      | 1      | <null> | <null> | <null> | 1145141919 | <null>     | 2026-01-25 20:42:10 |
| 15 | tourist    | 孙七      | 0      | 1.80   | 68.2   | 1.78   | 810114514  | wx_TOURIST | 2026-01-25 21:56:24 |
+----+------------+-----------+--------+--------+--------+--------+------------+------------+---------------------+
```

带默认值的左合并：

```sql
DELETE FROM user3_tb;

INSERT INTO user3_tb
    (user_name, create_time, real_name, gender, qq, wx,
     height, weight, rating)
SELECT
    u.user_name, u.create_time, u.real_name, u.gender, u.qq, u.wx,
    COALESCE(v.height, 9.99), COALESCE(v.weight, 999.9), COALESCE(v.rating, 9.99)
FROM user_tb u
LEFT JOIN user2_tb v
ON u.user_name = v.user_name;

SELECT * FROM user3_tb;
```

```text
user3_tb
+----+------------+-----------+--------+--------+--------+--------+------------+------------+---------------------+
| id | user_name  | real_name | gender | height | weight | rating | qq         | wx         | create_time         |
+----+------------+-----------+--------+--------+--------+--------+------------+------------+---------------------+
| 18 | nisemono   | 张三      | 0      | 1.77   |  60.0  | 1.0    | <null>     | wx_qwerty  | 2026-01-25 20:06:34 |
| 19 | shy-vector | 李四      | 1      | 9.99   | 999.9  | 9.99   | <null>     | shy_vector | 2026-01-25 20:06:34 |
| 20 | ice        | 王五      | 0      | 1.75   |  62.0  | 1.23   | 0D000721   | <null>     | 2026-01-25 20:39:45 |
| 21 | aha        | 赵六      | 1      | 9.99   | 999.9  | 9.99   | 1145141919 | <null>     | 2026-01-25 20:42:10 |
| 22 | tourist    | 孙七      | 0      | 1.80   |  68.2  | 1.78   | 810114514  | wx_TOURIST | 2026-01-25 21:56:24 |
+----+------------+-----------+--------+--------+--------+--------+------------+------------+---------------------+
```

**移动列**至开头、某列的后面，调整字段：

```sql
DESC user_tb;

ALTER TABLE user_tb
    MODIFY COLUMN create_time DATETIME NOT null DEFAULT CURRENT_TIMESTAMP FIRST,
    MODIFY COLUMN user_name VARCHAR(30) NOT null UNIQUE AFTER real_name;

DESC user_tb;
SELECT * FROM user_tb;

ALTER TABLE user_tb
    DROP COLUMN id,
    DROP COLUMN real_name,
    DROP COLUMN qq,
    DROP COLUMN wx;
    ADD COLUMN height DECIMAL(5,2),
    ADD COLUMN weight DOUBLE(6,3),
    ADD COLUMN rating FLOAT(3,2);

DESC user_tb;
```

```text
user_tb
+-------------+------------------+------+-----+-------------------+-------------------+
| Field       | Type             | Null | Key | Default           | Extra             |
+-------------+------------------+------+-----+-------------------+-------------------+
| id          | int              | NO   | PRI | <null>            | auto_increment    |
| user_name   | varchar(30)      | NO   | UNI | <null>            |                   |
| create_time | datetime         | NO   |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED |
| real_name   | varchar(20)      | NO   |     | <null>            |                   |
| gender      | tinyint unsigned | YES  |     | <null>            |                   |
| qq          | varchar(13)      | YES  | UNI | <null>            |                   |
| wx          | varchar(25)      | YES  | UNI | <null>            |                   |
+-------------+------------------+------+-----+-------------------+-------------------+

user_tb
+-------------+------------------+------+-----+-------------------+-------------------+
| Field       | Type             | Null | Key | Default           | Extra             |
+-------------+------------------+------+-----+-------------------+-------------------+
| create_time | datetime         | NO   |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED |
| id          | int              | NO   | PRI | <null>            | auto_increment    |
| real_name   | varchar(20)      | NO   |     | <null>            |                   |
| user_name   | varchar(30)      | NO   | UNI | <null>            |                   |
| gender      | tinyint unsigned | YES  |     | <null>            |                   |
| qq          | varchar(13)      | YES  | UNI | <null>            |                   |
| wx          | varchar(25)      | YES  | UNI | <null>            |                   |
+-------------+------------------+------+-----+-------------------+-------------------+

user_tb
+---------------------+----+-----------+------------+--------+------------+------------+
| create_time         | id | real_name | user_name  | gender | qq         | wx         |
+---------------------+----+-----------+------------+--------+------------+------------+
| 2026-01-25 20:06:34 | 1  | 张三      | nisemono   | 0      | <null>     | wx_qwerty  |
| 2026-01-25 20:06:34 | 2  | 李四      | shy-vector | 1      | <null>     | shy_vector |
| 2026-01-25 20:39:45 | 3  | 王五      | ice        | 0      | 0D000721   | <null>     |
| 2026-01-25 20:42:10 | 4  | 赵六      | aha        | 1      | 1145141919 | <null>     |
| 2026-01-25 21:56:24 | 6  | 孙七      | tourist    | 0      | 810114514  | wx_TOURIST |
+---------------------+----+-----------+------------+--------+------------+------------+

user_tb
+-------------+------------------+------+-----+-------------------+-------------------+
| Field       | Type             | Null | Key | Default           | Extra             |
+-------------+------------------+------+-----+-------------------+-------------------+
| create_time | datetime         | NO   |     | CURRENT_TIMESTAMP | DEFAULT_GENERATED |
| user_name   | varchar(30)      | NO   | PRI | <null>            |                   |
| gender      | tinyint unsigned | YES  |     | <null>            |                   |
| height      | decimal(5,2)     | YES  |     | <null>            |                   |
| weight      | double(6,3)      | YES  |     | <null>            |                   |
| rating      | float(3,2)       | YES  |     | <null>            |                   |
+-------------+------------------+------+-----+-------------------+-------------------+
```

**就地合并**

```sql
UPDATE user_tb u
JOIN user2_tb v ON u.user_name = v.user_name
SET u.height = v.height, u.weight = v.weight, u.rating = v.rating;

SELECT * FROM user_tb;

UPDATE user_tb
SET
    height = IFNULL(height, 9.99),
    weight = IFNULL(weight, 999.9),
    rating = IFNULL(rating, 9.99)
WHERE height IS null OR weight IS null OR rating IS null;

SELECT * FROM user_tb;
```

```text
user_tb
+---------------------+------------+--------+--------+--------+--------+
| create_time         | user_name  | gender | height | weight | rating |
+---------------------+------------+--------+--------+--------+--------+
| 2026-01-25 20:42:10 | aha        | 1      | <null> | <null> | <null> |
| 2026-01-25 20:39:45 | ice        | 0      | 1.75   | 62.0   | 1.23   |
| 2026-01-25 20:06:34 | nisemono   | 0      | 1.77   | 60.0   | 1.0    |
| 2026-01-25 20:06:34 | shy-vector | 1      | <null> | <null> | <null> |
| 2026-01-25 21:56:24 | tourist    | 0      | 1.80   | 68.2   | 1.78   |
+---------------------+------------+--------+--------+--------+--------+

user_tb
+---------------------+------------+--------+--------+--------+--------+
| create_time         | user_name  | gender | height | weight | rating |
+---------------------+------------+--------+--------+--------+--------+
| 2026-01-25 20:42:10 | aha        | 1      | 9.99   | 999.9  | 9.99   |
| 2026-01-25 20:39:45 | ice        | 0      | 1.75   |  62.0  | 1.23   |
| 2026-01-25 20:06:34 | nisemono   | 0      | 1.77   |  60.0  | 1.0    |
| 2026-01-25 20:06:34 | shy-vector | 1      | 9.99   | 999.9  | 9.99   |
| 2026-01-25 21:56:24 | tourist    | 0      | 1.80   |  68.2  | 1.78   |
+---------------------+------------+--------+--------+--------+--------+
```
