---
title: SQL进阶学习
date: 2018-07-30 11:33:06
tags: SQL
categories:
- 后端
- SQL
copyright: true
---
## SQL高级教程
  
### SELECT TOP，LIMIT，ROWNUM
  
指定返回的记录数量。MySQL支持LIMIT子句，Oracle支持ROWNUM子句。  
  
表lottery_code_repository  
  
<!--more-->  
  
![](https://i.imgur.com/VAq6OD0.png)
  
ps：数据只贴出来第一页。  
  
* LIMIT
  
```sql
SELECT *
FROM lottery_code_repository
LIMIT 5 
```
  
![](https://i.imgur.com/k0NFjRk.png)
  
* SELECT TOP
  
```sql
SELECT TOP 2 * 
FROM Customers
WHERE Country='Germany'
```
  
查询前百分之五十的记录：  
  
```sql
SELECT TOP 50 PERCENT * 
FROM Customers
WHERE Country='Germany'
```
  
* ROWNUM
  
```sql
SELECT * 
FROM Customers
WHERE Country='Germany' AND ROWNUM <= 3
```
  
### LIKE运算符
  
通配符：  
  
* % 表示零个，一个或多个字符
* _ 表示单个字符
  
| LIKE运算符 | 描述 |
|-----------|------|
| LIKE 'a%' | 查找以"a"开头的任何值 |
| LIKE '%a' | 查找以"a"结尾的任何值 |
| LIKE '%or%' | 在任何位置查找具有"or"的值 |
| LIKE '_r%' | 在第二个位置查找任何具有"r"的值 |
| LIKE 'a_%_%' | 查找以"a"开头且长度至少为3个字符的值 |
| LIKE 'a%o' | 查找以"a"开头，以"o"结尾的值 |
  
### Wildcards通配符
  
* [charlist] 定义要匹配的字符的集合和范围
* [^charlist]或[!charlist]定义不匹配字符的集合和范围
  
```sql
SELECT * 
FROM Customers 
WHERE City LIKE '[bsp]%'
```
  
MySQL好像不支持，运行不报错，但是查询结果为空。  
  
### IN运算符
  
```sql
SELECT *
FROM lottery_code_repository
WHERE lottery_code IN ('BYH0IU', 'BYYOQI', 'BY7W87')
```
  
![](https://i.imgur.com/Ce4BjWj.png)
  
### BETWEEN运算符
  
```sql
SELECT *
FROM lottery_code_repository
WHERE lottery_code BETWEEN 'BYH0IU' AND 'C7GNIY'
```
  
![](https://i.imgur.com/3MdLKGT.png)
  
### UNIOM运算符
  
```sql
SELECT *
FROM lottery_code_repository
WHERE lottery_code IN ('BYH0IU', 'C7GNIY')
UNION 
SELECT *
FROM lottery_code_repository
WHERE lottery_code = 'C7GNIY'
```
  
![](https://i.imgur.com/23wqQqq.png)
  
```sql
SELECT *
FROM lottery_code_repository
WHERE lottery_code IN ('BYH0IU', 'C7GNIY')
UNION ALL
SELECT *
FROM lottery_code_repository
WHERE lottery_code = 'C7GNIY'
```
  
![](https://i.imgur.com/Hw66Iu1.png)
  
### SELECT INTO
  
从一个表中复制数据，将数据插入到另一个新表中。
  
```sql
SELECT *
INTO lottery
FROM lottery_code_repository
WHERE lottery_code IN ('BYH0IU', 'C7GNIY')
```
  
![](https://i.imgur.com/A6fITQr.png)
  
MySQL不支持SELECT INTO，只能先创建表然后插入数据。
  
```sql
CREATE TABLE lottery
(SELECT *
FROM lottery_code_repository
WHERE lottery_code IN ('BYH0IU', 'C7GNIY'))
```
  
![](https://i.imgur.com/MMEC9Ak.png)
  
### INSERT INTO SELECT
  
从表中复制数据，并将数据插入现有的表中。  
  
```sql
INSERT INTO lottery
(SELECT *
FROM lottery_code_repository
WHERE lottery_code IN ('BYH0IU', 'C7GNIY'))
```
  
![](https://i.imgur.com/hm1UPRu.png)
  
目标表中的任何现有行都不会受到影响。  
  
### Date函数
  
MySQL内置日期函数：  
  
| 函数 | 描述 |
|-----|------|
| NOW() | 返回当前的日期和时间 |
| CURDATE() | 返回当前的日期 |
| CURTIME() | 返回当前的时间 |
| DATE() | 提取日期或日期/时间表达式的日期部分 |
| EXTRACT() | 返回日期/时间的单独部分 |
| DATE ADD() | 向日期添加指定的时间间隔 |
| DATE SUB() | 从日期减去指定的时间间隔 |
| DATEDIFF() | 返回两个日期之间的天数 |
|DATE FORMAT() | 用不同的格式显示日期/时间 |
  
MySQL DATE 数据类型：  
  
* DATE -格式：YYYY-MM-DD
* DATETIME -格式：YYYY-MM-DD HH:MM:SS
* TIMESTAMP -格式：YYYY-MM-DD HH:MM:SS
* YEAR -格式：YYYY或YY
  
表date  
  
![](https://i.imgur.com/uRthT8i.png)
  
```sql
SELECT *
FROM date
WHERE date = '2018-7-30'
```
  
![](https://i.imgur.com/AFCCIvl.png)
  
```sql
SELECT *
FROM date
WHERE datetime = '2018-7-30'
```
  
查询结果为空。  
  
```sql
SELECT *
FROM date
WHERE datetime = '2018-7-30 15:32:11'
```
  
![](https://i.imgur.com/BIERc9e.png)
  
## SQL进阶
  
### Aliases别名
  
as高级用法：  
  
```sql
SELECT CONCAT(lottery_code, ',', use_mark) as lottery
FROM lottery_code_repository
WHERE lottery_code = 'BYH0IU'
```
  
![](https://i.imgur.com/1t9dd9r.png)
  
### TRUNCATE TABLE 命令
  
删除现有数据表中的所有数据，不删除表结构。DROP TABLE会删掉数据和表结构。  