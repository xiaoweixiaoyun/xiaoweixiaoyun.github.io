---
title: int(1)和int(10)有区别吗
date: 2023/11/07
tags: 面试,问答
categories: 每日一问
description:  int(1)和int(10)有区别吗？
keywords: mysql、int
cover: /img/md/mysql.png
---

## 先给答案
没有区别

## 背景
>工作中不乏见过定义各种类型像varchar(4),char(2)等,里面的数字一般指的是字符长度。所以很多人也会觉得int(1)里面只能存1位数，殊不知是理解错了。下面具体讲解一下！
## 讲解
INT[(M)] [UNSIGNED] [ZEROFILL]
普通大小的整数。
带符号的范围是-2147483648到2147483647
无符号的范围是0到4294967295。
INT(1) 和 INT(10)本身没有区别，但是加上(M)值后，会有显示宽度的设置。

### 测试1
```sql
create table test(id int(3));
insert into test values(12);
insert into test values(1234);
```
```sql
mysql> select * from test;
+------+
| id   |
+------+
|   12 |
| 1234 |
+------+
```
从这个测试可以看出定义的id int(3)是可以存下1234的，接着带上ZEROFILL做测试。

### 测试2
和测试1一样创建一个表，也是定义id int(3) 带上zerofill，测试结果如下：
```sql
mysql> create table test1(id int(3) zerofill);
Query OK, 0 rows affected (0.52 sec)

mysql> insert into test1 value(12);
Query OK, 1 row affected (0.15 sec)

mysql> insert into test1 value(1234);
Query OK, 1 row affected (0.16 sec)

mysql> insert into test1 value(666666);
Query OK, 1 row affected (0.14 sec)

mysql> select * from test1;
+--------+
| id     |
+--------+
|    012 | # 这个地方不够三位宽度会自动补零
|   1234 |
| 666666 |
+--------+
3 rows in set (0.00 sec)

```

### 测试3
利用 test表（不带zerofill）和 test1表（带zerofill） 测试插入负数看看。

test表(可以正常存进去)
```sql
mysql> insert into test value(-1234);
Query OK, 1 row affected (0.07 sec)

mysql> select * from test;
+-------+
| id    |
+-------+
|    12 |
|  1234 |
| -1234 |
+-------+
3 rows in set (0.00 sec)
```

test1表
```sql
mysql>  insert into test1 value(-1234);
ERROR 1264 (22003): Out of range value for column 'id' at row 1
```
从这个测试可以发现添加zerofill的时候，无法添加负数到表里。
```sql
mysql>  insert into test1 value(0);
Query OK, 1 row affected (0.10 sec)

mysql> select * from test1;
+--------+
| id     |
+--------+
|    012 |
|   1234 |
| 666666 |
|    000 |
+--------+
4 rows in set (0.00 sec)
```
## 小结
以上测试中用到的 zerofill 翻译过来就是 零填充。

int后面的数字不能表示字段的长度，int(num)一般加上zerofill，才有效果。zerofill的作用一般可以用在一些编号相关的数字中，比如学生的编号 001 002 … 999这种，如果mysql没有零填充的功能，但是你又要格式化输出等长的数字编号时，那么你只能自己处理了。