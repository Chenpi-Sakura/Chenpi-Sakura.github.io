---
title: MySQL 在单一表格中获取数据
autor: Chenpi
date: 2025-06-15 21:42:00 +0800
categories: [数据库, 笔记]
tags: [数据库, 笔记]

math: true
mermaid: true
---

## 选择语句
```sql
USE store

SELECT *
FROM customers
WHERE customer_id < 4
ORDER BY first_name
```

---
### SELECT 字句
`SELECT`最简单的用法是选择所有列或者指定列。
```sql
SELECT *
FROM table_name;
-- 这里是选中了所有列

SELECT 
  column1, 
  column2, 
  ...
FROM table_name;
-- 这里是选中了指定列
```
我们还可以使用数学公式对列进行变换，并使用`AS`关键字为列指定**新名称**。
```sql
SELECT 
  c1 * 1.1 AS d1, 
  c2 / 0.9 AS d2, 
  ...
FROM table_name
```
我们还能使用`DISTINCT`对`SELECT`进行修饰，以**去除**结果中的**重复值**。
```sql
SELECT DISTINCT country
FROM customers;
-- 从 customers 表中选出所有不同的国家，每个国家只出现一次

SELECT DISTINCT 
  first_name, 
  last_name
FROM employees;
-- 只有当某一行的 first_name 和 last_name 两列的组合完全相同时，才会被视为“重复”，被去除掉。
```

---
### WHERE 语句
`WHERE`是用于行筛选的条件子句，它会对表中的每一行（每条记录）逐一进行判断，筛选出**满足条件**的行。
```sql
SELECT *
FROM employees
WHERE salary > 5000
-- 仅返回薪资在5000以上的
```
> 还存在的比较条件有：`<`,`!=`,`>=`,`<=`,`=`(注意不同于一般的编程语言，SQL中的相等为单个等号)
{: .prompt-tip}

---
### AND,OR,NOT 运算符
此处与Python中的语意类似，故不累述。

---
### IN 运算符
有时候我们会写如下代码：
```sql
WHERE department = 'HR' OR department = 'Finance' OR department = 'IT'
```

此时我们可以使用`IN`来简化操作：
```sql
WHERE department IN ('HR', 'Finance', 'IT')
```

`IN`运算符用于判断某个字段的值是否属于指定的多个值中的一个。
> 我们还能使用`NOT`进行修饰以达到相反的结果。
{: .prompt-tip}

---
### BETWEEN 运算符
有时候我们会写如下代码：
```sql
WHERE salary >= 5000 AND salary <= 8000;
WHERE order_date >= '2024-01-01' AND order_date <= '2024-12-31'
-- SQL中标准的日期表达为 YYYY-MM-DD
```

此时我们可以使用`BETWEEN`来简化操作：
```sql
WHERE salary BETWEEN 5000 AND 8000;
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';
```

`BETWEEN`运算符用于判断某个字段的值是否在**两个值之间（包含边界）**。常用于筛选数值范围和日期范围。

---
### LIKE 运算符
`LIKE`运算符用于在`WHERE`子句中进行模糊匹配，通常与通配符`%`或`_`搭配使用，用于查找符合特定模式的字符串。
`%`	匹配**任意数量的任意字符**（可以是 0 个）
`_`	匹配**任意单个字符**
```sql
SELECT code
FROM products
WHERE code LIKE '%A_1%';
```
此处我们将匹配一个子串，字串第一位为`A`，第二位为任意字符，第三位为`1`。
> 我们同样能加上`NOT`来进行修饰
{: .prompt-tip}

---
### REGEXP 运算符
正则表达式相比于`LIKE`更强大，可用于复杂模式的字符串筛选。
常用的正则表达式符号
|符号|含义|
|:---:|:---:|
|^|匹配开头|
|$|匹配结尾|
|.|匹配任意单个字符|
|*|匹配前一个字符重复 0 次或多次|
|[abc]|匹配 a 或 b 或 c 中的任意一个|
|[a-c]|匹配一个 a 到 c 的任意字母|
|\||或逻辑符号|

~~好麻烦啊哪个天才想出来的~~
示例
```sql
SELECT name
FROM users
WHERE name REGEXP '[aeiou]';
-- 匹配包含任一元音字母的名字。

SELECT name
FROM users
WHERE name REGEXP '^A.*n$';
/* 
  匹配以 A 开头，以 n 结尾的子段
  ^A：以 A 开头
  .*：中间任意数量的字符
  n$：以 n 结尾
*/
```

---
### IS NULL 运算符
用于判断某个字段的值是否为 NULL，也就是检查该字段是否缺失或未赋值。
~~这么简单就不讲了~~
```sql
SELECT name, email
FROM customers
WHERE email IS NULL;
-- 这会查找那些没有填写电子邮箱地址的客户记录。
```
> 注意反过来不是 `NOT IS NULL` 而是符合语法表达的 `IS NOT NULL`
{: .prompt-tip}
> 还需要注意 `NULL` 与 `''`并不一样
{: .prompt-warning}

---
### ORDER BY子句
`ORDER BY`用于对查询结果进行排序，默认是按升序（从小到大）排列的。
```sql
SELECT *
FROM customers
WHERE customer_id < 4
ORDER BY first_name;
```
你还可以根据**多个列**排序：
```sql
ORDER BY department, salary DESC
-- 先按部门排，再在部门内部按工资从高到低排。
```

不仅可以写列名，还可以写：
* **数学表达式**，比如：
  ```sql
  ORDER BY price * quantity
  ```
* **没出现在 SELECT 的列**（MySQL 支持，其他数据库可能不行）
* **列的别名**（MySQL 支持，也可以是常数定义出来的别名）
```sql
SELECT 
  name, 
  salary * 12 AS yearly_salary
FROM employees
ORDER BY yearly_salary DESC;
```
这个就用了一个在 `SELECT` 中定义的别名 `yearly_salary` 作为排序依据。
> 排序方向可以在每个列名后加 `DESC`（降序）或 `ASC`（升序，默认）
{: .prompt-tip}

---
### LIMIT 子句

`LIMIT` 子句用于限制返回结果的行数。

```sql
SELECT *
FROM products
LIMIT 5;
-- 返回products表中的前5行，不管总共有多少条数据。
```

也可以配合 `ORDER BY` 一起使用，选出“最前”或“最后”的数据：
```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC
LIMIT 3;
--给出工资最高的前三位
```

某些数据库（如 MySQL）支持**分页查询**的写法：
```sql
LIMIT 10 OFFSET 20
```

意思是：**从第 21 行开始，取 10 行**（因为 OFFSET 是从 0 开始数的）
也可以合并写成：
```sql
LIMIT 20, 10
-- 含义相同，先跳过 20 行，再取 10 行。
```
> 有些数据库（如 SQL Server）不支持 `LIMIT`，而是用 `TOP` 或 `FETCH FIRST` 等关键词。
{: .prompt-warning}
