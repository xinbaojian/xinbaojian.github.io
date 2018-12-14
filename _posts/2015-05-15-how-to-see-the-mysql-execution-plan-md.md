---
layout:     post
title:      "如何查看MySql的执行计划"
subtitle:   " \"Spring Boot\""
date:       2015-05-15 16:22:55
author:     "Baojian"
header-img: "img/post-bg-2015.jpg"
tags:
    - 技术
    - Java
    - Spring Boot
---

> 查看执行计划

#### 如何查看MySql的执行计划

mysql的查看执行计划的语句很简单，explain+你要执行的sql语句就OK了。

举一个例子

EXPLAIN SELECT * from employees where employees.gender='M' 

返回的结果如下：

#### 这些结果都代表什么？

- id是一组数字，表示查询中执行select子句或操作表的顺序。
- 如果id相同，则执行顺序从上至下。
- 如果是子查询，id的序号会递增，id越大则优先级越高，越先会被执行。
- id如果相同，则可以认为是一组，从上往下顺序执行，所有组中，id越高，优先级越高，越容易执行。
- selecttype有simple，primary，subquery，derived(衍生)，union，unionresult。
- simple表示查询中不包含子查询或者union。
- 当查询中包含任何复杂的子部分，最外层的查询被标记成primary。
- 在select或where列表中包含了子查询，则子查询被标记成subquery。
- 在from的列表中包含的子查询被标记成derived。
- 若第二个select出现在union后，则被标记成union，若union在from子句的子查询中，外层的select被标记成derived。
- 从union表获取结果的select被标记成union result。
- type叫访问类型，表示在表中找到所需行的方式，常见类型有all，index，range，ref，eq_ref，const，system，NULL 性能从做至右由差至好。
- ALL，即full table scan，mysql将遍历全表来找到所需要的行。
- index为full index scan，只遍历索引树。
- range表示索引范围扫描 ，对索引的扫描开始于一点，返回匹配的值域的行，常见于between，<，>的查询。
- ref为非唯一性索引扫描，返回匹配某个单独值的所有行，常见于非唯一索引即唯一索引的非唯一前缀进行的查找。
- eq_ref表示唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配，常见于主键或者唯一索引扫描。
- const，system表示当对查询部分进行优化，并转化成一个常量时，使用这些类型访问。比如将主键置于where列表中，mysql就能把该查询置成一个常量。sy
- tem是const的一个特例，当查询表中只有一行的情况下使用的是system。
- NULL表示在执行语句中，不用查表或索引。
- possiblekey表示能使用哪个索引在表中找到行，查询涉及到的字段上若存在索引，则该索引被列出，但不一定被查询使用。
- key表示查询时使用的索引。若查询中使用了覆盖索引，则该索引仅出现在key中举个例子
<!--more-->
>	mployee中gender上有一个索引。使用如下语句
	```
	EXPLAIN SELECT gender from employees
	```
>	则结果如下：1   SIMPLE   employees   index   IND_GEN   1   300695   Using index
>	```
	EXPLAIN SELECT first_name from employees
	```
>	则结果如下  1   SIMPLE   employees   ALL   300695   


- keylen表示索引所使用的字节数，可以通过该列结算查询中使用的索引长度
- ref表示上述表的链接匹配条件，即哪些列或常量可被用于查找索引列上的值。
- rows表示根据mysql表统计信息及索引选用情况，估算找到所需记录要读取的行数。
- extra表示不在其他列并且也很重要的额外信息。
- using index表示在相应的select中使用了覆盖索引。
- usingwhere表示存储引擎搜到记录后进行了后过滤(POST-FILTER)，如果查询未能使用索引，usingwhere的作用只是提醒我们mysql要用where条件过滤z结果集。
- using temporay表示用临时表来存储结果集，常见于排序和分组查询。
- usingfilesort，mysql中无法用索引完成的排序成为文件排序。

#### 关于覆盖索引的一些概念如下：


>MySQL可以利用索引返回select列表中的字段，而不必根据索引再次读取数据文件 包含所有满足查询需要的数据的索引称为 覆盖索引（Covering Index） 
>如果要使用覆盖索引，一定要注意select列表中只取出需要的列，不可select *，因为如果将所有字段一起做索引会导致索引文件过大，查询性能下降

#### mysq的执行计划有一定局限性

- EXPLAIN不会告诉你关于触发器、存储过程的信息或用户自定义函数对查询的影响情况
- EXPLAIN不考虑各种Cache
- EXPLAIN不能显示MySQL在执行查询时所作的优化工作
- 部分统计信息是估算的，并非精确值
- EXPALIN只能解释SELECT操作，其他操作要重写为SELECT后查看执行计划