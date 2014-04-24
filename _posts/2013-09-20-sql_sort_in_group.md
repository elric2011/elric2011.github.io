---
layout: post
title: "SQL分组排序Limit"
permalink: /a/sql_sort_in_group.html
description: ""
category: 
tags: [技术原创, mysql]
---
{% include JB/setup %}

问题
------

业务场景中经常遇到这样类似的需求：

> How do I find the most popular item from each category? 
>
> How do I find the top score for each player? 
	
这类问题可以描述为`select the extreme (or top N) from each group`

例如下面的原始数据（fruits表）：

| type   | variety    | price |
|--------|------------|-------|
| apple  | gala       |  2.79 |
| apple  | fuji       |  0.24 |
| apple  | limbertwig |  2.87 |
| orange | valencia   |  3.59 |
| orange | navel      |  9.36 |
| pear   | bradford   |  6.05 |
| pear   | bartlett   |  2.14 |
| cherry | bing       |  2.55 |
| cherry | chelan     |  6.33 |

查询各type中价格最高的行，结果应该如下：

| type   | variety    | price |
|--------|------------|-------|
| apple  | limbertwig |  2.87 |
| orange | navel      |  9.36 |
| pear   | bradford   |  6.05 |
| cherry | chelan     |  6.33 |

	select type, max(price),variety from fruits group by type;

错误的SQL
------

	SELECT * FROM fruits GROUP BY TYPE ORDER BY price DESC LIMIT 1;

查询的结果：

| type   | variety    | price |
|--------|------------|-------|
| pear   | bradford   |  6.05 |


方法一：存储过程
------

思路是通过程序循环查询每一组的 Top N，将查询结果合并起来。

	create procedure sp_demo()
	begin
		declare _done int default 0;
		declare _type varchar(20);
		declare rs cursor for select distinct type from fruits;
		declare continue handler for sqlstate '02000' set _done = 1;
		
	    drop table if exists temp_table;
		create table temp_table (
			type varchar(20),
			variety varchar(20),
			price decimal(8,2)
		) engine=innodb default charset=utf8;
		
	    open rs;
	    fetch next from rs into _type;
	    repeat
	    	insert into temp_table (type, variety, price) select type, variety, price from fruits where type = _type order by price desc limit 1;
	    	fetch next from rs into _type;
	    until _done end repeat;
	    close rs;
	    select * from temp_table;
	    drop table if exists temp_table;
	end


方法二：使用self-join或subquery
------

这一类方法的主要思路都是：

> step 1: finding the desired value of price;
> 
> step 2: selecting the rest of the row based on that.

### self-join ###

	select f.type, f.variety, f.price
	from (
	   select type, max(price) as maxprice
	   from fruits group by type
	) as x inner join fruits as f on f.type = x.type and f.price = x.maxprice;

### subquery ###

	select type, variety, price
	from fruits
	where price = (select max(price) from fruits as f where f.type = fruits.type);

### 查询Top N ###

	select type, variety, price
	from fruits
	where (
	   select count(*) from fruits as f
	   where f.type = fruits.type and f.price >= fruits.price
	) <= 2;


方法三：使用分析函数
------

分析函数语法：

	FUNCTION_NAME(<argument>,<argument>...)
	OVER
	(<Partition-Clause><Order-by-Clause><Windowing Clause>)

参考
------

- [http://www.xaprb.com/blog/2006/12/07/how-to-select-the-firstleastmax-row-per-group-in-sql/](http://www.xaprb.com/blog/2006/12/07/how-to-select-the-firstleastmax-row-per-group-in-sql/)
- [http://www.blogjava.net/loocky/archive/2007/11/13/160213.html](http://www.blogjava.net/loocky/archive/2007/11/13/160213.html)